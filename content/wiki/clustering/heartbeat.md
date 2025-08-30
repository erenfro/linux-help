---
{"publish":true,"title":"Heartbeat","created":"2025-08-29","modified":"2025-08-30T00:14:49.010-04:00","published":"2025-08-29","tags":["clustering","heartbeat"],"cssclasses":""}
---

# Setting up Heartbeat

The first thing is to install heartbeat and ldirectory if they're not already installed. If your distribution has them included in it's package repositories then all the better. Once you have it all installed, everything you will be doing will be in the /etc/ha.d directory.

Lets take a look at what should be there, provided you have it installed successfully:

* harc
* conf/
* rc.d/
* resource.d/

This is the common base structure for what is in /etc/ha.d. resource.d contains the resource scripts Heartbeat will use to do various things, such as setting an IP address, gratuitous arp, etc.

We'll be adding a few configuration files specifically for our setup process for heartbeat and ldirector with the following new files:

* authkeys
* ha.cf
* haresources

Lets first start with authkeys, which is what defines how two heartbeat servers would authenticate to each other in a secure fashion.

**authkeys:**

~~~~~~~~~~~~~~~~~~~~~~~~
auth 1
1 sha1 s0m3n1c3p4s5w0rdh3re.
~~~~~~~~~~~~~~~~~~~~~~~~

That's pretty much how simple it is to setup the authentication. This file should be chmod 600 to be read only by the user running heartbeat, which unfortunately is usually the root user because of what it will have to do, starting and stopping services, what a CRM does.

The next file is the heartbeat configuration file itself. It's what defines how heartbeat should work, communicate, logging, etc.

**ha.cf:**

~~~~~~~~~~~~~~~~~~~~~~~~~
# File to write debug messages to:
debugfile /var/log/ha-debug

# File to write other messages to:
logfile /var/log/ha-log

# Facility to use for syslog()/logger:
logfacility local0

# Heartbeat Interval, how many seconds between each beat:
keepalive 2

# Deadtime, how long before declaring the host dead:
deadtime 30

# Warntime, hos long before issuing a 'late heartbeat' warning:
warntime 10

# Heartbeat protocol, multicast interface information to use:
mcast lan0 255.0.0.1 694 1 0

# Auto failback to master when recovered? Basically Master/Slave nodes.
auto_failback on

# Nodes to include in cluster, each node should match what's in uname -n's output:
node router1.mydomain.tld
node router2.mydomain.tld
~~~~~~~~~~~~~~~~~~~~~~~~~

This is a simple configuration setup for Heartbeat. There are more options which are not used here such as STONITH measures mentioned before, but we're not going there for now.

Here's where the actual Cluster Resource Management routines comes in next, is the actual haresources file. Here you will actually define all the resources to use. For this tutorial I will cover a basic setup using two virtual IP's (or VIPs).

**haresources:**

~~~~~~~~~~~~~~~~~~~~~~~
router1.mydomain.tld \
  IPaddr2:xx.xx.xx.xx/24 net0 \
  IPaddr2:yy.yy.yy.yy/24 net0 \
  ldirectord \
  lighttpd
~~~~~~~~~~~~~~~~~~~~~~~

The IP's delimited here would be two VIPs, whether they were externally accessible IP's or internally accessible IP's, the difference matters not itself, but that they are the IP's intended for directing traffic going to them to another server or series of load balanced servers. IPaddr2 is a heartbeat native resource provided in the earlier described resources.d/ directory. There are two types, IPaddr and IPaddr2. The main difference in the two is IPaddr uses the old ifconfig interface tool which is not recommended for this, and IPaddr2 uses the ip tool from iproute2, the only recommended interface configuration tool for Linux these days.

ldirectord is started for the LVS director which we will discuss very soon now.

lighttpd is started to provide a fail-over web server that it if ever came down to it, “We're sorry but we're experiencing technical difficulties at this time.” as the last ditch effort to save face and still be alive enough to respond.

That's it, really for resources. For each server you would add a new line, escaped by \ for easy readability as shown here. If you wanted to add a dns server or dhcp server in here, you would simply add in the script's base name as you would see in /etc/init.d in there, or if you need something provided by heartbeat's resource.d/ you would add it in with Resourcename:parameters.

This is everything you need to know on how to setup heartbeat in the simplest yet useful manner.
