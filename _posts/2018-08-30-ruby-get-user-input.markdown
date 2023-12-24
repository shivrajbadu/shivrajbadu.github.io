---
layout: post
title:  "Get user input in Ruby!"
date:   2018-08-30 09:23:58 +0545
categories: [Ruby, Get user input]
tags: [ruby, get_user_input]
---

#### Getting user input

gets keyword is used to get the user input as a string.

```
#!/usr/bin/ruby

puts 'what is your name?'
name = gets.chomp
puts "How are you #{name}"
```

String#chomp method returns string after removing extra line.