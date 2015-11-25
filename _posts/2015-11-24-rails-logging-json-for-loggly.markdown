---
layout: post
title: "Rails logging with loggly in production"
date: 2015-11-24 16:00:00
comments: true
categories: ruby rails logging production
---

Logging has become increasingly important for web application developers looking to take advantage of their data to better optimize their marketing and development efforts.  Until recently I accepted the way that Rails wrote logs as gospel, not wanting to upset the Rails gods I left them alone in both production and development environments.  Until recently....

My company has been using [loggly](https://www.loggly.com/) for a while to observe behavior changes and events happening in, they provide a great interface for querying and filtering log data and allow you to view those queries with a large number of visualization options.  Additionally they archive all logs to S3 for future analysis. 

One feature of loggly's that sets it apart from just storing logs is it's ability to filter and correlate structured data.  If you send your data to loggly as a json doc you can instantly filter on attributes and sub attributes as if they were columns in a database.  This turns out to be hugely useful if you are trying to understand your application in aggregate.

Typical Rails logs do not output json, instead they output a fairly verbose string describing what happened.  This is great for humans as it is fairly easy to read, but not particularly great for machines that handle structured data much better.  JSON would be better in many cases, as it is immediately parseable by a huge number of code libraries.

Another gripe I have with default rails logging is that it's very difficult to track all the data related to one particular request.  The problem is that multiple processes write to the log at once leading to an overlap of log information in production.  

Lastly, current_user tracking is an absolute must if you want your logging to double as an audit trail which is something we've always wanted at my company but up until this point have not had a great way of achieving.  Taking all these requirements into consideration I decided to fix our logging to give us more of what we need and remove most of what we don't in production.  The following is the code we are using to make our logging much much better.  I tried to comment the code where appropriate, but if you have questions feel free to comment and I can address any concerns.


{% highlight ruby %}

# file config/initializers/instrumentation.rb

module Instrumentation

  # don't want to use default logging 
  # for actions_view, action_controller, or active_record
  def self.remove_existing_log_subscriptions
    ActiveSupport::LogSubscriber.log_subscribers.each do |subscriber|
      case subscriber
      when ActionView::LogSubscriber
        unsubscribe(:action_view, subscriber)
      when ActionController::LogSubscriber
        unsubscribe(:action_controller, subscriber)
      when ActiveRecord::LogSubscriber
        unsubscribe(:active_record, subscriber)
      end
    end
  end

  # tricky bit of code that removes existing subcribers
  def self.unsubscribe(component, subscriber)
    events = subscriber.public_methods(false).reject { |method| method.to_s == 'call' }
    events.each do |event|
      ActiveSupport::Notifications.notifier.listeners_for("#{event}.#{component}").each do |listener|
        if listener.instance_variable_get('@delegate') == subscriber
          ActiveSupport::Notifications.unsubscribe listener
        end
      end
    end
  end

  # next we make three classes that format our new log entries
  # this one is called when a request first hits the controller
  class JsonPageRequestStart
    def call(name, started, finished, unique_id, payload)
      # rid is the request id.  this should be accessible in all the logs
      # that way you can easily correlate a single request...
      # using Thread we can make sure that we have a request Id 
      # assigned to all entries that are part of the same request
      Thread.current[:rid] = SecureRandom.uuid 
      Rails.logger.info({
        cf: 'JsonPageRequestStart',
        name: name,
        started: started,
        finished: finished,
        unique_id: unique_id,
        payload: payload,
        rid: Thread.current[:rid]
      }.to_json)
    rescue
      Rails.logger.error({ error: "failed to log start_processing.action_controller"}.to_json)
    end
  end

  # this one is called when the request is fully completed including any views rendered.
  # this should only trigger if the request was successful.  
  class JsonPageRequestComplete
    def call(name, started, finished, unique_id, payload)
      # we use log level 'info' to be consistent with old log level
      Rails.logger.info({
        cf: "JsonPageRequestComplete",
        name: name,
        started: started,
        finished: finished,
        unique_id: unique_id,
        payload: payload,
        rid: Thread.current.fetch(:rid, "unknown")
      }.to_json)
    rescue
      Rails.logger.error({ error: "failed to log process_action.action_controller"}.to_json)
    end
  end

  class JsonDatabaseQuery
    def call(name, started, finished, unique_id, payload)
      Rails.logger.info({
        cf: "DatabaseQuery",
        name: name,
        started: started,
        finished: finished,
        unique_id: unique_id,
        payload: payload,
        rid: Thread.current.fetch(:rid, "unknown")
      }.to_json)
    rescue
      Rails.logger.info({ error: "failed to log sql.active_record"}.to_json)
    end
  end

end

# if you want to run this only in production then wrap this 
# code in an if statement for production use only
# if Rails.env.production?
Instrumentation.remove_existing_log_subscriptions
ActiveSupport::Notifications.subscribe('start_processing.action_controller', Instrumentation::JsonPageRequestStart.new)
ActiveSupport::Notifications.subscribe('process_action.action_controller', Instrumentation::JsonPageRequestComplete.new)
ActiveSupport::Notifications.subscribe('sql.active_record', Instrumentation::JsonDatabaseQuery.new)
# end
{% endhighlight %}

So far the code above handles unique requestID's, removal of old log format, and replacing with new JSON format. What we are missing is user tracking for future analytics and auditing needs.  The following code should be added to the same initializer.  The basic idea is that we need to improve our payload to hold additional data about the request being made.  While we could use the Thread.current object, Rails gives a better way to handle this so I present the following solution.

{% highlight ruby %}
module AddExtraRequestLogData

  # this is the extra info we want to add to each request, note that this 
  # data would be missing on any active record instrumentation.  This is 
  # generally fine since we can correlate data via the unique request id.
  def append_info_to_payload(payload)
    super
    payload[:host] = request.host
    payload[:fwd] = request.remote_ip
    payload[:env] = Rails.env
    if current_user
      payload[:user_id] = current_user.id
      payload[:user_type] = current_user.class.name
      payload[:guest] = false
    else
      payload[:guest] = true
    end
  end
end
# because we are using ruby 2.1 we can use prepend instead
# of clunky aliasing to override existing functionality.
ActionController::Base.send :prepend, AddExtraRequestLogData

{% endhighlight %}

This is what we run in production at [Music Xray](http://www.musicxray.com) if you found this at all useful or take issue with the implementation, please feel free to leave a comment and we can debate about it.



