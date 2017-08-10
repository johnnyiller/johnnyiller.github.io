---
layout: post
title: "Static asset headers in rails 5"
date: 2017-8-9 16:00:00
comments: true
categories: ruby rails caching assets headers
---

First things first.  Conventional wisdom tells us we should be serving our static assets from something like nginx because it's optimized for doing such a task.  Generally speaking I disagree with that thinking and instead offer a different solution.  One should be serving static content out of a CDN and that CDN should be caching your content at edge locations.

Given we want to cache content at edge locations we will want a mechanism for specifying in our rails application how we would like our our CDN to cache the content.  As in, how long should the CDN hold onto the content before returning to our origin server to fetch it again?  There are two ways that you might accomplish this.

1. Configure the CDN to cache the correct thing by overwriting headers produced by your application.
2. When a resource is requested from your origin provide the proper headers so that the CDN knows how to handle the asset.

In my opinion option 2 is much preferred.  Allowing your application to specify headers gives you more control and keeps the logic of your application where it belongs.  Assuming I've convinced you to use a CDN and configure the caching mechanism to work via headers specified at at the origin.  The logical question is how do I configure the proper headers such that my CDN will cache my assets.

The following code works with the fastly CDN service but probably works well with many others.

This code configures the browser cache and your CDN to cache assets for 10 years.  Since we are digesting all assets, meanings we are creating new versions if anything changes, we can cache these assets forever.

{% highlight ruby %}
# in your config/application.rb file
config.public_file_server.headers = {
  'Surrogate-Control' => 'max-age=94608000',
  'Cache-Control' => 'maxage=94608000, public, no-check',
  'Expires' => 1.year.from_now.httpdate,
  'Date' => 4.days.ago.httpdate,
  'Last-Modified' => 4.days.ago.httpdate
 }
{% endhighlight %}

The result of this config behind a proper CDN is that you assets will end up being fetched exactly once from your rails application and then never again until you clear the cache.  Even then, the browser cache will ensure that only new visitors to your site will need to request the assets.  In my opinion, Rails is perfectly well suited to serving up the occassional asset in this context and will likely have little impact on overall performance.  Furthermore keeping the caching logic in the code base has obvious utility including allowing caching headers to be tested more easily via unit tests.

As always, leave a comment if you found this useful.
