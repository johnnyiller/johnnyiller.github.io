---
layout: post
title: "Using form_for in rails to filter a data"
date: 2016-1-13 16:00:00
comments: true
categories: ruby rails form_for design patterns
---

Have you ever wondered how to filter data for admin users in a rails app?  If so, I present a reasonble way of doing it, such that you can use your existing form helpers via form_for.  At first, filtering data in rails seems simple.  You can just create a custom form using the form_tag helper right?  The problem with using the form_tag helper is that once you are using form_tag you are no longer using a form builder to create the fields in your form.  This would not be an issue if you were using the default builder that ships with Rails.  

In my opinion the default builder is pretty lacking from a useability standpoint.  This is the primary reason gems like [simple_form](https://github.com/plataformatec/simple_form) exist.  They give you a better starting point and quicker API for creating forms.  At my day job, we have lovingly created our own form builder that makes building forms from our models very quick and easy.  The problem we were facing was that our admin interfaces did not share the look and feel of the rest of the forms on our site because there wasn't a simple way to use form_for to filter lists of data. 

Enter, the FormFilterObject.  The FormFilterObject is an object in our application that mimics an active record instance just enough to get form_for to work.  Now that you know the punchline, let me show you some code to illustrate the point.


### First the form builder.

{% highlight ruby %}
    
class BootstrapFormBuilder < ActionView::Helpers::FormBuilder

  def field_settings(method, options = {}, tag_value = nil)
    field_name = "#{@object_name}_#{method.to_s}"
    default_label = tag_value.nil? ? "#{method.to_s.gsub(/\_/, " ")}" : "#{tag_value.to_s.gsub(/\_/, " ")}"
    label = options[:label] ? options.delete(:label) : default_label
    options[:class] ||= ""
    options[:class] += options[:required] ? " required" : ""
    [field_name, label, options]
  end

  def check_box(method, options={}, checked_value = "1", unchecked_value = "0")
    field_name, label, options = field_settings(method, options)
    options[:help_placement] = "top" unless options[:help_placement]
    options[:help_trigger] = "hover" unless options[:help_trigger]
    str = ""
    str << %Q{<div class="checkbox #{options[:class]}">}
    str << %Q{<label for="#{field_name}">#{super} #{label}}
    str << %Q{&nbsp;<a data-toggle="modal" data-target="##{field_name}_modal"><i class="glyph-help"></i></a>} if options[:help_modal]
    str << %Q{&nbsp;<a data-toggle="popover" data-target="##{field_name}_popover" data-placement="#{options[:help_placement]}" data-title="#{options[:help_header]}" data-trigger="#{options[:help_trigger]}"><i class="glyph-help"></i></a>} if options[:help_popover]
    str << %Q{</label>}
    str << %Q{</div>}
    if options[:help_modal]
      str << %Q{<div class="modal fade" id="#{field_name}_modal"><div class="modal-dialog"><div class="modal-content"><div class="modal-header"><button type="button" class="close" data-dismiss="modal">&times;</button>}
      str << %Q{<h3>#{options[:help_header]}</h3>} if options[:help_header]
      str << %Q{</div><div class="modal-body">#{options[:help_modal]}</div><div class="modal-footer"><a href="#" class="btn btn-default" data-dismiss="modal">Close</a></div></div></div></div>}
    end
    str << %Q{<span class="hide" id="#{field_name}_popover">#{options[:help_popover]}</span>} if options[:help_popover]
    str.html_safe
 end

  def radio_button(method, tag_value, options = {})
    field_name, label, options = field_settings(method, options)
    options[:help_placement] = "top" unless options[:help_placement]
    options[:help_trigger] = "hover" unless options[:help_trigger]
    str = ""
    str << %Q{<div class="radio #{options[:class]}">}
    str << %Q{<label for="#{field_name}">#{super} #{label}}
    str << %Q{&nbsp;<a data-toggle="modal" href="##{field_name}_modal"><i class="glyph-help"></i></a>} if options[:help_modal]
    str << %Q{&nbsp;<a data-toggle="popover" data-target="##{field_name}_popover" data-placement="#{options[:help_placement]}" data-title="#{options[:help_header]}" data-trigger="#{options[:help_trigger]}"><i class="glyph-help"></i></a>} if options[:help_popover]
    str << %Q{</label>}
    str << %Q{</div>}
    if options[:help_modal]
      str << %Q{<div class="modal fade" id="#{field_name}"><div class="modal-dialog"><div class="modal-content"><div class="modal-header"><button type="button" class="close" data-dismiss="modal">&times;</button>}
      str << %Q{<h3>#{options[:help_header]}</h3>} if options[:help_header]
      str << %Q{</div><div class="modal-body">#{options[:help_modal]}</div><div class="modal-footer"><a href="#" class="btn btn-default" data-dismiss="modal">Close</a></div></div></div></div>}
    end
    str << %Q{<span class="hide" id="#{field_name}_popover">#{options[:help_popover]}</span>} if options[:help_popover]
    str.html_safe
  end

  def select(method, choices, options = {}, html_options = {})
    field_name, label, options = field_settings(method, options)
    options[:help_placement] = "top" unless options[:help_placement]
    options[:help_trigger] = "hover" unless options[:help_trigger]
    options[:feedback] = "has-feedback has-required" if options[:required] && options[:feedback] !~ /required/
    options[:feedback] = options[:feedback].to_s + " has-help" if options[:help_popover] && (!options[:feedback] || options[:feedback] !~ /help/)
    (html_options[:class] ||= "") << " input-lg form-control"
    str = ""
    str << %Q{<div class="form-group #{options[:icon]} #{options[:feedback]}">}
    str << %Q{<label for="#{field_name}"#{(options[:sr_only] ? ' class="sr-only"' : "")}>#{label}}
    str << %Q{&nbsp;<a data-toggle="modal" href="##{field_name}_modal"><i class="glyph-help"></i></a>} if options[:help_modal]
    str << %Q{&nbsp;<a data-toggle="popover" data-target="##{field_name}_popover" data-placement="#{options[:help_placement]}" data-title="#{options[:help_header]}" data-trigger="#{options[:help_trigger]}"><i class="glyph-help"></i></a>} if options[:help_popover]
    str << %Q{</label>}
    str << %Q{<span id="helpBlock" class="help-block">#{options[:help_block]}</span>} if options[:help_block]
    str << %Q{#{super}}
    str << %Q{</div>}
    if options[:help_modal]
      str << %Q{<div class="modal fade" id="#{field_name}"><div class="modal-dialog"><div class="modal-content"><div class="modal-header"><button type="button" class="close" data-dismiss="modal">&times;</button>}
      str << %Q{<h3>#{options[:help_header]}</h3>} if options[:help_header]
      str << %Q{</div><div class="modal-body">#{options[:help_modal]}</div><div class="modal-footer"><a href="#" class="btn btn-default" data-dismiss="modal">Close</a></div></div></div></div>}
    end
    str << %Q{<span class="hide" id="#{field_name}_popover">#{options[:help_popover]}</span>} if options[:help_popover]
    str.html_safe
  end

  def file_field(method, options = {})
    options[:type] = "file"
    text_field(method, options)
  end

  def password_field(method, options = {})
    options[:type] = "password"
    text_field(method, options)
  end

  def text_field(method, options = {})
    field_name, label, options = field_settings(method, options)
    options[:help_placement] = "top" unless options[:help_placement]
    options[:help_trigger] = "hover" unless options[:help_trigger]
    (options[:class] ||= "") << " input-xlg form-control"
    options[:placeholder] = "#{label}" if options[:glyph]
    options[:feedback] = "has-feedback has-required" if options[:required] && options[:feedback] !~ /required/
    options[:feedback] = options[:feedback].to_s + " has-help" if options[:help_popover] && (!options[:feedback] || options[:feedback] !~ /help/)
    str = ""
    str << %Q{<div class="form-group #{options[:icon]} #{options[:feedback]}#{(options[:help_block] ? ' has-help' : "")}">}
    if [label, options[:glyph]].any?(&:present?)
      str << %Q{<label for="#{field_name}"#{(options[:glyph]||options[:sr_only] ? ' class="sr-only"' : "")}>#{label}}
      str << %Q{&nbsp;<a data-toggle="modal" href="##{field_name}_modal"><i class="glyph-help"></i></a>} if options[:help_modal]
      str << %Q{&nbsp;<a data-toggle="popover" data-target="##{field_name}_popover" data-placement="#{options[:help_placement]}" data-title="#{options[:help_header]}" data-trigger="#{options[:help_trigger]}"><i class="glyph-help"></i></a>} if options[:help_popover]
      str << %Q{</label>}
    end
    str << %Q{<span id="helpBlock" class="help-block">#{options[:help_block]}</span>} if options[:help_block]
    if options[:glyph]
      str << %Q{<div class="input-group">}
      str << %Q{<div class="input-group-addon glyph-#{options[:glyph]} size32"></div>}
    end
    str << %Q{#{super}}
    str << %Q{</div>} if options[:glyph]
    if options[:required]
      str << %Q{<span class="size32 form-control-feedback" aria-hidden="true">*</span>}
    else
      str << %Q{<span class="#{options[:feedback_glyph]} size32 form-control-feedback" aria-hidden="true"></span>}
    end
    str << %Q{</div>}
    if options[:help_modal]
      str << %Q{<div class="modal fade" id="#{field_name}"><div class="modal-dialog"><div class="modal-content"><div class="modal-header"><button type="button" class="close" data-dismiss="modal">&times;</button>}
      str << %Q{<h3>#{options[:help_header]}</h3>} if options[:help_header]
      str << %Q{</div><div class="modal-body">#{options[:help_modal]}</div><div class="modal-footer"><a href="#" class="btn btn-default" data-dismiss="modal">Close</a></div></div></div></div>}
    end
    str << %Q{<span class="hide" id="#{field_name}_popover">#{options[:help_popover]}</span>} if options[:help_popover]
    str.html_safe
  end

  def text_area(method, options = {})
    field_name, label, options = field_settings(method, options)
    options[:help_placement] = "top" unless options[:help_placement]
    options[:help_trigger] = "hover" unless options[:help_trigger]
    options[:feedback] = "has-feedback has-required" if options[:required] && options[:feedback] !~ /required/
    options[:feedback] = options[:feedback].to_s + " has-help" if options[:help_popover] && (!options[:feedback] || options[:feedback] !~ /help/)
    classes = options.fetch(:class,"").gsub(/\s+/m, ' ').strip.split(" ")
    classes << "form-control"
    classes << "input-xlg" if (classes & ["input-sm","input-lg","input-xlg","input-xxlg"]).size.zero?
    options[:class] = classes.join(" ").strip

    str = ""
    str << %Q{<div class="form-group #{options[:icon]} #{options[:feedback]}">}

    if [label].any?(&:present?)
      str << %Q{<label for="#{field_name}"#{(options[:sr_only] ? ' class="sr-only"' : "")}>#{label}}
      str << %Q{&nbsp;<a data-toggle="modal" href="##{field_name}_modal"><i class="glyph-help"></i></a>} if options[:help_modal]
      str << %Q{&nbsp;<a data-toggle="popover" data-target="##{field_name}_popover" data-placement="#{options[:help_placement]}" data-title="#{options[:help_header]}" data-trigger="#{options[:help_trigger]}"><i class="glyph-help"></i></a>} if options[:help_popover]
      str << %Q{</label>}
    end

    str << %Q{<span id="helpBlock" class="help-block">#{options[:help_block]}</span>} if options[:help_block]
    str << %Q{#{super}}
    if options[:required]
      str << %Q{<span class="size32 form-control-feedback" aria-hidden="true">*</span>}
    else
      str << %Q{<span class="#{options[:feedback_glyph]} size32 form-control-feedback" aria-hidden="true"></span>}
    end
    str << %Q{</div>} unless options[:noformgroup]
    if options[:help_modal]
      str << %Q{<div class="modal fade" id="#{field_name}"><div class="modal-dialog"><div class="modal-content"><div class="modal-header"><button type="button" class="close" data-dismiss="modal">&times;</button>}
      str << %Q{<h3>#{options[:help_header]}</h3>} if options[:help_header]
      str << %Q{</div><div class="modal-body">#{options[:help_modal]}</div><div class="modal-footer"><a href="#" class="btn btn-default" data-dismiss="modal">Close</a></div></div></div></div>}
    end
    str << %Q{<span class="hide" id="#{field_name}_popover">#{options[:help_popover]}</span>} if options[:help_popover]
    str.html_safe
  end


end

{% endhighlight %}

Once you have a useful form builder that does what you want for form_for you can then use it by passing it into your form_for helper as follows.

{% highlight erb %}
<% # where @setting is an instance of and ActiveRecord based model %>
<%= form_for(@setting, :builder => BootstrapFormBuilder do |f| %>
  <%= f.text_field :nickname %>
  <%= f.check_box :never_send_me_an_email_for_any_reason, :label=>"Never send me an email for any reason" %>
  <%= f.submit %>
<% end %>
{% endhighlight %}

While this is great for a single model it does not help you sort through and filter multiple models in a useful way.  This is where FormFilterObject comes in handy.  Here is an example of what a FormFilterObject might look like.

{% highlight ruby %}

  # goes in controller or in some sort of helper....
  class AdminSettingController < ApplicationController
    def index 
      klass = Class.new do 
        include ActiveModel::Model
        def self.name
          "SettingFilterObject"
        end
        def method_missing(m, *args, &block)
          self.open_struct ||= OpenStruct.new
          self.open_struct.send(m, *args)
        end
      end
      @instance = klass.new(params[:setting_filter_object] || {items_per_page: 10})
      @settings = Setting.all
      [:nickname, :never_send_me_an_email_for_any_reason].each do |filter|
        if @instance.send(filter).present?
          @settings = @settings.where(filter: @instance.send(filter)) 
        end
      end
      @settings = @settings.paginate(page: params[:page], per_page: @instance.items_per_page)
    end
  end

{% endhighlight %}

your view code would look something like this

{% highlight erb %}
  <%= form_for(@instance, as: :setting_filter_object, url: "", builder: BootstrapFormBuilder, method: :get do |f| %>
    <%= f.text_field :nickname %>
    <%= f.check_box :never_send_me_an_email_for_any_reason, :label=>"Never send me an email for any reason" %>
    <%= f.text_field :items_per_page %>
    <%= f.submit %>
  <% end %>

  <ul>
  <% @settings.each do |setting| %>
    <li><%= setting.id %></li> 
  <% end %>
  </ul>
{% endhighlight %}

That should be enough to get you started with more forms that filter ActiveRecord Object.  I realize this post is fairly specific and many won't find it generally useful, but if you do please leave me a quick comment.








