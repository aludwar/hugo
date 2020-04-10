---
title: Home lab switch replacement
author: Andrew Ludwar
type: post
date: 2016-10-30T23:48:16+00:00
url: /blog/2016/10/30/home-lab-switch-replacement/
dsq_thread_id:
  - 5310943156
categories:
  - enterprise
  - networking
tags:
  - hardware
  - networking

---
A few years back, I purged nearly all of my computer components that were kicking around the house, thus essentially abandoning my home lab. I had learned what I needed to with it, and had enough equipment at work to get done what I needed, so off it went. Old PCs, switches, cables, parts, etc. were all sold off, and it felt good to finally declutter. Fast forward a few years and I find myself with a different job, tackling different problems, new technology is out and the need to invest in a home lab has become important again. When I was looking at routers/switches, [mikrotik][1] came up and looked like a good option &#8211; pretty feature full and open source. I bought a couple of these, and later found out the LACP implementation had some [limitations][2]. This didn&#8217;t matter much until recently when I&#8217;ve began using 30+ VMs on a hypervisor. After some light research, and a helpful [/r/homelab][3] community, I decided on a Dell PowerConnect 6224 from ebay.

The Dell&#8217;s are pretty feature-full as well, and their OS has been modeled after Cisco&#8217;s. I&#8217;ve done work with Cisco gear in the past, so that was a good fit for me and a chance to get my hands dirty in networking again. I found a good deal from [Geekfurb][4] and spent this weekend setting it up.

When I first turned it on, it was quite a bit louder than expected. Certainly too loud to sit in the same room with. I got a few [replacement fans][5], which managed to reduce the noise considerably from about 40 dBA, to about 18 dBA. Luckily I did some googling before replacing the fans as I ran into a known issue with the fan wiring. Dell has a different wire sequence in their fans, so you&#8217;ll want to watch [this youtube video][6] when replacing them. Because my fans are 4.53 CFM and the OEM ones were 7.5 CFM, I&#8217;ve got a red fan status light on my switch. I think the [lower RPM is throwing it off][7]. All my fans work, and the switch powers on and passes diagnostics just fine. Here&#8217;s a before and after sound comparison for the replacement:

**<span style="text-decoration: underline;">Before:</span>**

{{< youtube 2DLii8vm5Mc >}}

**<span style="text-decoration: underline;">After:</span>**

{{< youtube yU09AiYyZl4 >}}

Much better. Even with three fans at 18 dBA each, it&#8217;s much quieter than before, and is easy to be in the same room as. As for connections, this switch has the old DB9 serial port on it, instead of the RJ45 management port. For clarity sake, there&#8217;s a couple ways network switches can be connected to. The older variety of gear will use the DB9 male serial port, which you&#8217;ll need a null modem cable and likely a DB9 to USB converter cable to use with a modern laptop. The newer variety of gear will have the RJ45 management port on the switch, which you&#8217;ll need a DB9 to RJ45 cross-over cable, and also DB9 to USB converter. Unless you&#8217;re using an old enough PC/laptop that has a serial port. Here&#8217;s what they look like:

[<img class="alignleft wp-image-513" src="https://calgaryrhce.ca/wp-content/uploads/2016/10/db9-nullmodem.jpg" alt="db9 null modem cable" width="460" height="242" />][8][<img class="aligncenter wp-image-512" src="https://calgaryrhce.ca/wp-content/uploads/2016/10/db9-usb.jpg" alt="db9 to usb converter" width="369" height="273" />][9]{.alignnone}[<img class="alignleft wp-image-509" src="https://calgaryrhce.ca/wp-content/uploads/2016/10/cisco-db9-rj45.jpg" alt="cisco db9 male to RJ45" width="323" height="274" />][10]

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

And the rear of the switch:

[<img class="alignleft wp-image-511 size-full" src="https://calgaryrhce.ca/wp-content/uploads/2016/10/dell-pc-serial.jpg" alt="dell switch serial port" width="500" height="139" />][11]

&nbsp;

&nbsp;

&nbsp;

&nbsp;

You can use any terminal emulator software to connect to the switch. Putty is a popular option that&#8217;s easy to use, there&#8217;s a package available for linux. In my case, the serial tty I&#8217;m using comes up as \`/dev/ttyUSB0\`. After I connect through that to the switch, I reset it to factory settings from the boot menu, and went to work. The main configuration I&#8217;m after is setting up link aggregation groups, and configuring the switch ports:

<pre class="lang:sh decode:true ">$ show running-config

# switchport configuration
!
interface ethernet 1/g7
channel-group 1 mode auto
description 'LACP for hypervisor 1'
exit
!
interface ethernet 1/g8
channel-group 1 mode auto
description 'LACP for hypervisor 1'
exit
!

# port-channel configuration
interface port-channel 1
description 'LACP group hypervisor 1'
exit

$ show interfaces status 

Port   Type                            Duplex  Speed    Neg  Link  Flow Control
                                                             State Status
-----  ------------------------------  ------  -------  ---- ----- ------------
1/g7   Gigabit - Level                 Full    1000     Auto Up     Active     
1/g8   Gigabit - Level                 Full    1000     Auto Up     Active     

...

Ch   Type                            Link
                                     State
---  ------------------------------  -----
ch1  Link Aggregate                  Up      


</pre>

With that work done, on each hypervisor I&#8217;ve created a linux bond with the two NICs, then a linux bridge on top of that for attaching virtual networks. The detailed steps are [here][12], but this is what it looks like:

[<img class="alignnone size-full wp-image-508" src="https://calgaryrhce.ca/wp-content/uploads/2016/10/bonded-bridge-1.png" alt="linux bond with bridge" width="500" height="389" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/10/bonded-bridge-1.png 500w, https://calgaryrhce.ca/wp-content/uploads/2016/10/bonded-bridge-1-300x233.png 300w" sizes="(max-width: 500px) 100vw, 500px" />][13]

All finished! Testing with iperf shows I&#8217;m getting close to line speed on the interfaces (~940 Mbit on 1 Gigabit NICs), so the setup is correct. To really test the aggregate bandwidth I&#8217;ll have to simulate multiple simultaneous connections. I&#8217;ll leave that for another day. Perhaps let this sit for a while and attach some switch statistics to back up the bandwidth numbers. All in all, for less than $150 I&#8217;ve finally got all my lab systems setup with LACP.

 [1]: https://routerboard.com/
 [2]: http://forum.mikrotik.com/viewtopic.php?t=86498
 [3]: https://www.reddit.com/r/homelab/
 [4]: http://www.ebay.com/usr/geekfurb
 [5]: http://www.fractal-design.com/home/product/case-fans/silent-series-r2-40mm
 [6]: https://www.youtube.com/watch?v=WkMBa2yfW4o
 [7]: http://forums.bit-tech.net/showthread.php?t=297341
 [8]: https://calgaryrhce.ca/wp-content/uploads/2016/10/db9-nullmodem.jpg
 [9]: https://calgaryrhce.ca/wp-content/uploads/2016/10/db9-usb.jpg
 [10]: https://calgaryrhce.ca/wp-content/uploads/2016/10/cisco-db9-rj45.jpg
 [11]: https://calgaryrhce.ca/wp-content/uploads/2016/10/dell-pc-serial.jpg
 [12]: https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s2-networkscripts-interfaces_network-bridge.html
 [13]: https://calgaryrhce.ca/wp-content/uploads/2016/10/bonded-bridge-1.png