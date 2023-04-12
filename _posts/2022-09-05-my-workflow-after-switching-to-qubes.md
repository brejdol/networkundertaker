**Background - Workflow**

I have been using Linux on my desktops/laptops since 2006-ish. When I started doing consulting work, I settled for a Debian-based distro + VmWare Workstation. That just clicked with my workflow - Some work was done in the base os, some in a vm (usually a windows 10, a Free-BSD, and/or a Kali Linux).

With that setup, I could have one VPN running in the base os, and another in a vm without interference. Company secure fileshare (Nextcloud w. 2FA) was mounted in the base os, and a shared folder was presented as a network drive in the windows vm. Both vm and base os was backed up to a local fileshare in my NAS w. Veeam. The fileshare was backed off-site to BackBlaze/OneDrive every week.

This setup wasn't obviously flawed - I could do what I wanted, I had a stable system, and changing computer was fairly easy - I just installed OS and VmWare Workstation, and copied/restored all the files/vm, and off I went.

However, I always ended up with a lot of work-related stuff in my "private" download folder - Config backups, scripts, ISOs, you name it. Most of it should  either be discarded after use, or put into the corresponding customer folder in the company Nextcloud file share.

And I failed this manual step, over and over. Over time, this heap of unsorted stuff grew and grew until I had to go through thousands of files, many of which I had forgotten completely what they were, so I had to look into them and try to cross reference them back to where they did belong. Not good, nor secure.

![Hit computer](/hit-computer.jpg)

If I wanted to use something temporarily, that still meant installing a vm. VmWare Workstation in conjunction with Ubuntu also started to fail me - It went from being very solid to more shaky - I had crashes, and the vm got corrupted on more then one occasion. Not fun when you are in the middle of a really busy work day! Ubuntu also started to feel more and more bloated, and it was somewhat hard to get to terms with all design choices made within the OS itself. Plain Debian was much better, but I still wanted to tighten my workflow.

**In walked Qubes**

**[Qubes](https://qubes-os.org)** turns your pc/laptop into a Xen-based virtualization host. Inside of it, you will run qubes, where each qube is a compartmentalized mini-vm derived from a template based on Debian or Fedora, or a full blown vm, created to your specs and needs.

![Qubes OS](/qubes.png)

The qubes comes in different types:

> AppVM

Has a persistent home, but volatile root. So everything except from your /home will be lost at shutdown/reboot. Based on a template VM, so if you want to install programs, you boot your template and install the apps, then shut it down. To get the new app in the qubes based on that template, just shut them down and start them again. And here is the good part - You can run one application from a Qube in a window, on your desktop. Windows from your different Qubes are color-coded, so that you can see which browser window belongs to your work Qube, and which belongs to your personal Qube.

> TemplateVM

Is the other way around - Volatile home, persistent root. Here you can install the specific apps you want to use. You can create as many templates as you need. So you might want a "desktop-template", witch word processing, different browsers, slack and every day apps etc. Then you'll have specific needs for your work side Qubes - Maybe tools for coding - Visual Studio, git etc. Who knows, maybe a "media-template", with video players, Spotify app etc.

> StandaloneVM

A "standard" full virtual machine. In my case, this is the Windows 10 vm with Office, Visio, and specific VPN-clients.

> DisposableVM

Everything is volatile. When you shut a qube based on a DisposableVM, it is overwritten with the template. What happen in Vegas stays in Vegas!

**Installation**

Qubes can be picky with the hardware, but I always go with the simplest possible hardware setup on my laptops - All Intel CPU/GFX/Nic, no added GFX. You will also need plenty of RAM if you want to utilize the full potential of Qubes, or any virtualization really. I have a Dell Inspiron 5420 with a Core i7, 32Gb of RAM and a 1Tb HD. Not super fancy, not crap. Installation is pretty straight forward, disk will be fully encrypted (obviously), and you get to chose which OS you want in your templates (Fedora36 is default, Debian11 is optional), but you can change that later. You will have a few templates from start, based on Debian11 and Fedora36 (In latest Qubes 4.1). You will also have a few whonix qubes, with a preinstalled setup using Tor. You can throw away what you don't want to use, and/or create new qubes, all to your liking.

**Internal Networking Setup**

From start, you will not have any networking - This is by default. You will have to map your hardware (NICs), ethernet and WiFi to your sys-net Qube. This is your default gateway to the outside world. You just have to right-click the sys-net in Qube Manager, shut it down, then choose "Settings" -> "Devices", and add your ethernet and wifi controller, then start the sys-net qube. I also added and imported my PKI chain into the sys-net Qube, since I use 802.1x in my home network. It works just fine, and connects to the outside. The sys-net Qube does SNAT out via its interface IPs.
You will also have a sys-firewall. It should have the sys-net Qube set as "NetVM", since it use sys-net to get out of the machine. The sys-firewall Qube handles all connections from the individual Qubes. It also does SNAT out towards the sys-net Qube. 

What about IPs? It is all handled auto-magical within QubeOS. You'll get an IP from a per-defined (and pretty hard-coded) range depending on which type of Qube it is. 10.137.x.x is used for AppVMs, templates and StandaloneVMs, 10.138.x.x is used for volatile Qubes. The internal networking is all L3, so no Qube can reach any other Qube - The only way out is via the sys-firewall -> sys-net -> out of the machine. Default firewall ruleset is to allow everything out from all Qubes, but that is configurable. You can also remove the networking completely for a completely isolated Qube etc. 

**IPv6**

IPv6 is still pretty experimental, but you can enable it via the cli in dom0 (the base-os, or controller). It however uses ULA-addressing (hard-coded deep within their Python code base), so if you want to make any use of IPv6, you would have to change the priority for fc00::/7 in /etc/gai.conf in all templates you use. NAT66 is used in sys-firewall and sys-net, to make sure connectivity works regardless of your surrounding network environment. Except for the obviously wrong choice of using ULA, this works well - After changing the prio of fc00::/7 so that it is preferred over IPv4, IPv6 works as is should, completely transparent for the user.

**USB**

You will also find a volatile Qube named sys-usb. This is handling all usb-ports by default. You can tell it to always accept USB-keyboards/mouse if you dare (I dared), then everything just works when you boot up your PC. If you connect another USB item, for example a USB-HD, you can choose which Qube it should be attached to. This is awesome security-wise, since nothing in this Qube is persistent.

**My Setup**

I ended up with one Qube for work (with a Nextcloud client, connecting to our company fileshare), one personal, one for blogging, one disposable for "dangerous" stuff, and a Windows10 vm. Windows btw works fine. If you install the qube-tools for Windows, you get copy-paste, sound and other integrations. My earlier VmWare Workstation on Ubuntu was crippled by the kcompactd0 bug, where you will get CPU going full on all cores trying to compact the swap file. Windows on Xen/Qubes is a pretty damn pleasant experience in comparison - It is swift and isn't hogging too much of your system resources. If you use more then one screen, you can create a setup profile so that you always get the same look. You can also use workspaces to move windows to.
What I now have is a setup where it is impossible for me to mix the work-side with the private side of things.

**Backup**

Qubes has a built-in backup utility, based on lvm snapshots. It is a full Qube/vm backup, and it works fine, but it has no incremental logic at all, so every backup is a FULL backup. I backup every week to a locally attached 2Tb USB SSD-drive. I then have a dedicated Backup-qube where I run DejaDup to my NAS. This means I at least get dedup/incremental backups in the second hop in the backup. The fileshare is then backed to OneDrive/BackBlaze every week (w. dedup/incremental/encryption). My 2Tb SSD-drive holds around 20 full backups of my system, that is well and enough.

In each Qube where I want to preserve anything more granular, I have setup DejaDup to backup to my NAS every day (Work, Personal, and blog Qubes are backed, Windows is backed with Veeam client to the same fileshare). Those backups are also synced to OneDrive/BackBlaze every week. What I want is to be able to both restore a file in my home folder, and restore the full Qube. I have tested both, and it just works.

**Security**

Qubes is very secure in comparison to any other desktop operating system. All Qubes are isolated from each other, and the stuff they share is volatile, and will not survive a reboot. It is possible to copy files between Qubes - You can right-click and choose to copy to another AppVM, and files end up in a folder in the destination Qube that is one-way. No drag-and-drop (for obvious reasons). You can tune and tweak the firewall on your individual Qubes to your liking. You can also setup internal routing - I, for example, have a "VPN-Qube", where I have gathered a few VPN connections. Instead of messing with my personal qube, I can enable the tunnel in this separate Qube, then reach the remote networks via inter-Qube routing. Me like!

**Conclusion**

I guess Qubes can be seen as pretty hard-core. It demands quite a bit of Linux and virtualization knowledge, and may not be for the faint-hearted. The instant reward for me using this setup is that I get a very secure base operating system that is highly configurable and adaptable. It instantly helped me to achieve what I wanted, where my old setup was missing the goal by a mile. It has been very stable, updates are frequent, and its user base is growing (which for an Operating System is a good thing...). On a regular day, I have Hamsket running in my work Qube (blue window border), configured with my work collaborations (Slack, teams, mails), work consoles and browsers. Then another Hamsket running in my personal Qube (Yellow window border), configured with my personal collaborations (Slack, mail, discord etc). A Windows10 vm running on the other screen running the stuff I need there. And maybe the blogging-Qube running too (green window borders), with Visual Studio and git (which is what this is written on).

![Hug computer](/hug-computer.jpg)

I went straight from my old setup to this on my primary work computer, and I haven't looked back once.
Well, maybe once, when I found out that I forgot to copy my .ssh folder with all the keys... :-)
But that is, as they say, another story...  

[Start page]({{ '/' | absolute_url }})

