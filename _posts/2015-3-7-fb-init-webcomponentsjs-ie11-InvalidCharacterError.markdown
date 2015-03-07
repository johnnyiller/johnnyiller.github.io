---
layout: post
title: "FB.init webcomponentsjs ie11 InvalidCharacterError"
date: 2015-3-7 16:00:00
comments: true
categories: facebook sdk webcomponents ie11 bug 
---

[music xray](https://www.musicxray.com) is in the process of a major overhaul to both our backend and frontend technologies.  One move we decided to make was to start using webcomponents.  As you may know, web components is not fully supported on every browser.  That's where [webcomponents.js](https://github.com/webcomponents/webcomponentsjs) comes in.  Webcomponents.js attempts to polyfill old browsers until they implement the web components specification.

Everything seemed to be going well until we included the facebook sdk on the page.  Just placing these two libraries on the page leads to following error.

{% highlight javascript %}
SCRIPT5022: InvalidCharacterError
File: webcomponents.js, Line: 3940, Column: 9
{% endhighlight %}

The error seemed cyptic and when you click on it you are taken to a function in the webcomponents library. After a ton of digging I finally figured out why.  Webcomponents.js overwrites the document.createElement function in it's effort to polyfill for old browsers.  Facebook on the other hand, assumes that you are on the standard document.createElement function.  This assumption means that facebook blows up in the presence of webcomponents.js

The answer that was given by the folks over at [webcomponents.js](https://github.com/webcomponents/webcomponentsjs/issues/208)  was that facebook is calling old code and they should fix it.  Well unfortunately in the meantime I need the site to actually work, so I put on my hacker hat and monkey patched document.creatElement to handle the junky call by the facebook sdk.

If you look at the [debug version of the facebook sdk](http://connect.facebook.net/en_US/sdk/debug.js) you will see the following javascript code about a quarter of the way down to page.

{% highlight javascript %}
if (hasNamePropertyBug()) {
  frame = document.createElement('<iframe name="' + name + '"/>');
} else {
  frame = document.createElement("iframe");
  frame.name = name;
}
{% endhighlight %}

The problem with the code comes from trying to create the iframe element using an html tag literal.  With a proper version of createElement, this operation is [no longer supported](http://www.w3schools.com/jsref/met_document_createelement.asp), thus the error.  To fix this issue, I load the following code immediately following webcomponents.js

{% highlight javascript %}

(function(){
  var old_function = document.createElement;
  document.createElement = function(argument){
    var matcher = /^<iframe/g;
    if(matcher.test(argument)){
      var dummy = old_function.call(document,'div');
      dummy.innerHTML = argument;
      var retel = dummy.querySelector('iframe');
      retel.name = retel.name;
      return retel;
    }else{
      return old_function.apply(document,arguments);
    }
  };
})();

{% endhighlight %}

Basically we wrap the document.createElement function calling the existing version under normal circumstances and our custom code to compensate for facebook's shitty code when needed.  Notice how we make a dummy element and set the innerHTML.  This is done so that we don't have to acutally try and parse html code with regular expressions, which can be difficult, we then find the element we are looking for and return it.

I understand that this code is a bit ugly, but desperate times call for desperate measures.  Please feel free to comment if you find this solution helpful.  I know that having this article available would have saved me quite a bit of time.


