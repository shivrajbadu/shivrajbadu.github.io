---
layout: post
title:  "Ruby String !"
date:   2018-08-30 09:23:58 +0545
categories: [Ruby, String]
tags: [ruby, string]
---

#### About String

String holds and manipulates an arbitrary sequence of bytes which is group of characters. String in ruby is defined using single quote and double quote as:

```Ruby
strvar = 'this is string'
strvar1 = "this is string #{some_dynamic_var}"
```

#### Find string length

size() and length() are used to find string length

```Ruby
"string".size
=> 6
```

#### String Interpolation

```Ruby
str = "String"
puts "This Is #{str}"
```

Ruby calls to_s() on the string interpolation block which is used to convert object itself into string.

#### Extract a Substring

```Ruby
str = "longstring"
str[0,4]
# long
str[4,6]
# string
str[0..-2]
# longstrin
str[0..3] = ''
# string
```

#### include? is used to find if string contains another string

```Ruby
str = "My name is Mr. ABC"
str.include?("ABC")
# true
```

index() can be used to find the start position / index position of the string
```Ruby
str = "My name is Mr. ABC"
str.index("ABC")
# 15
```

In Ruby String add more string like this:

```Ruby
str = "string"

str.rjust(18, "0")
 => "000000000000string"

str.ljust(18, "0")
 => "string000000000000" 
```

#### Case in String

```Ruby
var1 = "str"
var2 = "Str"
var1.upcase == var2.upcase
=> true
var1.casecmp?(var2) # casecmp? Case-insensitive version of String
=> true
```

#### Trim a String & Remove a White Space

```Ruby
str = "   string   "
str.strip
=> "string"
```

#### Trim left and right string
```Ruby
str = "   string   "
str.lstrip
 => "string   "
str.rstrip
 => "   string" 
```

#### String prefix and suffix

start_with?, end_with?

```Ruby
str = "a red car"
str.start_with?("a")
# true
str.start_with?("car")
# true
```

#### Ruby 2.5 has two methods delete_prefix & delete_suffix

```Ruby
str = "a red car"
str.delete_prefix("a red")
 => " car" 
str.delete_suffix("red car")
 => "a "
```

#### Convert string to array of characters

```
str = "string"
str.split("")
=> ["s", "t", "r", "i", "n", "g"]
```

#### Convert arrary to string
```Ruby
arr = ["s", "t", "r", "i", "n", "g"]
arr.join
=> "string"
arr.join("-")
 => "s-t-r-i-n-g"
```

#### Count specific characters

```Ruby
"lophophorous".count("o")
=> 4
```

 #### Convert string to integer
```Ruby
"str".to_i
=> 0

"50".to_i
=> 50
```

#### check string is a number
match() is introduced in Ruby 2.4

```Ruby
"123".match?(/\A-?\d+\Z/)
=> true

"123sadf".match?(/\A-?\d+\Z/)
=> false
```

Append Characters

```Ruby
str = ""
str << "Ruby"
str << " "
str << "Rails"
# "Ruby Rails"
```
Note: When you use += for string concatenation, this way new string will be created every time which is not good for performance.

Loop through characters
```Ruby
"hello world".each_char {|ch| puts ch}

"hello world".chars
 => ["h", "e", "l", "l", "o", " ", "w", "o", "r", "l", "d"] 
```

String Case

```Ruby
"hello".upcase
=> "HELLO"
"HELLO".downcase
=> "hello"
```

Multiline Strings

```Ruby
b = <<-STRING
hello
world
STRING

a = %Q(hello
world
)
 => "hello\nworld\n"
```

Gsub() replace text

```Ruby
str = "The color of car is red"
str.gsub("red", "blue")
=> "The color of car is blue"

str = "helloooo"
str.gsub("o", '')
=> "hell"

str = "my id is 5"
str.gsub(/\d+/, '1001')
=> "my id is 1001" 
str.gsub(/\w+/) {|w| w.capitalize}
 => "My Id Is 5"
```

#### Remove last character of a string
```Ruby
"hello".chomp("o")
=> hell
```

#### Remove first and last character if first and last letter satisfied some value

```Ruby
str = "{'a','b','c'}"

str[1..-1] if str.chars.first == '{'
str[0...-1] if str.chars.last == '}'
```

#### Change string encodings

```Ruby
"string".encoding
 => #<Encoding:UTF-8>

"string".force_encoding("UTF-8")
```

#### Find out number of occurrence of each character in a given string

```Ruby
str = "hello world"
arr = str.split("")
arr.uniq.each {|x| p "Count of #{x} = #{str.count(x)}" if x != " "

output:
"Count of h = 1"
"Count of e = 1"
"Count of l = 3"
"Count of o = 2"
"Count of w = 1"
"Count of r = 1"
"Count of d = 1"
```

{% include inarticle-adsense.html %}
