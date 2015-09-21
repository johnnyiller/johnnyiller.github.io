---
layout: post
title: "Using arel instead of strings in rails 4"
date: 2015-9-21 16:00:00
comments: true
categories: ruby rails arel mysql
---

## First, what is Arel?
As I understand it, Arel is the lazy loading sql generation library that backs ActiveRecord or more specifically ActiveRelation.  It allows framework developers to reduce coupling between the famework  and the sql database by offering a standardized API to generate sql queries.  Arel is not an ORM but it can be used to build an ORM (object relational mapper).

## Why would you want to use Arel?
At first you may think that arel is one of these things that should be reserved for times when you need to do something wildly complex.  I'm here to urge you to consider using it for any condition that you would normally try to write a string for.  The result will be more stable code that will last a longer time and will make you look like a badass. Additionally you will get sql injection attack prevention for free.

To illustrate, the following are three ways you might retrieve the same information using active record.

{% highlight ruby %}

# first and worst way to write the query notice the use
# of my age directly in the query string, this is just aweful
# and must be avoided.  never pass a variable directly to a 
# sql string like this, you open youself up to all sorts of attacks.
User.where("age > #{myage}")

# better, certainly more secure but it has issues.  If you 
# try to compose this query with another table that has an
# age column you will see an error because your sql server won't know
# which age you are refering to.
User.where(["age > ?", myage])

# This is the best choice for several reasons. First, name is
# escaped to prevent injections but additionally the age column
# is scoped to the 'users' table no matter what kind of join you make 
# additionally the entire statment is codified so a change to the
# sql implementation would not impact this codes ability to perform
User.where(User.arel_table[:age].gt(myage))

{% endhighlight %}

This is a simple example and in my opinion arel would be the obvious choice for this type of condition, but you'd be shocked at how many times I've seen people write  strings in order to build this type of query in rails.

In case it isn't clear why this the third option is better, it basically comes down to composability.  ActiveRecord allows you to merge scopes on a join.  If your database table that you were joining on just happened to have an age column you would get an error when you go to use the string version of the scope.  This can be completely avoided by using Arel and so you should just use it by default.


Another common problem that Rails 4 has is that it doesn't do so well with OR conditions.  Basically it's unsupported via scopes and most of the documentation would have you write an OR condition using a crazy hacked up string.  I urge you to resist writing code based on that documentation.  Instead, you should use Arel to codify a complex or conditions whenevery possible.  

Let's keep with the age problem.  Let's say we had a list of age ranges and we wanted a user to be able to search a database of users based on age ranges they were interested in.  This problem is a bit complex right. You'll want to find all the users that are in each of X number of age ranges. Let's say a user choose the following from our selection list.

1. ages 0 - 12
2. ages 24 - 35
3. ages 45 - 60
4. ages 66 - 68

How would we write a method to handle this in rails.  First we would want to parse out the ages.  Let's just assume we have code that does that and returns an array of ranges. We would get something like this.

{% highlight ruby %}

# ages in range form
ranges = [0..12, 24..35, 45..60, 66..68]
table = User.arel_table

# pop the last one off so we have a good starting point
starting = table[:age].in(ranges.pop)
query = ranges.inject(starting) do |result, element|
  # add the last or to the previous or condition...
  result.or(table[:age].in(element))
end
# query will be an arel relation.  We need to use a where clause
# to wire it up with ActiveRecord
User.where(query)
{% endhighlight %}

## If you still aren't convinced.  Consider this

Executing a sql query is the equivalent of evaling javascript or ruby code.  It's very difficult to find errors and debug when things go wrong because the entire program is a string that doesn't get executed until runtime.  In contrast codifying your sql will allow you to catch syntax errors before they make it into any of you environments.  

If you'd like to learn more about how ActiveRecord constructs queries and more generally you want to become a rails ninja then I encourage you to explore the [README and documentation on github](https://github.com/rails/arel).











