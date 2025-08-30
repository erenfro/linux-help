---
{"publish":true,"title":"Resizing Ext Filesystems","created":"2025-08-29","modified":"2025-08-30T00:26:59.903-04:00","published":"2025-08-29","tags":["filesystems","ext2"],"cssclasses":""}
---

Resizing Linux filesystems is something you may need to do from time to time. Often times you can even do this while it remains mounted and in use.

# Growing ext3 & ext4

Online growing has been supported in ext3 for a long time (exact date needed), however ext4 has not always has online resizing functionality. Only since kernel 3.x (need exact version). Though this functionality has been backported, per-distribution, to older kernels.

## Partitioned Filesystem

If your filesystem is on a partition, you will first need to resize the actual partition itself. You can do so with the fdisk utility, but for this to work, it has to have room to grow into. Assuming that is the case, bring up fdisk and look at your partition table:

~~~~~
fdisk /dev/sda
~~~~~

Use this tool to examine if you have enough space after your partition you need to resize has ample room to grow. If not, you will need to use alternative utilities to resize, reallocate, and realign the data, such as parted &amp; gparted can do. To resize the partition, simply remember the “Start” sector number, and the end sector, delete the partition, re-create it with the same Start sector, and increase the end sector as needed for the growth you desire.

Once that is done, resizing the underlying filesystem is as easy as:

~~~~~
resize2fs /dev/sda4
~~~~~

That's suggesting that we resized our 4th partition into new space, this command would grow the filesystem into the new available space and increase the available capacity.

## Logical Volume Management

If your filesystem is on an LVM(Logical Volume Manager) LV(Logical Volume), then resizing it becomes very easy. Even easier than partition based, depending on how your LVM is setup of course.

Full-disk LVM is the easiest to work with as there is no underlying partition tables to even worry about at all. This is the most ideal way to use LVM in fact. However, LVM PV(Physical Volume)'s can be contained within a partition as well, so you would have to possibly resize the partition if you're trying to grow out your physical volume allocation.

### Full-Disk LVM

To grow your LV(Logical Volume) on a full-disk LVM you first need to grow your PV(Physical Volume). After the underlying disk has grown, geometry updated, etc (as from [Hardware](/wiki/hardware/resize-hdd)), you can run this command:

~~~~~
pvresize /dev/sdX
~~~~~

This will resize your LVM PV(Physical Volume) to the new size allowing you to allocate more space to your LV(Logical Volume). Before you actually start resizing your LV(Logical Volume), you should check your VG(Volume Group) so you can make sure it recognizes the new space, and how many extents you have available for growth to use in resizing.

~~~~~
vginfo
~~~~~

Example output:

~~~~~
-- Volume group --

VG Name dbd

  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               240.00 GiB
  PE Size               4.00 MiB
  Total PE              61439
  Alloc PE / Size       30719 / 120.00 GiB
  Free  PE / Size       30720 / 120.00 GiB
  VG UUID               ZnKrCO-n1nG-tdOy-sqE5-45E5-7NNo-0UPrWd
~~~~~

In the example above, our disk was 120GB, but was increased to 240GB from the hypervisor. We can see we have 30720 extents (120 GiB) to use in growing our LV(Logical Volume). So, lets initiate the resize:

~~~~~
lvresize -l +30720 /dev/dbd/mysql
~~~~~

Now our mysql volume has been resized, however we still need the final step of resizing the actual filesystem itself:

~~~~~
resize2fs /dev/dbd/mysql
~~~~~

There we go! Our mysql database now has 240GB instead of 120GB available for use, and we did this all without ever shutting down the MySQL server.

### Partition-Based LVM

FIXME
