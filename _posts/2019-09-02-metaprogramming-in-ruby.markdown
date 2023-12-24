---
layout: post
title:  "Metaprogramming in Ruby"
date:   2019-09-02 09:23:58 +0545
categories: tutorials
---

## Metaprogramming in Ruby

Metaprogramming is a programming concept which treats other programs as their data and computer programs are written in such a way that is executed at runtime instead of compile time.

It helps in reducing development time by minimizing the lines of codes, also efficiently manages the programs with new solutions without recompilation.

Metaprogramming includes :
- compile code generation or Runtime code generation (or both)
- Aspect-Oriented Thinking or Aspect Oriented Programming
- DRY Thinking

It is advisable to mastering Metaprogramming before using it as it is very powerful.

Examples

Make a getter methods which return instance variables if they are not nil, if they are nil set it to some default value and return it.

```
class Foo
    def foo
        @foo ||= 0
    end
end
```
Suppose if you have multiple such getters then instead of writing them all we can use metaprogramming like this:

```
class Foo
    {foo: 0, bar: '', baz: []}.each do |method_name, default_value|
        define_method method_name do
            instance_var = :"@#{method_name}"
            instance_variable_get(instance_var) ||
            instance_variable_set(instance_var, default_value)
        end
    end
end
```

```
module GettersWithDefault
    def getters_with_default(spec)
        spec.each do |method_name, default_value|
            define_method method_name do
                instance_var = :"@#{method_name}"
                instance_variable_get(instance_var) ||
                instance_variable_set(instance_var, default_val)
            end
        end
    end
end

class Foo
    include GettersWithDefault

    getters_with_default foo: 0, bar: '', baz: {}
end
```

A common example of Metaprogramming

```
class Post
    def initialize(status)
        @status = status
    end

    %w(published unpublished draft).each do |possible_status|
        define_method("#{possible_status}?") do
            @status == possible_status
        end
    end
end
```

It seems like it saves time, because we don't need to write separate methods for published?, unpublished?, and draft?. However, there are tradeoffs. For example, metaprogramming like this makes searching for method definitions later difficult. It's certainly faster to type, but it's harder to find and read later. Since we spend so much more time reading code than writing it, code that's easier to write than read is actually a bad tradeoff.

Domain Specific Language

A Domain Specific Language or DSL is a custom language that solves a specific domain or problem. In Ruby's case, a DSL is written in Ruby but looks different from standard Ruby code. Some examples of Ruby DSL are Rails Routes, Rspec, Factory Girl, etc. Factory Girl has cmplicated internal code but it allows you to write expressive, declarative code.

```
FactoryGirl.define do
    sequence :github_username do |n|
        "github_#{n}"
    end

    factory :user do
        description "Learn all about Git"
        github_username

        trait :admin do
            admin true
        end
    end
end
```

DSL Structure

```
describe "User" do
  # ...
end

FactoryGirl.define do
    # ...
end

Rails.application.routes.draw do
  # ...
end
```
Be careful
If there is a less-complicated solution to a problem, reach for that first. Metaprogramming is usually not a good first solution to a problem, and DSLs require a good understanding of the problem's domain. Once you do understand the problem well, though, DSLs are a great option.

# Talk about Monkeypatching

-------------------
Code Discoverty and Readability

One problem with metaprogramming solutions are their obstruction of code discovery. When entering a new project or simply trying to re-familiarize onself with existing one, tracing code executiion in a text editor can be quite difficult if method definitions do not exist.

For example we can assume that a User class exists with a set of metaprogrammed methods:

```
class User
    [
        :password,
        :email,
        :first_name,
        :last_name
    ].each do |attribute|
        define_method(:"has_#{attribute}?") do
            self.send(attribute).nil?
        end
    end
end
```

Although a little contrived, this code is a list of simple convenience methods on a User class. This solution is easily extended to include additional attributes without a full method definition per attribute.

However, these methods can not be found using grep, silver searcher, or other "find all" tools. Since the method has_password? is never explicitly defined in the code, it is not discoverable.

A Work Around:

To combat this issue, some developers choose to write a comment listing the defined method names above metaprogramming block. This simple solution can greatly help the readability of the code.

```
class User
    # has_password?, has_email?, has_first_name?, has_last_name? method definitions
    [
        :password,
        :email,
        :first_name,
        :last_name
    ].each do |attribute|
        define_method(:"has_#{attribute}?") do
            self.send(attribute).nil?
        end
    end
end
```

Performance

Depending on the amount of times a piece of code is executed, performance considerations can be extremely important. "Hot code" is a term used to describe code that is called frequently during an application's request cycle. Since not all code is created equally, understanding the performance implications of different metaprogramming approaches is imperative when writing or modifying hot code.
