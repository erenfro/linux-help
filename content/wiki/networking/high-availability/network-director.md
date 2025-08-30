---
{"publish":true,"title":"Network Director","created":"2025-08-29","modified":"2025-08-30T11:09:39.079-04:00","published":"2025-08-29","tags":["networking"],"cssclasses":""}
---

# Setting up ldirectord

ldirector is the more complicated part of the setup process as it has a lot more defined possibilities that need to be considered. In the example used in this documentation, I will be covering webserver load balancing primarily. There is only one file you will be working with in this package, also in the /etc/ha.d directory, or simply /etc itself in more modern distros, especially not using heartbeat:

* ldirectord.cf

In the following configuration example, our VIP will be denoted as xx.xx.xx.xx for VIP1, and yy.yy.yy.yy for VIP2. Each Real IP, or RIP, will be denoted as aa.aa.aa.## for RIP block 1 where ## is 01-03 for each server in that load balanced block, and bb.bb.bb.## for RIP2. Note that each one would be actual different IP's, for for the purpose of this documentation, VIP's are external IP's and RIP's could either be IP's that are within the DMZ, accessible directly by the each of the heartbeat director servers.

**ldirectord.cf:**

~~~~~
checktimeout=8
checkinterval=5
autoreload=yes
logfile="local0"
quiescent=no
fork=yes
callback="/usr/local/sbin/lb_sync.sh"
emailalert="my@emailaddress.com"
~~~~~

The checktimeout is how long each check will have before expiring and timing out considering the test failed. Do not have this too low or you will experience a lot of interruptions you may not enjoy. The checkinterval is how long between each cycle to probe. Together it will have checked all servers within (8 + 5) * RIPs seconds and start checking again for the next cycle. autoreload tells ldirectord to reload it's configuration when the file changes.

An important one to note is the quiescent. With it set to no, if a server fails a check, it will be deleted from the LVS table completely and re-added later, making any further requests in route to the RIP stop and get directed elsewhere next attempt. By setting it to yes it will adjust weights to make it less likely to be used but allow connections to still come in to it.

fork is a wonderful and should-be-used feature for anyone running more than a couple RIPs or services in general on their director router. When enabled it spawns forks of ldirectord check services to probe individually per virtual service. Disabled, it will do each virtual service one by one and can take a very long time if you have a lot of services.

callback in this example is used on the master ldirectord host to sync the configuration changes to the slave heartbeat director router. An example will be shown later but it is very simple.

Now we're ready to get down into it with the virtual services. I'll begin with one virtual service and explain how it all works, then provide further examples below. Note that these are all within the same file, just below the above provided configuration contents. Here is our first webserver with 3 real servers behind 1 virtual IP:

~~~~~
virtual = xx.xx.xx.xx:80
    fallback = xx.xx.xx.xx:80 gate
    real = aa.aa.aa.01:80 gate 10
    real = aa.aa.aa.02:80 gate 20
    real = aa.aa.aa.03:80 gate 30
    scheduler = wrr
    persistent = 7200
    protocol = tcp
    service = http
    request = "/areyouthere.php"
    receive = "OK web[0-9]+"
    checktype = negotiate
~~~~~

That is a simple but yet powerful virtual director that receives requests directed at xx.xx.xx.xx on port 80, the http default port, and decides how to direct it to 3 real servers or a fallback server as a backup plan. The gate option in there defines it to use direct routing which takes the IP and forwards it to one of the servers. There are two other methods of routing including IP Tunneling and NAT. Direct Route is often the most preferred because the reply back goes by the way of whatever route the webservers have to the internet, which could be directly connected with each their own external IP or re-routed back to another router. IP Tunneling is fairly expensive in the end result but allows for more complex routes, such as connecting regionally different servers. NAT is another common option, which just changes the IP to the real server's IP and forwards the packet. IP Tunneling and NAT both have to return back through the director in reply from the service before responding back to the origin requester. Direct Routing is the least expensive because it doesn't change the packet itself at all, just forwards it to the real server IP, which will need to be able to receive requests addressed to the VIP. This will be explained later for Direct Routing how to set that up.

Breaking down each option further, fallback is for the fallback server to use when all real servers are in the fault state. The real options designate each RIP and their weights. Notice the scheduler is wrr? That's Weighted Round Robin, meaning each server will be passed in turn like a revolving door. In this example case, I setup three webservers with each incremented weights from 10, to 20, to 30. The reason for this is the persistent time in this example is set to 7200 seconds (2 hours). Ideally in most situations, it would be better to set persistent to the lowest possible time needed to execute any call to the service. Had that been so, persistent would have been probably around 10-30 seconds, and as a result the weights all equally at 10. Unfortunately in some cases, such as when sessions are not shared across all servers, it may be necessary to alter the plan a bit. I altered the weights in this example to help allow round robbin to pick different servers more accurately given the persistence value.

The rest of the settings are in relation to how to probe the service to check it's health. Since it is a webserver, it's using the tcp protocol, http service, and negotiating actual responces from the request page /areyouthere.php. A good use of the request would be something that ties into other parts of your website so that it can do checks and balances, but in doing this you should also consider load time it would take. In this example, the check is expecting to receive “OK web#”, where # could be anything from 0 to 100, or anything numerical. Any other kind of respond would result in the check being considered a failure and would cause ldirectord to delete the server from the LVS routes for the interval.

Here are further examples which includes providing SSL support and a second cluster of 3 webservers:

~~~~~
virtual = xx.xx.xx.xx:443
    fallback = xx.xx.xx.xx:443 gate
    real = aa.aa.aa.01:443 gate 10
    real = aa.aa.aa.02:443 gate 20
    real = aa.aa.aa.03:443 gate 30
    scheduler = wrr
    persistent = 7200
    protocol = tcp
    service = https
    request = "/areyouthere.php"
    receive = "OK web[0-9]+"
    checktype = negotiate

virtual = yy.yy.yy.yy:80
    fallback = yy.yy.yy.yy:80 gate
    real = bb.bb.bb.01:80 gate 10
    real = bb.bb.bb.02:80 gate 10
    real = bb.bb.bb.03:80 gate 10
    scheduler = wrr
    persistent = 30
    protocol = tcp
    service = http
    request = "/areyouthere.php"
    receive = "OK web[0-9]+"
    checktype = negotiate

virtual = yy.yy.yy.yy:443
    fallback = yy.yy.yy.yy:443 gate
    real = bb.bb.bb.01:443 gate 10
    real = bb.bb.bb.02:443 gate 10
    real = bb.bb.bb.03:443 gate 10
    scheduler = wrr
    persistent = 30
    protocol = tcp
    service = https
    request = "/areyouthere.php"
    receive = "OK web[0-9]+"
    checktype = negotiate
~~~~~

In the second webserver cluster persistence is set lower allowing better balancing between all the servers rather than keeping the requester locked onto the same server for long periods of time resulting in less efficient load balancing.

There are other scheduling methods you can use and routing methods but I will not cover those here. If you would like to learn about them you can always man ldirectord and the manpage describes them and how to use them.

The final step is to setup specific TCP/IP IPv4 settings through sysctl. To do this you can use /etc/sysctl.d/##-rulename.conf. I'll use /etc/sysctl.d/20-lvshost.conf in this configuration:

~~~~~
net.ipv4.ip_forward=1
net.ipv6.conf.eth1.forwarding=1
net.ipv4.vs.expire_quiescent_template=1
~~~~~

This sets up the forwarding globally, enables forwarding for IPv6 as well for the specified eth1 device, and enables quiescent expiration. To load these settings now you can run sysctl -p /etc/sysctl.d/20-lvshost.conf and it will load in the new settings immediately.

# Setting Up The Real Servers

Earlier, when I mentioned direct routed servers forward the packet to the real server IP without changing it which requires the real server to accept packets for the VIP as if it was it's own? Here I will describe how to do this for both Linux and Windows 2008 R2 (maybe 2008 R1 as well). I would cover earlier editions of Windows but it is more complicated, though still possible.

The trick is to assign the VIP itself to the loopback network interface with a netmask of 255.255.255.255. The reason is is to prevent it from trying to ARP it which would cause an IP conflict with multiple systems having the same IP.

## Linux

There's two things to do on Linux to set this up. First is to set it up the TCP/IP options for sysctl. I'll use /etc/sysctl.d/20-lvsdest.conf with the following contents:

~~~~~
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
net.ipv4.conf.eth0.arp_ignore = 1
net.ipv4.conf.eth0.arp_announce = 2
~~~~~

What these options do is effectively tell the kernel to ignore arp requests not local to it. For lo, the “local” IP would effectively be 127.0.0.1, your VIP's wouldn't be in that reserved network at all so it would completely prevent sending out arps about it, as well as it would ignore any arps coming in about it. In arp_announce, this tells it to only broadcast if the address is local to the primary IP of it for IP's bound to it. This would allow you to still have multiple IP aliases bound to it and be publically announced.

As for the VIPs themselves, many different distributions have different ways of handling how to set your VIPs up using the loopback adapter, add an IP alias of the VIP with the netmask of 255.255.255.255 or cidr netmask of 32, or even use the actual netmask for the appropriate network. Below I'll provide some case examples for various distributions:

### Gentoo

In /etc/conf.d, edit net:

~~~~~
config_lo=(
    "xx.xx.xx.xx/32"
    "yy.yy.yy.yy/32"
~~~~~

### Debian or Ubuntu

In /etc/network/interfaces you can setup your loopback aliases similar to the follow:

~~~~~
auto lo:1
iface lo:1 inet static
        address x.x.x.x
        netmask 255.255.255.x
~~~~~

For every VIP you will need another lo:X device for.

### OpenSUSE

More later.

FIXME

### RHEL, Fedora, CentOS

More later.

FIXME

## Windows 2008 R2

Windows by default only has a dummy loopback interface for 127.0.0.1 and that's it. To get this to work in Windows we have to first install the Virtual Loopback Adapter. To do this, bring up the Control Panel and go to Device Manager. Expand open Network adapters and first check if you have the Microsoft Loopback Adapter installed, if not, on the menu go to Action → Add Legacy Hardware. Click Next until you get a question relating to searching for and install hardware automatically or manually. Choose manually because the loopback will not be autodetected. Next, look through the list for Network Adapters and Next again. You'll be provided with a list of Manufacturers and adapters. First activate Microsoft, then on the right select Microsoft Loopback Adapter. Once you choose Next again it will confirm to install and Next will install it. It doesn't take long, and likely will not require a reboot.

Now to setup the loopback adapter itself to accept IP's directed to the VIP. Open up the Control Panel again but this time open Network and Sharing Center, then on the right pane, Change adapter settings. You will see the Loopback adapter interface there. I always recommend naming all your adapters to something useful than “Local Area Connection” with the optional suffix of #1, or #2, etc. I name each interface respective to it's purpose or network connection basis, such as “Internet Connection”, for interfaces directly facing the internet, “LAN Connection” for interfaces restricted within the local LAN, “DMZ Connection” for interfaces in the Demilitarized Zone, and “Loopback” for the loopback virtual adapter.

Right click on the Loopback adapter and choose Properties, select the Internet Protocol Version 4 (TCP/IPv4) and click Properties. Here you will need to choose to use the following IP address and fill in the details of your VIP address with the subnet mask of 255.255.255.255. Do not set a gateway or any DNS server addresses.

If you need further VIPs, click on Advanced and add each VIP address in the IP addresses list.

# Managing LVS

This section explains how to manage a running ldirectord server to temporarily remove servers from the queue for performing maintenance or for any other reason, adding new servers, viewing the current director statistics, etc.

LVS uses the tool ipvsadm to setup the routes used.

First, lets take a look at our routes as they are in the provided examples from above. To do this use the command:

~~~~~
ipvsadm -L -n
~~~~~

Which would output something like:

~~~~~
IP Virtual Server version 1.2.1 (size=########)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  xx.xx.xx.xx:80 wrr persistent 7200
  -> aa.aa.aa.01:80               Route   10     12         31
  -> aa.aa.aa.02:80               Route   30     13         28
  -> aa.aa.aa.03:80               Route   20     12         33
TCP  yy.yy.yy.yy:80 wrr persistent 7200
  -> bb.bb.bb.01:80               Route   10     42         76
TCP  xx.xx.xx.xx:443 wrr persistent 7200
  -> aa.aa.aa.01:443              Route   20     76         134
  -> aa.aa.aa.02:443              Route   10     75         128
  -> aa.aa.aa.03:443              Route   30     77         142
TCP  yy.yy.yy.yy:443 wrr persistent 7200
  -> yy.yy.yy.yy:443              Local   1      23         30
~~~~~

Uh oh! We got trouble! You see that last entry on yy.yy.yy.yy:443 how it's being routed locally? This means that, from our example, the SSL website is not responding as expected from any of the three real servers in the cluster and has went to the fallback on the running lighttp fallback. Even worse is, it's got active connections, so something needs to get done quick. Usually this means the servers are all down, or your response query is not getting what is expected, those are the first things to check for sure. It is usually not a good idea to directly modify the LVS table with ldirectord running but if you're sure it's up you could call something like:

~~~~~
ipvsadm -a -t yy.yy.yy.yy:443 -r bb.bb.bb.01:443 -g -w 10
~~~~~

This will force the entry into place to add director in. But the fallback is still there as you would notice from another ipvsadm -L -n result. Lets remove that quickly!

~~~~~
ipvsadm -d -t yy.yy.yy.yy:443 -r yy.yy.yy.yy:443
~~~~~

That deletes the old route that was used in the fallback. You can get further details by looking up the ipvsadm manpage. To simplify reading understanding, -A, -E, and -D, adds the equivalent of a virtual = xx.xx.xx.xx:port, the root definition, and -a, -e, and -d manges route information for a service.

Now, lets say you need to temporarily take a server offline. You can either set it's weight to 0 or delete it from the list. You could also edit the ldirector.cf file and comment out the line and reload ldirectord. To set the weight to 0 you would do:

~~~~~
ipvsadm -e -t xx.xx.xx.xx:80 -r aa.aa.aa.03:80 -w 0
ipvsadm -e -t xx.xx.xx.xx:443 -r aa.aa.aa.03:443 -w 0
~~~~~

To set both services currently assigned to the RIP of aa.aa.aa.03 to not be used. Give it a few minutes before you start taking things offline though. People may still be using the service at the time and they will not be cut off.

You can also add and remove service groups and services that are not controlled by ldirectord similarly. I won't cover that here and simply just recommend reading the ipvsadm manpage.

## Setting up the Linux Kernel


Most binary based distributions of Linux will likely (though not guaranteed), have LVS support built in natively or as modules. This section does not cover those because it can vary greatly between distributions. Instead I will cover how to actually compile a kernel which distributions like Gentoo you would have to do anyway.

It is assumed you know somewhat a little about compiling a Linux kernel. If you are not, I would suggest reading up about it before you do, especially if your distribution may have issues with a custom kernel as most of the time they have lots of patches done to them and and extremely modularized to be able to support a large variety of configuration options.

First lets get to the configuration of the kernel. Usually this is going to be in the /usr/src/linux directory. In the directory where you have the kernel source, run make menuconfig to start configuring.

~~~~~
Networking Support ->
  Networking options ->
    [*] TCP/IP networking
      [*] IP: multicasting
      [*] IP: advanced router
      [*] IP: policy routing
      [*] IP: tunneling
      [*] IP: GRE tunnels over IP
      [*]   IP: broadcast GRE over IP
      [*] IP: multicast routing
      [*]   IP: PIM-SM version 1 support
      [*]   IP: PIM-SM version 2 support
      [*] IP: ARP daemon support (EXPERIMENTAL)
      [*] IP: TCP syncookie support (disabled per default)
      [*] IP: IPComp transformation
      [*] IP: IPsec transport mode
      [*] IP: IPsec tunnel mode
      [*] IP: IPsec BEET mode
      [*] Network packet filtering framework (Netfilter) ->
        [*] Advanced netfilter configuration
            Core Netfilter Configuration ->
              Enable or module all.
        [*] IP virtual server support ->
          Enable or module all.
          IP: Netfilter Configuration ->
            Enable or module all but IP Userspace queueing via NETLINK (OBSELETE)
          IPv6: Netfilter Configuration ->
            Enable or module all as needed for IPv6, as abve, not Userspace queueing.
      [*] Security Marking
      [*] 802.1Q VLAN Support
      [*]   GVRP (GARP VLAN Registration Protocol) support
      [*] QoS and/or fair queueing ->
        Enable or module all as needed/desired.
~~~~~

You do not have to enable all of these, but for a routing server, this and more is suggested, to allow full expansion of any needs that may arise.
