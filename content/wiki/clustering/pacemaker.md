---
{"publish":true,"title":"Pacemaker","created":"2025-08-29","modified":"2025-08-30T00:15:17.460-04:00","published":"2025-08-29","tags":["clustering","pacemaker"],"cssclasses":""}
---

# About Pacemaker

Pacemaker is a CRM(Cluster Resource Manager) with a lot of active development and functionality. It is the predecessor to heartbeat and even uses Resource Agents from heartbeat for it's functionality.Pacemaker, like heartbeat can use both lsb-init scripts that are in /etc/init.d/* and ocf resource agent scripts specifically designed for pacemaker under the same original standards as heartbeat originally.

To use lsb-init scripts, you have to verify a few things before anything. All lsb scripts need to have proper error code returns and handling of specific command parameters, stop, start, and status at the very minimum. Check out Appendix A to check if your scripts are lsb compatible.

# Configuring the Cluster

In configuring pacemaker, you need to understand what all is involved, plan out how you want the cluster to work and if your operating systems differ any, what compatibility issues there may be in joining multiple systems to the same DC(Domain Controller). Pacemaker is run through corosync which is the cluster group communication system which handles the majority of communication. Along with that there's openais which is the Application Interface Specification layer. Some things will depend on corosync while others may need all of openais to work. Smaller parts of pacemaker's stack is the cluster-glue and heartbeat which are needed mostly in building and using the entirety of the system.

I will mostly detail how to work with installing pre-built pacemaker stacks from various distributions of Linux.

# Installing Pacemaker

## Debian 6

Unlike Debian 5, pacemaker is now finally part of Debian and easy to install. Debian 6 comes with pacemaker 1.0.x so keep that in mind with any compatibility concerns as will be mentioned later in this document. The following will install the pacemaker suite for Debian:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
aptitude install pacemaker corosync openais
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## OpenSUSE 11.x

OpenSUSE 11 has come with a very good pacemaker suite of packages out of the box, and are well maintained. You will get install pacemaker 1.1.x with openSUSE 11.x so keep that in mind for compatibility concerns if you will have any systems in the same cluster group running pacemaker 1.0. Installing them is simple as the following:

~~~~~~~~~~~~~~~~~~~~~
zypper in openais pacemaker
~~~~~~~~~~~~~~~~~~~~~

## Ubuntu 10.04 LTS

Ubuntu 10.04 has pacemaker packaged and they work for the most part, but they are not perfect to say the least. Some problems you may face by the stock packages are with ocfs2, gfs2, clvm, or other clustered filesystems and volume management systems. If you don't need these then you're fine to install the basic packages.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
aptitude install pacemaker corosync openais
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Alternative to that, there is a PPA for the Lucid Cluster Stack for installing pacemaker on Ubuntu 10.04.

More later.

## CentOS 5.x

FIXME

# Configuring Pacemaker

Once your distribution's pacemaker cluster stack is installed, configuring the main portion is relatively simple. The single most important part is configuring /etc/corosync/corosync.conf for the Domain Controller communication to start taking effect once it's all starting up.

I will detail out pacemaker 1.0 and pacemaker 1.1 configuration files for corosync as there's minor but important differences that may need to be noted.

## Configure Pacemaker 1.0

FIXME

## Configure Pacemaker 1.1

Here is an example working configuration file for one of my cluster groups.

**/etc/corosync/corosync.conf:**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Please read the corosync.conf.5 manual page
compatibility: whitetank

aisexec {
        # Run as root - this is necessary to be able to manage
        # resources with Pacemaker
        user:           root
        group:          root
}

service {
        # Load the Pacemaker Cluster Resource Manager
        ver:            0
        name:           pacemaker
        use_mgmtd:      yes
        use_logd:       yes
}

totem {
        # The only valid version is 2
        version:        2

        # How long before declaring a token lost (ms)
        token:          5000

        # How many token retransmits before forming a new configuration
        token_retransmits_before_loss_const: 10

        # How long to wait for join messages in the membership protocol (ms)
        join:           60

        # How long to wait for consensus to be achieved before starting
        # a new round of membership configuration (ms)
        consensus:      6000

        # Turn off the virtual synchrony filter
        vsftype:        none

        # Number of messages that may be sent by one processor on
        # receipt of the token
        max_messages:   20

        # Limit generated nodeids to 31-bits (positive signed integers)
        clear_node_high_bit: yes

        # Disable encryption
        secauth:        off

        # How many threads to use for encryption/decryption
        threads:        0

        # Optionally assign a fixed node id (integer)
        # nodeid:       1234

        interface {
                ringnumber:     0

                # The following values need to be set based on your environment
                bindnetaddr:    172.17.5.2
                mcastaddr:      226.94.1.1
                mcastport:      5451
        }
}

logging {
        fileline:       off
        to_stderr:      no
        to_logfile:     no
        to_syslog:      yes
        syslog_facility: daemon
        debug:          off
        timestamp:      off
}

amf {
        mode: disabled
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There's only really two parts of this configuration you need to adjust for it to work on your system, usually. Under totem and interface, there's the bindnetaddr, mcastaddr, and mcastport. My setup here binds the network address to the internal LAN IP of my cluster group housed on each server, so this changes between each node in the cluster group. The mcastaddr is the multicast address to send to, and the mcastport is the port to transmit through. All you need to change is the bindnetaddr for each node to match your network setup.

Once you have setup your corosync.conf on each node it's time to start up the cluster suite on each node. On openSUSE this is /etc/init.d/openais start, usually on others it is /etc/init.d/corosync start.

It will take a few moments for a Domain election to complete but once it is done, you can check the status of the communication by using:

~~~~~~~~
crm status
~~~~~~~~

**or:**

~~~~~~~
crm_mon
~~~~~~~

Both will show output similar to this:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
============
Last updated: Wed Apr 20 15:15:50 2011
Stack: openais
Current DC: system1 - partition with quorum
Version: 1.1.5-ecb6baaf7fc491b023d6d4ba3e0fce22d32cf5c8
2 Nodes configured, 2 expected votes
0 Resources configured.
============

Online: [ system1 system2 ]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When you see that Current DC: with a node specified as the DC, you know that it is working. Keep in mind though, if you get only 1 node total listed then it is not communicating. In a cluster group of 6 nodes, the Current DC on each will all be the same node. One node is elected to be the Domain Controller until either that node is turned off, shut down, restarted, or otherwise taken out of the group for any period of time, which causes a re-election of a new Domain Controller to be appointed. Yes, Pacemaker is a democracy and only one appointed leader is granted at a time.

# Configuring Pacemaker Resources

Configuring resources is the hardest aspect of working with pacemaker, especially if you have never worked with any kind of cluster resource manager before. It is like speaking an entirely new language to say the least. I will demonstrate a couple examples I use in production environments.

To configure pacemaker resources, you need to use the corosync resource manager tool, crm. crm is a shell interface to managing corosync and pacemaker. Familiarize yourself a little with this tool and understand how it works by doing similar to the following:

~~~~~~~~~~~~~~~~~~~
# crm
crm(live)# help
crm(live)# configure
crm(live)configure# help
crm(live)configure# show
crm(live)configure# cd
crm(live)resource# help
crm(live)resource# show
crm(live)resource# exit
~~~~~~~~~~~~~~~~~~~
As you can see, it's fully interactive, you can get help on everything, see what's configured already, etc.

## Example 1: Dual Firewalls

For this to work you will need a reasonably current version of the conntrackd resource-agent available from pacemaker's github resource-agent repository available at: github:resource-agents. You shouldn't have to modify much of this to make it work for your distribution, mostly just where it reads the ocf initialization scripts, then you're set. Put this into your normal /usr/lib/ocf/resource.d/heartbeat directory and continue after making the necessary changes that relate to the OCF_FUNCTIONS_DIR and ocf-shellfuncs, usually near the top after all the comments. The other resource-agents you will see used here will be available in our file repository when it becomes available, those will be part of the ocf:hds:* resource-agents.

This example is a non-symmetric cluster that uses some more advanced methods of using attributes, location constraints, and custom resource agents I personally created for my firewall setup. The configuration is as shown below:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
node fw1 \
        attributes firewall="100"
node fw2 \
        attributes firewall="50"
primitive lan-gw ocf:heartbeat:IPaddr2 \
        params ip="172.17.1.0" cidr_netmask="16" nic="eth0"
primitive net-gw ocf:heartbeat:IPaddr2 \
        params ip="xx.xx.xx.180" cidr_netmask="29" nic="eth1"
primitive srv_conntrackd ocf:heartbeat:conntrackd
primitive upnp-gw lsb:linux-igd
primitive vip1 ocf:hds:proxyarp \
        params ip="xx.xx.xx.178" ext_iface="eth1" int_iface="eth0"
primitive vip2 ocf:hds:proxyarp \
        params ip="xx.xx.xx.179" ext_iface="eth1" int_iface="eth0"
group gateway lan-gw net-gw upnp-gw \
        meta target-role="Started"
group vips vip1 vip2 \
        meta target-role="Started"
ms conntrackd srv_conntrackd \
        meta master-max="1" master-node-max="1" clone-max="2" clone-node-max="1" notify="true" target-role="Started"
location conntrackd-run conntrackd \
        rule $id="conntrackd-run-rule-0" -inf: not_defined firewall or firewall number:lte 0 \
        rule $id="conntrackd-run-rule-1" firewall: defined firewall
location gateway-loc gateway \
        rule $id="gateway-loc-rule-0" -inf: not_defined firewall or firewall number:lte 0 \
        rule $id="gateway-loc-rule-1" firewall: defined firewall
location vips-loc vips \
        rule $id="vips-loc-rule-0" -inf: not_defined firewall or firewall number:lte 0 \
        rule $id="vips-loc-rule-1" firewall: defined firewall
colocation conntrackd-loc inf: conntrackd:Master gateway:Started
colocation vips-on-gateway inf: vips net-gw:Started
order vips-after-gateway inf: gateway:start vips:start
property $id="cib-bootstrap-options" \
        cluster-infrastructure="openais" \
        expected-quorum-votes="2" \
        stonith-enabled="false" \
        placement-strategy="utilization" \
        symmetric-cluster="false" \
        no-quorum-policy="ignore"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this example you will see many things. An internal LAN IP resource, an Internet IP resource, those of which are being used as a gateway IP for the network infrastructure internally and externally. These gateway IP's are called VIPs, or Virtual IP addresses.

You will also see two VIPs that are not actually binding IP's but setting up proxyarp resources. Another system actually binds those IP's but they are passed through the firewall of the active firewall node and forwarded to the actual machine binding the IP.

You will also see two application resources, conntrackd and linux-igd (or upnpd). conntrackd you can read about in this wiki in another section. linux-igd is the application server used to provide IGD-like UPnP support in my network, since this is a home network it can be useful, however not recommended in a corporate environment.

As for location constraints, you may have noticed the attributes on both nodes with firewall=“100” and firewall=“50”. Those are weight scores for which node is going to be active and fail-over. The gateway-loc constraint insures that it will not be active on any node with a firewall score less than or equal to 0 or not even defined at all, and then based on score. In this example, the node fw1 will be the dominant server and become primary whenever it is online and ready. Should fw1 go down for any reason, fw2 will immediately take over.

## Example 2: libvirt managed Virtual Servers

FIXME

# Appendixes

## Appendix A: Checking LSB Init Compatibility

The relevant part of LSB spec can be found at: [LSB-Init-Refspec](http://refspecs.freestandards.org/LSB_3.1.0/LSB-Core-generic/LSB-Core-generic/iniscrptact.html). It includes a description of all the return codes listed here.

Assuming some_service is configured correctly and currently not active, the following sequence will help you determine if it is LSB compatible:

1. Start (stopped): /etc/init.d/some_service start ; echo “result: $?”
    1. Did the service start?
    2. Did the command print result: 0 (in addition to the regular output)?
2. Status (running): /etc/init.d/some_service status ; echo “result: $?”
    1. Did the script accept the command?
    2. Did the script indicate the service was running?
    3. Did the command print result: 0 (in addition to the regular output)?
3. Start (running): /etc/init.d/some_service start ; echo “result: $?”
    1. Is the service still running?
    2. Did the command print result: 0 (in addition to the regular output)?
4. Stop (running): /etc/init.d/some_service stop ; echo “result: $?”
    1. Was the service stopped?
    2. Did the command print result: 0 (in addition to the regular output)?
5. Status (stopped): /etc/init.d/some_service status ; echo “result: $?”
    1. Did the script accept the command?
    2. Did the script indicate the service was not running?
    3. Did the command print result: 3 (in addition to the regular output)?
6. Stop (stopped): /etc/init.d/some_service stop ; echo “result: $?”
    1. Is the service still stopped?
    2. Did the command print result: 0 (in addition to the regular output)?
7. Status (failed):
    1. This step is not readily testable and relies on manual inspection of the script. The script can use one of the error codes (other than 3) listed in the LSB spec to indicate that it is active but failed. This tells the cluster that before moving the resource to another node, it needs to stop it on the existing one first.

The script can use one of the error codes (other than 3) listed in the LSB spec to indicate that it is active but failed. This tells the cluster that before moving the resource to another node, it needs to stop it on the existing one first.

If the answer to any of the above questions is no, then the script is not LSB compliant. Your options are then to either fix the script or write an OCF agent based on the existing script.

One thing to note that I have noticed a lot of init scripts that utilize common lsb-init script functions provided by the distribution to use things such as 'set -e' inside the script before doing anything: This is extremely bad as any error in any portion of the script or anything that returns a non-zero exit code will always return an exit code that is not properly handled, or given the proper error return code expected for LSB compliance.

FIXME
