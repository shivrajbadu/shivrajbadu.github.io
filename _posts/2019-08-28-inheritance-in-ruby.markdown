---
layout: post
title:  "Inheritance in Ruby"
date:   2019-08-28 09:23:58 +0545
categories: [Ruby, Inheritance]
tags: [ruby, inheritance]
---

### Inheritance in Ruby

Inheritance is the feature of OOP in which characteristics & behaviours of one class inherits into another class. The class which is inheriting behaviour is called subclass and class it inherits from is called superclass. Inheritance can also be used to remove duplication in your code and helps to achieve DRY "Don't Repeat Yourself" principle.


#### Class Inheritance

```
class Animal
    def eat # this method will overrides on other inherited classes
        puts "Animal eats grasses, water, etc"
    end
end

class Cat < Animal
end

class Cow < Animal
end
```

Here Animal is the superclass and Cat, Cow is the subclass.

```
cat = Cat.new
cow = Cow.new
cat.eat # => Animal eats grasses, water, etc
cow.eat # => Animal eats grasses, water, etc
```

#### Method overriding

```
class Animal
    def eat
        puts 'Animal eats grasses, water, etc'
    end
end

class Cat < Animal
    attr_accessor :food_name

    def initialize(food_name)
        @food_name = food_name
    end

    def eat
        puts "Cat eats #{food_name}"
    end
end

class Dog < Animal
end
```

```
cat = Cat.new("Milk and water")
cat.eat
# => Milk and water
dog = Dog.new
dog.eat
# => Animal eats grasses, water, etc
```

#### super

super is the inbuilt function of Ruby, which is used to call the methods up the inheritance hierarchy.

```
class Animal
  def eat
    "Animal"
  end
end

class Cat < Animal
  def eat
    super + " - cat - eats milk and water."
  end
end

cat = Cat.new
cat.eat        # => "Animal - cat - eats milk and water."
```

#### Module Mixins in Ruby

Modules are a way of grouping together methods, classes, and constants. Modules provide a namespace and prevent name clashes, and it implement the mixin facility. Mixins is like multiple inheritence. 

```
module ModuleName
    def module_method
        puts "I am a module method"
    end
end

class ClassName
    include ModuleName
end
```

```
obj = ClassName.new
obj.module_method
# => I am a module method
```
