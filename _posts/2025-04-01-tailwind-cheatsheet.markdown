---
layout: post
title: "Tailwind CSS Mastery Guide"
date: 2025-04-01 12:20:18 +0545
categories: [Tailwind CSS, UI, cheatsheet]
tags: [css, tailwind_css, UI, cheatsheet]
---

Here's a comprehensive list of Tailwind CSS utility categories and their class name prefixes to help you master Tailwind:

# Tailwind CSS Mastery Guide

## Layout

| Category              | Prefixes/Classes                                                                |
| --------------------- | ------------------------------------------------------------------------------- |
| Container             | `container`                                                                     |
| Display               | `block`, `inline-block`, `inline`, `flex`, `inline-flex`, `grid`, `inline-grid` |
| Box Sizing            | `box-border`, `box-content`                                                     |
| Float                 | `float-right`, `float-left`, `float-none`                                       |
| Clear                 | `clear-left`, `clear-right`, `clear-both`, `clear-none`                         |
| Position              | `static`, `fixed`, `absolute`, `relative`, `sticky`                             |
| Top/Right/Bottom/Left | `top-0`, `right-0`, `bottom-0`, `left-0` (with various sizes)                   |
| Visibility            | `visible`, `invisible`, `collapse`                                              |
| Z-Index               | `z-0` to `z-50`, `z-auto`                                                       |

## Flexbox

| Category        | Prefixes/Classes                                                          |
| --------------- | ------------------------------------------------------------------------- |
| Flex Direction  | `flex-row`, `flex-row-reverse`, `flex-col`, `flex-col-reverse`            |
| Flex Wrap       | `flex-wrap`, `flex-wrap-reverse`, `flex-nowrap`                           |
| Flex            | `flex-1`, `flex-auto`, `flex-initial`, `flex-none`                        |
| Flex Grow       | `grow`, `grow-0`                                                          |
| Flex Shrink     | `shrink`, `shrink-0`                                                      |
| Order           | `order-1` to `order-12`, `order-first`, `order-last`, `order-none`        |
| Justify Content | `justify-start`, `justify-end`, `justify-center`, `justify-between`, etc. |
| Align Items     | `items-start`, `items-end`, `items-center`, `items-baseline`, etc.        |

## Grid

| Category              | Prefixes/Classes                                  |
| --------------------- | ------------------------------------------------- |
| Grid Template Columns | `grid-cols-1` to `grid-cols-12`, `grid-cols-none` |
| Grid Column Start/End | `col-start-1`, `col-end-3`, etc.                  |
| Grid Template Rows    | `grid-rows-1` to `grid-rows-6`, `grid-rows-none`  |
| Gap                   | `gap-0` to `gap-96` (also `gap-x-*`, `gap-y-*`)   |

## Spacing

| Category      | Prefixes/Classes                                                      |
| ------------- | --------------------------------------------------------------------- |
| Padding       | `p-0` to `p-96` (also `pt-*`, `pr-*`, `pb-*`, `pl-*`, `px-*`, `py-*`) |
| Margin        | `m-0` to `m-96` (also `mt-*`, `mr-*`, `mb-*`, `ml-*`, `mx-*`, `my-*`) |
| Space Between | `space-x-*`, `space-y-*`                                              |

## Sizing

| Category  | Prefixes/Classes                                                  |
| --------- | ----------------------------------------------------------------- |
| Width     | `w-0` to `w-96`, `w-auto`, `w-full`, `w-screen`, `w-min`, `w-max` |
| Min-Width | `min-w-0`, `min-w-full`, `min-w-min`, `min-w-max`                 |
| Height    | `h-0` to `h-96`, `h-auto`, `h-full`, `h-screen`                   |

## Typography

| Category    | Prefixes/Classes                                         |
| ----------- | -------------------------------------------------------- |
| Font Family | `font-sans`, `font-serif`, `font-mono`                   |
| Font Size   | `text-xs` to `text-9xl`                                  |
| Font Weight | `font-thin` to `font-black`                              |
| Text Color  | `text-{color}-{shade}` (e.g., `text-red-500`)            |
| Text Align  | `text-left`, `text-center`, `text-right`, `text-justify` |

## Backgrounds

| Category            | Prefixes/Classes                                        |
| ------------------- | ------------------------------------------------------- |
| Background Color    | `bg-{color}-{shade}` (e.g., `bg-blue-500`)              |
| Background Opacity  | `bg-opacity-0` to `bg-opacity-100`                      |
| Background Position | `bg-bottom`, `bg-center`, `bg-left`, etc.               |
| Background Gradient | `bg-gradient-to-{direction}` (e.g., `bg-gradient-to-r`) |

## Borders

| Category      | Prefixes/Classes                                                    |
| ------------- | ------------------------------------------------------------------- |
| Border Radius | `rounded`, `rounded-t`, `rounded-r`, `rounded-b`, `rounded-l`, etc. |
| Border Width  | `border`, `border-0` to `border-8`, `border-t`, `border-r`, etc.    |
| Border Color  | `border-{color}-{shade}` (e.g., `border-gray-300`)                  |

## Effects

| Category   | Prefixes/Classes                                                           |
| ---------- | -------------------------------------------------------------------------- |
| Box Shadow | `shadow-sm`, `shadow`, `shadow-md`, `shadow-lg`, `shadow-xl`, `shadow-2xl` |
| Opacity    | `opacity-0` to `opacity-100`                                               |

## Transitions & Animation

| Category            | Prefixes/Classes                                                |
| ------------------- | --------------------------------------------------------------- |
| Transition Duration | `duration-75` to `duration-1000`                                |
| Animation           | `animate-none`, `animate-spin`, `animate-ping`, `animate-pulse` |

## Interactivity

| Category    | Prefixes/Classes                                          |
| ----------- | --------------------------------------------------------- |
| Cursor      | `cursor-auto`, `cursor-pointer`, `cursor-wait`, etc.      |
| User Select | `select-none`, `select-text`, `select-all`, `select-auto` |

## Pseudo-class Variants

- Hover: `hover:`
- Focus: `focus:`
- Active: `active:`
- Responsive: `sm:`, `md:`, `lg:`, `xl:`, `2xl:`
- Dark mode: `dark:`

{% include inarticle-adsense.html %}