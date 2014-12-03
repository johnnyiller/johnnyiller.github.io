---
layout: post
title: "Upsert, a necessity for modern api design"
date: 2014-12-3 16:00:00
comments: true
categories: api architecture design
---

I've had the good fortune of being able to work with a few NoSql databases lately.  One feature that they all share in common is that they each provide a really easy way to perform an upsert.  An upsert is where you insert a new entry or record if one has not been created else if one has been created you simply update what's already there with the new values.  This is a nice clever trick to remove logic from the application and save some code in the process.  Using mongo db, your code might look like the following, without upsert.

{% highlight ruby %}

client = MongoClient.new
db = client['example-db] 
coll = db['example-collection']
# assuming you don't know whether or not you have
# data in the collection, you need to query to find out

docs = coll.find({"email"=>"addr@example.com"})

if docs.size > 0
  coll.update({"email" => "addr@example.com"}, {"age" => 24})
else
  coll.insert({"email" => "addr@example.com", "enabled" => 24}) 
end

{% endhighlight %}

The example looks contrived, and it is if your age is always 24.  But if each person has a different age and you are now requiring age on signup but also allowing older users to fill in their age on a separate screen then this type of function would make a lot of sense.  So now that I've convinced you that this type of feature is generally useful, let's do the operation much more efficiently and without as much conditional logic in our code and without two separate database calls.

{% highlight ruby %}
client = MongoClient.new
db = client['example-db] 
coll = db['example-collection']
coll.update({"email" => "addr@example.com"}, {"age" => 24}, {upsert: true})
{% endhighlight %}

By simply adding upsert true to the update command we get the exact same functionality, but only make a single call to the database.  The database will take care of merging our age with our email address and inserting it if it needs too or just updating the record if that's what's called for.  Much much nicer.

Redis has a similar construct for upserting using the SET command.  This is really useful for establishing a temporary lock as redis also supports expire value.  The following code illustrates how to set a lock using upserting in redis.

{% highlight ruby %}
# assuming you've created a redis connection
# set the key mytestkey to a value of 100 with 
# an expiration of 30 seconds if and only if the key 
# does not currently exist.
redis.set("mytestkey", 100, {ex: 30, nx: true })
{% endhighlight %}

I won't go into all the details about how useful this is, but if you can think about trying to do something like this before nx was an option on the set command you would see that you would have to do additional work in your application logic.  Additionally, having one command helps avoid race conditions as it reduces the amount of time you are manipulating and syncing code in your application layer.

One last example to convince you that that upserting should not be an optional feature in modern api design. Amazon dynamodb includes it in their [UpdateItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_UpdateItem.html) api call by default. 

So clearly modern data manipulation API's should allow for upserting of data.  Recently I had to do some integration work with the contant contact api so that our marketing department could segement users by interest.  The experience left me longing for a better implementation.  One that included, you guessed it, *upsert* capability. 

### Failure 1
If one reads the [API documention](http://developer.constantcontact.com/docs/contacts-api/contacts-collection.html?method=POST) for creating a new contact they can see clearly that if they issue a post against the contacts endpoint and the email address is already in the system then you will receive the following 409 error.

*The email address provided is already in use*

### Failure 2
Looking at the [API documention](http://developer.constantcontact.com/docs/contacts-api/contacts-resource.html?method=PUT) for updating a contact one can see that you can not even update a user based on any other attribute other than the ID of the contact.  Clearly upserting doesn't work in this case because I have to find the id anyway in order to update the data.

This is wrong on a couple of levels 

1.  email addresses must be unique as established earlier, yet I need a special id to update one?  That does not follow how you would expect an api to work.
2.  I have no way of changing just a piece of the data even if I have the contact id.  I need to override the contact entirely.  This means I have to first pull down the existing contacts data merge in my new data in code and then send back the new merged data.  A lot of work that should be done by the API provider and not the API consumer.

The whole point of this post is to beg the API developers of the world (especially public companies like constant contact) to please consider how an application would actually go about integrating with your API and to develop the functionality that moves the data management to the API and away from the client application. One way to move in the right direction is to allow upsert capability.  If databases are doing it then so should your API's. 





