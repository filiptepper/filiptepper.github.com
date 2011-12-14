---
title: Sharing ActiveRecords objects via memcached in Ruby 1.8 and Ruby 1.9
excerpt: Where Ruby encodings can hurt your applications.
layout: post
---
At our team at [Blip](http://blip.pl/, "Blip") we're working right now on a transition from <code>Ruby 1.8</code> to <code>Ruby 1.9</code>. For almost a year now we've been using <code>Ruby 1.9</code> for all of our backend code and due to increasing load of asynchronous processesing we wanted to share application cache with backend.

This seems like a pretty simple task - adding namespacing is more than enough if you plan on using the same releases of <code>Ruby</code> at both ends.

Now, we all know that <code>Ruby 1.9</code> comes with an enhanced support for character encoding. Also, we want to use <code>UTF-8</code> encoding in our <code>Ruby 1.9</code> application. First we tried hacking [<code>memcached</code>](http://github.com/fauna/memcached/, "memcached") gem, but the simplest solution turned out to be a simple patch for <code>Marshal</code> module:

{% highlight ruby %}module Marshal
  class << self
    @@prok = Proc.new { |o| o.is_a?(String) ? Iconv.iconv("UTF-8//IGNORE", "UTF-8", o.force_encoding("UTF-8").encode("UTF-8")).first : o }

    alias old_load load

    def load(source)
      old_load source, @@prok
    end
  end
end{% endhighlight %}

Keep in mind that this is not 100% bulletproof solution - this one works for us and should cover most of your needs.