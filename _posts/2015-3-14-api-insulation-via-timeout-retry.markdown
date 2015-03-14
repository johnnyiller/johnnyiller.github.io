---
layout: post
title: "Ruby retry with timeout"
date: 2015-1-12 16:00:00
comments: true
categories: ruby retry timeout
---

The following post is mostly code as I believe that the code and tests are self explanatory.  The following code was written in an attempt to make API calls more resilient before failing.  Basically the idea is that 3rd party API's can have intermittent network issues and in production you'd be better off re-trying your code once or twice before completely failing and having your users see a 503. Really whenever you are making a call over the nextwork, it's probably good to wrap your code in something like the following.  Tests are provided to help the user understand how to use the utility function.

{% highlight ruby %}

require 'minitest/autorun'

module Utilities
  require 'timeout'
  # tries is the number of tries to execute the code
  # on is an array of the errors
  # timeout give an upper limit to the amount of time this method will run
  # and delay specifies the number of seconds to sleep between each retry
  def self.with_retry(retries: 2, on: [StandardError], timeout: 30, delay: 1, rescued_callback: nil)

    # http://ruby-doc.org/stdlib-2.1.1/libdoc/timeout/rdoc/Timeout.html
    Timeout::timeout(timeout) do 
      begin
        return yield 
      rescue *on => e
        sleep(delay)
        # callback code in case you want to do something special on 
        # failure like write to a log or what have you.
        if rescued_callback
          rar = rescued_callback.arity 
          if rar == 0
            rescued_callback.call
          elsif rar == 1
            rescued_callback.call(e)
          end
        end
        # retry if we have retries left
        retry unless (retries -= 1).zero?
        # re-raise the error if we don't
        raise e
      end
    end
      
  end

end

# custom errors for test purposes
class CustomError < StandardError; end
class CustomError2 < StandardError; end

class TestUtilities < MiniTest::Test

  def test_timeout_after_5_seconds

    start_time = Time.now.to_i
    assert_raises Timeout::Error do
      ::Utilities.with_retry(retries: 20, timeout: 5 ) do 
        raise "an error"
      end
    end
    end_time = Time.now.to_i
    
    assert_equal 5, end_time-start_time

  end

  def test_delay_works

    called = 0
    assert_raises Timeout::Error do
      ::Utilities.with_retry(retries: 20, timeout: 4, delay: 2 ) do 
        called += 1
        raise "an error"
      end
    end
    assert called <= 3

  end

  def test_tries_works

    called = 0
    assert_raises RuntimeError do 
      ::Utilities.with_retry(retries: 20, delay: 0 ) do 
        called += 1
        raise "an error"
      end
    end
    assert_equal 20, called

  end

  def test_wont_catch_just_any_exception

    called = 0
    assert_raises CustomError do
      ::Utilities.with_retry(retries: 3, delay: 0, on: CustomError2 ) do 
        called += 1
        raise CustomError.new("my custom error")
      end
    end

    assert_equal 1, called

  end

  def test_catches_specified_exceptions

    called = 0

    assert_raises CustomError do 
      ::Utilities.with_retry(retries: 3, delay: 0, on: [CustomError, CustomError2] ) do 
        called += 1
        raise CustomError.new("my custom error")
      end
    end
    
    assert_equal 3, called

  end

  def test_rescued_callback_works
    
    called = 0
    myerr = nil
    callback = ->(err){ called += 1; myerr = err }

    assert_raises RuntimeError do
      ::Utilities.with_retry(retries: 3, timeout: 4, delay: 0, rescued_callback: callback ) do 
        raise "test errr"
      end
    end
    assert_equal 3, called
    assert_equal "test errr", myerr.to_s
  end

end
{% endhighlight %}

If you find this function at all useful please comment below...
