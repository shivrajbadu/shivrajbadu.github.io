---
layout: post
title:  "Tailwind CSS Navigation"
date:   2024-01-02 11:40:18 +0545
categories: [Tailwind CSS, Navigation]
tags: [css, tailwind_css, UI]
---

Let's explore utility classes for working with navigation menus and dropdowns in Tailwind CSS.

Horizontal Navigation Menu: You can create a horizontal navigation menu using flexbox utilities to align menu items horizontally.

```
<nav class="flex">
    <a href="#" class="p-2">Home</a>
    <a href="#" class="p-2">About</a>
    <a href="#" class="p-2">Services</a>
    <a href="#" class="p-2">Contact</a>
</nav>
```

Vertical Navigation Menu: Similarly, you can create a vertical navigation menu using flexbox utilities to align menu items vertically.

```
<nav class="flex flex-col">
    <a href="#" class="p-2">Home</a>
    <a href="#" class="p-2">About</a>
    <a href="#" class="p-2">Services</a>
    <a href="#" class="p-2">Contact</a>
</nav>
```

Dropdown Menu: You can create dropdown menus using nested lists and CSS. Tailwind CSS provides utility classes for styling dropdowns and managing their visibility.

```
<nav>
    <ul class="flex">
        <li>
            <a href="#" class="p-2">Home</a>
        </li>
        <li>
            <a href="#" class="p-2">About</a>
        </li>
        <li>
            <a href="#" class="p-2">Services</a>
            <ul class="absolute hidden bg-gray-100 p-2">
                <li><a href="#" class="block">Service 1</a></li>
                <li><a href="#" class="block">Service 2</a></li>
                <li><a href="#" class="block">Service 3</a></li>
            </ul>
        </li>
        <li>
            <a href="#" class="p-2">Contact</a>
        </li>
    </ul>
</nav>
```

Responsive Navigation: You can use responsive classes to create navigation menus that adapt to different screen sizes.

```
<nav class="flex flex-col lg:flex-row">
    <!-- Menu items here -->
</nav>
```

These are some examples of how you can create navigation menus and dropdowns using Tailwind CSS utility classes. Experiment with these classes to create navigation structures that match your design requirements.

```
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Let's explore Tailwind CSS</title>
        <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    </head>
    <body>
        <!-- Your content goes here -->
        <div class="container mx-auto py-6">
            <h1 class="text-3xl font-bold text-center text-gray-800">Welcome! Let's walkthrough with Tailwind CSS</h1>
            <p class="text-lg text-center text-gray-600">Let's explore Tailwind!</p>
        </div>

        <div class="container mx-auto py-6">
            <p class="text-lg text-left text-gray-600 font-bold">Horizontal Nav</p>
            <nav>
                <ul class="flex">
                    <li>
                        <a href="#" class="p-2">Home</a>
                    </li>
                    <li>
                        <a href="#" class="p-2">About</a>
                    </li>
                    <li class="relative">
                        <a href="#" class="p-2">Services</a>
                        <ul class="dropdown-menu hidden bg-gray-100 absolute top-full left-0">
                            <li><a href="#" class="block p-2">Service 1</a></li>
                            <li><a href="#" class="block p-2">Service 2</a></li>
                            <li><a href="#" class="block p-2">Service 3</a></li>
                        </ul>
                    </li>
                    <li>
                        <a href="#" class="p-2">Contact</a>
                    </li>
                </ul>
            </nav>

            <p class="text-lg text-left text-gray-600 font-bold">Vertical Nav</p>
            <nav class="flex flex-col">
                <a href="#" class="p-2">Home</a>
                <a href="#" class="p-2">About</a>
                <a href="#" class="p-2">Services</a>
                <a href="#" class="p-2">Contact</a>
            </nav>
        </div>

        <!-- JavaScript -->
        <script type="text/javascript">
            document.addEventListener("DOMContentLoaded", function() {
                // Get all dropdown toggle buttons
                var dropdownToggleButtons = document.querySelectorAll('.relative > a');
                
                // Add event listeners to toggle dropdown visibility
                dropdownToggleButtons.forEach(function(button) {
                    button.addEventListener('click', function(event) {
                        event.preventDefault(); // Prevent default anchor behavior
                        var dropdownMenu = this.parentElement.querySelector('.dropdown-menu');
                        dropdownMenu.classList.toggle('hidden');
                    });
                });

                // Close dropdowns when clicking outside
                document.addEventListener('click', function(event) {
                    if (!event.target.closest('.relative')) {
                        var dropdownMenus = document.querySelectorAll('.dropdown-menu');
                        dropdownMenus.forEach(function(menu) {
                            menu.classList.add('hidden');
                        });
                    }
                });
            });
        </script>
    </body>
</html>
```

{% include inarticle-adsense.html %}