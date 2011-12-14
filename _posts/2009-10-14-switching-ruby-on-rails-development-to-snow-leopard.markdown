---
title: Switching Ruby on Rails development to Snow Leopard
excerpt: Ease the pain of switching to Snow Leopard.
layout: post
---
Just a couple of random notes on switching to Snow Leopard.

<h4>taf2-curb gem</h4>
Compiling <code>taf2-curb</code> also requires (like <a href="http://filiptepper.com/2009/10/13/compiling-memcached-gem-on-snow-leopard/"><code>memcached</code></a> does) a custom environment flag:

{% highlight bash %}sudo port install curl
sudo env ARCHFLAGS="-arch x86_64" gem install taf2-curb{% endhighlight %}

<h4>Subversion bundle in TextMate</h4>

I didn't quite work out all the issues with <code>TextMate's</code> <code>Subversion</code> bundle, but at least <code>CommitWindows</code> pops up (I still can't revert changes). Setting <code>TM_RUBY</code> variable to <code>/usr/bin/ruby</code> (for default <code>Snow Leopard</code> <code>Ruby</code>) in Properties / Advanced /  Shell Variables tab fixes this issue.
