---
layout: post
title: "Zopim custom button"
date: 2014-11-19 16:00:00
comments: true
categories: zopim javascript chat customer service
---

Recently, I was charged with integrating [zopim](https://www.zopim.com) with [musicxray.com](http://www.musicxray.com) in an effort to improve our customer support experience.  I must say, the software is really nice and functions about as good as I could have hoped for.

That said, if you have a design that does not allow for an ever present chat widget located in one of the four corners of you site, then you will have to use the [zopim api](https://api.zopim.com/files/meshim/widget/controllers/LiveChatAPI-js.html) and implement your own button.

![This is what we are going for](https://drive.google.com/folderview?id=0B6VpqbAXvH1qUVIxd3c5MzFuUlU&usp=sharing)

Notice the button on the right side of the page.  It fits with the theme of the site and is in a location of our choosing with an icon of our choosing.  The html markup to achieve this button on our site is something like the following.

### important: for this to work, you need to make sure that your zopim widget is hidden by default, you can do so using the zopim widget configuration interface

{% highlight html %}
<div class="live_chat_available hidden">
  <br/><br/>
  <a href="#" class="btn btn-warn">
    <i class="glyph-conversation multiline size24"></i>
    <span>
      we are here to help<br/> 
      live chat now
    </span>
  </a>
</div>
{% endhighlight %}

Your markup may be slightly different but for illustration purposes this should work.  The important thing is that the "hidden" class hides the contents of the div.  So you'll need to specify some css to make that happen.  Once you've got that markup on your site you will need to write a bit of javascript.  We are using jQuery for dom manipulation but that isn't a requirement.  The following should give you a good starting point for getting things working.

{% highlight javascript %}

// wrap everything in a closure so to avoid declaring variables and functions on the window object.

(function(){
  // this function gets called after the $zopim object is loaded via async script 
  function setUpCustomerServiceZopim(){
    $(document).ready(function(){

      // move the chat window up just a bit so that it 
      // doesn't cover the footer.  We are nudging the 
      // widget position just a bit, but you could 
      // customize in other ways as well.
      $zopim.livechat.window.setOffsetVertical(52);

      // it's pretty unlikely that a user wants the window 
      // open after they stop chatting...
      $zopim.livechat.setOnChatEnd(function(){
        $zopim.livechat.window.hide();
      });

      // when we start a chat we will fetch some extra data via ajax and sync it up
      // so that the customer support agent knows who they are dealing
      // the resp object looks like {email: "email@example.com", name: "username", tags: ["Artist"]}
      $zopim.livechat.setOnChatStart(function(){
        $.ajax("/public_contents/supplimental_chat_data").success(function(resp){
          $zopim.livechat.set(resp);
          $zopim.livechat.addTags.apply($zopim.livechat.addTags,resp.tags);
        });
      });
      
      // make the chat links on the site hide and show based
      // on whether or not we have an agent present. The key here
      // is to listen for status events and adapt the UI
      $zopim.livechat.setOnStatus(
        function(status){ 
          var chatLinks = $('.live_chat_link_item');
          $('.live_chat_available').addClass('hidden');
          // if there is an agent online then show the button
          // otherwise don't show the button. Also attach
          // an event handler that launches the widget 

          if(status == "online"){
            // removing hidden will show the div when agents are online
            $('.live_chat_available').removeClass('hidden');
            $('.live_chat_available a').on('click', function(event){
              event.preventDefault();
              $zopim.livechat.window.show();
            });
          }

        }
      );

    });
  }

  // this is a slightly modified version of the widget embed code
  // the embed code is loaded asyncronously so we need to register a
  // onload listener on the script tag and make a function that calls
  // our setUpCustomerServiceZopim() function that we just defined.
  // additionally you will want to swap out [YOUAPIKEY] with your 
  // actual API key
  window.$zopim||(function(d,s){var z=$zopim=function(c){z._.push(c)},$=z.s=
      d.createElement(s),e=d.getElementsByTagName(s)[0];z.set=function(o){z.set.
      _.push(o)};z._=[];z.set._=[];$.async=!0;$.setAttribute('charset','utf-8');
      $.src='//v2.zopim.com/?[YOURAPIKEY]';z.t=+new Date;$.
      type='text/javascript';e.parentNode.insertBefore($,e);
      $.onload=function(){setUpCustomerServiceZopim()}})(document,'script');

}());

{% endhighlight %}

If you found this little snipped helpful or harmful then please leave a comment so we can discuss thins further.



