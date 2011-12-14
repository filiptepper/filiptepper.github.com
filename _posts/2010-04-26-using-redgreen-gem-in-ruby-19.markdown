---
title: Using redgreen gem in Ruby 1.9
excerpt: Tips and tricks for you testing pleasure.
layout: post
---
We all love [<code>autotest</code>](http://zentest.rubyforge.org/ZenTest/, "ZenTest"), don't we? It's an essential tool for TDD software development, no doubt about it.

[<code>redgreen</code>](http://rubygems.org/gems/redgreen "redgreen") adds a little sugar to how your test results are displayed on the console. It doesn't work out-of-the box with Ruby 1.9, however it's just a matter of a simple tweak in your <code>~/.autotest</code> file.

Here's an example of my <code>~/.autotest</code>:

{% highlight ruby %}
require "autotest/fsevent"
require "autotest/growl"

unless ENV["RSPEC"]
  PLATFORM = RUBY_PLATFORM unless defined? PLATFORM
  require "redgreen/autotest"
end

Autotest.add_hook :initialize do |at|
  %w{.svn .hg .git vendor}.each {|exception| at.add_exception exception}
end
{% endhighlight %}

For Ruby 1.9 be sure to define <code>PLATFORM</code> constant, and if you want to use both <code>autotest</code> and <code>autospec</code> you need the <code>unless ENV["RSPEC"] ... end</code> block.