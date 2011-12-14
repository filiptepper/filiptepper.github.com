---
title: Submitting forms with JavaScript
excerpt: Cross-browser proof form submissions with JavaScript.
layout: post
---
From time to time some of our users file a bug report concerning an accidental form submit. We weren't able to track it down until today and there seems to be a nasty bug in some browsers (appeared in <code>Firefox 3.6</code> on <code>Mac OS X</code>).

We used to attach an event to a <code>textarea</code> element (we're using <code>Prototype</code>, this is not the exact code but the general idea of it):

{% highlight javascript %}
$("input-textarea").observe("keyup", function() {
  if (event.keyCode == Event.KEY_RETURN) {
    return formSubmit();
  }
});
{% endhighlight %}

This worked in most scenarios except for this one:
*  User enters some text into <code>textarea</code>,
*  User clicks on the browser's address bar and enters an URL,
*  User presses <code>Enter</code>,
*  The form was submitted, then location was changed.

Avoiding this kind of behavior can be achieved by switching from <code>onkeyup</code> to <code>onkeydown</code> event.

{% highlight javascript %}
$("input-textarea").observe("keydown", function() {
  if (event.keyCode == Event.KEY_RETURN) {
    return formSubmit();
  }
});
{% endhighlight %}

This snippet works for us as expected.