---
layout: post
title: "A gentle introduction to Machine Learning"
date: 2016-12-1 16:00:00
comments: true
categories: machine learning
---

I recently completed a great course on Machine Learning, the course is offered by [coursera](https://www.coursera.org/learn/machine-learning) so check it out if you find this article interesting.  Until now, I've only been a passive consumer of machine learning in my own application development and home life [Amazon Echo](https://www.amazon.com/gp/product/B00X4WHP5E).  As a scientist and engineer, I always knew that some interesting algorithms were crunching data and then making complex decisions behind the scene, but I didn't really understand the mechanics of that process.  I believe writing is a great way to share information as well as solidify one's understanding about a particular subject and so it is with that reasoning that I write the following article.  If you don't care for understanding how machine learning works but are more interested in why it's important right now feel free to skip over the details.


# The basic idea behind machine learning.

Machine learning is concerned with looking at a source of data and figuring out (fitting) a mathematical function that characterizes the behavior captured in that data.  Let's say we have a farm and that farm uses water and fertilizer we record that data every year and then associate it with a corn yield.  So that table of data may look something like the following.

|fertilizer(lbs)|Water (G)  |Corn(lbs)  |
|---------------|-----------|-----------|
|100            |600        |800        |
|110            |400        |600        |
|80             |700        |500        |
|120            |600        |810        |

The challenge is to find a mathematical function such that given some value of water and some other value of fertilizer, we predict the amount of corn output we should see for the season.  Anyone familiar with basic economic analysis will recognize this as a simple linear regression problem than can be solved quickly with a little linear algebra.  Typically one would never use machine learning to solve such a simple problem as deterministic methods work just fine at this scale.  

## What does this have to do with machine learning?  

As the number of data points gets larger and you have more and more components contributing to the "Corn" yield, the calculations required to solve the problem via deterministic methods become more and more computationally expensive and complex.  At a large enough scale you could have a problem that would take an entire lifetime to solve.  Thus we have to use more involved mathematical techniques that estimate the solution rather than solving exactly.

In addition to complex linear problems, we might not be trying to find linear functions at all.  This leads us to explore techniques for finding more complex functions that can make better predictions given a new set of inputs (features).  It turns out that machine learning techniques can fit a wide variety of functions and thus have broad appeal across domain models.

# Least Squared Errors (linear models)

Let's say I have some data and I'm looking for a function to characterize that data.  A measure of how well my function does at classifying the data could be how far away from a line each data point was.  This is the error between my function and the data it represents.  In order to account for points below and above the line and count them equally, one might square the error.  If you add up all the squared-error values you get for a particular function, you have a way to evaluate how good your function "fits" the data you have.

It's worth noting that different problems involve different functions that measure the amount of error, but the premise remains the same.  The error produced is often referred to as the cost of the model and thus the error measuring function is referred to as a Cost function.

If you have a way of measuring how much error your function has then you have an objective way of deciding if one function is better than another.  Machine learning techniques offer us a way to discover the function that works the best without looking at an infinite number of possible solutions.

# Gradient Descent

The next logical question is how do I quickly find the best function given an infinite number of possible functions.  The answer lies in differential calculus.  Calculus tells us that if we take the derivative of function and set it equal to 0 then solve the equation we will get the optimal values.  Now since we don't know the function in advance, things aren't so simple, but we can exploit this truth. Mainly we can get a sense of the direction where the minima lies by calculating the partial derivatives given our current best function.  We can then use this information to take a small step in a direction towards a better function.  We do this until a specific number of iterations or until we are no longer making much progress at which point we stop and assume our solution is good enough.  Here is a video illustrating the concept clearly.

<iframe width="100%" height="400" src="https://www.youtube.com/embed/eikJboPQDT0" frameborder="0" allowfullscreen></iframe>

This technique forms the foundation for most machine learning problems. Solving different problems is simply a matter of formulating them so that they can be solved using this technique.


# Why is machine learning so exciting?

The reason machine learning is exciting is because a huge number of problems can be re-formulated as probabilistic models.  Meaning sometimes we only care that an answer is close enough to the correct answer and we'd be willing to trade compute time human or computer to get an answer that's 99.999% correct. Furthermore, some problems only make sense in terms of probabilities.  For instance, if you are predicting if it's going to rain 5 days from now it would not make sense to say there is 100% chance of anything happening in the future.

# why is machine learning so much more exciting lately?

The reason is pretty simple.  We now have access to massive amounts of data, the likes of which the human race has previously never recorded (the amount of data gathered per day is accelerating).  Secondly, the semiconductor industry has managed to make silicon that is particularly well suited for Linear Algebra calculations (even if they didn't, the fact that many linear algebra operations can be parallelized over multiple machines means we could just use more off the shelf computers to do our calculations anyway).  Thus, we now have the compute infrastructure to make use of this ever increasing data set.  

## So what? how does more data improve the solutions to our machine learning problems?  

It turns out that there are a huge number of problems that perform better, the more data one has.  Think about what we are trying to do. We are making mathematical models that estimate the real world, many times the data we have describes the real world, the better our description of the real world the better our understanding could potential be and thus the better our prediction about what might happen in that world might be.  Having more data and compute power allows computers to understand the world through data in ways that have never been possible.

# How does all this relate to Artificial Intelligence?

Think of artificial intelligence as systems of machine learning models adapted for human consumption.  The idea is that you want to make a system that can take some input, run it through 1..n different machine learning algorithms and other filtering processes and then spit out a useful result for a human.  So machine learning algorithms are just one of a number of sub-systems that make up an artificially intelligent system.  It could be that a system like Amazon's Alexa uses hundreds of different machine learning algorithms to present the user with one fluid experience. The average consumer will never interact directly with a machine learning model but almost everyone will interact with an artificial intelligence system.

# Conclusion and thoughts

Learning these concepts has been an eye-opening experience and I believe there are a lot of interesting applications that will be created as things progress in this space.  Clearly, this article is a Math lite summary about machine learning but I encourage you to learn more by taking the courera course (amazingly it's totally free).  I plan to use machine learning models to make more intelligent systems for hobby work (drones to chase the squirrels from my garden anyone?) or for profit (fraud detection, customer sentiment,product recommendations, and BI based decision making). I Can't wait to get started.
