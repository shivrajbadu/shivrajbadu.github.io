---
layout: post
title:  "Ruby - Switch Case Statement !"
date:   2018-09-01 07:12:18 +0545
categories: [Ruby, Swith Case]
tags: [ruby, switch_case]
---

Ruby uses the case expression with one or more when conditions. After execution it returns one of the when statement or default else case.

```
case gets.chomp

when '1'
  puts "You have entered 1"
when '2'
  puts "You have entered 2"
else
  puts 'You have entered number other than 1 & 2'
end
```