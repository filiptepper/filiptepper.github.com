---
title: Testing Feedzirra
excerpt: Couple of examples.
layout: post
---
For one of my side-projects I'm using <a href="http://github.com/pauldix/feedzirra"><code>Feedzirra</code></a>, a robust feed parsing library. Since we all TATFT I wanted to test parts of my code that depend on feed parsing.

The glitch is that <code>Feedzirra</code> doesn't work with <a href="http://fakeweb.rubyforge.org/"><code>FakeWeb</code></a>, since it's using <code>cURL</code> for remote connections. However <code>cURL</code> supports <code>file://</code> protocol, so you can fake external requests with local files. I'm using <a href="http://github.com/thoughtbot/shoulda"><code>Shoulda</code></a> in the following examples.

<code>app/models/feed_storage.rb</code>

{% highlight ruby %}# Excerpts extracted from model

class FeedStorage < ActiveRecord::Base
  def parse
    if self.marshaled.nil?
      parser = Feedzirra::Feed.fetch_and_parse self.feed.url
      entries = parser.entries
    else
      parser = Marshal.load self.marshaled
      parser = Feedzirra::Feed.update parser
      entries = parser.new_entries
    end

    self.update_attribute :marshaled, (Marshal.dump parser)
    entries
  end
end
{% endhighlight %}

<code>test/unit/feed_storage_test.rb</code>

{% highlight ruby %}# Excerpts extracted from model's tests
class FeedStorageTest < ActiveSupport::TestCase
  context "a new feed" do
    setup do
      @feed = Factory :feed
      @feed.update_attribute :url, "file://#{URI.escape(File.join(File.dirname(File.expand_path(__FILE__, Dir.getwd)), "..", "fixtures", "full_feed.rss"))}"
    end
    should "setup a new parser and parse all entries" do
      assert_equal 50, @feed.parser.parse.length
    end
  end

  context "a parsed feed" do
    setup do
      @feed_path = "#{URI.escape(File.join(File.dirname(File.expand_path(__FILE__, Dir.getwd)), "..", "fixtures"))}"
      `cp #{File.join @feed_path, "full_feed.rss"} #{File.join @feed_path, "feed.rss"}`

      @feed = Factory :feed
      @feed.update_attribute :url, "file://#{File.join @feed_path, "feed.rss"}"
      @feed.parser.parse

      `cp #{File.join @feed_path, "updated_feed.rss"} #{File.join @feed_path, "feed.rss"}`
    end
    should "parse all new entries" do
      assert_equal 1, @feed.parser.parse.length
    end
    teardown do
      `rm -f #{File.join @feed_path, "feed.rss"}`
    end
  end
end
{% endhighlight %}

The logic behing <code>FeedStorage</code> is quite simple - for every feed in the database I store its feed parser (a marshaled Ruby object, might switch to something else if I run into performance issues). <code>Feed</code> class requires valid urls for <code>url</code> field, hence the <code>update_attribute</code> call.

Feel free to drop me a line in the comments - I'm sure there's some room for improvements!