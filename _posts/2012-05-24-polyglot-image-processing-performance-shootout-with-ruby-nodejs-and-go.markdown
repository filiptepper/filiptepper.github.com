---
title: Polyglot image processing performance shootout with Ruby, Node.js and Go
layout: post
---

### TL;DR

It doesn't matter what you use for on-the-fly image processing over HTTP.

### Idea

First I should put a giant YMMV on top of this post. Second I know that it's just benchmarks. And now for the real thing...

My [Durszlak](http://durszlak.pl/) project is a testing ground for different bits and pieces of code that I would like to use in the future with my clients. It's a real production website, with a pretty decent user base, handling up to 20 000 000 requests a month (application and assets) on just a single bare metal machine.

I've had this idea for this project for a while now. What if I just stored only original uploads and used a simple resizing service behind [`Varnish`](https://www.varnish-cache.org/) to serve thumbnails? Oldest thumbnails are rarely requested and I could save some disk and backup space and learn a thing or two. Some companies I've worked with previously used this solution with great success.

Since I'm the sole developer on the project I knew that I could use basically anything I wanted. Be it Ruby, Node.js or Go. Then I found [`HttpImageFilterModule`](http://wiki.nginx.org/HttpImageFilterModule) for [`nginx`](http://nginx.org/). Then came [other contributions](https://github.com/filiptepper/thumbnails/network/members) and some meaningful results came up.

But first some interesting findings.

### Memory versus Files

When I first started running my benchmarks I was surprised to learn that Node.js outperformed Rubies (MRI, JRuby and Rubinius) by 8 to 12 times. I took some time to review [`gm`](https://github.com/aheckmann/gm) library I was using with Node.js to notice one important difference between `gm` and Ruby libraries.

`gm` can be called for inline streaming of converted image while all Ruby libraries write converted image to disk first.

For Node's `gm`:

1. File is read from disk.
1. Output is read into memory.
1. Output is sent back to user via HTTP.

For various Ruby libs:

1. File is read from disk.
1. Output is stored on disk.
1. Output is read from disk.
1. Output is sent back to user via HTTP.

Look at all this IO! Seems like not much but impact on performance is huge.

Then I replaced `mini_magick` with this simple piece code to handle thumbnails:

    command = "gm convert -size 100x100 ../image.jpg -resize 100x100 JPG:-"
    image = ""
    IO.popen(command) do |result|
      while part = result.read(1024)
        image << part
      end
    end

Thumbnail is never stored on disk and Ruby's performance is almost equal Node.js.

I did the same thing for `Go` example (first example was based on [`https://github.com/tobi/imagery-go`](https://github.com/tobi/imagery-go)).

### Benchmarks

Some number - simple benchmarks run with `ab` on a mid-2009 2.53 GHz dual core MacBook Pro. Unfortunately, I was not able to install `PythonMagick` on my setup so no `Python` / `Tornado` examples.

Source code for benchmarks is available on [GitHub](http://github.com/filiptepper/thumbnails).

#### Goliath, Ruby 1.9.3-p194

		Concurrency Level:      10
		Time taken for tests:   2.175 seconds
		Complete requests:      100
		Failed requests:        0
		Write errors:           0
		Total transferred:      261400 bytes
		HTML transferred:       251900 bytes
		Requests per second:    45.98 [#/sec] (mean)
		Time per request:       217.478 [ms] (mean)
		Time per request:       21.748 [ms] (mean, across all concurrent requests)
		Transfer rate:          117.38 [Kbytes/sec] received

		Connection Times (ms)
		              min  mean[+/-sd] median   max
		Connect:        0    0   0.2      0       2
		Processing:   214  217   1.5    217     220
		Waiting:      154  212  16.1    216     220
		Total:        215  217   1.6    218     220

-

		Concurrency Level:      100
		Time taken for tests:   22.212 seconds
		Complete requests:      1000
		Failed requests:        0
		Write errors:           0
		Total transferred:      2614000 bytes
		HTML transferred:       2519000 bytes
		Requests per second:    45.02 [#/sec] (mean)
		Time per request:       2221.244 [ms] (mean)
		Time per request:       22.212 [ms] (mean, across all concurrent requests)
		Transfer rate:          114.92 [Kbytes/sec] received

		Connection Times (ms)
		              min  mean[+/-sd] median   max
		Connect:        0    1   1.4      1       7
		Processing:   474 2139 298.3   2213    2469
		Waiting:      255 1920 298.8   1992    2252
		Total:        481 2140 297.1   2213    2472

#### Goliath, JRuby 1.6.7.2

    Concurrency Level:      10
    Time taken for tests:   24.638 seconds
    Complete requests:      100
    Failed requests:        0
    Write errors:           0
    Total transferred:      259200 bytes
    HTML transferred:       251900 bytes
    Requests per second:    4.06 [#/sec] (mean)
    Time per request:       2463.782 [ms] (mean)
    Time per request:       246.378 [ms] (mean, across all concurrent requests)
    Transfer rate:          10.27 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    0   0.3      0       1
    Processing:   671 2409 418.2   2469    2960
    Waiting:      671 2376 403.2   2468    2958
    Total:        672 2410 418.1   2469    2961

-

For concurrency of 100 server failed to respond.

#### Goliath, JRuby 1.7.0.preview1

    Concurrency Level:      10
    Time taken for tests:   22.330 seconds
    Complete requests:      100
    Failed requests:        0
    Write errors:           0
    Total transferred:      259200 bytes
    HTML transferred:       251900 bytes
    Requests per second:    4.48 [#/sec] (mean)
    Time per request:       2232.985 [ms] (mean)
    Time per request:       223.299 [ms] (mean, across all concurrent requests)
    Transfer rate:          11.34 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    0   0.5      0       3
    Processing:  1123 2232 352.3   2230    3339
    Waiting:     1123 2119 329.6   2226    2670
    Total:       1126 2233 352.0   2230    3340

-

For concurrency of 100 server failed to respond.

#### EventMachine, Ruby 1.9.3-p194

    Concurrency Level:      10
    Time taken for tests:   1.014 seconds
    Complete requests:      100
    Failed requests:        0
    Write errors:           0
    Total transferred:      256100 bytes
    HTML transferred:       251900 bytes
    Requests per second:    98.61 [#/sec] (mean)
    Time per request:       101.410 [ms] (mean)
    Time per request:       10.141 [ms] (mean, across all concurrent requests)
    Transfer rate:          246.62 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    1   1.0      0       8
    Processing:    19   99  30.9     92     200
    Waiting:       19   98  31.0     91     200
    Total:         20  100  31.0     94     201

-

    Concurrency Level:      100
    Time taken for tests:   10.387 seconds
    Complete requests:      1000
    Failed requests:        0
    Write errors:           0
    Total transferred:      2561000 bytes
    HTML transferred:       2519000 bytes
    Requests per second:    96.28 [#/sec] (mean)
    Time per request:       1038.676 [ms] (mean)
    Time per request:       10.387 [ms] (mean, across all concurrent requests)
    Transfer rate:          240.79 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    1   1.1      1      11
    Processing:   139 1012 267.8   1045    2209
    Waiting:      139 1010 268.0   1040    2208
    Total:        142 1014 267.5   1047    2209

#### EventMachine, Rubinius 2.0.0dev (1.8.7 8732fc62 yyyy-mm-dd JI)

    Concurrency Level:      10
    Time taken for tests:   1.744 seconds
    Complete requests:      100
    Failed requests:        0
    Write errors:           0
    Total transferred:      256100 bytes
    HTML transferred:       251900 bytes
    Requests per second:    57.33 [#/sec] (mean)
    Time per request:       174.437 [ms] (mean)
    Time per request:       17.444 [ms] (mean, across all concurrent requests)
    Transfer rate:          143.37 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    1   1.4      0       6
    Processing:    95  170  56.7    147     318
    Waiting:       88  153  50.2    136     318
    Total:         96  171  56.7    147     318

-

For concurrency of 100 server failed to respond.

#### Node.js 0.6.18

    Concurrency Level:      10
    Time taken for tests:   0.986 seconds
    Complete requests:      100
    Failed requests:        0
    Write errors:           0
    Total transferred:      258300 bytes
    HTML transferred:       251900 bytes
    Requests per second:    101.40 [#/sec] (mean)
    Time per request:       98.616 [ms] (mean)
    Time per request:       9.862 [ms] (mean, across all concurrent requests)
    Transfer rate:          255.79 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    0   1.0      0      10
    Processing:    34   97  29.0     89     175
    Waiting:       34   88  30.1     77     160
    Total:         34   97  29.1     89     175

-

For concurrency of 100 server failed to respond.

#### Go 1.0.1

    Concurrency Level:      10
    Time taken for tests:   1.119 seconds
    Complete requests:      100
    Failed requests:        0
    Write errors:           0
    Total transferred:      260100 bytes
    HTML transferred:       251900 bytes
    Requests per second:    89.35 [#/sec] (mean)
    Time per request:       111.923 [ms] (mean)
    Time per request:       11.192 [ms] (mean, across all concurrent requests)
    Transfer rate:          226.95 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    1   1.7      0      12
    Processing:    53  108  23.3    107     163
    Waiting:       52  105  23.4    105     159
    Total:         53  108  23.6    108     163

-

    Concurrency Level:      100
    Time taken for tests:   11.453 seconds
    Complete requests:      1000
    Failed requests:        0
    Write errors:           0
    Total transferred:      2601000 bytes
    HTML transferred:       2519000 bytes
    Requests per second:    87.31 [#/sec] (mean)
    Time per request:       1145.333 [ms] (mean)
    Time per request:       11.453 [ms] (mean, across all concurrent requests)
    Transfer rate:          221.77 [Kbytes/sec] received

    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:        0    1   1.6      0      15
    Processing:    54 1092 192.1   1135    1395
    Waiting:       44 1088 191.8   1130    1390
    Total:         57 1093 191.5   1135    1395

#### nginx 1.2.0 (2 worker processes)

		Concurrency Level:      10
		Time taken for tests:   2.369 seconds
		Complete requests:      100
		Failed requests:        0
		Write errors:           0
		Total transferred:      308700 bytes
		HTML transferred:       289700 bytes
		Requests per second:    42.21 [#/sec] (mean)
		Time per request:       236.891 [ms] (mean)
		Time per request:       23.689 [ms] (mean, across all concurrent requests)
		Transfer rate:          127.26 [Kbytes/sec] received

		Connection Times (ms)
		              min  mean[+/-sd] median   max
		Connect:        0    0   1.4      0      12
		Processing:    43  225 113.9    217     507
		Waiting:       43  225 113.9    217     507

-

		Concurrency Level:      100
		Time taken for tests:   22.388 seconds
		Complete requests:      1000
		Failed requests:        0
		Write errors:           0
		Total transferred:      3087000 bytes
		HTML transferred:       2897000 bytes
		Requests per second:    44.67 [#/sec] (mean)
		Time per request:       2238.841 [ms] (mean)
		Time per request:       22.388 [ms] (mean, across all concurrent requests)
		Transfer rate:          134.65 [Kbytes/sec] received

		Connection Times (ms)
		              min  mean[+/-sd] median   max
		Connect:        0    1   2.1      0      35
		Processing:    44 2111 1109.8   2043    5615
		Waiting:       44 2110 1109.9   2042    5615
		Total:         48 2111 1110.0   2046    5615

### Raw processing benchmarks

With a little help from fellow Node.js experts [@medikoo](http://twitter.com/medikoo) and [@maciejmalecki](http://twitter.com/maciejmalecki) I've also created a benchmark of raw image processing, without HTTP.

All benchmarks are executed with `time` (U assumed that every time processing is run whole runtime is loaded from scratch). This has the biggest impact on JRuby's performance (JVM startup time).

#### Ruby 1.9.3-p194

		10.12s user 13.69s system 179% cpu 13.284 total
		9.94s user 13.21s system 179% cpu 12.904 total
		10.37s user 14.52s system 180% cpu 13.817 total

#### JRuby 1.6.7.2

		16.50s user 22.63s system 180% cpu 21.708 total
		15.68s user 19.77s system 183% cpu 19.297 total
		15.53s user 20.54s system 183% cpu 19.659 total

#### Rubinius 2.0.0dev (1.8.7 8732fc62 yyyy-mm-dd JI)

		13.68s user 16.40s system 181% cpu 16.557 total
		13.68s user 16.27s system 184% cpu 16.255 total
		13.86s user 16.57s system 171% cpu 17.772 total

#### Node.js 0.6.18

		9.29s user 7.00s system 177% cpu 9.154 total
		9.33s user 7.04s system 174% cpu 9.405 total
		9.31s user 7.02s system 178% cpu 9.154 total

### Summary

Ruby? Node.js? Go? As long as you're dealing with HTTP it doesn't really matter in this case in terms of performance. Since this factor is negligible, you may want to take a closer look at memory usage - Go uses 1/10 of MRI's and 1/2 of Node.js' memory. That's rougly 5MB, 10MB and 50MB for Go, Node.js and Ruby respectively.