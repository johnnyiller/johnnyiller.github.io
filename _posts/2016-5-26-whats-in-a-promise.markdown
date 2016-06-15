---
layout: post
title: "What's in a Promise (Javascript)"
date: 2016-5-26 16:00:00
comments: true
published: false
categories: javascript promises async 
---

I have been using javascript Promises in some capacity or another for the the past 4 years.  First as jQuery deferreds and then as Q promises and finally as the native Promise interface implemented in es6.  After using promises for so long it finally occurred to me that if somebody asked me to implement a simple Promise library I wouldn't really know how to do it without doing some research.  This article hopes to teach the basics of promises and how you might implement a simple Promise interface.  Please use a professionally maintained version in your production applications, this knowledge is meant to illustrate a concept not provide a production ready system.

<iframe width="100%" height="315" src="https://www.youtube.com/embed/llDikI2hTtk" frameborder="0" allowfullscreen></iframe>

# back to the code.

For the un-initiated, Promise based javascript interfaces offer a programmer a way to avoid callback hell and gracefully handle errors in an async javascript environment (you know like in the browser or node.js).  This technique is useful when you have say multiple ajax queries to multiple endpoints then you need to combine the result and render to a user interface.  Since each request executes independently you don't know which request will complete first. Perhaps promises are best illustrated with a simple example.


{% highlight javascript %}
// three ajax request without promises.....

function getUrl(url, callback){
  var request = XMLHttpRequest();
  // get list of comment resources for a user
  request.open('GET', 'http://example.com/endpointpath/user/1/comments');
  request.onload = function(){
    callback(request);
  };
  request.send();
}

function commentsGathered(commentDetailsArray){
  // display result to the user....
}

getUrl("http://example.com/endpointpath/user/1/comments", function(request){
  // returns.. { comments: ["http://example.com/endpointpath/comment/1", "http://example.com/endpointpath/comment/2"] }
  // we want to get details of each of the comments by calling these two endpoints
  // to handle this we need to introduce a callback after each comment completes
  // I will admit that this code is not the best option, in fact this is usually where I reach
  // for a promise library cause my head starts to hurt.
  var commentUrls = JSON.parse(request.response).comments;
  var commentDetails = {};
  
  commentUrls.forEach(function(url){
    getUrl(url, function(){
      commentDetails[url] = JSON.parse(request.response);
      if(Object.keys(commentDetails).length == commentUrls.length){
        commentsGathered(Object.values(commentDetails)); 
      }
    })
  });
  
});

// so you can see how this code might get complicated quickly.
// Imagine commentsGathered was an async process requiring a callback 
// and further, you wanted to handle error conditions if each or 
// any of the request had an issue....  Although the code isn't much
// shorter the promises interface offers a simpler way to organize your code.
// here is the Promise based example using es6 promises.

// returns a promise.... no callback specified...
function getUrl(url){
  return new Promise(function(resolve, reject){
    var request = XMLHttpRequest();
    // get list of comment resources for a user
    request.open('GET', 'http://example.com/endpointpath/user/1/comments');
    request.onload = function () {
      if (request.status === 200) {
        // parse request here cause it makes sense to do so.
        resolve(JSON.parse(request.response));
      } else {
        reject('This is not OK.');
      }
    };
    request.onerror = function () {
        reject('error could not reach endpoint');
    };
    request.send();
  });
}
// we've added error handling to the promise version cause it's
// actually manageable using promises.

// here is the implementation using our lovely promises interface

getUrl("http://example.com/endpointpath/user/1/comments").then(function(commentResp){
  var commentUrls = commentResp.comments;

  // generate array of promises based on the returned urls
  var commentDetailPromises = commentUrls.map(function(url) { return getUrl(url); });

  // create a new promise based on the  completion of an array of promises
  // returning the promise basically shifts the then context for the
  // next call to then...
  return Promise.all(commentDetailPromises);

}).then(function(commentDetails){
  // handle what you want to do with the detailed comments...
}).catch(function(err){
  // as a bonus you can easily handle any error you encounter...
});


{% endhighlight %}


I find the second example much easier to reason about and additionally much easier to change and maintain.  Since the root of all evil for software development is the requirement to change, it makes sense that we would write our code to changeable above most other requirements.  If nothing else I hope this example has convinced you to use promises in your code to simplify the async craziness.  

That said, we haven't yet explored in any detail how a promise library actually enables this sort of functionality.  The rest of this post will be dedicated to describing the basic mechanisms that get promises to be "thenable", that is, to have "then" method and use it correctly.  Additionally we will create our own class level "All" method to handle an array of async requests. 



