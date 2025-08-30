---
{"publish":true,"title":"Partitions and Filesystems","created":"2025-08-29","modified":"2025-08-30T10:32:05.665-04:00","published":"2025-08-29","tags":["partitions","filesystems"],"cssclasses":""}
---

In Linux, just like any other operating system, you have partitions. Partitions separate various areas where data is stored on your hard drives, which can be a good and bad thing if not done right.

For example, in setting up a new Linux system, I always recommend a minimal partitioning schema to use to separate data from operating system. I will go through that later, but before that, I will describe how exactly Linux handles it's file system a little more so you can understand further about why partitioning is important.

# Partitions

Every operating system uses partitions of some kind for setting up containers on your hard disk for storage uses. Linux supports a few different partition formats, but the one I'll cover here is the standard msdos partition table which is generally the native default of Linux still today.

Partitions are used to separate data in different portions of a hard disk drive. Think of partitions like dresser drawers for clothes. You wouldn't find it as useful to have 4 drawers and just random stuff in each one, would you? Well, maybe some of you wouldn't care, but a more organized approach would be to segment out each drawer's use for different types of clothing instead. This same logic applies to partitions on hard drive storage.

Many distributions also have the option to install to LVM Volumes, rather than partitions. LVM is Linux's Logical Volume Manager system which instead of seperating data portions by partitions, it does it by contained volumes. The approach is extremely similar, but less restrictive than the partition layout that msdos provides on ther underlying layers. Personally, I will prefer using LVM over partitions as much as possible, but I will cover LVM in a different section of this site as it does add some major differences as to how things work in more advanced or recovery measures needed.

# File Systems and Mounts

Linux uses mount points based on a singular and ever growing hierarchy. Everything has a root, and that root literally starts with the path of /. From there, you have many other directories based off your root, such as the most important ones, /bin, /boot, /dev, /etc, /home /lib, /sbin, /tmp, /usr, and /var, which looks something quite like this chart below:

* /
    * /bin
    * /boot
    * /dev
    * /etc
    * /home
    * /lib
    * /sbin
    * /tmp
    * /usr
    * /var

What do all these directories do? Well, lets run through them:

/

* This is the root directory. This is where the whole tree starts.

/bin

* This contains the majority of executable files that are general purpose but critical to the basic operating system needs in order to run the system. It includes executable programs that are needed to boot the system up in single user mode to bring the system up or repair it. Here you will also find the shell interpreter to which you can run commands, list directory contents, make directories, delete directories and files, and many more. Without the executables in /bin, you would not really have a usable operating system.

/boot

* Contains static files for the boot loader. This directory is where your kernel and boot loader data lives in order to actually boot up your system into a usable state.

/dev

* This directory is automatically generated mostly these days, and provides you with access to devices on your system. You can directly access devices like CD/DVD drives, hard disk drives, sound devices, and video devices.

/etc

* Contains configuration files which are local to the machine. Some larger packages, like X11, can have their own sub-directories below /etc. Site-wide configuration files may be placed here or in /usr/etc. Nevertheless, programs should always look for these files in /etc and you may have links for these files in /usr/etc. Along with that, this contains service scripts to start and stop various system services your system may need in order for a fully functional multi-user operating environment, including networking scripts and configurations.

/home

* Think of this as your home, because literally, it is. This is where all user accounts store their own data, personal configurations and everything. All your downloads, bookmarks, music, everything is generally stored here.

/lib

* This directory should hold those shared libraries that are necessary to boot the system and to run the commands in the root file system. This is like a library, literally. Instead of it being of books, however, this contains files that many other files share and use for operation. A primary example of how important this is, /lib contains a core library known as glibc, or libc. Everything on your system depends on this library file because they absolutely need it in order to run. Library files allows executable to share common functionality so that you don't have to have a whole bunch of unabridged books, per-se, thick, heavy, and full of redundant information in them, so that you are not wasting as much hard drive space or memory.

/sbin

* Like /bin, this directory holds commands needed to boot the system, but which are usually not executed by normal users. This contains privileged superuser executable files needed in order to not just use the system, but to administrate it. Tools such as that to add, remove, and update users, configure network interfaces, and setting up firewall rules, resides here.

/tmp

* This directory contains temporary files which may be deleted with no notice, such as by a regular job or at system boot up. This is like the bit-bucket for temporary data that you don't keep to keep all the time.

/usr

* This is kind of it's own root hierarchy for another tier of a lot of what's in /, but for other purposes. User executables are generally placed here, such as applications, software you use including Firefox.

/var

* Finally, we have variable data. This directory contains files which may change in size, such as spool and log files. It also contains files that store data for services, run-time information, email data, and database data. Generally, data used by software that needs to be persistent on disk.

With all those provided, there's some specific mentions to make note of. In most modern distributions of Linux, /dev and /tmp are usually virtualized file systems. /dev is virtual file system (of VFS) called devfs that automatically manages itself based on available devices it detects. Add a new hard drive, such as a USB thumb drive, and a new entry in /dev will appear for it, complete with all it's partition tables allocated on it. The other one, /tmp, is a virtual file system known as tmpfs which uses 50% of available RAM by default to allocate for /tmp. It only uses as much RAM as what's stored in /tmp however, so it doesn't use up all 50% of RAM just being present. If you need more available space for temporary files than 50% of your available RAM, then you would need to alter this after getting Linux all deployed and setup and make /tmp part of a file system instead.

Other virtual filesystems not mentioned are /proc and /sys, but for now I won't go into too much detail into those. They will be covered in more advanced topics.

# File Systems

Finally, one of the most important topics is, file systems. What you actually need on any partition to actually store and retrieve data in an organize manner on the hard disk drive. Linux actually supports many different file systems native to it, even some not native to it.

The file systems this chapter will cover that are native to Linux are:

**ext2, ext3, ext4:** The truly native file system for Linux. Since day 1, Linux has always come default with ext file systems. Usually this will be the most stable file system.

**XFS:** Provided by SGI for use in Linux and included in mainline Linux core for years now, this file system has many differences in stability and performance.

**JFS:** Provided by IBM for use in Linux and included in mainline Linux core for years now, this file system has many differences in stability and performance.

**ReiserFS:** Provided by Novell for use in Linux and has been in mainline Linux, but may at some point be deprecated as development of it is no longer maintained.

Each file system has it's own set of advantages and disadvantages, but all are native. I will try to give the best case scenario and requirements for each file system and what I could recommend for various use case purposes.

## ext2:

ext2 is the oldest native file system for Linux available. Today, it's barely ever used except in extremely low-end systems or mobile devices that don't want the overhead that ext3 or ext4 adds in. That said, ext2 is a very reliable file system but lacks in what modern file systems has which is journaling support. What journaling does is essentially pre-allocates some space to store files as they're being written, even partial files, and then when the system is able it migrates that into the actual final location it will be, glued together and everything to reduce any fragmentation that may have been caused.

## ext3:

ext3 is basically ext2 with journal capability. Many distributions still come with ext3 as the default file system to use simply because it has the most reliability track record and features at the moment. That said, ext3 is a very solid file system, and it's fully backwards compatible with ext2, to some degrees. When you mount an ext3 file system as ext2, all the data is available, usable, updatable, everything, but doing so will destroy the journal data it may still have and will need to be re-created at the next ext3 mount. I say this because there are some file system support for ext2 in other operating systems, such as Windows, FreeBSD, etc, but the same effect will cause the journal information to be destroyed likewise.

Some advantages of using ext3 are being able to resize the file system live. You can shrink it and grow it into new sizes while it's still mounted, and this poses a major benefit because for servers, you can enlarge a file system to provide more usable space without ever shutting it down.

## ext4:

ext4 is the newest native file system for Linux. This file system is semi-backwards compatible with ext3 but looses that compatibility when it is made fresh as ext4 from the start. ext4 uses both journals and extents to provide even more reliable and speedy access to data. That said, it also supports the same features as ext3 such as live resizing, however, live resizing was just recently added in Linux kernel 2.6.36. Even though it can't do it live on older versions of Linux, it could still do it while not mounted, or from a bootable disc not using the local drive for an operating system dependency.

## XFS:

XFS was open-sourced and provided to Linux by SGI originally used on SGI's Irix OS. It by itself is a very good file system but it can take some care to maintain it, and it needs a reasonable amount of RAM (2GB+ recommended) in order to use optimally. This file system does have potential for corruption if not on a system with redundant power systems to keep the system from uncleanly unmounting it, such as power failure. That said, this file system is extremely fast for both small files and large files. It was designed to be able to support large files very efficiently and even has a “real-time” option for working with such files. Along with that, though the file system itself prevents fragmentation as much as possible, it does have tools available to perform live defragmentation to it while it's in active use. This file system can grow in size to allow for more space allocation, but it cannot shrink in size without destroying and rebuilding it.

## JFS:

JFS is a file system made by IBM known as Journaling File System. It was originally designed for IBM's AIX operating system. This file system is very optimal for most uses as like any file system for Linux. It's in many ways, similar to XFS, ReiserFS, and ext4 as it does have journaling and extents. It does also have the same issue XFS has with resizing, you can grow the file system larger, but you cannot shrink it in size.

## ReiserFS:

ReiserFS is a pretty sturdy file system provided as a unique native file system designed specifically for Linux. It offered a totally new approach to how file systems could be made. That said, this file system is very efficient with small files, used to be optimally used for web server content because of that as an example. It's fully capable of being resized online and has the fastest of the boot-time file system checks of all the file systems. I do potentially see this file system becoming deprecated in Linux in it's future as development has stopped due to unpredictable situations of it's original developer.

# Structuring File System and Mounts

Putting this all together takes thought and knowing what to do in many cases. For you, we'll make it simple and give you a basic guideline based on a few different scenario choices to help you decide the best layout for your system.

First thing's first, I always recommend at the minimum 4 partitions (or 2 partitions and the rest in LVM volumes). The first one should always be for /boot, and unless you're using RHEL, CentOS, or Fedora or equivalent, this should only be about 120 MB in size. For the others mentioned, they recommend 500 MB primarily for their upgrade process actually consuming /boot for some of it's work.

Second most important filesystem is of course the root partition / itself. Most of the time this will never need to be more than about 20 GB because Linux is not all that large. 20 GB is more than plenty for practically everything you will need and using any more than this will most likely be a real waste of space usage.

The third important partition is the swap partition. I haven't gone over swap yet, but what it basically is is memory being stored onto the hard disk for various reasons including low physical memory available for use, or just taking software that's not very active and moving it to disk for the time being to allow more optimal use of memory for other things that are actually being used. Some people say swap should be the same size as how much RAM you have or even double what RAM you have. There's only two rules I follow for logic when deciding this factor. If this is for a system that you will want to hibernate and power down and bring the system back up in the exact state it was before powering down then your swap should be at least the size of your RAM, plus an additional 2GB as well. Otherwise, don't ever make this any higher than about 2GB, and no less than about 1GB. If you're system is running low on memory, the first thing it will do is try to swap out processes to disk, failing that it will start outright killing processes to recover memory by being somewhat self destructive in the purpose of survival. If you run out of RAM and swap combined there is a potential for the system to panic and completely lock up causing the system to be unusable until you reboot it.

Last partition is the largest one in most cases. The one to store /home in, your data, everything you want to keep. This can consume the remaining space of your hard drive, or you can split up the rest of it into other portions as needed.

Those are the most optimal settings for most every common use of Linux, but it lacks in some areas when considering different server usage purposes. For home use, it's perfect however. I'll give some areas of focus now towards what optional differences you may use for use in a server based environment.

## Mail Servers

Depending on how your mail server is going to be used, this can vary slightly. Using this on a system-wide mail server where all mail will be delivered to local users setup on this specific server, you will likely want to use what is called Maildir setup for each user's home directory (usually /home/username). If this mail server will handle mail in a virtual case scenario, where users may not exactly exist within the server itself, this would be similar, but stored in /var/spool/mail instead, so you would want to allocate space for that as needed.

Mail servers could also use the older traditional mailbox format known as mbox, but this format is not recommended. mbox actually stores every single email for a user into 1 single file, so as time goes on, this grows, and grows, and grows, unless mail is deleted of course. Maildir format uses a directory structure and 1 file per email, which is much more optimal.

Last thing to consider is the file system to use for mail server's data storage. ext3 or ext4 is fine for this purpose in most cases, as it is reliable and fast enough to handle most anything you throw at it. If you need any higher performance you would probably want to look into XFS or JFS, or even ReiserFS if you're dealing mostly with smaller emails with no or few attachments.

## Database Servers

MySQL and PostgreSQL, the two most common database servers used in Linux because they are free and open source projects, generally are setup to house their raw database files in /var/lib/mysql and /var/lib/pgsql respectively. When considering the storage for this, you would want to allocate space as desired in /var/lib/mysql or /var/lib/pgsql to house your data.

Considering filesystem use, you could use ext3 or ext4 which is fairly optimal, but I would more strongly recommend XFS for any database that is going to have a lot of use and growth to it. XFS would provide many benefits as it works extremely well with large files, which databases, most of the time all they do is grow, and grow.

## Web Servers

Web servers can very much differ from the actual content on it as to what it's going to need for proper tuning and allocation. The majority of most web servers are lots and lots of small files, such as html files or PHP scripts, etc. Most systems use /var/www for housing the content of websites, however I would strongly suggest instead to use /srv/www simply because that's what is set to be the current standard. You can choose to use either at your discretion of course. Whichever the case may be, you will want to allocate space dedicated to the purpose of this web server to use.

For the file system, there's 3 reasonably prime choices you could use: ext3, ext4, or JFS. XFS could also be used as well of course, but I would not personally suggest it for the purpose because it is not always the best at handling lots of smaller files. If you're dealing more with larger files like videos or large archives files then it might be ideal. ext3/4 is good in that it is well balanced for use in CPU load, reads, writes, and reasonably fast. JFS is one that may be ideal in the fact it uses less CPU the the others overall, allowing the web server to have more CPU for execution time running scripts such as PHP.

# Summing it all Up

Now that you have the basic idea of what kind of structure you need, I will provide a simple little hierarchy map with suggested sizes so you can visualize what this could look like.

## Home Desktop/Work Workstation

* / (20 GB)
    * /boot (120 MB)
    * /home (500 GB)
* swap (2 GB)

## Database Server (MySQL)

* / (20 GB)
    * /boot (120 MB)
    * /var/lib/mysql (100 GB)
    * /home (50-100 GB – Just in case you need to export/import databases)
* swap (2 GB)

## Mail Server (Virtual Users)

* / (20 GB)
    * /boot (120 MB)
    * /var/spool/mail (100 GB)
* swap (2 GB)

## Mail Server (Local Users with real users)

* / (20 GB)
    * /boot (120 MB)
    * /home (100 GB)
* swap (2 GB)
