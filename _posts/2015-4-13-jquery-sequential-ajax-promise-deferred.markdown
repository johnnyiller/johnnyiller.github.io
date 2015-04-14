---
layout: post
title: "Call $.ajax sequentially via jquery deferred's"
date: 2015-4-13 16:00:00
comments: true
categories: jquery sequential javascript ajax  
---

This is just a quick tip for anyone searching the interwebs wondering how you can make a bunch of ajax calls on a page one right after another without turning async to false on the ajax request.  The problem you see is that if you call $.ajax directly in a for loop or as part of an each statement you will trigger a crap pile of ajax requests all at once.   In theory this is a good way to do things.  But in practice this could lead to one or more page loads consuming a large amount of server resources and slowing down your app for other users.  

Enter jQuery Deferred's. The following code outlines the jquery based code that would be required in order to execute the ajax requests sequentially:


{% highlight javascript %}

function getAjaxDeferred(url){
  return function(){
    // wrap with a deferred
    var defer = $.Deferred();
    $.ajax({ url: url, method: 'GET'}).complete(function(){
      // resolve when complete always.  Even on failure we 
      // want to keep going with other requests
      defer.resolve();
    });
    // return a promise so that we can chain properly in the each 
    return defer.promise();
  };
}
  
// using GET endpoints, will work with POST as well
var urls = ["http://www.example.com/url1","http://www.example.com/url2","http://www.example.com/url3"];

// this will trigger the first callback.
var base = $.when({});
$.each(urls, function(index, url){
  base = base.then(getAjaxDeferred(url));
});

{% endhighlight %}

If this is at all useful to you in your development efforts please hit me up with a comment at the bottom of the page.

