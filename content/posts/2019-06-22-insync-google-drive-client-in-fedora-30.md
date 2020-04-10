---
title: InSync Google Drive client in Fedora 30
author: Andrew Ludwar
type: post
date: 2019-06-22T16:09:45+00:00
url: /blog/2019/06/22/insync-google-drive-client-in-fedora-30/
xyz_smap:
  - 1
categories:
  - open source
  - storage
tags:
  - cloud
  - fedora
  - storage

---
After dropbox [ended its support][1] for linux clients, I went looking for a different cloud storage solution to sync data between my linux systems. I ended up purchasing 100GB with Google Drive (now Google One) and also purchasing a linux client that would connect to Google Drive and sync data between the cloud and my linux systems. I ended up with [InSync][2], and am really enjoying this new setup.

After upgrading all my systems to Fedora 30 (1 workstation, 2 laptops), I had trouble starting the InSync client on my Fedora 30 workstation. I reached out to InSync support, and in a few days had a resource reply with a fix. I wanted to document the fix here in case this helps others with a similar issue.

For starters, the InSync team is still populating a Fedora 30 repo but you can use the Fedora 29 repo to install/update the client. Just change the $releasever variable in the repo file to be hardcoded for the 29 release.

So this:

<pre class="lang:sh decode:true ">$ cat /etc/yum.repos.d/insync.repo
[insync]
name=insync repo
baseurl=http://yum.insynchq.com/fedora/$releasever/
gpgcheck=1
gpgkey=https://d2t3ff60b2tol4.cloudfront.net/repomd.xml.key
enabled=1
metadata_expire=120m
</pre>

Becomes this:

<pre class="lang:sh decode:true ">$ cat /etc/yum.repos.d/insync.repo 
[insync]
name=insync repo
baseurl=http://yum.insynchq.com/fedora/29/
gpgcheck=1
gpgkey=https://d2t3ff60b2tol4.cloudfront.net/repomd.xml.key
enabled=1
metadata_expire=120m
</pre>

My error was the InSync client was not starting up at all, it was core dumping even with a &#8211;no-daemon flag passed. I&#8217;m running my desktop session in Wayland as opposed to X11:

<pre class="lang:sh decode:true">$ insync start --no-daemon
qt.qpa.plugin: Could not load the Qt platform plugin "wayland" in "" even though it was found.
libGL error: MESA-LOADER: failed to open radeonsi (search paths /usr/lib64/dri)
libGL error: failed to load driver: radeonsi
libGL error: MESA-LOADER: failed to open swrast (search paths /usr/lib64/dri)
libGL error: failed to load driver: swrast
Segmentation fault (core dumped)</pre>

So the fix for me, as suggested by InSync support was to remove this library file. I renamed it to accomplish the same thing:

<pre class="lang:sh decode:true ">$ sudo mv /usr/lib/insync/libstdc++.so.6 /usr/lib/insync/libstdc++.so.6.rm.bak</pre>

&nbsp;

After that, the InSync client was able to start.

&nbsp;

 [1]: https://linux.slashdot.org/story/18/08/10/2120248/dropbox-is-dropping-support-for-all-linux-file-systems-except-unencrypted-ext4
 [2]: https://www.insynchq.com/