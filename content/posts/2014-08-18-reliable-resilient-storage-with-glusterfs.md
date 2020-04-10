---
title: Reliable, resilient storage with GlusterFS
author: Andrew Ludwar
type: post
date: 2014-08-19T05:42:26+00:00
url: /blog/2014/08/18/reliable-resilient-storage-with-glusterfs/
dsq_thread_id:
  - 5075804661
categories:
  - devops
  - enterprise
  - open source
  - storage
tags:
  - devops
  - gluster
  - software defined storage

---
A need came up lately for some inexpensive resilient storage that was easily expandable, and that spanned multiple datacentres.  Having recently been playing with GlusterFS and Swift in our OpenStack trial, I was quick to point out that these were strong use-cases for a GlusterFS architecture. We needed something right away, and something that also wasn&#8217;t terribly expensive, which Gluster caters to quite well.  Typically we would purchase a SAN technology for this, but recently having some discussions on cost and business agility, we decided against that and opted to try a popular open source alternative that&#8217;s been gaining both momentum and adoption at the enterprise level.

I began researching the latest best-practise architecture for GlusterFS.  It&#8217;s been about a year since I&#8217;ve given it a deep investigation, so I started re-reading their documentation.  I was pleasantly surprised to see that quite a bit more work had been done on the documentation since I last checked-in with the project.  They&#8217;ve recently moved a lot of documentation into [github][1], and redesigned their [website][2] as well.  I found some good information on their website [here][3], but found the most useful information in github [here][4].

Our storage needs for this project were very simple.  We need resilient, highly available storage that can expand quickly, but don&#8217;t need a ton of IOPs.  Databases and other intense I/O applications are out of scope of this project, so what we&#8217;re mostly looking for out of this architecture is basic file storage for things like:

  * ISOs
  * Config file backups
  * Restored file location
  * Static web files
  * FTP/Dropbox/OwnCloud file storage

After perusing through [Gluster.org][2]&#8216;s documentation, I came across the architecture that would best suit our needs.  [Distributed replicated volumes][5].  We wanted two servers in each datacentre to have a local copy of this data, and also have two copies stored in another datacentre for resiliency.  This way we can suffer a server loss, a server loss in each datacentre, or an entire datacentre loss and still have our data available.  The documentation even gives you the command syntax to create this architecture; perfect!  I wanted to change the architecture slightly, but that wasn&#8217;t a big deal.  There&#8217;s enough detail in the documentation that I was able to understand how to do this.

  1. I racked 2 CentOS 6.5 servers in each datacentre (for a total of 4. I had four Dell R420s that were decomissioned from another project &#8211; commodity hardware)
  2. Installed gluster-server on them all via the [glusterfs-repo][6]
  3. Made sure their DNS was resolvable for both forward and reverse
  4. Chopped up their RAID10 1TB hard disk with LVM leaving approx 80GB for the OS, and the rest for /gfs/brick1

&#8230;and I was ready to run their provided command &#8211; slightly modified of course.  Since I wanted the data to essentially be replicated across all 4 nodes, I changed the replica count from 2 to 4.  This means if I ever need to expand, I&#8217;ll need to do it 4 nodes at a time.  I then ran this command on one of the cluster members to create the gluster volume named &#8216;gfs&#8217;:

<pre>gluster volume create gfs replica 4 transport tcp server1:/gfs/brick1 server2:/gfs/brick1 server3:/gfs/brick1 server4:/gfs/brick1</pre>

For a visual reference, this is what this architecture looks like:<figure id="attachment_117" aria-describedby="caption-attachment-117" style="width: 720px" class="wp-caption aligncenter">

[<img class="wp-image-117 size-large" src="http://calgaryrhce.ca/wp-content/uploads/2014/08/GlusterFS-1024x685.png" alt="GlusterFS - Distributed Replicated Volume" width="720" height="481" srcset="https://calgaryrhce.ca/wp-content/uploads/2014/08/GlusterFS-1024x685.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2014/08/GlusterFS-300x200.png 300w, https://calgaryrhce.ca/wp-content/uploads/2014/08/GlusterFS-272x182.png 272w, https://calgaryrhce.ca/wp-content/uploads/2014/08/GlusterFS.png 1153w" sizes="(max-width: 720px) 100vw, 720px" />][7]<figcaption id="caption-attachment-117" class="wp-caption-text">GlusterFS &#8211; Distributed Replicated Volume</figcaption></figure> 

That&#8217;s it! I could now install the glusterfs and glusterfs-fuse client software on any node that needed to access this volume, and mount it with

<pre>mount -t glusterfs server1:/gfs /mnt</pre>

After succesfully mounting the volume, I did a few file writing tests with bash for loops, and dd, just to test the speed and see that any created files were replicating.  I then did a few failover tests by shutting down an interface on a node, rebooting a node completely, and shutting down two nodes in a datacentre.  Since I am using the native glusterfs client to connect to the volume, the failover and self-healing features are automatically handled unbeknownst to the user.  After the 42 second timeout, services failed over nicely, and files were replicated across active hosts.  When the downed nodes came back up and joined the volume again, the files they missed were automatically copied over.  Perfect!  And just like that &#8211; I was done.

I was surprised at how simple this was to architect and setup.  When I need additional disk space, I&#8217;ll rack another 4 nodes into the architecture, and expand it by using the gluster volume add-brick command documented [here][8].

In another article, I plan to do some tweaking to the 42 second TCP timeout &#8211; Gluster provides documentation on all the [options and tuneables][9] you can set.  It&#8217;s also worth looking at using an IP-based load balancing technology.  (Either an F5, or CTDB).  This should increase the service availability even further.  In yet another article, I would also like to explore the new [geo-replication][10] technology that spans WANs and the internet.  For now, I have a 1TB highly available and resilient file storage volume to play with.

&nbsp;

 [1]: http://github.com
 [2]: http://gluster.org
 [3]: http://gluster.org/documentation/
 [4]: https://github.com/gluster/glusterfs/tree/master/doc/admin-guide/en-US/markdown
 [5]: https://github.com/gluster/glusterfs/blob/master/doc/admin-guide/en-US/markdown/admin_setting_volumes.md#creating-distributed-replicated-volumes
 [6]: http://download.gluster.org/pub/gluster/glusterfs/LATEST/CentOS/glusterfs-epel.repo
 [7]: http://calgaryrhce.ca/wp-content/uploads/2014/08/GlusterFS.png
 [8]: https://access.redhat.com/documentation/en-US/Red_Hat_Storage/2.1/html/Administration_Guide/sect-User_Guide-Managing_Volumes-Expanding.html
 [9]: https://github.com/gluster/glusterfs/blob/master/doc/admin-guide/en-US/markdown/admin_managing_volumes.md
 [10]: https://github.com/gluster/glusterfs/blob/master/doc/admin-guide/en-US/markdown/admin_geo-replication.md