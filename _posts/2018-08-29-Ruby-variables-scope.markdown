---
layout: post
title:  "Ruby Variables Scope!"
date:   2018-08-29 09:23:58 +0545
categories: [Ruby, Variable Scope]
tags: [ruby, variable]
---

#### Scope of Ruby variables

* Global Variable

Global Variables can be accessed inside classes and it's methods. Global variable are available everywhere. It is defined by prefacing the variable name with $ symbol. Before initialization it has value nil.

```ruby
$global_variable = 'This is a global variable !'
class Example
  def test_global
    puts $global_variable
  end
end

# instantiation and call
obj = Example.new
obj.test_global # This is a global variable !
```

* Instance Variable

Instance Variable is accessible in any instance method in a particular instance of a class. It is defined by prefacing the variable name with @ symbol.

```ruby
class Vehicle
  def initialize(name, color)
    @name = name
    @color = color
  end

  def full_info
    puts "Name of vehicle is: #{@name} with color #{@color} !"
  end
end

# instantiate
vehicle = Vehicle.new('Car', 'Red');

# method call
vehicle.full_info # Name of vehicle is: Car with color Red !
```

* Local Variable

Local variable has local scope which be accessed inside the code where they are declared, that is when local variable is decared inside method or loop it cannot be used outside of method or loop. It is defined by small letter or begin with underscore.

```ruby
class LocalVariable
  def fun
    local_var1 = 'one'
    _LocalVar2 = 'two'

    puts local_var1 + _LocalVar2
  end
end

# instantiation and call
LocalVariable.new.fun # onetwo
```

* Class Variable

A class variable is a variable that is shared amongst all instances of the class. Class variable are declared with @@ sign. Class variable are called on the class itself. Class variables are like global variable but inside the class scope.

```ruby
class Vehicle
  @@name = 'Honda'

  def self.name
    puts @@name
  end
end

Vehicle.name
```

* Ruby Constant

Ruby constant are the values whose value cannot be changed once it is assigned. Constant declared within a class are available anywhere within the context of class, and when declared outside of class are assined with a global scope. Constants are written in uppercase letter with underscore to seperate different word.

```ruby
PROJECT_VALUE=100
```