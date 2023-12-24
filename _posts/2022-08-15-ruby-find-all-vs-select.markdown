---
layout: post
title:  "Ruby find_all vs. select"
date:   2022-08-15 10:40:18 +0545
categories: [Ruby, select vs. find_all]
tags: [ruby, find_all, select]
---

## About find_all vs. select Ruby method

`find_all` or `select` returns an array which contains all elements of enum for which the given block returns a true value, and, if no block is given, an Enumerator is returned.

Here are some examples:

```
arr = 1..8 
h = {a: 1, b: 2, c: 3, d: 4, e: 5, f: 6, g: 7, h: 8}
```

```
arr.select{|x| x.even?} # => [2, 4, 6, 8]
a.find_all{|x| x.even?} # => [2, 4, 6, 8]
```

On hash select returns hash and find_all returns array.

```
h.select { |k, v| v.even? }   # => {:b=>2, :d=>4, :f=>6, :h=>8}
h.find_all { |k, v| v.even? } # => [[:b, 2], [:d, 4], [:f, 6], [:h, 8]]
```