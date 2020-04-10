---
title: Troubleshooting performance of VDO and NFS
author: Andrew Ludwar
type: post
date: 2018-10-20T17:43:07+00:00
url: /blog/2018/10/20/troubleshooting-performance-of-vdo-and-nfs/
xyz_smap:
  - 1
categories:
  - cloud
  - enterprise
  - networking
  - open source
  - performance tuning
  - storage
tags:
  - hardware
  - I/O
  - linux kernel
  - open source
  - performance tuning
  - private cloud
  - software defined storage
  - SSD
  - storage
  - virtualization

---
In setting up a local virtualization environment a little while back, I thought I&#8217;d try the [recently GA&#8217;d VDO capabilities][1] in the RHEL 7.5 kernel. These include data compression and de-duplication natively in the linux kernel (through a kernel module). This was Red Hat&#8217;s efforts behind the Permabit acquisition. Considering a virtualization data store is a prime candidate for a de-duplication use-case, I was anxious to reclaim some of my storage budget 🙂 . I was also curious to see what the extra overhead was like (if any), and understand the general performance characteristics of VDO.

I found VDO quite easy to setup, [I followed this guide][2] basically verbatim. Given my NFS virtualization back-end stores mostly the same OS images, I was happy to see excellent deduplication stats on my VDO device:

<pre class="">[root@nfs vms]# vdostats --si
Device                    Size      Used Available Use% Space saving%
/dev/mapper/vdo0          2.5T    239.3G      2.3T   9%           60%</pre>

After a few months and several VMs spawned later, I noticed some slowness whenever I was doing high IO work like database updates and copying several gigs of data to disks all at once. (When my Satellite server downloads 50+ GB for a new content repo, and index&#8217;s it for example). My other VMs would notice a bit of a slowdown during this time. Given I hadn&#8217;t done much for tuning in this environment, it was probably time to look into it. I&#8217;ve also been debating upgrading my home lab to 10G networking and this seemed to line up with what I was seeing for storage performance. I thought I finally was being bottlenecked by the network, given I&#8217;ve got an SSD array in my NFS server, with a 4-port 1GB NIC in LACP. But before I went crazy buying 10G networking gear, I looked at tuning what I had.

The [official documentation][3] is fairly good at explaining the performance characteristics of VDO, and what you might want to tune. I also went into NFS server/client tuning as I hadn&#8217;t done much for this either. Given there&#8217;s a few things at play here (disk hardware performance, network performance, VDO optimization, NFS optimization) I quickly went down a few rabbit holes and realised I needed to do some basic benchmarking and baselining so I could understand which areas in this stack were actually performing well, and which ones were candidates for more tuning. In addition to the VDO tuning docs, here&#8217;s what I used for reference:

  * [How to increase the number of threads created by the NFS daemon in RHEL 4, 5 and 6?][4]
  * [How can I improve the performance of my RHEL NFS server?][5]
  * [Initial baseline data collection for NFS client streaming I/O performance][6]
  * [High I/O wait to NFS share on NAS][7] 
  * [RHEL network interface dropping packets][8]

Firstly, it&#8217;s important to troubleshoot things in isolation, and use a benchmarking method that&#8217;s complimentary to isolation as well. I used the iperf3 utility for network benchmarking and fio utility for disk benchmarking. With this I&#8217;d do a series of sequential read, sequential write, random read, and random read and write tests, and these would be done both on local disk filesystems and over the network filesystem. For reference, here&#8217;s the fio commands:

<pre class=""># Sequential read
# fio --name TEST --eta-newline=5s --filename=fio-tempfile.dat --rw=read --size=500m --io_size=10g --blocksize=1024k --ioengine=libaio --fsync=10000 --iodepth=32 --direct=1 --numjobs=1 --runtime=60 --group_reporting

# Sequential write
# fio --name TEST --eta-newline=5s --filename=fio-tempfile.dat --rw=write --size=500m --io_size=10g --blocksize=1024k --ioengine=libaio --fsync=10000 --iodepth=32 --direct=1 --numjobs=1 --runtime=60 --group_reporting

# Random read
# fio --name TEST --eta-newline=5s --filename=fio-tempfile.dat --rw=randread --size=500m --io_size=10g --blocksize=4k --ioengine=libaio --fsync=1 --iodepth=1 --direct=1 --numjobs=1 --runtime=60 --group_reporting

# Random read and write
# fio --name TEST --eta-newline=5s --filename=fio-tempfile.dat --rw=randrw --size=500m --io_size=10g --blocksize=4k --ioengine=libaio --fsync=1 --iodepth=1 --direct=1 --numjobs=1 --runtime=60 --group_reporting<code></code></pre>

After reading the above guides and doing some basic investigating and benchmarking, this is what I ended up tuning first:

Overall the networking looked alright. I saw some dropped packets, but I&#8217;ve been doing a fair amount of cable pulling, stop/starting hosts, and VPN up/down. The NICs on NFS server and clients weren&#8217;t using their full ring buffer, so I changed this. I don&#8217;t think this was much of a candidate for the dropped packets, but this tuning couldn&#8217;t hurt.

<pre class="">[root@nfs]# ethtool -g eno1
Ring parameters for eno1:
Pre-set maximums:
RX: 2047
RX Mini: 0
RX Jumbo: 0
TX: 511
Current hardware settings:
RX: 200
RX Mini: 0
RX Jumbo: 0
TX: 511

# ethtool -G eno1 rx 2047
# ethtool -G eno2 rx 2047
# ethtool -G eno3 rx 2047
# ethtool -G eno4 rx 2047

# ethtool -g eno1
Ring parameters for eno1:
Pre-set maximums:
RX: 2047
RX Mini: 0
RX Jumbo: 0
TX: 511
Current hardware settings:
RX: 2047
RX Mini: 0
RX Jumbo: 0
TX: 511


[root@curie]# ethtool -g enp1s0
Ring parameters for enp1s0:
Pre-set maximums:
RX: 511
RX Mini: 0
RX Jumbo: 0
TX: 511
Current hardware settings:
RX: 200
RX Mini: 0
RX Jumbo: 0
TX: 511

# ethtool -G enp2s0 rx 511
# ethtool -G enp1s0 rx 511

# ethtool -g enp1s0
Ring parameters for enp1s0:
Pre-set maximums:
RX: 511
RX Mini: 0
RX Jumbo: 0
TX: 511
Current hardware settings:
RX: 511
RX Mini: 0
RX Jumbo: 0
TX: 511</pre>

I also increased the default number of NFS threads on the NFS server. Considering I&#8217;ve got 15+ VMs, each VM looks to use 2-3 nfs threads depending on number of disks, tuning the default of 8 to 20 should help for concurrent disk activity:

<pre class=""># egrep COUNT /etc/sysconfig/nfs
RPCNFSDCOUNT=20</pre>

Similarly, the VDO device I created only had 1 thread allocated in several places, so I upped these as well and doubled the cache size:

<pre class="">vdo modify --all --vdoLogicalThreads=4 --vdoPhysicalThreads=4 --vdoBioThreads=6 --vdoCpuThreads=6 --vdoAckThreads=2 --blockMapCacheSize=256M</pre>

After these changes, I still wasn&#8217;t seeing any significant performance change. I was getting fairly abysmal speeds even on a local SSD filesystem on the NFS server, not even going over the network. I started to isolate this, and started to suspect a hardware/SSD tuning related issue. After updating my DL360p 420i storage controller firmware, making sure the RAID controller cache was disabled, [SSD smart path was on][9], it started to dawn on me. Previously, these six SSD drives had been used in a RAID5 configuration that saw a ton of heavy disk IO. I had dedicated this host to an OpenStack environment and had done several builds hammering these disks. RAID5 is not an optimal SSD configuration, parity calculation is expensive, and this would add a ton of disk IO and disk wear that wouldn&#8217;t be present in a RAID0 or RAID10 configuration. Essentially, I&#8217;ve got worn SSDs. I needed to do a [secure erase of these SSDs][10] to return their cells to as close to original factory condition as one could get. These disks are approx 3 years old and haven&#8217;t had this done yet.

After doing an enhanced secure erase, I saw my local disk storage speeds come back up about 5 fold. This was more in line with the newer SSDs in other servers. Doing disk tests over the network saw the same speed increase. So it looks like my problem was entirely hardware related :). As I&#8217;m now turning on all the VMs, I&#8217;m seeing a much quicker response when doing the high IO activities. There&#8217;s more than 16 NFS threads consistently in use now and I&#8217;m monitoring to see the change in VDO related performance. I need to research a utility to get accurate VDO stats, I think this likely will be with a PCP module. But at first glance, with not much concrete data to go on yet, I \*think\* the VDO tuning has helped as well.

While I learned a bit about VDO and NFS performance tuning, it looks like I might need to spend that 10G networking budget on new SSDs instead. There&#8217;s diminishing life left in these.

 [1]: https://www.redhat.com/en/blog/look-vdo-new-linux-compression-layer
 [2]: https://rhelblog.redhat.com/2018/04/17/how-to-set-up-a-rhel-nfs-server-with-vdo-data-reduction/
 [3]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/vdo-ig-tuning-vdo
 [4]: https://access.redhat.com/solutions/2216
 [5]: https://access.redhat.com/articles/333973
 [6]: https://access.redhat.com/articles/1172303
 [7]: https://access.redhat.com/solutions/415263
 [8]: https://access.redhat.com/solutions/21301
 [9]: https://kallesplayground.wordpress.com/useful-stuff/hp-smart-array-cli-commands-under-esxi/
 [10]: https://grok.lsu.edu/Article.aspx?articleid=16716