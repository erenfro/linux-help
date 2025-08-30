---
{"publish":true,"title":"Donations","created":"2025-08-29","modified":"2025-08-30T00:06:55.334-04:00","published":"2025-08-29","tags":["donate"],"cssclasses":""}
---

Why Donate?
Well, first of all, I provide this site just because I want to, and I thank anyone whom wants to pitch in and help out any way they wish to, so for those that can't help out by giving assistance to technical docs this site is all about I will gladly accept donations to help account for the out of pocket expense I have by running this site.

How are Donations used?
Donations for this site go towards costs to host this site and everything it entails, as well as to Open Source Project developers that spend their private time developing useful and good quality software. If you have any specific requests for OSS projects, please feel free to put it in the donation comments.

What Donations cover?
I'll tell you now, what empowers this site, so you can understand the cost it is to run:

We are now hosted at Linode, where our site now runs faster and better than before, and reduces hosting costs, Linode customer service is absolutely amazing, and the console level access you get when you need it is just outright perfect. This site is not simply just web, but also email and database servers to keep things very solid. This site runs off several Linode instances at $71/mo.

[![Linode](Linode_updated_logo.png)](https://www.linode.com/lp/refer/?r=ad01cbad178ccd0a3112024fcbb3d7f0d29c2246)

Even if you don't wish to donate to us, if you do not currently have an account and are looking for hosting solutions, try out DigitalOcean.com following the image link below to give us credit for it. That can count towards a cool donation too since we get $25 for every referral once they get a bill pay $25 or more we get $25 back! We do run a few things on DO just to keep total seperation from one service provider, and for personal reasons.

[![Digital Ocean](digitalocean-horizontal-eps.png)](http://www.digitalocean.com/?refcode=fc9680d34c9f)

Also, as a secondary option if you're interested, Vultr, another VPS provider, provides rather reliable and slightly more flexible options. We use Vultr as well for similar reasons as above, but more for our BBS systems running Decker's Heaven BBS. So check these people out as well, if you're interested. They are reasonably fast and reliable, in their own ways. Not as overall fast as DigitalOcean or AWS, but they give you low-cost flexible options from SSD for high speed, or Magnetic disks for high capacity storage.

[![Vultr](vultr-logo1.png)](http://www.vultr.com/?ref=6851553)


The Origin
This is where Linux-Help.org started at with a much higher cost base, running out of my own personal home with 7 physical machines. 3 of the servers are storage servers, 4 of them are hypervisors running Linux KVM (Kernel Virtual Machine).

The hypervisors ran even more servers within them! I ran two firewalls, one of which was active, the other was a fail-over to takeover when the primary fails for whatever reason. I also ran two network directors which directs incoming traffic to the website between two different web servers, which again is setup in an active/fail-over setup so if one goes down, the other took over where it left off. Along with that, I ran two web servers that runs the Apache and PHP servers that were running this site. Connecting that together are two PostgreSQL servers handling the back-end of all the data involved. For search indexes there were two Solr servers running. In total, that's actually 10 virtualized servers that was running the show here. The hypervisors, however, are what actually cost because of electricity, cooling, and of course there's the internet which I pay for quality business class service for my home run operations here. This basically ran me about $175~$250/mo in electrical costs to run and maintain due to the amount of power needed.

The three storage servers actually housed the website data using a replicated clustered & distributed file engine known as Ceph. It provided high availability block data for the VM's using RBD, and provided high available shared file access through CephFS's filesystem mount. Internally within the Ceph network was replicated data for both providing truly high available storage at extremely high speed. This was much better than the former method which was just 1 server providing NFSv4 for qcow2 based disk images for the virtual machines, and NFSv4 on another node to provide share filesystem access to the web server content, that was 2 single points of failure.

For further speed, images, css, and javascript files were being stored in a Cloud Distribution Network (CDN) thanks to Amazon CloudFront Services. This allowed the images and content to be provided in edge networks that provide the content closer to where you live now, and on high speed networks. This service does cost some money, but it's pennies each month for what little storage usage and bandwidth usage there is.

For power redundancy, I had 2 APC Back-UPS XS 1000 which provide battery backup power for 2 hypervisors each. The storage servers, 1 APC Back-UPS XS 1300 provided battery backup for 1 storage server, and 1 APC Back-UPS XS 1500 provided the other 2 storage server's battery backup.

The Network was setup using two HP ProCurve 1810-24G web-managed switches for the VLAN and LACP setup. The 3 storage servers were trunked into LACP load balanced networks attached by Intel Pro 1000 Pt 2 port NIC's for speed and redundancy.

So you see, 7 physical servers in all, 10 virtualized servers beneath that, 3 APC backups, and a whole lot going on, just for the sake of doing it. How about that?

Closing Statement
Anyway, I gladly accept anyone that wants to join this little community and provide technical material to help others by, and also donations to help cover my out of pocket personal expenses. So please, feel free to do one or the other, or even both if you're feeling generous.

We do also offer professional services for contracted paid training sessions and infrastructural architecture for specialized needs. Doing so via Google Hangouts, Skype, or Phone and some form of screen sharing so you can see and even record what's going on as you need, so you can go over and review. This service is provided for anyone that wishes to learn advanced techniques on server architecture for high available, and scaled environments. If you would like to learn hands-on, please feel free to contact us on the Professional Services section of this website, and we can setup an initial review conference to spec out the initial goals, setup a course schedule and rates.

<form action="https://www.paypal.com/cgi-bin/webscr" method="post" target="_top">
<input type="hidden" name="cmd" value="_s-xclick">
<input type="hidden" name="hosted_button_id" value="8FXCRDPJ75L3E">
<input type="image" src="https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif" border="0" name="submit" alt="PayPal - The safer, easier way to pay online!">
<img alt="" border="0" src="https://www.paypalobjects.com/en_US/i/scr/pixel.gif" width="1" height="1">
</form>

Psi-Jack
Linux Systems Engineer and Founder of this Techni-Wiki