---
layout: post
title: "amazon product advertising api ruby signing requests"
date: 2015-5-17 16:00:00
comments: true
categories: ruby rails aws signed request
---

This week the marketing department at my company decided that they wanted to make a new landing page with a focus on social validation.  Social validation means that the advertisement references other people who like the service in an effort to pursuade people that if the service works for these people then it might also work for them.

As part of this effort the marketing team wanted to show Amazon Product Reviews for a book that was written about our website.  Normally I would just suggest copying a few reviews or taking a screenshot and pasting it, but recently Amazon.com began cracking down on this sort of behavior.  My guess is that Amazon.com wants to manage the presentation of the data and get proper attribution links. 

Thus, we needed to implement a programmatic way to query Amazon and get an iframe for displaying reviews on our landing page.  Documentation for gettting user reviews can be found [here](http://docs.aws.amazon.com/AWSECommerceService/latest/DG/EX_RetrievingCustomerReviews.html).  Notice that the link has a "RequestSignature" parameter.  It turns out that the primary challenge to getting the api request to work is in calculating this signature.  Luckily calculating this signature is [well documented](http://docs.aws.amazon.com/AWSECommerceService/latest/DG/rest-signature.html) and I was able to come up with a solution. I broke out my trusty text editor and came up with this piece of code.  Hopefully this saves you a couple minutes of your time so you can go back to drinking coffee and mingling at the water cooler.

{% highlight ruby %}

  class AmazonProductAdvertisingApiSigner
    
    attr_accessor :url, :secret

    def initialize(url:, secret:)
      self.url = url
      self.secret = secret
    end

    def sign
      host_and_path, params = *self.url.split("?")
      params = params.gsub(",","%2C").gsub(":","%3A")
      canonical = params.split("&").sort.join("&")
      data = ['GET', 'webservices.amazon.com', '/onca/xml', canonical].join("\n")
      sha256 = OpenSSL::Digest::SHA256.new
      sig = OpenSSL::HMAC.digest(sha256, self.secret, data)
      signature = Base64.encode64(sig).strip
      signature = signature.gsub("+", "%2B").gsub("=", "%3D")
      "#{host_and_path}?#{canonical}&Signature=#{signature}"
    end

  end

  # class usage.  
  # Note, you must use your own secret and accesskey get them by signing up at the following url
  # https://affiliate-program.amazon.com/gp/flex/advertising/api/sign-in.html

  signer = AmazonProductAdvertisingApiSigner.new(secret: "1234567890", url: "http://webservices.amazon.com/onca/xml?Service=AWSECommerceService&AWSAccessKeyId=AKIAIOSFODNN7EXAMPLE&AssociateTag=mytag-20&Operation=ItemLookup&ItemId=0679722769&ResponseGroup=Images,ItemAttributes,Offers,Reviews&Version=2013-08-01&Timestamp=2014-08-18T12:00:00Z")
  signed_url = signer.sign

{% endhighlight %}

For those of you developing a rails application this is how I ended up using the class in my controller.

{% highlight ruby %}

  def book_reviews
    # cache result, the result returned lasts 24 hours before needing to be refreshed
    cached_iframe_url = Rails.cache.fetch("book_review_iframe_url", expires_in: 23.hours) do 
      url = "http://webservices.amazon.com/onca/xml?AWSAccessKeyId=XXXXXXXXXXXX&AssociateTag=httpmusicxcom-20&ItemId=0977751228&IdType=ISBN&SearchIndex=Books&Operation=ItemLookup&ResponseGroup=Reviews&Service=AWSECommerceService&Timestamp="
      # timestamp format is important must be in UTC 
      timestamp_str = Time.now.utc.strftime("%Y-%m-%dT%H:%M:%SZ")
      url << timestamp_str
      signer = AmazonProductAdvertisingApiSigner.new(secret: "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX", url: url)
      signed_url = signer.sign
      # get the xml from amazon
      clnt = HTTPClient.new
      body = clnt.get(signed_url).content

      # use nogogiri to get the element we are after and pull out the iframe url
      xml_doc  = Nokogiri::XML(body)
      iframe_url = xml_doc.css('IFrameURL').text
      raise "not a good url" unless iframe_url.present? and iframe_url =~ /amazon\.com\/reviews\/iframe/i
      iframe_url
    end

    # lastly redirect the user to the iframe. 
    redirect_to cached_iframe_url 

  end
{% endhighlight %}

Finally put the iframe on the page

{% highlight html %}
  <iframe width="100%" height="600" src="/public_contents/book_reviews" frameborder="0"></iframe>
{% endhighlight %}

This page changes but if you want to see an example of the final work, [check it out](http://www.musicxray.com/i-am-a-music-artist).

I realize i'm mostly talking to myself on this blog but if anyone in on the interwebs finds this useful please leave a comment.
