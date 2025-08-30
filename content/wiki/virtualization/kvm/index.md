---
{"publish":true,"title":"KVM","created":"2025-08-29","modified":"2025-08-30T11:17:52.752-04:00","published":"2025-08-29","tags":["virtualization"],"cssclasses":""}
---

Linux's native virtual machine hypervisor is KVM(Kernel Virtual Machine). This one is based on QEMU which was a software only based virtual machine that didn't use any hardware to enhance the hypervisor environment. KVM actually does require CPU features for virtualization in order to work. The main advantage of this is that you get a virtual machine that is 99% as fast as the physical hardware that it's running under.

You can read up some about Linux KVM at their website: [Linux-KVM](http://www.linux-kvm.org/)

When using KVM and yet wanting to secure the Host OS with a stateful firewall at the same time, you can disable full binding to the bridge interfaces specifically. This will prevent it from directly effecting traffic travelling through the bridges which could cause strange and unusual anomalies (See Fedora bug #512206). To prevent this you can use these sysctl settings:

~~~~~
net.bridge.bridge-nf-call-arptables = 0
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
net.bridge.bridge-nf-filter-pppoe-tagged = 0
net.bridge.bridge-nf-filter-vlan-tagged = 0
~~~~~

Putting these sysctl settings into /etc/sysctl.d/kvm.conf or similar will allow it to be applied at every boot in most distributions.
