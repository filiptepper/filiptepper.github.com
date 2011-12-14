---
title: Rails 3.1 asset pipeline zero downtime deployments
excerpt: Asset pipeline deployments made easy with Capistrano.
layout: post
---
Are you ready for Rails 3.1? Are you ready for
[asset pipeline](http://guides.rubyonrails.org/asset_pipeline.html)?!

I thought I was. Well, I was. And everything worked just perfect for as long as I was using [Passenger](http://www.modrails.com/). But Unicorn delivers the promise of zero downtime deployments and I was tempted to finally use it in production.

However I noticed that for a brief moment after restarting Unicorn the _old master_ workers were using assets that were already removed from current application.

If you deploy with [Capistrano](https://github.com/capistrano/capistrano) then there isn't much that you need to do. Just add the following to the `config/deploy.rb` file:

{% highlight ruby %}load "deploy"
load "deploy/assets"{% endhighlight %}

Yes, with Capistrano it's just that easy!