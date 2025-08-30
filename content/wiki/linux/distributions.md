---
{"publish":true,"title":"Distributions","created":"2025-08-29","modified":"2025-08-30T10:24:32.665-04:00","published":"2025-08-29","cssclasses":""}
---

This section is all about Linux and Linux distributions

# Solus

Website: <https://solus-project.com/>

FIXME needs review

# Debian

Website: <http://www.debian.org/>

Debian is a very well put-together distribution. It doesn't always have the latest and greatest software, but Debian, for a server, is very good. As a desktop platform, it's never going to be my recommendation. Debian is always my secondary choice for servers as it has reliable and well tuned server software with it. Through backports you can get some later versions at the sake of potential security risks added, but as long as you know what you're doing you can keep a well oiled distribution like Debian secure.

Debian almost always has grade A support for just about anything. It's been around as one of the oldest distributions still around today, originally based on SLS.

I use Debian as my Proxmox VE clustered kvm & OpenVZ virtual servers.

# Fedora

Website: <http://fedoraproject.org/>

Fedora is an excellent distribution for it's bleeding edge qualities. It's not recommended to be used as a server, but as long as you can stand updating every 6 months it's possible to do reasonably well, but you may find yourself completely changing how you run your servers with every new release. As a workstation and as a desktop, it's actually quite formidable, comparable to openSUSE in fact, though it does lack the one thing that truly makes openSUSE desktop class, yast2. It does have some tools for managing simple configurations, and even a very nice firewall editor.

I didn't like Gnome 3 at first, but after understanding it, using it for a week, I grew to enjoy it and how it helped better organize things. About the only minor issue I have with it is the notification area can be a little annoying to use, to isolate out a specific notification icon so you can close it, or do something with it, since they accordion out.

If you prefer to use the properiatery Nvidia or ATI drivers, this distro will cause you problems in the long run, especially because they do not provide these drivers natively, and the 3rd party repos are usually several days behind all kernel releases, and they don't use DKMS to allow the sane auto-recompile of critical parts to work with the kernel through each update.

# openSUSE

Website: <http://www.opensuse.org/>

This is my distribution of choice for everyday desktop use as well as server use. This is a very well developed distribution and with YaST2 it makes everything conveniently in one tool to manage almost every system and server aspect of your system.

The only downsides to OpenSUSE is the lack of support for some software projects such as Virtualmin. Plan out your needs and if you find something that just won't work well because of lacking support, then think carefully on alternatives mentioned below.

I use OpenSUSE for iSCSI Targets, LDAP authentication servers, and database servers primarily. I also use it in virtual servers that run my communications software, such as ZNC and OpenFire. It's also used for my monitoring software which includes Zabbix, Splunk, and Nessus.

This distro does have official repositories, provided by ATI and nVidia, but they are not always going to match up to a brand new kernel revision being installed. If you use and prefer the prorpriatery drivers, you will occassionally have bumps of problems between software updates that include the kernel.

# Gentoo

Website: <http://www.gentoo.org/>

Gentoo is a rolling-release distribution where you can fine-tune compile every single piece of software you want, each with specific fine-tuned options of what you want that software to be compiled against. That said, it's a very good distribution if you don't mind compiling everything and setting up that fine-tuned control over everything.

As a server, however, this is okay, standalone, but in clusters or multiples, Gentoo can quickly get out of hand even if you know how to maintain package centralization.

I do not use, nor personally recommend Gentoo Linux for production use of any kind.

# CentOS

Website: <http://www.centos.org/>

CentOS is based on RHEL, an enterprise grade distribution. CentOS almost always has grade-A support from other software and that makes it a key choice for many people. As a server it's excellent. If you take the time to learn SELinux fully, you can secure everything down on it locked tighter than a drum. CentOS is also pretty decent as a workstation, but not exactly your everyday “desktop”.

CentOS's downsides is it's older software packages available. With 5.5 having Linux 2.6.18, you may find some hardware issues due to the kernel version being so old, this can be a bad thing by itself. CentOS 6 is still pending any kind of release at the time of this writing so I can only comment for now up to 5.5. Clustering with CentOS is not so pleasant as well, depending on what exactly you're trying to do. RHCS in 5.5 and below is okay for some things, but not so good for others, especially when you need resources that are Master/slave such as DRBD. There is pacemaker for CentOS, but it lacks some things you may need elsewhere, like pacemaker-compatible dlm_controld and gfs2_controld.

I use CentOS presently only for my business's Scalix mail server, in a clustered environment where there's two redundant smtp floater servers on kvm guests, and one master web access, imap, pop3, etc, services running the rest.

# Arch Linux

Website: <http://www.archlinux.org/>

Arch Linux brings rolling-release to a new era. Arch has both binary packages from the core, extra, and community repositories, while having compiled 3rd party packages from AUR. This allows for very simplified management of the core system itself making up the distribution so that development on it can be kept up-to-date more rapidly and easily than other traditional distributions. As a result of this, Arch Linux is running the very latest kernels available, utilizing systemd and journald for boot up and logging, and giving the easy to maintain approach concept to the distribution.

AUR, or Arch User Repository, is the repository that is managed by the community and compiled by the user wanting to use it. Similar to Gentoo's approach in some aspects, but you only download what you need when you want it. There are tools to automate this process, such as packer and yaourt. Arch's simplified build system, PKGBUILD, is basically a bash script with structured definitions. This makes creating new packages, and updating existing ones very easy.

The pros of Arch is, it is what you make of it, to most of the idea behind it. Beyond the core installation, there is nothing but choices. For people whom have been using Linux for years, this distribution will definitely teach you new things for sure. Old tools you are used to aren't installed by default, so if you haven't already learned your way around iproute2's ip and ss commands, it's beyond time to.

On the downside, the cons of Arch is that everything has to be configured by hand. What you get with Arch is plain vanilla everything. You can easily spend days to even months just tweaking the look and feel of your desktop or server. For servers, this may be a downer as well, however, not always true. Servers are usually held back many versions with proven reliable software. This is where Arch may or may not thrive depending on the specific uses. Newest is not always the greatest.

With every distribution of course, their are it's differences. All in all, Arch has made it to solid standing in the community all on it's own, and it's expected to continue to grow.

# Ubuntu

Website: <http://www.ubuntu.com/>

Ubuntu has a lot of haters and lovers combined. It also has a lot of experienced and new users using it. Ubuntu is actually based on Debian, but with newer software, some less tested, some just more current and stable. Every 2 years Ubuntu comes out with an LTS (Long Term Support) which are actually their strong points. Each LTS release is probably the most stable Ubuntu releases there is, as it seems the in-between versions they experiment and play with new techniques, software, etc. Ubuntu is good both as a server and workstation.

Ubuntu's main problem is in it's init system, upstart. upstart is barely even half-baked with how it's available before it even is mature enough, and has the tools to properly work with it all. Upstart is an event-based init system. So for example, if you have an init process that needs 'networking', it will be triggered to start when networking is up. This has benefits and downsides to it the way upstart is designed, but I won't go into that kind of detail here.

I will say this about Ubuntu, initial new releases usually tend to have a lot of problems, more-so especially with all the in-between LTS release versions than anything else, but once those issues are ironed out, the stability usually is remarkably strong in the long run. Ubuntu also has a good track with keeping up with kernel and nvidia propriatery drivers so that when you do update your kernel, the display drivers will continue to work as expected. This is the best distro for this case scenario for certain. If, however, you just need or use the opensource drivers then you will likely have no problems in this regard.

I presently use Ubuntu for most my personal desktops, servers, and work desktop and servers for various purposes.

# Mandriva

Needs reviewer. FIXME
