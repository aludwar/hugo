---
title: Performance Tweaking – SSDs, I/O Scheduler, Virtual Memory, Browser Cache
author: Andrew Ludwar
type: post
date: 2014-07-13T22:15:31+00:00
url: /blog/2014/07/13/performance-tweaking-ssds-io-scheduler-virtual-memory-browser-cache/
dsq_thread_id:
  - 5107877395
categories:
  - open source
  - performance tuning
  - storage
tags:
  - I/O
  - performance tuning
  - SSD
  - storage

---
Having recently been exposed to some SSD tweaking at work, I thought I&#8217;d do the same with my home PCs.  Prior to this weekend, I&#8217;ve just had four 1TB SATA drives in a RAID 5 configuration for a disk setup.  Performance has always been satisfactory, but with recent SSD prices coming down quite a bit, my late 2008 MacBook Pro feeling it&#8217;s age, it was time for an upgrade.  Also, it&#8217;s been a while since I&#8217;ve invested in some new gear for myself, so why not now? ;).

For the laptop, a [256 GB Samsung 840 Pro SSD.][1]  For the tower, the [128 GB][2] version.

[<img class="alignnone size-medium wp-image-106" src="http://calgaryrhce.ca/wp-content/uploads/2014/07/SSDs-web-300x197.jpg" alt="SSDs" width="300" height="197" srcset="https://calgaryrhce.ca/wp-content/uploads/2014/07/SSDs-web-300x197.jpg 300w, https://calgaryrhce.ca/wp-content/uploads/2014/07/SSDs-web.jpg 400w" sizes="(max-width: 300px) 100vw, 300px" />][3]

These are higher grade SSDs, with quite a bit higher IOP rating than average.  Generally speaking, the larger the hard drive, the more IOPs you will get also.  The laptop would be using the entire drive for all data, while the tower would be using the SSD for just the OS.  My laptop runs OSX Mavericks, and the tower CentOS 6.5.  For now, we&#8217;ll just talk about the CentOS box.

### Tuning the filesystems &#8211; optimizing data structure for SSD use, eliminating journal access time writes, temp filesystems to RAM, enabling TRIM

I&#8217;ve split up the OS into five filesystems that will sit on the SSD. Root, var, usr, boot, and ssd. The /ssd filesystem is just set aside for anything that I might need quick disk for.

The advantage of SSDs is read-performance, and they do wear when used, so ideally you want the files that are frequently read on the SSDs, but reduce the writes and file changes as much as you can to save wear and increase the SSDs life span. For this reason I&#8217;ve used the &#8220;noatime&#8221; and &#8220;nodiratime&#8221; mount options on the SSD ext4 filesystems. Since ext4 is a journaling filesystem, linux will record information about when files were created, last modified, as well as when it was last accessed. Turning this feature off not only reduces the writes to the SSD, it also can improve performance on often accessed and frequently changing files. Since no important application logs sit on these filesystems, and /var/log/messages includes timestamps within the file, I can safely turn this off. My tower /etc/fstab looks like this:

<pre>UUID=ebc6b3e8-48e5-4d1d-b7c1-0cd5f8738bca /boot ext4 defaults 1 2
/dev/mapper/vg_os-lv_root / ext4 noatime,nodiratime 1 1
/dev/mapper/vg_os-lv_ssd /ssd ext4 noatime,nodiratime 1 2
/dev/mapper/vg_os-lv_usr /usr ext4 noatime,nodiratime 1 2
/dev/mapper/vg_os-lv_var /var ext4 noatime,nodiratime 1 2
/dev/mapper/vg_os-lv_swap swap swap defaults 0 0
/dev/mapper/vg_os-lv_export /export ext4 defaults 1 2
/dev/mapper/vg_os-lv_home /home ext4 defaults 1 2
tmpfs /tmp tmpfs defaults,noatime,nodiratime 0 0
tmpfs /var/tmp tmpfs defaults,noatime,nodiratime 0 0
</pre>

Since swap can be frequently accessed, I&#8217;ve opted to not put that on the SSD. Also, /export and /home contain important data that needs to be revivable if I ever lose a couple disks. SSDs are not good for archival use; once they wear out, you won&#8217;t be able to scrape the drive for any useable data. Once it goes, your data is gone with it.

I&#8217;ve also made some other tweaks on the filesystem, making /tmp and /var/tmp use RAM instead of disk for file storage. This will speed up temp file access by quite a bit. I have 32GB of RAM on the system, so I&#8217;m not crunched for memory.

I&#8217;ll also need to enable TRIM support. There&#8217;s a good article [here][4] that talks about enabling it properly. I&#8217;ve enabled TRIM for LVM, and have placed a simple script in the weekly cron to handle the discarding of blocks on the SSD.

### Change the kernel I/O scheduler

The I/O scheduler optimizes disk operations for speed. The default scheduler is CFQ (Completely Fair Queuing), which is optimized for the mechanical rotational latency of regular HDDs. Since that doesn&#8217;t apply to SSDs, consider using the deadline scheduler instead which prioritizes reads over writes. This change will help take advantage of the SSD performance. You can read more about the I/O scheduler options [here][5]. Further to that, RedHat-based systems also have the tuned-adm command that can set pre-defined performance profiles based on your hardware and optimization purpose. (See man tuned-adm for more.)

To enable this change scheduler change permanently, you&#8217;ll need to modify GRUB to pass the &#8220;elevator=deadline&#8221; parameter on the kernel boot line.

<pre>vi /etc/grub.conf
...
title CentOS (2.6.32-431.20.3.el6.x86_64)
root (hd0,0)
kernel /vmlinuz-2.6.32-431.20.3.el6.x86_64 ro root=/dev/mapper/vg_os-lv_root rd_NO_LUKS KEYBOARDTYPE=pc KEYTABLE=us rd_LVM_LV=vg_os/lv_swap LANG=en_US.UTF-8 rd_LVM_LV=vg_os/lv_root rd_MD_UUID=0b5dd8e0:26c65ad9:72ad5b48:a8447401 rd_MD_UUID=adfcdb87:7b4dcd6e:683df3bf:f6056163 rhgb crashkernel=auto quiet SYSFONT=latarcyrheb-sun16 rd_NO_DM <strong>elevator=deadline</strong>
initrd /initramfs-2.6.32-431.20.3.el6.x86_64.img
...
</pre>

### Tuning virtual memory &#8211; Reduce swapping

Although I&#8217;ve placed my swap partition on the HDDs, swap file tuning is still an important tuneable to look at. Swap space is hard drive space that is used when RAM is exhausted and the current proccesses need more memory. The system will also move data to the swap file from RAM if it hasn&#8217;t been accessed in a while. This preemptively free&#8217;s up RAM for other processes. This is incredibly slow compared to RAM, and really should only be used as a fall-back when your system is running out of RAM. If your system is frequently swapping, go buy more RAM and use the swap file as a temporary work around.

In the kernel, there&#8217;s a setting that controls the degree to which the system swaps. Since I have a lot of RAM to use, I only want the system to swap when it&#8217;s absolutely necessary. A high value in the swappiness setting aggressively swaps processes into physical memory when they&#8217;re not active. A low value avoids swapping processes for as long as possible. So I&#8217;ve set the vm.swappiness parameter to zero. You can read more about this tuning [here][6].

I&#8217;ve added this parameter to /etc/sysctl.conf, and made it active with sysctl -p. This will ensure this is persistent through a reboot.

<pre># Only swap if absolutely necessary
vm.swappiness = 0</pre>

### Moving Firefox cache to RAM

This is a nice tweak that tells firefox to use RAM instead of disk for it&#8217;s cache. This should speed up browsing frequently visited sites. To enable it, go into the about:config, set browser.cache.disk.enable to false, set browser.cache.memory.enable to true, and browser.cache.memory.capacity to the number of KB you want to assign. I&#8217;ve used -1 to let Firefox dynamically determine my cache size depending on how much RAM the system has.

<pre>about:config
browser.cache.disk.enable = false
browser.cache.memory.enable = true
browser.cache.memory.capacity = -1
</pre>

You can use the about:cache entry to see your newly changed cache information!

### Results

Now for the results. I&#8217;ve used [iozone][7] as my benchmarking tool because it performs a series of tests quite easily, and graphs the output nicely for me in excel. Attached are [the results][8] of the read report statistics. As you can see, considering OS cache amongst a few other things blur the true speed differences a little bit, the SSD has roughly triple the performance in read speed vs the four HDDs in RAID5.

 [1]: http://www.memoryexpress.com/Products/MX43872
 [2]: http://www.memoryexpress.com/Products/MX43630
 [3]: http://calgaryrhce.ca/wp-content/uploads/2014/07/SSDs-web.jpg
 [4]: http://blog.neutrino.es/2013/howto-properly-activate-trim-for-your-ssd-on-linux-fstrim-lvm-and-dmcrypt/
 [5]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/ch06s04s02.html
 [6]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-memory-tunables.html
 [7]: http://iozone.org/
 [8]: http://calgaryrhce.ca/wp-content/uploads/2014/07/SSDvsHDD.xlsx