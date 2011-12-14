---
title: Fetching favicons with Ruby
excerpt: Sweeten your links with favicons.
layout: post
---
With <code>Ruby</code> and [<code>Google</code>](http://google.com/ "Google"), to be exact.

[<code>Favg</code>](http://github.com/filiptepper/favg "Favg") is a simple gem that wraps one of <code>Google</code>'s <code>S2</code> services and can be used to fetch and save websites' favicons.

To fetch a favicon:

{% highlight ruby %}
require "favg"
Favg.fetch "killingcreativity.com"
 => "\x89PNG\r\n\x1A\n\x00\x00\x00\rIHDR\x00\x00\x00\x10\x00\x00\x00\x10\b\x06\x00\x00\x00\x1F\xF3\xFFa\x00\x00\x00\x04sBIT\b\b\b\b|\bd\x88\x00\x00\x00OIDAT8\x8Dc\xFC\xFF\xFF\xFF\x7F\x06\n\x00\x13%\x9AG\r\x80\x00\x16\\\x12\x8F\x1E=b\xD8\xB7o\x1F\x03\x03\x03\x03\x83\x93\x93\x13\x83\x9C\x9C\x1Ci\x06\xEC\xDB\xB7\x8F\xE1\xF6\xED\xDBp~BB\x02Vu\xB4\xF3\x82\x93\x93\x13V6:`\x1CM\x89\x83\xC0\x00\x00g\xEB\x15\x94e\x1Er\x85\x00\x00\x00\x00IEND\xAEB`\x82"
{% endhighlight %}

To save a favicon to a file:
{% highlight ruby %}
require "favg"
Favg.fetch "killingcreativity.com", "favicon.png"
 => #<File:favicon.png>
{% endhighlight %}