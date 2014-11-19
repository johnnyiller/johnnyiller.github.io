---
layout: post
title: "Convert json to a useful ruby object"
date: 2014-11-19 16:00:00
comments: true
categories: ruby rails json oop tips tricks
---
In a quest to find the absolute easiest way to create a useable object from a json doc, I read some of the ruby stdlib documentation and like so many issues I've come across, I quickly discovered that the briliant ruby community already had a simple solution for me.

Many 3rd party RESTful API's deliver json as the response.  The issue, is that I would like to take a json structure like the following and interact with it in a sane manner. 

{% highlight javascript %}
{
  "firstname": "jeff",
  "lastname": "durand",
  "address": {
    "street": "22 charlotte rd",
    "zipcode": "01013"
    "residents": 1
  }
}
{% endhighlight %}

Before figuring out how to use the JSON parse options in Ruby, I thought that I would just have to navigate the array.  So my old code looked like this.

{% highlight ruby %}
require 'json'
json_string = '{"firstname":"jeff", "lastname":"durand", "address": { "street":"22 charlotte rd", "zipcode":"01013", "residents": 3 }}'
hash = JSON.parse(json_string)
puts hash["address"]["street"]

# result: 22 charlotte rd

{% endhighlight %}

I thought there must be a more natural rubyish way to handle the data instead of traversing a hash.  Turns out that the JSON.parse command actually takes a number of options.  One of those options is *object_class*.  If you combine this object_class with *OpenStruct*, you get a really easy way to generate usable objects from raw json documents.

The final code looks like
{% highlight ruby %}
require 'json'
# OpenStruct is not included by default so you have to add it.
require 'ostruct'

json_string = '{"firstname":"jeff", "lastname":"durand", "address": { "street":"22 charlotte rd", "zipcode":"01013", "residents": 3 }}'
json_object = JSON.parse(json_string, object_class: OpenStruct)
puts json_object.address.street

# result: 22 charlotte rd

{% endhighlight %}

The object_class can be any class, so it's completely possible to create your own class that inherits from OpenStruct but has better methods for object serialization or access. A really nice combination would be to use [ahoward's map gem](https://github.com/ahoward/map) 

That's all for this post, hope you found it useful

