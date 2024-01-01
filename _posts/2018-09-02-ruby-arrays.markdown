---
layout: post
title:  "Ruby - Arrays !"
date:   2018-09-02 09:01:18 +0545
categories: [Ruby, Arrays]
tags: [ruby, arrays]
---

An array is an ordered collection of elements that can be of any type. Each element in an array is referred to by an index. Array can have objects like integer, string, float, Fixnum, Hash, Symbol.

Creating / Initialization of an Array

```Ruby
arr = Array.new
```

Set array size

```Ruby
arr = Array.new(50)
puts arr.size # 50
puts arr.length # 50
```

Assign a value of an array

```Ruby
arr1 = Array.new([1,2,3,4,5])
arr2 = [1,2,3,4,5]
```

Accessing elementes

```Ruby
arr1[3] => 4
arr1[50] => nil
arr1[-3] => 3
arr1[0, 4] => [1,2,3,4] # 0 is the indexing value and 4 is 4 items including indexing value 0
arr1[1..3] => [2,3,4]
arr1.at(1) => 2
arr1.fetch(4) => 5
arr1.first => 1
arr1.last => 5
arr1.take(3)
=> [1, 2, 3]
arr1.drop(3)
=> [4, 5]

arr = [1,2,3,4,5]
arr.count => 5
arr.empty? # => false
arr.include?(2) => true
```

Check if two array objects are equal

```Ruby
# checks the value
arr1.eql? arr2
=> true

# checks the references of the object, object id
arr1.equal? arr2
=> false
# This is because object_id for both object is different
arr1.object_id = 11544620
arr2.object_id = 11617600

arr1 = arr2
arr1.equal? arr2 
=> true
# Now object_id for both object is same
arr1.object_id = 11617600
arr2.object_id = 11617600
```

Check if there are same elements in both arrays

```Ruby
array1 = [1,2,3]
array2 = [2,3,1]
array1.to_set == array2.to_set
=> true

array1 = [1,2,3,4]
array2 = [1,2,3]
array1.to_set == array2.to_set
=> false

#### In Ruby >= 2.6 we can use array1.intersection(array2) method, if both are same it returns empty array []
[ 1, 2, 3 ].difference([ 3, 2, 1 ])
[ 1, 2, 3 ].difference([ 1, 2, 3 ])
=> []
```


Word array create an array in which each entry is a single word.

```Ruby
count  = %w{one two three four five}
```

This is equivalent to

```Ruby
count = ["one", "two", "three", "four", "five"]
```

Nested Array: Array can contains other arrays

```Ruby
staffs_info = [
  ["Ram", "0012", "Manager"],
  ["Shyam", "0013", "HR Manager"],
  ["Hari", "0014", "Receptionist"]
]
```

Accessing value of nested array

```Ruby
staffs_info[0][1]
=> "0012"
```

Adding Data to Array

```Ruby
count << "six"
 => ["one", "two", "three", "four", "five", "six"]

count.push("seven")
=> ["one", "two", "three", "four", "five", "six", "seven"]

# insert first position of an array
count.unshift("zero")
=> ["zero", "one", "two", "three", "four", "five"]

# insert at any position
count.insert(5, "between 4 & 5")
=> ["zero", "one", "two", "three", "four", "between 4 & 5", "five"]

# insert multivalues once
count.insert(3, 'bet 2-3 1', 'bet2-3 2', 'bet2-3 3')
=> ["zero", "one", "two", "bet 2-3 1", "bet2-3 2", "bet2-3 3", "three", "four", "between 4 & 5", "five"]
```

Adding Data to nested Array

```Ruby
staffs_info[0] << "Rs. 80,000"
=> ["Ram", "0012", "Manager", "Rs. 80,000"]
staffs_info
=> [["Ram", "0012", "Manager", "Rs. 80,000"], ["Shyam", "0013", "HR Manager"], ["Hari", "0014", "Receptionist"]]
```

Array pop / Remove items from an array

```Ruby
arr = [1,2,3,4,5]
arr.pop
=> 5
arr
=> [1,2,3,4]

# array.unshift(0) will add o to start of an array while array.shift will remove first element
arr = [1,2,3,4,5]
arr.shift 
=> 1
arr
[2,3,4,5]

# delete an item at particular index use delete_at(index_position)
arr = [1,2,3,4,5]
arr.delete_at(2)
=> 3
arr
[1,2,4,5]

# compact() is used to remove nil value from an array
arr = [nil, 1, 2, 3, nil, 4, nil, 5]
arr.compact
[1,2,3,4,5]
arr
=> [nil, 1, 2, 3, nil, 4, nil, 5]
arr.compact!
=> [1,2,3,4,5]
arr
=> [1,2,3,4,5]

arr = [1,1,2,2,3,3,4,4,5,6,7]
arr.uniq
=> [1, 2, 3, 4, 5, 6, 7]
```

Iterating over an Array

```Ruby
arr = [1,2,3,4,5]
arr.each {|item| p item+10}
=> it prints 11,12,13,14,15

arr = [1,2,3,4,5]
arr.reverse_each {|item| p item+10}
=> it prints 15,14,13,12,11

arr = [1,2,3,4,5]
arr.map {|item| p item+10}
=> it prints 11,12,13,14,15
arr
=> 1,2,3,4,5

arr.map! {|item| p item+10}
arr
=> [11, 12, 13, 14, 15]

```

Selecting items from an Array

Non-destructive Selection

```Ruby
arr = [1,2,3,4,5,6,7,8]
arr.select {|a| a > 3}
# => [4,5,6,7,8]
arr.reject {|a| a < 3}
# => [3,4,5,6,7,8]
arr.drop_while {|a| a < 5}
# => [5,6,7,8]
arr
=> [1,2,3,4,5,6,7,8] 
```

Destructive Selection

Destructive methods are select! and reject!

```Ruby
arr.select! {|a| a > 3}
=> [4, 5, 6, 7, 8]
arr
=> [4, 5, 6, 7, 8]


> arr = [1,2,3,4,5,6,7,8]
=> [1, 2, 3, 4, 5, 6, 7, 8] 
> arr.delete_if { |a| a < 4 }
=> [4, 5, 6, 7, 8] 
> arr
=> [4, 5, 6, 7, 8]

arr = [1, 2, 3, 4, 5, 6, 7, 8]
arr.keep_if { |a| a < 4 }
[1,2,3]
arr
=> [1, 2, 3]
```

Public methods like &, |, &&, ||

```Ruby
a = [1, 2, 3, 4]
b = [3, 4, 5, 6]

Set intersection:
a & b
=> [3, 4]


a | b
=> [1, 2, 3, 4, 5, 6]

a || b
=> [1, 2, 3, 4]

a && b
=> [3, 4, 5, 6]

```

Concatenating two arrays:

```Ruby
["a", "b"] + ["c", "d"]
=> ["a", "b", "c", "d"]
````

Difference of arrays:

```Ruby
["a", "b", "c", "d", "e"] - ["c", "d"]
=> ["a", "b", "e"]
```

Arrarys can be chained together and returns an array (arr << obj -> arr)

```Ruby
["a", "b"] << 10 << ["c", "d"]
=> => ["a", "b", 10, ["c", "d"]]
```

array <=> another_array -> -1, 0, +1 or nil

```Ruby
a = [1,2]
b = [3,4]
a <=> b => -1

a = [1, 2]
b = [1, 2]
a <=> b => 0

 a = [1, 2, 3]
 b = [1, 2]
 a <=> b => 1

 a = [1, 2, 3]
 b = [1, 2, :v]
 a<=>b => nil
```

arr == another_arr -> bool

```Ruby
[1,3] == [1,3] #=> true
[1,3] == [1,3,4] #=> false
```

bsearch {|x| block} -> elm

Binary search finds a value from this array which meets the given condition in O(log n) where n is the size of the array.

```Ruby
arr = [0,1,2,3,4,5]
arr.bsearch {|x| x >= 2}
=> 2
```

Clear an Array

```Ruby
arr = [1,2,3,4,5]
arr.clear
# => []

a = [1,2,3,4,5]
a.collect {|x| x.to_s+"!"}
=> ["1!", "2!", "3!", "4!", "5!"]

a.collect.with_index {|x,i| p i}
=> [0, 1, 2, 3, 4]

a.map.with_index {|x,i| p i}
=> [0, 1, 2, 3, 4]
```

Combination

```Ruby
a = [1,2,3,4,5]
a.combination(1).to_a
#=> [[1], [2], [3], [4], [5]]
a.combination(2).to_a
=> [[1, 2], [1, 3], [1, 4], [1, 5], [2, 3], [2, 4], [2, 5], [3, 4], [3, 5], [4, 5]]
a.combination(3).to_a
=> [[1, 2, 3], [1, 2, 4], [1, 2, 5], [1, 3, 4], [1, 3, 5], [1, 4, 5], [2, 3, 4], [2, 3, 5], [2, 4, 5], [3, 4, 5]]
```

compact
```Ruby
[1,2,nil,'a','b',4].compact
#=> [1,2,'a','b',4]
```

concat
```Ruby
[1,2].concat([5,6])
#=> [1,2,5,6]
```

cycle
Calls the given block for each element n times or forever if nil is given.
Does nothing.
```Ruby
a=["a","b","c"]
a.cycle {|x| puts x} # infinite loop
a.cycle(2) {|x| puts x} # => a b c a b c 
```

Array fill

```Ruby
arr = [1,2,3]
arr.fill('a')
=> ["a", "a", "a"]
```

flatten

```Ruby
arr = [[1,2], 3,4,[5]]
arr.flatten
# => [1,2,3,4,5]
```

replace

```Ruby
arr = ['a', 'b']
arr.replace([1,2])
=> [1,2]
```

sort
```Ruby
a = [5,4,6,8]
a.sort
=> [4, 5, 6, 8]
```

Conversion

```Ruby
to_s => returns the string
to_h => returns hash i.e. [key, value] pairs
> [[1,:b], [2,:c]].to_h
=> {1=>:b, 2=>:c}
to_a => returns self
to_ary => returns self
```

transpose matrix

```Ruby
a = [[1,2], [3,4], [5,6]]
a.transpose
=> [[1,3,5], [2,4,6]]
```
