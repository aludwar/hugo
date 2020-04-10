---
title: Retro gaming with RetroPie
author: Andrew Ludwar
type: post
date: 2019-01-03T22:58:42+00:00
url: /blog/2019/01/03/retro-gaming-with-retropie/
xyz_smap:
  - 1
categories:
  - open source
tags:
  - open source
  - raspberri pi

---
I&#8217;ve been using a raspberry pi for some occasional tinkering, but have finally found it a more permanent home. I recently got an itch to play some old console games and thought I&#8217;d build a RetroPie. I had all the hardware I needed, but in case you don&#8217;t there are [kits available that include everything][1] one needs to get a retro gaming system going. These are a great option to get started.

With how easy these are to build, I don&#8217;t think I&#8217;d consider the classic NES or classic SNES products that have recently been released. I like being able to customize the platform, and it&#8217;s really easy to combine NES/SNES/N64 and other emulators in the same piece of hardware, thereby cutting down how many systems you need to buy and HDMI ports required of your TV. I can also easily add new games to the system, and would rather donate money to the RetroPie project over buying others&#8217; hardware. So this is exactly what I did!

Once you have the hardware (raspberry pi, pi power cable, microsd card, USB stick, HDMI cable), everything you need to get started is on the [RetroPie wiki][2]. After copying the retropie image to the sd card, I was able to boot into retropie and use the USB stick to start copying my ROMs over. As soon as you copy ROMs into their respective emulator folder, the emulator will show the SNES/NES option for selection. I opted to load my pi with some NES/SNES classics I still have in a box in the basement.

The raspberry pi ecosystem is quite large, so unsurprisingly I was able to find a [really cool SNES case][3] for my pi:

[<img class="alignnone wp-image-727" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-1-1024x824.jpg" alt="" width="521" height="419" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-1.jpg 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-1-300x241.jpg 300w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-1-768x618.jpg 768w" sizes="(max-width: 521px) 100vw, 521px" /> ][4][<img class="alignnone wp-image-726" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-2-768x1024.jpg" alt="" width="317" height="423" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-2-768x1024.jpg 768w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-2-225x300.jpg 225w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-2.jpg 1024w" sizes="(max-width: 317px) 100vw, 317px" /> ][5][<img class="alignnone wp-image-725" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-3-1024x914.jpg" alt="" width="474" height="423" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-3.jpg 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-3-300x268.jpg 300w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-3-768x686.jpg 768w" sizes="(max-width: 474px) 100vw, 474px" />][6][<img class="alignnone wp-image-724" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-4-1024x408.jpg" alt="" width="522" height="208" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-4.jpg 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-4-300x120.jpg 300w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-4-768x306.jpg 768w" sizes="(max-width: 522px) 100vw, 522px" /> ][7][<img class="alignnone wp-image-723" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-5-1024x470.jpg" alt="" width="455" height="209" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-5-1024x470.jpg 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-5-300x138.jpg 300w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-5-768x352.jpg 768w" sizes="(max-width: 455px) 100vw, 455px" />][8]

It relocates the power to functional on/off and reset buttons on the top of the case like the classic SNES, and also includes some cooling, and easy access to ports and storage. With a couple of USB SNES gaming pads for $8 off Amazon, and about an hour&#8217;s worth of assembly and install &#8211; I was up and running:

[<img class="alignnone wp-image-729" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-7-912x1024.jpg" alt="RetroPie emulation" width="588" height="660" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-7-912x1024.jpg 912w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-7-267x300.jpg 267w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-7-768x863.jpg 768w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-7.jpg 1024w" sizes="(max-width: 588px) 100vw, 588px" /> ][9][<img class="alignnone wp-image-730" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-6-1024x727.jpg" alt="Final Fantasy 2" width="762" height="541" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-6.jpg 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-6-300x213.jpg 300w, https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-6-768x545.jpg 768w" sizes="(max-width: 762px) 100vw, 762px" />][10]

Very cool!

 [1]: https://www.memoryexpress.com/Products/MX72345
 [2]: https://github.com/RetroPie/RetroPie-Setup/wiki/First-Installation
 [3]: https://www.amazon.ca/gp/product/B079PHRHLG/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1
 [4]: https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-1.jpg
 [5]: https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-2.jpg
 [6]: https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-3.jpg
 [7]: https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-4.jpg
 [8]: https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-5.jpg
 [9]: https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-7.jpg
 [10]: https://calgaryrhce.ca/wp-content/uploads/2019/01/SNES-6.jpg