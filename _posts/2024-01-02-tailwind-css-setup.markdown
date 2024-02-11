---
layout: post
title:  "Tailwind CSS Setup"
date:   2024-01-02 11:40:18 +0545
categories: [Tailwind CSS, Setup]
tags: [css, tailwind_css, UI]
---

To begin using Tailwind CSS in your project, you first need to set up a project environment. Here's how you can set up Tailwind CSS in a simple HTML project:

*  Create an HTML file: Let's create an index.html file where we'll write our HTML code.

* Include Tailwind CSS: You can include Tailwind CSS in your HTML file by either linking to a CDN or by installing it via npm and including the compiled CSS file. For simplicity, we'll use the CDN approach here.

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Tailwind CSS Project</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
</head>
<body>

<!-- Your content goes here -->

</body>
</html>
```

Start using Tailwind classes: Now you can start using Tailwind CSS classes directly in your HTML elements to style them. For example:

```
<div class="container mx-auto py-6">
    <h1 class="text-3xl font-bold text-center text-gray-800">Welcome to My Tailwind CSS Project</h1>
    <p class="text-lg text-center text-gray-600">Let's explore Tailwind together!</p>
</div>
```

In this example, `container`, `mx-auto`, `py-6`, `text-3xl`, `font-bold`, `text-center`, `text-gray-800`, `text-lg`, `text-gray-600` are Tailwind CSS utility classes that style the HTML elements.

That's it! You've successfully set up Tailwind CSS in your project. Now you can continue exploring and using Tailwind's utility classes to style your HTML elements further.

{% include inarticle-adsense.html %}