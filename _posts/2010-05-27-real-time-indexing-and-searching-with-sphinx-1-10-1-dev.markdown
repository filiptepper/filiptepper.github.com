---
title: Real-time indexing and searching with Sphinx 1.10.1-dev
excerpt: Couple of easy steps and the results are great.
layout: post
---
Real-time indexing and searching is one of the major goals of web development in 2010. As of version 1.10.1 [<code>Sphinx</code>](http://www.sphinxsearch.com/, "Sphinx") is added to the list of search engines that deliver the promise of real-time.

__Remember__, in this post I'm using a developers' trunk, checked out from repository. Get your's copy here:

{% highlight bash %}$ svn checkout http://sphinxsearch.googlecode.com/svn/trunk sphinx
$ cd sphinx
$ ./configure
$ make
$ sudo make install{% endhighlight %}

Worked for me out of the box on a Debian _etch_ box after failing on Snow Leopard.

First you need to setup your <code>sphinx.conf</code> file:

{% highlight bash %}index rt {
  type = rt
  path = /usr/local/sphinx/data/rt
  rt_field = message
  rt_attr_uint = message_id
}

searchd {
  log = /var/log/searchd.log
  query_log = /var/log/query.log
  pid_file = /var/run/searchd.pid
  workers = threads
  listen = 192.168.24.11:9312:mysql41
}{% endhighlight %}

You need <code>listen</code> switch in the <code>searchd</code> section to set up a MySQL protocol based server. However, a couple of quick benchmarks show that it's significantly slower that the regular <code>searchd</code>.

After launching <code>search</code> I used the MySQL protocol to index my data. Given the structure (<code>message</code>, <code>message_id</code>) I can now connect to my server and start indexing:

{% highlight bash %} $ mysql -P 9312 -h 127.0.0.1
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 1.10.1-dev (r2309)

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> INSERT INTO rt VALUES (1, 'this message has a body', 1);{% endhighlight %}

__Remember__, always use single quotes.

Querying real-time index is just as easy as typing:

{% highlight bash %}mysql> SELECT * FROM rt WHERE MATCH('message');
+------+--------+------------+
| id   | weight | message_id |
+------+--------+------------+
|    1 |   1500 |          1 |
+------+--------+------------+
1 row in set (0.00 sec){% endhighlight %}

And that's it, now you're ready to start your real-time search!

All the details can found in the <code>doc</code> folder of the repository. This is just a brief write up to get you started.