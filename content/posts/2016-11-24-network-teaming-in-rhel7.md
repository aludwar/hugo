---
title: Network teaming in RHEL7
author: Andrew Ludwar
type: post
date: 2016-11-24T22:32:26+00:00
url: /blog/2016/11/24/network-teaming-in-rhel7/
dsq_thread_id:
  - 5394029631
categories:
  - enterprise
  - networking
  - open source
tags:
  - linux
  - networking
  - open source

---
If you&#8217;ve adopted or are just starting to read up on the new features included in Red Hat Enterprise Linux 7, you may have come across the new networking feature called teaming. It essentially is a replacement for bonding that offers more modularity, increased link monitoring features, higher network performance, and easier management of interfaces.

Habitually, network interface configuration is something I&#8217;m used to setting up once, and forgetting about it. I don&#8217;t usually re-visit my networking configuration unless some hardware changes, or I need to re-IP a system. With the advancement of SDN, this might change in the not-so-distant future, so I thought I&#8217;d give teaming a try. And hey, if it offers even marginally greater performance, why not get the most out of my OS?``

I first started with reading a comparison of [Network Bonding to Network Teaming][1], and the new network teaming daemon &#8220;teamd&#8221;, with it&#8217;s concept of &#8220;runners&#8221;. Network teaming has basically assigned a daemon to a link aggregate that allows management and monitoring of that interface through the daemon. I guess this is similar to systemd modularizing init scripts, we&#8217;ve got a daemon wrapper that will manage the config files for us in a programmatic way. After going through the [docs][2], I found it pretty easy to get this setup and running. If you&#8217;re lazy (read efficient) like me, there&#8217;s adequate example configs in _/usr/share/doc/teamd*/example-ifcfgs/_ that are easily modified (there&#8217;s one for LACP).

On one of my hypervisors, I&#8217;ve got 2 NICs, an LACP bond (will be replaced by an LACP team), and a bridge device. My new interface configs look like this:

<pre class="lang:sh decode:true">$ cat ifcfg-eno1 
DEVICE="eno1"
HWADDR="70:71:bc:5c:bd:b9"
DEVICETYPE="TeamPort"
ONBOOT="no"
TEAM_MASTER="team0"
NM_CONTROLLED=no
$ cat ifcfg-enp4s0 
DEVICE="enp4s0"
HWADDR="00:17:3f:d1:31:d8"
DEVICETYPE="TeamPort"
ONBOOT="no"
TEAM_MASTER="team0"
NM_CONTROLLED=no
$ cat ifcfg-team0 
DEVICE="team0"
DEVICETYPE="Team"
ONBOOT="yes"
BRIDGE=br0
BOOTPROTO=none
TEAM_CONFIG='{"runner": {"name": "lacp", "active": true, "fast_rate": true, "tx_hash": ["eth", "ipv4", "ipv6"]},"link_watch":    {"name": "ethtool"},"ports":    {"eno1": {}, "enp4s0": {}}}'
$ cat ifcfg-br0 
IPV6INIT=yes
IPV6_AUTOCONF=yes
BOOTPROTO=static
NM_CONTROLLED=no
IPADDR=192.168.122.10
NETMASK=255.255.255.0
GATEWAY=192.168.122.1
DNS1=8.8.8.8
DNS2=8.8.4.4
DEVICE=br0
STP=yes
DELAY=7
BRIDGING_OPTS=priority=32768
ONBOOT=yes
TYPE=Bridge
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=br0</pre>

The only real differences between bonding and teaming as far the config files are concerned is that there&#8217;s new device types called &#8220;TeamPort&#8221;, and &#8220;Team&#8221;, and then inside the team config there&#8217;s a JSON formatted entry that replaces the traditional bonding options. In here you define the runner (or method of link management) you want for your slave ports, the link monitoring tool, and which interfaces are slaves (ports) of the team. You can use the same bridge options with the team as you did the bond, that doesn&#8217;t change. One important rule to note though, bringing up the team interface won&#8217;t also bring up the slave interfaces, and in order to add slaves to a team, the slaves need to be in a link down state prior to addition to the team. This is the reason my ONBOOT=no for the NICs, and when systemd brings up the network, it will add the downed slaves to the team, then link up the slaves, then link up the team. I guess this is the modularity intent, let the slaves be managed on their own, then manage the team based on the slave behaviour.

After modifying my files as above, restarting the network service, waiting a few seconds for LACP negotiation to occur, my team is up and running. You can use the new teamdctl command to query and [control][3] the interfaces:

<pre class="lang:sh decode:true">$ sudo teamdctl team0 state
setup:
  runner: lacp
ports:
  eno1
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
        down count: 0
    runner:
      aggregator ID: 2, Selected
      selected: yes
      state: current
  enp4s0
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
        down count: 0
    runner:
      aggregator ID: 2, Selected
      selected: yes
      state: current
runner:
  active: yes
  fast rate: yes</pre>

Nice! That was less daunting than I thought it might be. The team driver also has good integration with NetworkManager, should you chose to [manage your team interfaces with the GUI][4].

Some others have done [comparison benchmarking of bonding vs teaming][5]. Although probably quite marginal for my use-case, the benchmarking shows additional bandwidth throughput, with reduced CPU load.

 [1]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Comparison_of_Network_Teaming_to_Bonding.html
 [2]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/ch-Configure_Network_Teaming.html
 [3]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html-single/Networking_Guide/index.html#sec-Controlling_teamd_with_teamdctl
 [4]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html-single/Networking_Guide/index.html#sec-Creating_a_Network_Team_Using_a_GUI
 [5]: http://rhelblog.redhat.com/2014/06/23/team-driver/