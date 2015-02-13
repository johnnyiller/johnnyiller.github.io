---
layout: post
title: "Bypass Strong parameters for a single ActiveRecord model"
date: 2015-2-13 16:00:00
comments: true
categories: ruby rails activerecord strong parameters
---

Upgrading to rails 4 from rails 3.2 has been pretty smooth with one exception: Strong parameters.  Strong parameters is the most current iteration on how rails handles parameters sent from the view to the controller then eventually passed to the model.  The idea is that the controller should be in charge of deciding which parameters ultimately make it the model.  

The problem that I'm seeing, in practice, is that I have a lot of models and classes that reference other models and so there isn't really a one to one relationship between the controller and the model in question.  Instead the model is hidden in a sea of other objects.  In some of these deeply nested objects I neither want nor need strong parameter protection.  Now as far as I know, and please correct me if i'm wrong, there is no convenient way to turn off strong parameters temporarily for one model only at certain times.  So after much searching I decided the easiest solution would actually be to write my own mini ActiveRecord plugin.  Basically the idea is to rewrite the new and create method in an unsafe manner.  Here is my simple solution.  Hope you find it useful.


{% highlight ruby %}
# place this code in an initializer config/initializers/unsafe_creation.rb
module UnsafeCreation

  def create_unsafe(params)
    n = self.new
    params.each { |k,v| n.send("#{k}=", v) }
    n.save
    yield(n) if block_given?
    n
  end

  def new_unsafe(params)
    n = self.new
    params.each { |k,v| n.send("#{k}=", v) }
    yield(n) if block_given?
    n
  end

end
# make sure all active record based models have these methods...
# if you want to only load it into specifc models then leave 
# this part off and just extend each model individually.
ActiveRecord::Base.send(:extend, UnsafeCreation)

# Usage:

# Assuming we have an ActiveRecord based model like the following
class Post < ActiveRecord::Base
  # fields in db..  body, title
end
# we could then get an instace of the model with our parameters like this
post = Post.new(body: "my small body", title: "my title")
post.save

{% endhighlight %}


