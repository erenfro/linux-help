---
{"publish":true,"title":"SuExec vs MPM-ITK","created":"2025-08-29","modified":"2025-08-30T11:22:55.573-04:00","published":"2025-08-29","tags":["web-server"],"cssclasses":""}
---

Running a multi-user webserver can be a hassle at times, especially if you want to actually run each virtual-host's executable content as the user that actually owns it. Well, some benchmark results are pretty clear how much difference there can be between running them through different means.

We'll compare running PHP code through SuExec fcgid methods, which uses a php-fcgid-wrapper approach under the worker MPM, a very minimal setup with no per-vhost php.ini, and compare it with the itk MPM which has per-user abilities right at the apache level.

The results of the bench, provided by ab, are below:

# SuExec Approach

~~~~~
Document Path:          /test.php
Document Length:        62967 bytes

Concurrency Level:      5
Time taken for tests:   540.931 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Total transferred:      63264000 bytes
HTML transferred:       62967000 bytes
Requests per second:    1.85 [#/sec] (mean)
Time per request:       2704.654 [ms] (mean)
Time per request:       540.931 [ms] (mean, across all concurrent requests)
Transfer rate:          114.21 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   6.9      0     155
Processing:  1634 2701 332.2   2604    4437
Waiting:      684 1059 162.9   1037    1950
Total:       1635 2702 331.9   2604    4437

Percentage of the requests served within a certain time (ms)
  50%   2604
  66%   2719
  75%   2778
  80%   2826
  90%   3119
  95%   3463
  98%   3722
  99%   3940
 100%   4437 (longest request)
~~~~~

# itk MPM Approach

~~~~~
Document Path:          /test.php
Document Length:        62110 bytes

Concurrency Level:      5
Time taken for tests:   359.724 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Total transferred:      62445000 bytes
HTML transferred:       62110000 bytes
Requests per second:    2.78 [#/sec] (mean)
Time per request:       1798.618 [ms] (mean)
Time per request:       359.724 [ms] (mean, across all concurrent requests)
Transfer rate:          169.52 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   3.9      0      87
Processing:   952 1796 280.9   1729    3451
Waiting:       11   12   6.3     11     136
Total:        952 1797 280.7   1729    3451

Percentage of the requests served within a certain time (ms)
  50%   1729
  66%   1749
  75%   1777
  80%   1855
  90%   2044
  95%   2407
  98%   2809
  99%   3017
 100%   3451 (longest request)
~~~~~

# Conclusion

The choice is pretty clear who's the winner here as far as performance. The itk MPM is absolutely provable to be faster in the short and long term process. What else though do you get between these differences?

Well, with the fcgid approach, everything executed gets called through suexec as a CGI, but this limits you to only those programs that are made for CGI and/or FastCGI models. What about wsgi, ruby on rails, etc? You can run these through FastCGI, but they would likely have the same slower results as the above statistics, but tun through mpm-itk, you can actually run mod_python, mod_wsgi, mod_passenger, and even mod_perl without wrapping them through a CGI interpretter on top of everything else.

On the opposite side, the worker MPM is a threaded MPM that, for static content and threaded applications can run with better speed. This is better for static content, like HTML files and images and such as well but not so much there I wouldn't imagine. You loose this threaded model when you use the itk MPM because it's based on the traditional prefork MPM which forks a process out for each call rather than threads.

FastCGI was good while it was there, but thanks to the Apache developers making the MPM framework for Apache 2.x, and developers that made the itk MPM, we have an alternative and clearly better solution to the overall performance and suexec concerns.

It's now your choice to decide what you want. Both configuration methods are available here at this wiki so you can try both out for yourself and see what you think for yourself.

# Testing Methods

These tests were performed by Apache's own ab tool against a php file, test.php, which only called phpinfo(). Both mod_php5 and php5-fcgi tests included the use of eAccelerator. The command line used to call it was:

~~~~~
ab -n1000 -c5 http://localhost/test.php
~~~~~

The test was performed under the Debian 5.0 distribution of Linux as well.
