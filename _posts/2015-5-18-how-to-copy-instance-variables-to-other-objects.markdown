---
layout: post
title: "How to pass instance variables from one object to another."
date: 2015-5-18 16:00:00
comments: true
categories: ruby metaprogramming 
---

I have been doing some mentoring for [the firehose project](http://www.thefirehoseproject.com/).  Recently my student was a bit puzzled as to why she could access an instance variable in a view but not access them in a model after declaring them in a controller.  Long story short is that rails doesn't try to pass instance variables to models (for good reason) so it is not done.

To further explain and shed some light on what I think rails is doing behind the scenes, I decided to whip up this quick example of how one might go about shuffling instance variables around.  

To be clear, I am not saying rails actually does things this way, but only that in ruby moving values from one object to another is very easy and so the tendency to do such things happens naturally when using the language.  That said, what's natural for a seasoned ruby programmer could seem downright insane for new developers or developers with limited ruby knowledge.  

Please feel free to take the code that follows, put it in a file and run it to see it in action.  Try adding instance variables to see that it works without additional implementation code.

{% highlight ruby %}

class SampleController

  attr_accessor :secondvar

  def initialize(firstvar: "myfirstvar", secondvar: "mysecondvar")
    @firstvar = firstvar
    self.secondvar = secondvar
  end

  def third_var=(thirdvar)
    @thirdvar = thirdvar
  end

  def copy_instance_vars_to_view(view_instance)
    
    self.instance_variables.each do |instance_variable|
      view_instance.instance_variable_set(instance_variable, self.instance_variable_get(instance_variable))
    end

  end

end

class SampleView


  def initialize
    @fourth_var = "myfourthvar"
  end

  def print_out_instance_variables
    self.instance_variables.each do |instance_variable|
      puts self.instance_variable_get(instance_variable)
    end
  end

end

controller = SampleController.new
controller.third_var = "something custom"

sample_view = SampleView.new

# pass the view into the controller.  In rails the controller might 
# actually instantiate the view but the same principle applies
controller.copy_instance_vars_to_view(sample_view)

sample_view.print_out_instance_variables


{% endhighlight %}

