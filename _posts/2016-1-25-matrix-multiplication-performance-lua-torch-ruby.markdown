---
layout: post
title: "Matrix multiplication performance standard lua vs torch vs ruby"
date: 2016-1-25 16:00:00
comments: true
categories: luajit lua torch ruby performance
---

I decided very recently to re-learn my college linear algebra course with the goal of eventually moving on to deep machine learning neural networks.  Additionally, I recently started reading [Make it Stick](http://www.amazon.com/Make-It-Stick-Successful-Learning/dp/0674729013) and found many of the arguments compelling for retaining what I'm learning.  One of the theories as is that if you combine multiple subjects and work through practice problems regularly your results will be much better than if you just read a textbook even multiple times.  Thus, I decided that while I'm re-learning linear algebra I would additionally learn how to program in the [Lua](http://www.lua.org/) programming language.  This is the product of [my work so far](https://github.com/johnnyiller/lua_linear_algebra).  

As you can see from the source code, I decided to do a speed test for matrix multiplication comparing my implementation written in lua, and executed via luajit vs torch vs ruby.  The reason why I tested against ruby is becuase, ruby is my language of choice (for it's beautiful syntax), when it comes to most things and I wanted to see just how good or bad it really was when it comes to raw computation.  The results speak for themselves, and although I wasn't surprised that ruby was the slowest, I was surprised *how much slower it was*.

For the interested, here are the results of the tests running all on the same hardware m3.medium instance on AWS (Amazon Web Services).

## Lua matrix multiplication [written by me](https://github.com/johnnyiller/lua_linear_algebra/blob/master/lib/matrix.lua)
{% highlight lua %}
local x = os.clock()
Matrix = require("lib.matrix")

A = Matrix.new(10,10,0.0,1.0)
for i=1, 10 do 
  for j=1, 10 do 
    A.set(i,j,math.random())
  end
end

for i=1, 100000 do
  local result = A * A
end
print(string.format("elapsed time: %.2f\n", os.clock() - x))

--[=====[
takes approximately: 2.91 seconds
--]=====]
{% endhighlight %}

## Ruby matrix multiplication [Stdlib](http://ruby-doc.org/stdlib-2.2.3/libdoc/matrix/rdoc/Matrix.html)
{% highlight ruby %}

require('matrix')
require('benchmark')
n = 10000

Benchmark.bm do |x|
  x.report {
    A = Matrix.build(10,10) do 
          rand
        end
    n.times do 
      result = A * A
    end
  }
end
=begin
here are the results for ruby: time in seconds

user     system      total        real
   7.550000   0.000000   7.550000 (  7.895253)
=end
{% endhighlight %}

## Lua matrix multiplication [Torch framework](http://torch.ch/) 
{% highlight lua %}
local x = os.clock()
require 'torch'
A = torch.rand(10,10)

for i=1, 100000 do
 local B = A * A
end

print(string.format("elapsed time: %.2f\n", os.clock() - x))
--[===[
takes approximately: 0.39 seconds
--]===]
{% endhighlight %}

To summarize, **Torch** (written for lua) is amazingly fast, literally **20 times faster than ruby** and **7 times faster than my simplistic lua implementation**. Another interesting datapoint is that the simple lua program I wrote is almost 3 times as fast as the ruby implementation suggesting that luajit is just a higher performance language.  

The moral of the story is this, if you want to learn linear algebra or play around with toy problems then by all means program your own framework or use something like ruby.  However, if you are planning to do serious computation, it pays to look at tools that are optimized for raw computation.

With promising results like this, I'm looking forward to doing more work with Torch once I've refreshed my memory with the basic computation and practical application of linear algebra. 

