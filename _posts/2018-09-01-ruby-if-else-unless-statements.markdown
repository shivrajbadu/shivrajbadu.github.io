---
layout: post
title:  "Ruby - Control Flow Statement !"
date:   2018-08-30 09:23:58 +0545
categories: tutorials
---

#### if Statement in Ruby


if, elsif and else block in Ruby controls decision based on the condition to true/false resulting in the different execution of the code.

```
key = 10
if key > 15
  puts 'Key is greater than 15'
elsif key < 8
  puts 'key is less than 8'
else
  puts 'key is between 8 and 15'
end 
```

unless statement is inverse of if statement. unless statement is executed if expression is not true

```
num = 10
unless num == 9
  puts "Selected number is not 10"
end
```

#### Ternary Operator

Ternary operator is short hand for if else expression. Two symbols ? : are used.

```
x = 2
x > 5 ? 'Greater' : 'Smaller' 
```