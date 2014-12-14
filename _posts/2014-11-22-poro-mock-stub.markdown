---
layout: post
title: "PORO and Minitest for mocking and stubbing in tests (part1: Dependency injection)"
date: 2014-11-22 16:00:00
comments: true
categories: ruby rails testing minitest poro mock_stub
---

Let me start by saying I've never been a huge fan of rspec.  I think the magic it provides shields new developers from learning and understanding how ruby works, learning and understanding is partly what unit testing aims to offer in the first place.  Over the years I've decided that if a mock or a stub is necessary to test a piece of code that I would simply write it using plain old ruby and where necessary using using Minitest::Mock. 

The easiest way to test your code is to design it with with testability as a concern.  To be clear; if you can write code that is *clear, concise, and testable that's great*.  However, you should not sacrifice clear and concise code just to make it testable. 

One thing I do to make my code more testable, is, I try to pass in dependencies where possible (dependency injection) vs allowing the class itself to instantiate another object.  This allows the developer to easily pass in a mock object.  Here is a concrete example:

{% highlight ruby %}
class SimpleObject
  def say_hello
    "hi"
  end
end
class DifficultToTest

  def greeting
    obj = SimpleObject.new # this is the kind of code you want to avoid for unit testing
    "#{obj.say_hello} sir"
  end

end

# to use this class you would do the following
dtt = DifficultToTest.new
puts dtt.greeting

# result: hi sir

{% endhighlight %}

If you wanted to write the same code, but write it with testability in mind you would write it the following way.

{% highlight ruby %}
class SimpleObject
  def say_hello
    "hi"
  end
end

class EasyToTest

  def greeting(obj:)
    "#{obj.say_hello} sir"
  end

end

# to use this class you would do the following
ett = EasyToTest.new
obj = SimpleObject.new
puts ett.greeting(obj: obj)

# result: hi sir

{% endhighlight %}

This is a very very very simple example, but the difference is huge when it comes to testability.  We would like to use a mock object in place of the SimpleObject class while testing both the DifficultToTest class and the EasyToTest class.  To test the DifficultToTest class with a mock for the SimpleObject we would have to do the following using something like minitest.

{% highlight ruby %}

mock_simple = MiniTest::Mock.new
def mock_simple.say_hello; "hello"; end 

SimpleObject.stub :new, mock_simple do 
  dtt = DifficultToTest.new
  assert_equal "hello sir", dtt.greeting
end

{% endhighlight %}

If it's not clear why we are using a mock for this test then let me explain a bit more.  We are using a mock to isolate the function of the SimpleObject class from the DifficultToTest class.  If for example say_hello was a function that required a network call or some lengthy computation, you wouldn't want to call it over and over again in your tests.  You'd want to test that one lengthy function call once and then in all other cases use the mock object.  Keeping tests fast is what keeps you running tests often and running tests often is the key to making sure they are actually passing and your code isn't broken.

Back to the second example.  With the SimpleToTest class we would write the test in the following way.
 
{% highlight ruby %}
mock_simple = MiniTest::Mock.new
def mock_simple.say_hello; "hello"; end 
ett = EasyToTest.new

assert_equal "hello sir", ett.greeting(obj: mock_simple)

{% endhighlight %}

I hope it is clear to see, that the second example is far clearer and easier to understand.  This would be even more evident if we had multiple dependencies inside of the greeting method.  One thing I am missing in this example is expectation checking.  In a simple case I would likely do the following if I wanted to test that the say_hello method was actually called.

{% highlight ruby %}
obj = Object.new
def obj.say_hello
  @__said_hello__ ||= 0
  @__said_hello__ += 1
  "hello"
end

ett = EasyToTest.new
assert_equal "hello sir", ett.greeting(obj: obj)
assert_equal 1, obj.instance_variable_get(:@__said_hello__)
  
{% endhighlight %}

now we have a test that has no use of an external magic mocking and stubbing library yet we have all the benefit and clear understanding of how the ruby language works.  That's all for now. 


