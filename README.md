# How to Mount an Anicent Iomega Zip100 Drive on Debian

This is in case anyone else runs into this crazyiness.  It's cobbled together from a variety of sources and my own experience.  This has been tested to work on:

#### Internal Iomega Zip 100 IDE interface

It might also work for Zip 250, Jaz drives, I don't know. 

Situation: My company, Punkadyne Labs, needed to mount and check for some old material stored as ancient backups on Zip 100 disks.  The only Zip 100 drive we could get at a ham radio swap meet was sold by an old Egyptian man who mysteriously vanished when we looked back after the sale. After applying some anti-curse charms, we had to find a PC that still had an IDE channel and a ribbon cable.  Luckily, we had a spare one being used to prop up a sagging server rack of old VT 100 terminal servers. 

    user@localtoast-:~$ sudo dmidecode | grep -A 2 -i "base board"
    Base Board Information
      Manufacturer: Intel Corporation              
      Product Name: D850GB  

It still had a version of Debian 6 on it, which we had to upgrade to Debian 9 for reasons not specific to this document. 

    user@localtoast-:~$ lsb_release -a
    No LSB modules are available.
    Distributor ID:	Debian
    Description:	Debian GNU/Linux 9.12 (stretch)
    Release:	9.12
    Codename:	stretch
    
It had 512mb RAM still not stolen out of it, 20gb hard drive, and a 1.8gz Pentium 4 chip.  We set to work.  First we wanted to make sure that the BIOS saw the Zip drive, and it did. Then we booted her up, and looked for it in dmesg:

    user@localtoast-:~$ sudo dmesg | grep -i iomega
    [    2.048336] ata1.01: ATAPI: IOMEGA  ZIP 100       ATAPI, 13.A, max PIO2, CDB intr
    [    2.101347] scsi 0:0:1:0: Direct-Access     IOMEGA   ZIP 100          13.A PQ: 0 ANSI: 5


Thankfully, someone had this problem before, probably when debian was still on 20 floppies. They created a set of libraries and a tool called **jazip**.

    user@localtoast-:~$ sudo apt search iomega
    Sorting... Done
    Full Text Search... Done
    
    jazip/oldstable 0.34-15.1+b2 i386
      Mount and unmount Iomega Zip and/or Jaz drives

    user@localtoast-:~$ sudo apt install jazip
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following additional packages will be installed:
      libforms2
    The following NEW packages will be installed:
      jazip libforms2
    0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
    Need to get 456 kB of archives.
    After this operation, 1,312 kB of additional disk space will be used.
    Do you want to continue? [Y/n] 
    Get:1 http://deb.debian.org/debian stretch/main i386 libforms2 i386 1.2.3-1.3+b1 [376 kB]
    Get:2 http://deb.debian.org/debian stretch/main i386 jazip i386 0.34-15.1+b2 [79.7 kB]
    Fetched 456 kB in 0s (503 kB/s)
    Preconfiguring packages ...
    Selecting previously unselected package libforms2.
    (Reading database ... 61839 files and directories currently installed.)
    Preparing to unpack .../libforms2_1.2.3-1.3+b1_i386.deb ...
    Unpacking libforms2 (1.2.3-1.3+b1) ...
    Selecting previously unselected package jazip.
    Preparing to unpack .../jazip_0.34-15.1+b2_i386.deb ...
    Unpacking jazip (0.34-15.1+b2) ...
    Setting up libforms2 (1.2.3-1.3+b1) ...
    Processing triggers for menu (2.1.47+b1) ...
    Processing triggers for libc-bin (2.24-11+deb9u4) ...
    Processing triggers for man-db (2.7.6.1-2) ...
    Setting up jazip (0.34-15.1+b2) ...
    Processing triggers for menu (2.1.47+b1) ...
    
    user@localtoast-:~$ sudo jazip
    In fl_initialize() [flresource.c:995]: jazip: Can't open display 
    In fl_bgn_form() [forms.c:347]: Missing or failed call of fl_initialize()

The first error was it couldn't open our diosplay, but that's because we were connected via ssh (the VGA display was nearly unreadable).  So we reconnected via ssh -X and got this error:

    Can't read configuration file /etc/jazip.conf.

Luckily, [there are manpages](https://www.google.com "there are manpages"), and they tell us we need to make a conf file:

    user@localtoast-:~$ sudo jazipconfig
    There are currently no entries in /etc/jazip.conf.

    There are no Zip devices detected on the system.

    There are no Jaz devices detected on the system.

    Available commands:
     (c)reate an entry from scratch.
     (q)uit without saving.
     (e)xit and save changes.
                               ? c

    Note: Do not specify partition number in device.
    What device do you want to add? (e.g. /dev/sda) /dev/sdb

    What mount point? (e.g. /zip) /zip
    --------------------------------------------
    These are the entries currently selected for /etc/jazip.conf:

      1:   Device /dev/sdb   Mount point /zip

    There are no Zip devices detected on the system.

    There are no Jaz devices detected on the system.

    Available commands:
     (d)elete an entry from /etc/jazip.conf
     (c)reate an entry from scratch.
     (q)uit without saving.
     (e)xit and save changes.
                               ? e

    Mount point /zip does not exist. Create it? (y/n) [y]: y
    Creating /etc/jazip.conf

Wait, what? It can't detect any drives?  Was that old Egyptian man lying?  Did he sell us a dud?

    user@localtoast-:~$ df -h
    Filesystem      Size  Used Avail Use% Mounted on
    udev            240M     0  240M   0% /dev
    tmpfs            50M  1.9M   48M   4% /run
    /dev/sda2       7.7G  2.5G  4.8G  35% /
    tmpfs           249M     0  249M   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           249M     0  249M   0% /sys/fs/cgroup
    tmpfs            50M     0   50M   0% /run/user/1000
    
It's not mounted... is it even listed as a block device?

    user@localtoast-:~$ lsblk
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    sda      8:0    0    19G  0 disk 
    ├─sda1   8:1    0  10.8G  0 part 
    ├─sda2   8:2    0   7.9G  0 part /
    ├─sda3   8:3    0     1K  0 part 
    └─sda5   8:5    0 384.3M  0 part [SWAP]
    sdb      8:16   1    96M  0 disk 
    sr0     11:0    1  1024M  0 rom  
    sr1     11:1    1  1024M  0 rom  
    
It is!  Still /dev/sdb.  And now it shows us 96MB of space, which is fine for an empty disk we have mounted. What if we manually mount it?

    user@localtoast-:~$ sudo mount /dev/sdb /zip
    
    puser@localtoast-:~$ df -h
    Filesystem      Size  Used Avail Use% Mounted on
    udev            240M     0  240M   0% /dev
    tmpfs            50M  3.1M   47M   7% /run
    /dev/sda2       7.7G  2.5G  4.8G  35% /
    tmpfs           249M     0  249M   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           249M     0  249M   0% /sys/fs/cgroup
    tmpfs            50M     0   50M   0% /run/user/1000
    /dev/sdb         94M     0   94M   0% /zip

There we go!  User is part of the group **floppy** which the instructions tell us. We launch the jazip and... get an ancient X-windows tool that looks like it was created with Mac OS in the mid 1990s.  I mean, look at this thing:

Reference-style: 
![This is the kind of thing you'd expect a weird old man to sell you][logo]

[logo]: https://raw.githubusercontent.com/GrigLars/IomegaJazZipDrive/d419ff21ac183f9f17fc2c598c52f12d6710dad4/jazip_screenshot.png "This is the kind of thing you'd expect a weird old man to sell you"

