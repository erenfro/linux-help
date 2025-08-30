---
{"publish":true,"title":"Virtualization","created":"2025-08-29","modified":"2025-08-30T11:17:02.257-04:00","published":"2025-08-29","cssclasses":""}
---

Virtualization is a new and still growing, but powerful technology being used today. Virtualization allows you to pocket various system services into their own isolated virtualized environment. What this allows you to do is many important things. Hardware is virtualized so you don't have to worry about hardware comparability within the virtual machine as much. You can move virtual machines from physical computer to different physical computers, allowing portability to be much higher, but even more so, many virtual machine hypervisors allow you to do this live, while the system is still running. With virtual machines you can also setup disk pools in a more efficient manner as well, allowing you to store your virtual machine “disks” that run the OS, as well as the disks holding the data that you need, such as website content, etc.

These options you have from virtual machines gives you a great new world of exploration into many different areas of interest. One major one is high availability, because you can migrate your virtual machines from physical machine to physical machine, you could essentially move a machine to another to do maintenance on the physical machine it was running on, then move it back when you're done. This provides little to no downtime for this.

There are several hypervisors that can run virtual machines today, most of the ones I will be covering are free and/or opensource options available from Linux.

* [KVM](kvm)
* [LXC](lxc)
* [OpenVZ](openvz)
* [Xen](xen)
