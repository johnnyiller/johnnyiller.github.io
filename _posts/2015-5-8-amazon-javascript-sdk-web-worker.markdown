---
layout: post
title: "AWS SDK dynamodb web worker"
date: 2015-5-8 16:00:00
comments: true
categories: amazon web services web worker
---

A while back I decided to start tracking all the song playing data on [musicxray.com](http://www.musicxray.com).  Looking for a lightweight solution that wouldn't impact the performance of our application servers, I decided to send data directly from the browers to dynamodb.  

The first step you will want to do is create a custom user that has one permission enabled for put operations only.  This is important as you don't want your data being deleted or viewed by the outside world.  Basically we have a data bucket that we can send values into but can't get them out without a different user with more permissions.

Once you have your dynamodb table set up you'll want to start sending data to it.  I choose a very simple schema.  There are three fields on the table one id which serves as the hash and one date that serves as the range.  Lastly we have a "data" field that stores a stringified blob of json.

On the front end my first attempt was to send the data in the main javascript loop.  This turned out to work pretty well.  However, I wasn't satisfied with the UI responsiveness of the solution.  I decided to employ web workers to really push the data tracking task to the background as quickly as possible.

I ran into some minor issues with loading the amazon sdk in the web worker.  Hopefully the following helps save someone else a little bit of time.

{% highlight html %}
<!-- create an inline web worker important to specify the type so that the browser doesn't run the code -->
<script id="song_tracking_web_worker" type="javascript/worker">
  // this was the trick I needed to get the aws sdk to load.
  // web workers don't have a 'window' object but the library assumes 
  // there is a window object
  window = {};
  importScripts('https://sdk.amazonaws.com/js/aws-sdk-2.1.27.min.js');

  // initialize our dynamodb table and get it ready to accept values
  window.AWS.config.update({accessKeyId: 'XXXXXXXXXX', secretAccessKey: 'XXXXXXXJJJJXXXXX'});
  window.AWS.config.region = 'us-east-1';
  var table = new window.AWS.DynamoDB({params: {TableName: 'song_player_metrics'}});

  // take values we get from the main loop and jam them into the worker.
  // we could add a queue here if the order was important.  Since each request
  // is timestamped this was not a concern for me.
  self.onmessage = function(msg) {
    table.putItem(msg.data, function(e) {});
  };

</script>

<script type="text/javascript">
  /* <![CDATA[ */

  (function(){
    var user_id = '46';
    var session_id = '12929219212'; 
    var ip = "127.0.0.1";

    // start up the web worker.  This is a very handy trick so that you
    // can load up a webworker on the same page that defines it.
    var blob = new Blob([document.querySelector('#song_tracking_web_worker').textContent], { type: "text/javascript" })
    var worker = new Worker(window.URL.createObjectURL(blob));


    // this is our function that gets called after each second of audio finishes playing
    // some default data is passed in and we simply add to it before serializing it and
    // posting to the web worker
    $.mXray.mp3playing = function(data){

      var date = new Date().getTime();
      data.user_id = user_id;
      data.ip = ip;
      data.session_id = session_id;
      data.date = date;

      // using it as a very large persistent reliabe key value store can grow to a couple of terabytes before
      // we need to archive and start a new table....
      var itemParams = {
        Item: {
          song_id: {S: "wwwmusicxraycom:song_player:" + data.id}, 
          date: {N: "" + date},
          data: {S: JSON.stringify(data)}
        }
      };

      // fire and forget we are just logging listening activity.
      worker.postMessage(itemParams);

    };
  })();

  /* ]]> */
</script>

{% endhighlight %}

So that's my solution for tracking every second of play activity on our site in a low maintenance infinitely scaleable way.  If you find this at all useful then please feel free to comment.

