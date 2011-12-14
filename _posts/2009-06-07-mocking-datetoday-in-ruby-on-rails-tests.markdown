---
title: "Mocking Date.today in Ruby on Rails tests"
excerpt: "Freeze time in your tests."
layout: post
---
Since for the last couple of months I'm all about <a href="http://en.wikipedia.org/wiki/Test-driven_development">test-driven development</a> I wanted to test a couple of <code>ActiveRecord</code> callbacks that relied on <code>Date.today</code> method.

Here's a quick hack for your <code>test_helper.rb</code> file that allows you to specify the date returned by <code>Date.today</code>.

{% highlight ruby %}class Date
  class << self
    @@mocked_today = nil

    alias :unmocked_today :today

    def today
      @@mocked_today || unmocked_today
    end

    def with_mocked_today(mocked_today)
      if block_given?
        @@mocked_today = mocked_today
        begin
          yield
        ensure
          @@mocked_today = nil
        end
      end
    end
  end
end{% endhighlight %}

And a quick example how to use it:

{% highlight ruby %}Date.with_mocked_today Date.parse("2008-01-01") do
  // your code here...
end{% endhighlight %}
