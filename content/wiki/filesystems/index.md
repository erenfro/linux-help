---
{"publish":true,"title":"Filesystems","created":"2025-08-29","modified":"2025-08-30T11:34:59.181-04:00","published":"2025-08-29","tags":["filesystems"],"cssclasses":""}
---

Different filesystems have many different features provided by them. The filesystems I will cover, and their features will be primarily only POSIX complaint filesystems supported by Linux.

## POSIX Filesystems

* [ext2](ext)
* [ext3](ext)
* [ext4](ext)
* [btrfs](btrfs)
* xfs
* jfs

[Resizing ext](ext/resizing-ext)

## Comparisons

None yet. FIXME

## Features

#### Access Control Lists

ACLs are actually quite useful in filesystems for setting up specific directory trees and their files with permissions specifically adjusted for multi-user and multi-group access to them.

Resizing ext2/3/4 can be done with resize2fs

## Volume Management

LVM, or Logical Volume Manager, is the Linux way to manage volumes. LVM has a lot of powerful features including snapshots, resizing, striping for speed, mirroring for redundancy and more.

Resizing LVM can be done by simply using a command similar to: lvextend -L+20G volume

## RAID

RAID, or Redundant Array of Independent Disks, is a technology used to combine multiple hard drives together, similar to LVM but with much more on redundancy and speed. There are many levels of RAID which have different abilities.

* RAID-0 is block-level striping without parity or mirroring.
* RAID-1 is block-level mirroring without parity.
* RAID-2 is bit-level sriping with dedicated hamming-code parity. This RAID is not used much.
* RAID-3 is byte-level striping with dedicated parity. This RAID is not used much.
* RAID-4 is block-level sriping with dedicated parity. This RAID is not used much.
* RAID-5 is block-level striping with distributed parity.
* RAID-6 is block-level striping with double distributed parity.
* RAID-10 is otherwise known as RAID 1+0, is RAID-1 on RAID-9, or mirrored sets in a striped set. Requires at least 4 disks but allows for a lot of IOPS to be utilized with redundancy.

Software RAID provides the highest compatibility but can be the slowest due to it being strictly software-based. The fastest RAID hardware uses a battery backed buffer on the controller to allow for on-board memory to be used for write caching and has some level of fault tolerance due to the battery backup.

Resizing Software RAID arrays:

~~~~~
mdadm --create --verbose /dev/md0 --level=raid1 --raid-devices=2 /dev/hda2 /dev/hdb1
mkfs.ext3 /dev/md0
mount /dev/md0 /mnt/test
mdadm /dev/md0 --fail /dev/hda2 --remove /dev/hda2
cfdisk /dev/hda                                  # adjust partition size for hda2
mdadm /dev/md0 --add /dev/hda2
mdadm /dev/md0 --fail /dev/hdb1 --remove /dev/hdb1
cfdisk /dev/hdb                                  # adjust partition size for hdb1
mdadm /dev/md0 --add /dev/hdb1
mdadm --grow /dev/md0 --size=max
resize2fs /dev/md0
~~~~~

This works for online resizing if you read and understand what all is happening here with the mdadm tool, fdisk tool, and the final resize2fs as mentioned before in this documentation.

## Network Filesystems

Samba, NFS, Coda

### Clustered Filesystems

* GFS
* GFS2
* OCFS2

### Distributed Filesystems

* Ceph FS
* MooseFS
* DFS
* Sheepdog
* LustreFS

### Network Block Devices

* Ceph RBD
* iSCSI
* GNBD
* FoE
* AoE
