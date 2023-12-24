---
layout: post
title:  "Ruby Put and Print Commands !"
date:   2018-08-30 05:12:38 +0545
categories: [Ruby, Puts vs. Print]
tags: [ruby, print]
---

#### puts vs print

puts and print both are used to display the result of evaluating Ruby code.
Major difference between these two are: puts adds a newline after executing but print does not add new line.

```
puts "one two"
one two
=> nil

print "one two"
one two => nil

print [1,2,nil]
[1, 2, nil] => nil

puts [1,2,nil]
1
2

=> nil 
```
