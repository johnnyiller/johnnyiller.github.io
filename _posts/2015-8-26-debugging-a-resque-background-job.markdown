---
layout: post
title: "Debugging a resque background job for rails 4.2 using rails console"
date: 2015-8-26 16:00:00
comments: true
categories: ruby rails resque byebug debugger
---

As the title implies, this article assumes you are using rails 4.2 and resque for background jobs.  I suppose it also assumes that you want to quickly debug a piece of code.  To be clear, the best way to make sure your code doesn't have issues is to make sure you have full test coverage.  However, I had a problem recently that my test was saying was working but in production it wasn't working.  The only way I know to fix such a thing is to work through it with a debugger and fix and then write a test that recreates the situation.  Hopefully this short tutorial can help you if you find yourself in a similar spot.

## let's get started..

First, stop any background workers you might have running so that you don't process the job accidentally.

Next, If you are using ruby > 2.1 you'll want to be using byebug for all of your debugging needs.  Add it to your Gemfile.


{% highlight ruby %}
  # Gemfile
  gem "byebug"
{% endhighlight %}

Next run the following command from a terminal
{% highlight bash %}
bundle install
{% endhighlight %}

Now that we have *byebug* we are ready to get started.  Open up the offending code.  In my case the background job was being executed via the following code.

{% highlight ruby %}
  class GeneralBackgrounder < BaseTask
    extend Resque::Plugins::ExponentialBackoff
    @queue = :low_priority
    @backoff_strategy = [30, 60, 600, 3600, 7200]

    def self.perform(constant,action,options,*args)
      # add debugger stopping point here this will halt the execution and allow you to inpect what's happening.
      byebug 
      super(args)
      options ||= {}

      constant = const_get(constant)
      object = options[:object_id] ? constant.find(options[:object_id]) : nil
      if object.nil?
        object = options["object_id"] ? constant.find(options["object_id"]) : nil
      end
      item = object || constant
      execute_job_with_item(args, item, action, options)

    end

    def self.execute_job_with_item(args, item, action, options)
      if item.respond_to?(action)
        args.empty? ? item.send(action.to_sym) : item.send(action.to_sym, *args)
      end
    end

  end
{% endhighlight %}

The next thing you will have to do is trigger the action that queues up the background job so that you have at least one of the background job in your queue.  Once that's complete you should trigger the rails console to give yourself an interactive session.

{% highlight bash %}
./bin/rails c

# after session launches, debugging begins
# first we will want to make sure we are logging to to STDOUT

Resque.logger = Logger.new(STDOUT)

# now that we can see log messages easily we will work on the queue.
# replace [YOURQUEUENAME] with the name of the queue you where the job has been placed.
# this is the CLI for executing a resque worker.

Resque::Worker.new('[YOURQUEUENAME]', {}).work

{% endhighlight %}

When your job executes it will launch into the debugger from there you inspect variable and otherwise follow the code execution path.  Here is a [cheatsheet](http://fleeblewidget.co.uk/2014/05/byebug-cheatsheet/) to get you started.

If you found this article helpful then please leave a comment.




