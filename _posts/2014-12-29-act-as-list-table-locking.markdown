---
layout: post
title: "help: acts_as_list is locking up my database"
date: 2014-12-25 16:00:00
comments: true
categories: ruby rails acts_as_list
---

The Acts As List gem has been around for a pretty long time.  DHH originally wrote it in 2007.  That makes the plugin (now a gem) over 7 years old.  The fact that I am still using this in production shows just how useful the gem was.  Recently (yes, recently) my team and I decided to upgrade a very old rails 2.3 codebase.  In doing this move, I started removing plugins from the old plugin directory as they are no longer supported in rails > 4.0.  Long story short, we replaced the acts_as_list plugin with the acts_as_list gem.  You would think that such a simple gem would work the same from rails 2.3 to rails 3.2.  That's what I thought too...

Both the old acts_as_list plugin and the new acts_as_list gem make use of the update_all. 

{% highlight ruby %}
  # Newer example code from gem
  def increment_positions_on_all_items
    acts_as_list_class.unscoped do
      acts_as_list_class.where(
        scope_condition
      ).update_all(
        "#{position_column} = (#{position_column} + 1)"
      )
    end
  end
  
  # same method in the old plugin
  def increment_positions_on_all_items
    acts_as_list_class.update_all(
      "#{position_column} = (#{position_column} + 1)",  "#{scope_condition}"
    )
  end

{% endhighlight %}

However, as you can see, they make user of slightly different scoping mechanisms.  Well it turns out that if you use the newer gem code that uses a *where* statement to scope the query you can end up with a particularly nasty query that will lock up your database and generally ruin your day.  You'll likely only notice this on a large table.  The table I saw issues with had a little over 2 million records.  

The problem arrises when your scoping column is optional or is set after you create model.  Allow me to illustrate with a concrete example.


{% highlight ruby %}


class Song < ActiveRecord::Base

  belongs_to :album  

  # so I want to be able to order songs once I associate them with an album
  acts_as_list scope: [:album_id]

end 

class Album < ActiveRecord::Base
  
  has_many :songs

end

{% endhighlight %}

In this example, If I always create a song with an associated album then all is well in the world.  The update_all query is executed but it is scoped to the album_id so at most maybe 100 rows would have to be updated.  Updating a 100 rows in MySQL is pretty fast and unlikely to cause any real issues in your application.

Everything breaks down when you decide that you want to let users upload songs to your app and then at some other point in the future associated those songs with an album.  What happens when you upload a song without an album is that the album_id is nil.  When the album_id is nil the acts_as_list gem will still attempt to update the position for the provided scope.  What this means is that it will issue an update_all query that will include any song that doesn't have an album_id.  This could be 90% or more of your database table.  Trying to update 1,800,000 records everytime someone uploads a song will result in crippling database performance and a terrible user experience.

So now that we know what is wrong with act_as_list.  What can we do about it?  We on thing you can do is to change how your models work.  If you were to introduce a join model in between the the songs and albums model then you would ensure that you would never have an association that had null values and so you would always be calling update_all on a properly scoped model.

Secondly, you could ignore the acts_as_list plugin and move the song ordering onto a simple serialized array on the album model.  So changing your Album model to look something like this.  song_order_ids would be a set of song_ids where the order mattered. You could then use the song_order_ids with a custom sort_by routine that would order your songs.

{% highlight ruby %}

class Album < ActiveRecord::Base
  
  has_many :songs

  serialize :song_order_ids, Set
  

end

{% endhighlight %}

Of course doing things this way would require you to manually manage the song_order and make sure it was updated appropriately.Not really that difficult, and probably the best option if you only have one model in your app that needs to be ordered. 

Another option is to replace act_as_list with [ranked-model](https://github.com/mixonic/ranked-model).  Ranked-model is a more modern version of acts_as_list that goes out of it's way to avoid hitting the database when it doesn't need too.  This sounds like my kind of plugin.  I think I'm going to replace my acts_as_list dependent models with this gem.  After completing the migration I'll write another post outlining any difficulties or interesting data migration steps necessary.


