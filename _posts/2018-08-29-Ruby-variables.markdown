---
layout: post
title:  "Ruby Variables!"
date:   2018-08-29 09:23:58 +0545
categories: [Ruby, Variable]
tags: [ruby, variable]
---

#### What is Ruby variables?

Variables are like containers used to store information for later use. Values can be stored in the form of Integer, String, Boolean, Float, Decimal, Array, Hashes, etc.

Variable can be declared as:

```
num = 10
str = "this is string"
char = 'A'
arr = [0,1,2,3,4]
hash_var = {name: 'Shiv Raj', code: '00145', address: 'Nepal', "email" => 'shivrajbadu@gmail.com'}
bool_var = false

```

Many languages like C, Java are strong or static variable typing. That means you must define a variable type when declaring them e.g. if it is integer you must write int var_name, if string you must write varchar var_name. But Ruby is dynamically typed language which means we do not need to define type of the variable, and once variable is declared, you can later on change the variable type in the code, these are the advantage of dynamically typed lanaugage.


##### Way to know Ruby Variable Type

Use kind_of? method of Object class.
```
num = 10
num.kind_of?(Integer) # true
```

To get the class method name used by the variable
```
num.class
=> Fixnum

str = 'this is string'
str.class
=> String
```

* Variable type can be changed just by assigning new value

```
x = 10
x.class # Fixnum
x = "Ten"
x.class # String
```

* Convert the values of the variables

```
x=10
=> 10
x.to_f
=> 10.0
x.to_s
=> "10"
x.to_s(2) # convert to base 2 binary
=> 1010
x.to_s(16) # convert to hexadecimal
=> "a"
x.to_s(8) # convert to octal
=> "12"
```