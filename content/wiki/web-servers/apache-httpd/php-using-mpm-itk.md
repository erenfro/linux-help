---
{"publish":true,"title":"PHP Using MPM-ITK","created":"2025-08-29","modified":"2025-08-30T11:21:50.126-04:00","published":"2025-08-29","tags":["web-server"],"cssclasses":""}
---

When using Apache with mpm-itk you can setup every virtual host to run under a different user and group. This includes everything running by Apache itself, including modules like mod_python, mod_wsgi, and mod_passenger. This approach allows for a lot more possibility out of Apache and per virtual host security and is not limited to just PHP.

By using this approach versus running PHP and everything through suexec, you not only gain the ability to run python, wsgi applications, and ruby on rails as the user owning them, but they are faster and more native to the environment of which they were designed to run under. The difference in this method over the mpm-worker is mpm-itk is based on the traditional prefork MPM which doesn't do threading.

# Intended Audience

This article explains how to run Apache with per virtual host user and group rights. It is written for server administrators who want to have all or most virtual hosts as the user and group which owns the content. The usefulness of this is for security, comparability, and speed.

## What is mpm-itk

mpm-itk is an MPM(Multi-Processing Module) for the Apache web server. mpm-itk allows you to run each of your virtual hosts under a separate uid and gid – in short, the scripts and configuration files for one virtual host can be restricted from being readable for all other users with virtual hosts.

mpm-itk is based on the traditional prefork MPM, which means it's non-threaded. This means you can run non-thread-aware code (like many PHP extensions) without problems. On the other hand, you lose out to any performance benefit you'd get with threads, of course; you'd have to decide for yourself if that's worth it or not. You will also take an additional performance hit over prefork as there's an extra fork per request.

## Advantages of mpm-itk

1. Speed
    * mpm-itk provides near-native speed for any module used with Apache.
2. Configuration flexibility
    * mpm-itk makes every apache instance run as the user and group specified, including it's modules.
3. Ease of Configuration
    * mpm-itk has only 3 configuration directives, of which only 1 is needed to actually get started.

## Disadvantages of mpm-itk

1. Multiple Versions
    * mpm-itk doesn't allow you to run multiple versions of modules like you could through fastcgi wrapping. To do so you would have to run them through fastcgi methods still with the exception of needing suexec, but still adding an extra layer to execution time.

# Installation

## Prerequisites

This manual is designed for Debian Lenny servers. It expects a server running Apache 2.x. Basically this should work equally for other systems – they probably just use different paths and package names…

## Installation of Packages

The following packages are required:

* libapache2-mpm-itk
    * The Apache MPM itself providing itk support
* libapache-mod-php5
    * PHP module for Apache to run PHP scripts.

## Configuration

Install the necessary packages.

~~~~~
apt-get install libapache2-mpm-itk libapache2-mod-php5
~~~~~

Enable mod_php5 (this is usually done by default for you by aptitude)

~~~~~
a2enmod php5
~~~~~

Create a new virtual host /etc/apache2/sites-available/somevhost.conf:

~~~~~
AssignUserID someuser somegroup
ServerName somevhost
DocumentRoot /home/someuser/public_html
ErrorLog /var/log/apache2/somevhost_error_log
CustomLog /var/log/apache2/somevhost_access_log combined
ScriptAlias /cgi-bin/ /home/someuser/cgi-bin/
DirectoryIndex index.php index.shtml index.html index.htm

Options +Indexes +Includes +FollowSymLinks allow from all AllowOverride All

allow from all
~~~~~

Finally, restart Apache:

~~~~~
/etc/init.d/apache2 restart
~~~~~

## Additional configuration

More later.

FIXME

Intended additions is to include per-vhost php.ini or configurable ini settings a user can use to change for their needs.

# Appendix

## Further reading

Website of mpm-itk: <http://mpm-itk.sesse.net/>

# FAQ

* How fast is it?
    * Compared to suexec, the speed difference is much more superior. As far as multi-threaded applications, you may loose speed as a result of mpm-itk due to it only preforking.
