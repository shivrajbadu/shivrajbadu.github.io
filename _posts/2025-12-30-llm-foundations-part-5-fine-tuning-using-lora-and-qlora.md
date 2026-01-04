---
layout: post
title: "LLM Foundations Part 5: Fine-Tuning with LoRA and QLoRA"
date: 2025-12-30 10:30:00 +0545
categories: [AI, LLM, Machine Learning]
tags:
  [llm, fine-tuning, lora, qlora, parameter-efficient-fine-tuning, peft, mlx]
---

# LLM Foundations Part 5: Fine-Tuning with LoRA and QLoRA

## Introduction

Welcome to the fifth installment of our "LLM Foundations" series. In previous parts, we explored the architecture and pre-training of Large Language Models. Now, we turn to one of the most practical and impactful aspects of working with them: fine-tuning. While retraining an entire LLM is beyond the reach of most, fine-tuning allows us to adapt a pre-trained model for specific tasks.

However, even standard fine-tuning is incredibly resource-intensive. Modifying all the weights of a 7-billion-parameter model, for instance, requires immense VRAM and storage. This is where **Parameter-Efficient Fine-Tuning (PEFT)** comes in. PEFT methods allow us to achieve results comparable to full fine-tuning while only modifying a tiny fraction of the model's parameters.

In this post, we'll dive deep into two of the most popular PEFT techniques: **LoRA (Low-Rank Adaptation)** and its even more efficient successor, **QLoRA (Quantized LoRA)**.

## The Challenge of Full Fine-Tuning

A pre-trained LLM is a generalist. To make it an expert in a specific domain—like generating legal documents, writing in a particular brand's voice, or acting as a coding assistant—we need to fine-tune it on a relevant dataset.

The traditional approach, known as full fine-tuning, involves updating all the weights of the model during this process. For a model with billions of parameters, this presents significant challenges:

- **Catastrophic Forgetting:** The model may lose some of its general capabilities while learning the new task.
- **Computational Cost:** It requires a massive amount of GPU memory (VRAM), often necessitating multiple high-end GPUs.
- **Storage Inefficiency:** If you want to adapt the model for ten different tasks, you need to store ten separate copies of the massive model.

PEFT methods were created to solve these exact problems.

## What is LoRA? The Core Idea

Low-Rank Adaptation (LoRA) is a clever technique that avoids updating the original model's weights. Instead, it operates on a simple but powerful principle: it learns the _change_ or _delta_ needed to adapt the model, rather than relearning everything from scratch.

Here’s how it works:

1.  **Freeze the Base Model:** All the weights of the pre-trained LLM are frozen. This means they won't be updated during training, which saves a huge amount of memory.
2.  **Inject Adapter Layers:** LoRA injects small, trainable "adapter" layers alongside the original layers of the model (typically the attention layers).
3.  **Low-Rank Decomposition:** These adapter layers are composed of two much smaller matrices, often called `A` and `B`. Instead of learning a large weight matrix of size `d x k`, LoRA learns these two smaller matrices of sizes `d x r` and `r x k`, where `r` (the rank) is much smaller than `d` or `k`.

The number of trainable parameters is now just the sum of the parameters in `A` and `B`, which is a tiny fraction of the original model's size. At inference time, the outputs of the original layer and the adapter layer are simply added together.

**The benefits are immense:**

- **Drastically Fewer Trainable Parameters:** Training is faster and requires significantly less VRAM.
- **No Change to Base Model:** The original LLM weights are untouched.
- **Efficient Task Switching:** To switch tasks, you only need to swap out the small adapter weights (a few megabytes) instead of a whole new model (many gigabytes).

### LoRA in Practice: A Code Perspective

In a real-world implementation, you'd typically use a configuration object to define the LoRA parameters. The provided `LoRAConfig` class is a perfect example:

```python
class LoRAConfig:
    max_lora_rank: int
    max_loras: int
    max_cpu_loras: Optional[int] = None
    lora_dtype: Optional[torch.dtype] = None
    lora_extra_vocab_size: int = 256
    # ... validation logic ...

    def __post_init__(self):
        # Keep this in sync with csrc/punica/bgmv/bgmv_config.h
        possible_max_ranks = (8, 16, 32, 64)
        if self.max_lora_rank not in possible_max_ranks:
            raise ValueError(f"max_lora_rank ({self.max_lora_rank}) must be one of "
                             f"{possible_max_ranks}.")
        # ... more checks ...
```

The `max_lora_rank` here corresponds to the `r` value—the rank of the decomposition. A smaller rank means fewer parameters but potentially less expressive power. Ranks like 8, 16, or 32 are common and offer a great trade-off.

This configuration is then passed when the model is loaded, signaling to the framework that LoRA adapters need to be prepared.

```python
def load_model(self, *, model_config: ModelConfig, device_config: DeviceConfig, lora_config: Optional[LoRAConfig],
                   # ... other configs
                   ) -> nn.Module:
        with set_default_torch_dtype(model_config.dtype):
            with torch.device(device_config.device):
                model = _initialize_model(model_config, self.load_config, lora_config, #...
                                          )
        return model.eval()
```

## Enter QLoRA: Pushing Efficiency Even Further

QLoRA takes the efficiency of LoRA to a whole new level. It introduces a groundbreaking innovation: **backpropagation through a quantized, frozen model**.

Here’s the magic behind QLoRA:

1.  **4-bit Quantization:** The large, frozen, pre-trained model is quantized from its native 16-bit or 32-bit precision down to just 4-bits. This dramatically reduces the memory footprint. QLoRA introduces a new data type, the **4-bit NormalFloat (NF4)**, which is information-theoretically optimal for normally distributed weights.
2.  **Trainable LoRA Adapters:** Just like with LoRA, small LoRA adapters are added to the model, and only these adapters are trained.
3.  **Gradient Flow:** During the backward pass, gradients flow from the LoRA adapters _through_ the frozen 4-bit quantized weights to update the adapter parameters.

QLoRA also employs clever techniques like **Double Quantization** (quantizing the quantization constants themselves) and **Paged Optimizers** (similar to paged memory in operating systems) to handle memory spikes, further reducing the VRAM requirement.

The result? QLoRA can fine-tune models with performance very close to 16-bit fully fine-tuned models, but with a memory footprint that makes it accessible on consumer-grade hardware.

## A Practical Example: Fine-Tuning with MLX on Apple Silicon

The power of QLoRA is not just theoretical. Frameworks like MLX for Apple Silicon have made it a practical reality for developers and researchers.

As our testing confirmed, on machines with 32GB of unified memory, **MLX can support 4-bit quantized QLoRA fine-tuning of 7-billion-parameter models.** This is a remarkable feat of memory efficiency.

A typical workflow for fine-tuning with MLX and QLoRA looks like this:

1.  **Data Conversion:** A script like `data_transform.py` is used to convert your raw dataset into a format that MLX can efficiently process.
2.  **Training Execution:** A shell script (`train_by_mlx.sh`) kicks off the training process. This script reads a YAML configuration file where you can specify key parameters, especially the LoRA settings (rank, alpha, which layers to apply it to).
3.  **Model Conversion and Serving:** After training, a script like `convert_and_serve.sh` merges the trained adapter weights into the base model to create a new, fine-tuned model ready for deployment. It can then start a local server for testing.
4.  **Testing:** Finally, a script like `test_mlx.py` sends prompts to the served model to evaluate its performance on the new task.

This streamlined process demonstrates how QLoRA has democratized the ability to create custom, high-performing LLMs.

## Conclusion

LoRA and QLoRA represent a monumental shift in how we approach LLM customization. By focusing on adapting a small number of parameters instead of retraining an entire model, they drastically lower the barrier to entry for fine-tuning. QLoRA, in particular, pushes the boundaries of memory efficiency, enabling the fine-tuning of massive models on a single GPU.

These PEFT techniques empower developers, researchers, and businesses to create specialized models without the prohibitive costs associated with full fine-tuning, unlocking a new wave of innovation in applied AI.

{% include inarticle-adsense.html %}

## Suggested Reading

- [Original LoRA Paper: LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685)
- [Original QLoRA Paper: QLoRA: Efficient Finetuning of Quantized LLMs](https://arxiv.org/abs/2305.14314)
- [Hugging Face PEFT Library](https://github.com/huggingface/peft)
- [Apple's MLX Framework Documentation](https://ml-explore.github.io/mlx/build/html/index.html)
