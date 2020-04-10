---
title: Home lab, CAT6 wiring, and rack
author: Andrew Ludwar
type: post
date: 2017-09-13T18:05:48+00:00
url: /blog/2017/09/13/home-lab-cat6-wiring-and-rack/
dsq_thread_id:
  - 6141164266
categories:
  - networking
  - open source
  - OpenStack
tags:
  - homelab
  - networking
  - open source
  - openstack
  - private cloud

---
Between being busy with home renovations, and writing articles for work, I haven&#8217;t made much time for blogging this summer. Since it&#8217;s raining today and hampering my home renos, I&#8217;m finally getting to a post on the work I did wiring my house with CAT 6, and setting up a home lab rack in the basement. I realise having a rack in your house is a little ridiculous, but for those folks who like to tinker or need more resources than a desktop PC or laptop, it is incredibly useful (and affordable) to have. Like others, my home lab serves up some basic components for my home network, and also allows a testing ground that gets quite a bit of usage.

### **<span style="text-decoration: underline;">CAT6 Home Wiring:</span>**

This is a much easier project than most think. It goes a long way to offer connectivity to the various devices in your home, as well as increases your home&#8217;s market value. While I have home WiFi, its nice to be able to offer high speed bandwidth and reliability to every room in your house, and use the WiFi solely for areas that are difficult to run cable to. Moreover, its a much better WiFi experience to extend access points via wired switching than bridging antenneas. To put it candidly, I&#8217;ve never had to buffer any Netflix show off my PS3 or otherwise since running cable 🙂 .

**<span style="text-decoration: underline;">Materials list:</span>**

I bought this all off monoprice.com via Amazon, for a total cost of about $400 CDN. I already had the tools and some experience running cable from previous employment. Having the added benefit of living in a bungalow with an undeveloped basement, I only had to traverse one floor with the cabling. But the same steps apply for additional floors and hopefully you have tiled ceiling in the basement if it&#8217;s not undeveloped. Otherwise, you&#8217;ll need to make holes and patch it up afterwards.

&#8211; Cat 6 cable roll
  
&#8211; Cat 6 plugs, boots, RJ45 keystone connectors
  
&#8211; Low-voltage mounting bracket and wall plates
  
&#8211; Patch panel and patch panel hinged wall mount
  
&#8211; Cable tester, Cable stripper, Wire cutters, RJ45 crimper/punch tool
  
&#8211; Drywall saw, stud finder, drywall mud and trowel, drywall sandpaper, vacuum

First, I cut holes in the upstairs drywall adjacent to the coaxial cable outlets, as I know there is existing holes drilled behind these that lead to the basement. I piggy backed on these same holes, and made a couple new ones where I needed room. Hint, if you cut the drywall on a 45 degree angle facing inward to the wall (as opposed to 90 degrees) it makes it easier to set the drywall and mud back over it when you&#8217;re done. This allows the piece you cut out to sit back in the hole without falling in or forward.

[<img class="alignnone wp-image-595" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_130455-768x1024.jpg" alt="cat 6 wiring" width="393" height="524" />][1] [<img class="alignnone wp-image-583" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_171803-768x1024.jpg" alt="cat 6 wiring" width="394" height="525" />][2] [ <img class="alignnone wp-image-578" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_152304-1024x768.jpg" alt="IMG_20170225_152304" width="701" height="526" />][3][ <img class="alignnone wp-image-588" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170308_122712-768x1024.jpg" alt="IMG_20170308_122712" width="392" height="522" />][4][ <img class="alignnone wp-image-579" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_152329-768x1024.jpg" alt="IMG_20170225_152329" width="393" height="524" />][5][ <img class="alignnone wp-image-580" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_152437-768x1024.jpg" alt="IMG_20170225_152437" width="394" height="525" />][6][<img class="alignnone wp-image-581" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_152448-1024x768.jpg" alt="IMG_20170225_152448" width="404" height="303" />][7]

The cable runs follow the other low-voltage cable (coaxial/RJ11) and terminate near the electrical panel where I&#8217;ve added another wood backing. This gave me extra space to mount the patch panels, relocate other cabling, and have an electrician mount additional circuits that are used for the rack.

[<img class="alignnone wp-image-582" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_170725-768x1024.jpg" alt="IMG_20170225_170725" width="375" height="500" />][8] [<img class="alignnone wp-image-587" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170308_122701-768x1024.jpg" alt="IMG_20170308_122701" width="377" height="503" />][9] [ <img class="alignnone wp-image-586" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170308_111422-1024x768.jpg" alt="IMG_20170308_111422" width="669" height="502" />][10]

After running the cable, I used the punchdown tool to place the cable in the back of the patch port as well as at the other end in the keystone connector jack. There&#8217;s color coding in both to make it easy, but generally you should use [Type B][11] termination everywhere. With cables punched down, I tested the circuits with the cable tester to confirm termination, and confirmed I could grab an IP address from a router and did a bandwidth test.

[<img class="alignnone wp-image-584" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170226_134808-559x1024.jpg" alt="IMG_20170226_134808" width="359" height="658" /> ][12][<img class="alignnone wp-image-585" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170226_134812-e1505324117566-400x1024.jpg" alt="IMG_20170226_134812" width="257" height="658" />][13] [<img class="alignnone wp-image-589" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_103059-934x1024.jpg" alt="IMG_20170913_103059" width="600" height="658" />][14] [ <img class="alignnone wp-image-594" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_103243-1024x690.jpg" alt="IMG_20170913_103243" width="357" height="241" />][15]

Next came assembling the used rack found off the local classifieds, racking the gear, and making cable from the patch panel to the rack. Generally tried to keep things as tidy as possible for cabling.

[ ][15][ <img class="alignnone wp-image-592" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_103219-768x1024.jpg" alt="IMG_20170913_103219" width="470" height="627" />][16][ <img class="alignnone wp-image-591" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_103132-768x1024.jpg" alt="IMG_20170913_103132" width="468" height="624" />][17][<img class="alignnone wp-image-593" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_103234-1024x768.jpg" alt="IMG_20170913_103234" width="512" height="384" />][18]

Overall, I spent a couple of weekends completing the project, and currently have 8/24 cable runs populated. This covers my office, living room, and bedroom. Next up is patching in the kitchen, and the ceiling for an AP mount. In the rack I have:

&#8211; 2 Dell PowerConnect switches (1x 48 port, 1x 24 port, just the 48 port shown.)
  
&#8211; HP Z800, 2x Intel Xeon X5570, 16 core , 96 GB RAM, 3x 256 GB SSD in RAID0, 2x 2TB HDD in RAID1.
  
&#8211; Triplite 1.5 KW UPS, and APC 1.0 KW UPS.
  
&#8211; Tower PC, 1x Intel i7-2600K, 8 core , 32 GB RAM, 1x 256 GB SSD, 4x 1TB HDD in RAID5.
  
&#8211; HP DL360 G7, 2x Intel Xeon <span class="st">E5640, 24</span> core, 144 GB RAM, 3x 500 GB SSD in RAID0.

The cable modem and [Mikrotik CRS125-24G-1S-2HnD-IN][19] router sit upstairs in my office as they&#8217;re silent, and are trunked into the ToR switch. My workstation in the office upstairs is a Tower PC, 1x Intel i7-5930K, 12 core, 64 GB RAM, 1x 256 GB SSD M.2 NVMe, 5x 500 GB SSD in RAID5 with a boat load of screen real estate. 🙂 . The rack gear is all KVM hypervisors hosting a handful of VMs for DNS cache, grafana, monitoring, and a whack of OpenStack reproducing environments from Juno to Ocata.

[<img class="alignnone size-large wp-image-601" src="https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_114902-1024x687.jpg" alt="IMG_20170913_114902" width="1024" height="687" />][20]

 [1]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_130455.jpg
 [2]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_171803.jpg
 [3]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_152304.jpg
 [4]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170308_122712.jpg
 [5]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_152329.jpg
 [6]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_152437.jpg
 [7]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_152448.jpg
 [8]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170225_170725.jpg
 [9]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170308_122701.jpg
 [10]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170308_111422.jpg
 [11]: http://www.electriciantalk.com/attachments/f10/48169d1424101267-t568a-b-rj45.jpg
 [12]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170226_134808.jpg
 [13]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170226_134812-e1505324117566.jpg
 [14]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_103059.jpg
 [15]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_103243.jpg
 [16]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_103219.jpg
 [17]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_103132.jpg
 [18]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_103234.jpg
 [19]: https://mikrotik.com/product/CRS125-24G-1S-2HnD-IN
 [20]: https://calgaryrhce.ca/wp-content/uploads/2017/09/IMG_20170913_114902.jpg