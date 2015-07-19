UX.  We have been converting many of our UI over to the wonderful new web-components specification via the Polymer framework. With our move over Polymer we started reducing our reliance on jQuery.  jQuery has really great built in support for event handling that's both efficient and intuitive.  With the removal of jQuery I found myself wanting to user "addEventListener" more frequently.  To be clear, Polymer does have event delegation, it's just not the same as jQuery and so takes re-learning things.

This desire to start using addEventListener all over the place lead to a rather lengthy discussion with my lead UI architect. This lead me to ask the question.  Given advances in javascript execution engines, increase CPU, Memory and SSD drives do we still need to optimize event handling in the browser?

The answer in my humble opinion is resounding depends... Event delegation is quicker on initialization, takes less Memory, and calls the handler quicker.  That said, if you are handling fewer than 5000 events on a page then it makes absolutely no difference as far as I can tell.  So if you have a web page with a hundred or so elements and 10 or 20 of them need to handle events then just use "addEventListener" and bypass the complexity of event delegation if you aren't using jQuery.  The extra code maintenance and complexity just won't really be worth it.  

However, if you are building a professional grade complex UI with thousands of potential html elements and even more event handlers, then you have a responsiblity as a professional to use event delegation.  

To test things out, I made a really simple test page that can attach events using either a delegated model or addEventListener.  If you want to crash your browser then crank both numbers up pretty high and run the expiriment. [Event Handling Test Page]({{site.url}}/assets/examples/add_event_listener.html)

When running 100,000 buttons with 10 event handers per button (1,000,000 total event handlers) I found that it took about 2 seconds to attach the event handlers using event delegation and about 7 seconds without.  so about 4 times slower when using addEventListener.  Additionally I found that Chrome used 825MB of Memory to attach and generate the event handlers and elements when not using event delegation and 487MB when it was used.  Given the result is exactly the same in terms of functionality, it's clear that using delegation makes the most amount of sense.

For completeness and because this is a blog about writing code the following two code example illustrate using event delegation and using simple addEventListener to achieve the same thing (at least in chrome, not tested in other browsers).

<h4>Basic event handling</h4>
{% highlight javascript %}
  var btn = document.createElement('div');
  btn.innerHTML = "basic clickable element";
  btn.addEventListener('click', function(){
    alert('clicked the button'); 
  });
  document.querySelector('body').appendChild(btn);
{% endhighlight %}

<h4>Delegated event handling</h4>
{% highlight javascript %}
  document.addEventListener('click', function(evt){
    if(typeof evt.target['onClick'] === "function"){
      evt.target['onClick'].apply(evt.target['onClick'], arguments);
    }
  });
  var btn = document.createElement('div');
  btn.innerHTML = "delegated event handling";
  btn.onClick = function(){
    alert('clicked the button'); 
  };
  document.querySelector('body').appendChild(btn);
{% endhighlight %}

