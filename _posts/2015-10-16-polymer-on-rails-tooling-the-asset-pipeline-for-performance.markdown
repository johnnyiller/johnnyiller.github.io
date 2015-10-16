---
layout: post
title: "Polymer on Rails, tooling the asset pipeline for performance"
date: 2015-10-16 16:00:00
comments: true
categories: rails polymer asset pipeline performance vulcanize
---

[Music Xray](http://www.musicxray.com) is continuing our migration from a jQuery plugin based UI system to a component based UI system.  There are many competing frameworks out there including Angular, React, and Riot that help solve the declaritive UI problem.  We choose polymer for a number of reason.  But the biggest reason we chose it, is that it is attempting to be standards based and thus natively supported in browsers.  That said, many browsers have not fully implemented the web components specification leading to slow performance on browsers yet to update.  The end result is that chrome is blazing fast, firefox is a close second, safari can get slow and IE is the worst offender.

Additionally, we are a Rails development shop so we would like our development workflow to function within the context of the asset pipeline as adding an entire frontend toolchain would increase the overall complexity of our build process.  This lead us to explore how we can successfully deliver a web component based frontend via the Rails asset pipeline without using tools like [vulcanize](https://github.com/polymer/vulcanize).

As we started to develop more and more components on the page two things became clear.  First loading a bunch of Web Components from different sources all on one page gets slow quickly.  This is no surprise for a Web Developer as we have seen this type of thing for years with loading lots of images, javascript, or css.  The solution is a simple one.

## Always concatenate your components

You should be delivering all of you components for a specific page as one file that has javascript and CSS inline so that you aren't making thousands of concurrent network requests.  In Rails this means processing assets via the asset pipeline.  Luckily that piece of the puzzle has already been solved by using the [polymer-rails](https://github.com/alchapone/polymer-rails) gem.  Simply follow the instructions on the README get your components into an application.html.erb file in your components directory and you are off to the races.  This gem is great and gets you 90% there in terms of performance. However, it's missing a useful optimization for browsers that don't support Web Components.

If your browser doesn't support web components (Safari, IE) then everytime you stamp out an instance of your component using your fancy new tag you it will cost you a performance hit in the form of trying to load in the style of javascript as if it's never seen it before.  This behavior does not happen on chrome as it's smart enough to know that it can just use what you already have.  The solution to this it turns out is simple as well.

## Always pull your javascript out of dom-module and put it into one script tag

The polymer rails gem does not have the ability to do that and so my team and I were left to come up with a solution on our own.  Turns out that the solution is not that difficult but like everything the devil is in the details so.  The following is a small piece of code we used to shift our javascript all to the bottom of the concatenated web component using Nokogiri.  


{% highlight ruby %}
# dependencies
# 
# gem 'nokogiri'
# gem 'polymer-rails'
#
# file: config/initializers/assets.rb

# inherit from a Tilt::Template to adhere to processing interface
class ComponentHoistingProcessor < Tilt::Template

  def prepare; end

  # we don't use context locals or block but they are here for documentation purposes
  def evaluate(context, locals, &block)
    # it is important to parse the components using HTML5 or else your 
    # custom HTML attributes can be deleted leading to strange behavior
    doc = Nokogiri::HTML5 data

    # build a string of all the javascript tags that have inner content
    concat_string = doc.css('script').inject("") do |result, script|
      # we want to leave linked items alone as that is not our job in this plugin
      if script['src'].blank?
        inner_script = script.inner_html
        inner_script = inner_script.strip
        last_line = inner_script.lines.last
        # if your javascript doesn't have a ; at the end we put one there
        # otherwise your javascript might bleed into the next components javascript.
        unless last_line =~ /\/\/|;\s*$|\*\/\s*$/
          inner_script << ";"
        end
        if inner_script.present?
          result << inner_script 
          # remove the original script from the document
          script.remove
        end
      end
      result
    end
    # the HTML5 parse creates a body tag if one doesn't exist.  We don't need the body tag
    # so we just take what's inside of it. Finally we add our javascripts to the end.
    "#{doc.css('body')[0].inner_html}<script>#{concat_string}</script>"
  end
end

# there are several processors that one can use, but in our case we want to process the bundle
# this means we get the assets after they have been concatenated. 
Rails.application.assets.register_bundle_processor 'text/html', ComponentHoistingProcessor 
      
{% endhighlight %}

Once you have this in your initializer you will have to clear out your asset cache and clear out any compiled assets you might have lying around.

{% highlight bash %}
rm -rf tmp/cache
{% endhighlight %}

Restart your webserver and you should be good to go.  Please let me know in the comments if this helped you out at all.







