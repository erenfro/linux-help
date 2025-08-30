---
{"publish":true,"title":"Resizing HDD","created":"2025-08-29","modified":"2025-08-30T00:37:19.212-04:00","published":"2025-08-29","tags":["hardware","resize","hdd"],"cssclasses":""}
---

Whether you're hot swapping physical hardware out, adding a new virtual disk, or resizing an existing virtual disk, this guide will detail how to work with adding/removing new hard disk drives to your machine.

# Virtual Disks

When you add disks, or resize an existing disk, your virtual machine guest will not immediately know about it, and you will have to tell it about the changes.

Rescan underlying disk's geometry:

~~~~~
echo 1 > /sys/block/sdX/device/rescan
~~~~~

Rescan SCSI Bus:

~~~~~
echo - - - > /sys/class/scsi_host/hostX/scan
~~~~~
