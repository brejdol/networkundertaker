I have been using Linux on my desktops/laptops since 2006-ish. When I started doing consulting work, I settled for a Debian-based distro + VmWare Workstation. That just clicked with my workflow - Some work was done in the base os, some in a vm (usually a windows 10, a FreeBSD, and/or a Kali Linux).

With that setup, I could have one VPN running in the base os, and another in a vm without interference. Company secure fileshare (Nextcloud w. 2FA) was mounted in the base os, and a shared folder was presented as a network drive in the windows vm. Both vm and base os was backed up to a local fileshare in my NAS w. Veeam. The fileshare was backed off-site to BackBlaze/OneDrive every week.

This setup wasn't obviously flawed - I could do what I wanted, I had a stable system, and changing computer was fairly easy - I just installed OS and VmWare Workstation, and copied/restored all the files/vm, and off I went.

However, I always ended up with a lot of work related stuff in my "private" download folder - Config backups, scripts, ISOs, you name it. Most of it should  either be discarded after use, or put into the corresponding customer folder in the company Nextcloud file share.

And I failed this manual step, over and over. Over time, this heap of unsorted stuff grew and grew until I had to go through thousands of files, many of wich I had forgotten completely what they were, so I had to look into them and try to cross reference them back to where they did belong. Not good, nor secure.
