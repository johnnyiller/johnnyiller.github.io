---
layout: post
title: "python builder pattern"
date: 2018-2-6 16:00:00
comments: true
published: false
categories: serverless AWS python pattern software design
---

After years of ruby and javascript I've started doing some work with python recently after joining the team at [Agero](https://www.agero.com/careers).  One of the first things I noticed was that a lot of folks in the python world pass around a lot of named arguments as a convention.  My first example of this is the boto3 library used to interact with AWS in python [(for proof, click here)](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html#DynamoDB.Client.scan)

In an effort to reduce some of this parameter passing folks might be tempted to make their own class that reduces the number of parameters one needs to pass around.  I offer a simpler functional solution taken from some patterns I've seen used in the functional javascript community. 

Enter partial function application with a builder pattern.  The following code highlights how one might create reusable functions without creating a domain specific class hierarchy.

Take the boto3 library.  More specifically, take the scan function for a dyanmodb table.  The following is an example of how one could use this concept to produce a convenient way to iterate through all items in a dynamo collection.

{% highlight python %}
# ./function_builder.py
import functools

class FunctionBuilder:

    def __init__(self, func):
       self.func = func

    def having(self, *args, **kwargs):
        self.func = functools.partial(self.func, *args, **kwargs)
        return self

    def get(self):
        return self.func

# ./dynamo_iterator.py
class DynamoIterator:
    
    def __init__(self, func):
        """Given dynamodb function that returns items and a LastEvaluatedKey iterate
    
        Keyword arguments:
        func -- function that takes LastEvaluatedKey and returns { items:  [] }
        """
        self.func = func
        self.last_evaluated_key = None
        self.first_fetch = True

    def _get_items(self, db_result):
        return db_result.get('Items', [])

    def _has_more_results(self):
        return self.last_evaluated_key and len(self.data) == 0

    def __iter__(self):
        """Required to implement the Iterator interface in python"""
        return self

    def __next__(self):
        """Required to implement the Iterator interface in python"""
        if self.first_fetch or self._has_more_results():
            self.first_fetch = False
            result = self.func(LastEvaluatedKey=self.last_evaluated_key)
            self.last_evaluated_key = result.get('LastEvaluatedKey', None)
            self.data = self._get_items(result)

        if len(self.data) <= 0:
            raise StopIteration

        return self.data.pop()

    """Python 2.7 requireds next vs 3.7 that requires __next__ hence we alias the function"""
    next=__next__
{% endhighlight %}

Now that we have these two simple base classes we can iterate through a collection of dynamodb objects using the following code.

{% highlight python %}
# ./main.py
import boto3
from .dynamo_iterator import DynamoIterator
from .function_builder import FunctionBuilder

dynamodb = boto3.resource('dynamodb')
table = self.dynamodb.Table('my-table-name')

partial_scan_all = FunctionBuilder(table.scan).having(Limit=100, Select='ALL_ATTRIBUTES').get() 

# this will iterate through every item in a dynamodb table.  Obviously not meant for quickly returning values to a user interface.
for item in DynamoIterator(partial_scan_all):
    print(item)
{% endhighlight %}

Hopefully you found this code example useful. If you have any questions please leave a comment and I'll try to help with an answer.

