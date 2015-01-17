---
layout: post
title: "Collections of PORO (plain old ruby objects)"
date: 2015-1-16 16:00:00
comments: true
categories: ruby poro collections enumerable
---

Often times when writing a ruby based application (yes that includes Rails) you end up with a bunch of objects that you would like to treat like an array but with special powers (okay methods).  This usually comes up when you want to start asking questions about your array.  

Example Scenario: let's say you have a list of animal objects and you want to know how many of them have 4 legs.  In your code you could iterate through the array every time and check for legs and then tally them up.  Or you could have an AnimalCollection object and simply ask the collection how many have four legs and receive a proper integer.  Now that we have a use case, here is an example how I would generally do this sort of thing by using the enumerable mixin.

{% highlight ruby %}

class Dog
  def legs
    4
  end
end

class Bird
  def legs
    2
  end
end

class AnimalCollection

  include Enumerable
  attr_accessor :animals

  def initialize(animals=[])
    self.animals = animals 
  end

  def each(&block)
    self.animals.each(&block)
  end

  def how_many_with_four_legs?
    self.select { |animal| animal.legs == 4 }.size
  end

end

animals = [Dog.new, Bird.new, Dog.new, Bird.new, Bird.new, Bird.new]

animal_collection = AnimalCollection.new(animals)

puts "There are #{animal_collection.how_many_with_four_legs?} animals with four legs"

# output: There are 2 animals with four legs

{% endhighlight %}

It's worth noting that you could subclass your AnimalCollection Class from Array.  However I would consider this an antipattern as the mixin offer a clean interface for making a collection without altering the the objects class hierarchy. 

If you agree disagree or otherwise found this article useful, then please feel free to leave a comment and i'll be sure to get back to you.


