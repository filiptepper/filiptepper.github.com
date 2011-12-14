---
title: Compiling memcached gem on Snow Leopard
excerpt: Frustrated? Look no more.
layout: post
---
For production at <a href="http://blip.pl/"><code>Blip</code></a> we've recently switched from <code>memcache-client</code> to Evan Weaver's <a href="http://blog.evanweaver.com/files/doc/fauna/memcached/files/README.html"><code>memcached</code></a> gem (performance!). Compiling it on <code>Snow Leopard</code> for development requires <code>ARCHFLASH</code> to be set to <code>-arch x86_64</code>.

To fix this just uninstall and reinstall <code>memcached</code>:

{% highlight bash %}sudo gem uninstall memcached
sudo env ARCHFLAGS="-arch x86_64" gem install memcached<{% endhighlight %}

And you're done!
