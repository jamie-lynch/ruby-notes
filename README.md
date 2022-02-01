# Ruby Notes

This project contains a series of notes and snippets that I've collected as I learn Ruby. 
For now they all exist unorganised in the README but at some point I'll probably organise them a bit better. 

## Contents

- [Type Conversions](#type-conversions)
- [Class Reopening](#class-reopening)
- [Refinements](#refinements)
- [`__FILE__ == $0`](#__file__--0)

## Type Conversions

Ruby has got most of the methods you'd expect for type conversions, e.g.

```ruby
1.to_s
# "1"

"1".to_i
# 1

{foo: "bar", hello: "world"}.to_a
# [[:foo, "bar"], [:hello, "world"]]

[[:foo, "bar"], [:hello, "world"]].to_h
# {:foo=>"bar", :hello=>"world"}

"hello".to_sym
# :hello
```

The main gotcha to be aware of is with using `to_i` as this has odd behaviour for values that cannot be directly converted.

```ruby
1.5.to_i
# 1

"hello".to_i
# 0
```

Instead use `Integer(val)` which will throw and exception if the value cannot be converted

## Class Reopening

Class reopening (a.k.a Monkey Patching) can be used to add new methods or override existing methods on classes after they have been defined. 

```ruby
class Dog
  def talk
    puts "woof"
  end
end

dog = new Dog
dog.talk
# woof

class Dog
  def fetch
    :bone
  end
end

dog1 = new Dog
dog.talk
# woof
dog.fetch
# :bone
```

This can be useful in some situations but also confusing as that instance will retain those patches wherever it is passed around. 

An example of this in the wild is the `active_record/serialization` library which adds methods to built in classes. 

```ruby
[1,2,3].to_json
# NoMethodError (undefined method `to_json' for [1, 2, 3]:Array)

require "active_record/serialization"

[1,2,3].to_json
# "[1,2,3]"
```

## Refinements

Refinements offer similar functionality to [Class Reopening](#class-reopening) but with limited scope. This is useful to be able to make use of the power of class reopening without polluting the global signature of a class. Refinements are only active in the current scope after calling `using`. This means that they won't be active outside the file where it is loaded in, or when an instance of the class is passed outside of the file. 

```ruby
class Dog
  def bark
    puts "woof"
  end
end

module DogFetch
  refine Dog do
    def fetch
      :bone
    end
  end
end

dog1 = Dog.new
dog1.fetch
# NoMethodError (undefined method `fetch' for #<Dog:0x00007f7c1e2851d0>)

using DogFetch

dog2 = Dog.new
dog2.fetch
# :bone
```

## `__FILE__ == $0`

`__FILE__` is a magic variable which refers to the name of the current file. `$0` is a variable that refers to the name of the file used to launch the program. A common use case for this is to conditionally run some code if the current file is the one used to start the program.

The following example defines a `Dog` class which may be imported and used by other the parts of the application. If the file is run directly, however, a new `Dog` would be instantiated and the `bark` method called.

```ruby
# dog.rb
class Dog
  def bark
    puts "woof"
  end
end

if __FILE__ == $0
  dog = Dog.new
  dog.bark
end

# person.rb
require_relative "dog"

class Person
  def initialize(has_pet = true)
    if has_pet
      @pet = Dog.new
    end
  end

  def pet
    @pet
  end
end

p = Person.new
p.pet.bark
# woof

# terminal
# $ ruby dog.rb
# "woof"
```