---
{"publish":true,"title":"Apache Threaded PHP","created":"2025-08-29","modified":"2025-08-30T11:21:20.608-04:00","published":"2025-08-29","tags":["web-server"],"cssclasses":""}
---

Yet another way to handle PHP under Apache, this method actually proves to be quite interesting, and fast. Because Apache is using the Worker MPM on this, it's multi-threaded, which provides a major speed boost to things that are not just within PHP's nature, but within the webserver itself. PHP still runs preforked through the mod_fcgid FastCGI daemon and it runs incredibly fast.

We'll go through the simple basic setup this involves:

# Installing Apache & PHP

Install PHP5 as you normally would for your distribution, or use the following as a guide for my supported distributions.

## Debian/Ubuntu:

~~~~~
apt-get install apache2-mpm-worker libapache2-mod-fcgid libapache2-mod-macro php5-cgi
~~~~~

# Configuring Apache for FCGID

## Debian/Ubuntu:

Create a new configuration file in /etc/apache2/mods-available named php5_fcgid.conf with the contents:

~~~~~
<IfModule !mod_php5.c>
  <IfModule mod_fcgid.c>
    # Where to look for the php.ini file?
    DefaultInitEnv PHPRC        "/etc/php5/cgi"

    # Maximum requests a process handles before it is terminated
    MaxRequestsPerProcess       1000

    # Maximum number of PHP processes
    MaxProcessCount             30

    # Number of seconds of idle time before a process is terminated
    IPCCommTimeout              240
    IdleTimeout                 240

    #Or use this if you use the file above
    FCGIWrapper /usr/bin/php-cgi .php
    FcgidMaxRequestLen 1073741824

    AddHandler php-fcgi .php
    Action php-fcgi /fcgi-bin/php-fcgi-wrapper

    AddType application/x-httpd-php .php

    Alias /fcgi-bin/ /var/www/fcgi-bin.d/php5-default/

    <Location /fcgi-bin/>
      SetHandler fcgid-script
      Options +ExecCGI
    </Location>
  </IfModule>
</IfModule>
~~~~~

Once this is in place, you can enable this configuration setting using the command: a2enmod php5_fcgid

Finally, all you need is to setup your initial default wrapper. This is easily done using:

~~~~~
mkdir -p /var/www/fcgi-bin.d/php5-default
cd /var/www/fcgi-bin.d/php5-default
ln -s /usr/bin/php5-cgi php-fcgi-wrapper
~~~~~

Now, restart Apache, and you're ready to rock with PHP.
