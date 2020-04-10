---
title: Link Aggregation w/ LACP
author: Andrew Ludwar
type: post
date: 2014-09-15T19:14:00+00:00
url: /blog/2014/09/15/link-aggregation-w-lacp/
categories:
  - enterprise
tags:
  - lacp
  - redundancy

---
Link aggregation is not a new concept, yet I still see a lot of folks who don&#8217;t make regular use of it. With regard to server networking architecture, especially in heavily virtualized or highly available environments, it&#8217;s a crucial tool that provides redundancy, and slightly increased throughput. For how simple it is to implement, it is a no-brainer to consider for physical server networking.

RedHat covers its configuration nicely in [this document][1] for RHEL6 and it&#8217;s flavours. They even provide a [configuration helper app][2] that guides you through the configuration options, and provides a working configuration for you at the end.  You can either copy in the interface files they provide, or run a script that will implement the changes you&#8217;ve selected.

To gain the most benefit from LACP, you&#8217;ll want your switch (or better yet &#8211; switches) to support it.  However, you can still gain from enabling it in a single-switch environment.  That&#8217;s the configuration that I&#8217;m going to lay out here.  I&#8217;ve got two onboard NICs in my physical CentOS 6.5 server, and I&#8217;m going to bond them together using the linux bonding driver, in mode 4, which will implement LACP.  Here are my interface files:

<pre>server:/etc/sysconfig/network-scripts$ cat ifcfg-eth0
DEVICE=eth0
ONBOOT=yes
NM_CONTROLLED=no
SLAVE=yes
MASTER=bond0
server:/etc/sysconfig/network-scripts$ cat ifcfg-eth1
DEVICE=eth1
ONBOOT=yes
NM_CONTROLLED=no
SLAVE=yes
MASTER=bond0
server:/etc/sysconfig/network-scripts$ cat ifcfg-bond0
DEVICE=bond0
ONBOOT=yes
BOOTPROTO=static
NM_CONTROLLED=no
IPADDR=10.0.8.4
NETMASK=255.255.255.0
GATEWAY=10.0.8.1
DNS1=10.0.8.1
BONDING_OPTS="mode=4 miimon=100"</pre>

Now with a service network restart, I will have enabled LACP and have bonded these two physical NICs into one link aggregate or channel bond.  Should one NIC or its cable fail, the other will automatically take over. You can use the iperf tool to compare throughput/bandwidth performance between LACP-enabled and disabled configs.

 [1]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/sec-Using_Channel_Bonding.html
 [2]: https://access.redhat.com/labs/networkbondinghelper/