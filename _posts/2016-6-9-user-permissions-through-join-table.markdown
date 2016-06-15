---
layout: post
title: "User permissions using a join table in Rails"
date: 2016-6-9 16:00:00
comments: true
categories: ruby rails domain modeling user permissions
---

I've been mentoring students for [the firehose project](http://www.thefirehoseproject.com/mentors) for close to a year now and one thing that keeps tripping people up is how to allow for a flexible permissions system.  To be clear, what I mean by permissions system is a mechanism for tracking who is associated with a particular resource and what their access rights are.

A typical blog example would include a user that has full permission on a blog post they create but can give edit and view rights to a friend they want to make small changes for example.  In the same blog app we may have a comment system where we have moderators that can delete a comment if they find it in-appropriate.  Depending on your app you may describe more complex relationships.  But for now let's take this case and look at how you might model that domain in Ruby On Rails.

## The User Model


{% highlight ruby %}
# app/models/user.rb
class User < ActiveRecord::Base
  has_many :permissions

  # you can't set up a typical has_many relationship when dealing 
  # with polymorphic relationships, you must use source and source type 
  # to fully specify what type of objects you intend to retrieve

  has_many :posts, through: :permissions, source: :permitable, source_type: 'Post' do 
    # you can add any number of methods further scoping the has_many query.
    # it is advised to use Arel, however as of this writing Bitwise operations
    # are not supported in Arel so, sadly we use a string.

    def owner_of
      where('`permissions`.`bitmask` & ? > 0', Permission::OWNER )
    end
  end

  # comments looks just like posts, you could add in some meta programming to make this 
  # automatice whenever you include a module in your source class. Perhaps we'll explore
  # that later.

  has_many :comments, through: :permissions, source: :permitable, source_type: 'Comment' do
    def owner_of
      where('`permissions`.`bitmask` & ? > 0', Permission::OWNER )
    end
  end

end

# app/models/post.rb
class Post < ActiveRecord::Base
  # you have to use the as: option here to get the relationship to function
  # as expected

  has_many :permissions, as: :permitable
  has_many :users, through: :permissions
end

# app/models/comment.rb
# same as post for all intents and purposes
class Comment < ActiveRecord::Base
  has_many :permissions, as: :permitable
  has_many :users, through: :permissions
end

# app/models/permission.rb

# our permissions model holds all the logic for storing and defining permissions.
# in our case we are using a simple bitmask so the individual numbers are stored here as well
class Permission < ActiveRecord::Base
 
  # you have to use polymorphic true to make permissions use the 
  # permitable_id and permitable_type columns for the relationship
 
  belongs_to :permitable, polymorphic: true
  belongs_to :user
  
  # we are using bitwise calcs to determine permissions, you can use 
  # booleans in columns if it works better for your application
  ADMIN = 1
  OWNER = 2
  VIEW = 4
  EDIT = 8

end

# db/shema.rb

# obviously if you are using ActiveRecord you need a database schema backing all this up
# this is the most basic of schema's required to get this working.
ActiveRecord::Schema.define(version: 20160615181741) do

  create_table "comments", force: :cascade do |t|
    t.string   "email"
    t.text     "body"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

  create_table "permissions", force: :cascade do |t|
    t.integer  "permitable_id"
    t.string   "permitable_type"
    t.integer  "user_id"
    t.integer  "bitmask",         default: 0
    t.datetime "created_at",                  null: false
    t.datetime "updated_at",                  null: false
  end

  add_index "permissions", ["bitmask"], name: "index_permissions_on_bitmask"
  add_index "permissions", ["permitable_type", "permitable_id"], name: "index_permissions_on_permitable_type_and_permitable_id"
  add_index "permissions", ["user_id"], name: "index_permissions_on_user_id"

  create_table "posts", force: :cascade do |t|
    t.string   "title"
    t.text     "body"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

  create_table "users", force: :cascade do |t|
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

end


{% endhighlight %}

The primary motivation for writing this article is to remind myself how to set up some of the more complex relationships in rails and also to inform others who might be searching the interwebs for the same type of thing.  If you are reading this post, hopefully you found it useful.  Comments or questions always welcome.  Thanks for reading.



