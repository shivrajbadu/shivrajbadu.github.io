---
layout: post
title:  "Tailwind CSS Button Styling"
date:   2024-01-06 11:40:18 +0545
categories: [Tailwind CSS, Button Styling]
tags: [css, tailwind_css, UI]
---

Let's dive into buttons and interactive elements in Tailwind CSS.

## Button Styles

Tailwind CSS provides utility classes to easily style buttons. You can use classes like bg-blue-500, text-white, font-bold, py-2, px-4, rounded, etc., to create visually appealing buttons.

```
<button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
    Click me
</button>
```

## Button Sizes

You can adjust the size of buttons using utility classes like text-xs, text-sm, text-lg, etc.

```
<button class="bg-blue-500 text-white font-bold py-2 px-4 rounded text-xs">
    Small Button
</button>
<button class="bg-blue-500 text-white font-bold py-2 px-4 rounded text-lg">
    Large Button
</button>
```

## Button States

Tailwind CSS allows you to define styles for different states of buttons, such as hover, focus, active, and disabled.

```
<button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:ring-2 focus:ring-blue-500">
    Hoverable Button
</button>
<button class="bg-blue-500 active:bg-blue-700 text-white font-bold py-2 px-4 rounded">
    Clicked Button
</button>
<button class="bg-gray-300 text-gray-500 font-bold py-2 px-4 rounded cursor-not-allowed" disabled>
    Disabled Button
</button>
```

## Button Groups

You can group multiple buttons together using flexbox utilities to create button groups.

```
<div class="flex space-x-4">
    <button class="bg-blue-500 text-white font-bold py-2 px-4 rounded">Button 1</button>
    <button class="bg-blue-500 text-white font-bold py-2 px-4 rounded">Button 2</button>
    <button class="bg-blue-500 text-white font-bold py-2 px-4 rounded">Button 3</button>
</div>
```

These are some examples of how you can create and style buttons and interactive elements using Tailwind CSS utility classes.


{% include inarticle-adsense.html %}