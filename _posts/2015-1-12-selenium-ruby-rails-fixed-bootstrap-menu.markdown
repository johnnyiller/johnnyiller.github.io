---
layout: post
title: "Selenium testing fixed bootstrap menu (Ruby Driver)"
date: 2015-1-12 16:00:00
comments: true
categories: ruby rails selenium fixedmenu bootstrap
---

Recently we decided to implement [our](http://www.musicxray.com) website UI using bootstrap 3 as a base.  Along with this conversion came some really nice menus on the header and footer of the site.  I didn't think that this change would cause too many issues with our Selenium based testing.  Generally speaking, it didn't, except for one issue.  The menu is fixed, so when the selenium driver tries to click on element below the fold of the page it encounters the following error.

{% highlight bash %}
Selenium::WebDriver::Error::UnknownError:         unknown error: Element is not clickable at point (507, 684). Other element would receive the click: 
    (Session info: chrome=33.0.1750.146)
    (Driver info: chromedriver=2.10.267521,platform=Windows NT 6.3 x86_64)
{% endhighlight %}

Ugly business

This is because the automatic scrolling used by driver.find_element and similar method calls only scrolls the page up just high enough to bring the button or link into the containing window.  I'm sure there is a better solution which probably involves monkey patching the ruby driver but for the time being (we have to launch in 2 days and I need my integration tests working) I wrote the following method.

{% highlight ruby %}

# assuming you have already create an instance of a Selenium WebDriver 

def driver.scroll_by_selector(selector)
  self.execute_script("window.scrollBy(0,document.querySelector('#{selector}').getBoundingClientRect().top);")
end

# example usage 
driver.scroll_by_selector("#my_lower_on_the_page_div")
element = driver.find_element(:id, "my_lower_on_the_page_div")
element.click

{% endhighlight %}

This manual scrolling brings the element into view so that we no longer get the error, calling this code before calling driver.find_element solved my problem, hopefully it helps you as well.  Let me know in the comments if this works for you.
