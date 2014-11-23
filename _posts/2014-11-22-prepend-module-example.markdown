---
layout: post
title: "Meta-programming in ruby with prepend module"
date: 2014-11-22 18:00:00
comments: true
categories: ruby rails metaprogramming module prepend
---

Recently my team and I started using ruby 2.1.1 in production on [www.musicxray.com](https://www.musicxray.com).  With this change we now that the ability to use the prepend method inside of our classes.  The prepend functionality allows a fairly clean syntax for wrapping existing functionality in custom code in a dynamic way.

This is a very simple example but I think it illustrates some powerful concepts.  What we are trying to do is give ourselves a module that when mixed in allows us to list the methods we would like to silence errors on.  We could use this functionality to automatically log errors to the log instead of breaking a user interface for our users in a rails application.  Here is the example I came up with.

{% highlight ruby %}

module Silence

  # expose the silence_exceptions_for method to the class
  # that included our module
  def self.included(base)
    base.send(:extend, ClassMethods)
  end

  # here is the main functionality.
  module ClassMethods
    def silence_exceptions_for(*methods)
      methods.each do |method|
        silencer = Module.new do
          # define method and pass all arguments along for the ride.
          define_method method do |*args|
            begin 
              # we can call super here because we are using prepend
              # without prepend we would need to do some yucky method
              # aliasing to get this to work.
              super(*args)
            rescue => ex
              "called exception"
            end
          end
        end
        prepend silencer 
      end
    end
  end

end

class TestClass

  include Silence

  silence_exceptions_for :test_method, :test_method2, :test_method3

  def test_method
    "test method without args"
  end

  def test_method2(name)
    "test method with arg name #{name}"
  end

  def test_method3(name)
    raise "this is an error"
  end

end

tc = TestClass.new
puts tc.test_method
puts tc.test_method2("hello")
puts tc.test_method3("woot")

# output: 
# test method without args
# test method with arg name hello
# called exception

{% endhighlight %}

I find this method of method wrapping much cleaner than using method alias, what do you think?

