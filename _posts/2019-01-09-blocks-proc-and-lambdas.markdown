---
layout: post
title:  "Blocks, Lambdas and Proc"
date:   2019-01-07 08:01:18 +0545
categories: [Ruby, Block Lambda Proc]
tags: [ruby, block, lambda, proc]
---

# Lambdas and Proc

Lambdas and Proc are block executing statement.

Lambdas and Proc both are object of Proc.

Lambdas and Proc are executed by call().

Lambda declaration

```Ruby
x = lambda { p "This is lambda" }
x.call
=> "This is lambda"

obj = lambda do |x, y|
  x+y
end
obj.call(2,3)
=> 5

# if required arguments are not supplied lambda throws argument errors
obj.call(2,3,5)
ArgumentError (wrong number of arguments (given 3, expected 2))
```

Proc declaration

```Ruby
x = Proc.new {p "this is proc"}
x.call
=> "this is proc"

obj = Proc.new do |x, y|
  x+y
end
obj.call(2,3)
=> 5

# if required arguments are not supplied it won't throw argument error like lambda
obj.call(2,3,5)
=> 5
```

```Ruby
class Block
  def hello(*args, &block)
    yield *args
  end

  proc = Proc.new do |*args|
    puts *args.class
    arr = *args
    sum = 0
    arr.flatten.each do |num|
      sum = sum + num
    end
    puts sum
  end

  obj = Block.new
  obj.hello([1,10,15], &proc)
end
=> Array
26
```

# Blocks

Ruby blocks are anonymous functions are passed into methods. They are enclosed between {} brackets or in do/end statement.
It accepts multiple arguments as |arg1, ..., argn|. Blocks are used with `each`.
It allows to save code and use it later.

```Ruby
#### single line blocks
[20,30,40].each {|n| puts n}
# here code inside {} are block

#### multi-line blocks
[20,30,40].each do |n|
  puts n
end
```

### Ruby yield keyword
yield is a keyword that calls and run the code inside the block

```Ruby
def block_fun
  yield
end

block_fun { puts "Block is executing" }
```
