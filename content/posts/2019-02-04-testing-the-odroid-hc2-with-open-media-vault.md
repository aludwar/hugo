---
title: Testing the ODROID-HC2 with Open Media Vault
author: Andrew Ludwar
type: post
date: 2019-02-04T22:59:18+00:00
url: /blog/2019/02/04/testing-the-odroid-hc2-with-open-media-vault/
xyz_smap:
  - 1
categories:
  - open source
tags:
  - hardware
  - linux
  - storage

---
I&#8217;ve recently been tinkering with a few small Raspberry Pi-like boards. I&#8217;m anxious to get my hands on hardkernel&#8217;s [ODROID-H2][1] x86 board, I think they might have hit a sweet spot for a decently spec&#8217;d, low TDP board with a decent amount of RAM and NVMe storage options. I like the NUC platform, but the H2 might be a less expensive & more capable option. Until this product gets back in stock, I&#8217;ve been playing with the [ODROID-HC2.][2]

[<img class="alignnone size-large wp-image-748" src="https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152413-693x1024.jpg" alt="ODROID-HC2" width="693" height="1024" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152413-693x1024.jpg 693w, https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152413-203x300.jpg 203w, https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152413-768x1135.jpg 768w, https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152413.jpg 812w" sizes="(max-width: 693px) 100vw, 693px" /> ][3][<img class="alignnone size-large wp-image-746" src="https://calgaryrhce.ca/wp-content/uploads/2019/02/MVIMG_20190204_152420-750x1024.jpg" alt="ODROID-HC2" width="750" height="1024" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/02/MVIMG_20190204_152420-750x1024.jpg 750w, https://calgaryrhce.ca/wp-content/uploads/2019/02/MVIMG_20190204_152420-220x300.jpg 220w, https://calgaryrhce.ca/wp-content/uploads/2019/02/MVIMG_20190204_152420-768x1049.jpg 768w, https://calgaryrhce.ca/wp-content/uploads/2019/02/MVIMG_20190204_152420.jpg 1000w" sizes="(max-width: 750px) 100vw, 750px" />][4]

The HC2 board is very similar to a Raspberry Pi, except with a little more storage capability having a SATA3 port included. It&#8217;s a really good platform for a low TDP, inexpensive NAS appliance. Of course there&#8217;s no redundancy with just a single disk, but ideally you have the important data backed up elsewhere and just use this device for serving out content. It comes with a metal base that fits mounting either a 2.5&#8243; or 3.5&#8243; disk, and a plastic case enclosure. The USB piece shown is a WiFi adapter:

[<img class="alignnone size-large wp-image-747" src="https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152404-1024x615.jpg" alt="ODROID-HC2" width="1024" height="615" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152404-1024x615.jpg 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152404-300x180.jpg 300w, https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152404-768x461.jpg 768w, https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152404.jpg 1500w" sizes="(max-width: 1024px) 100vw, 1024px" />][5]

There&#8217;s a few [prebuilt images][6] for the device, including Android. I&#8217;ve opted for OpenMediaVault, and it&#8217;s worked pretty well thus far. I&#8217;ve added a 2 TB 3.5&#8243; disk, and have got it hosting some NFS shares, an HTTP server, have Docker installed on it, and am running a Plex container within.

[<img class="alignnone size-large wp-image-740" src="https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV-1024x730.png" alt="OMV" width="1024" height="730" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV-1024x730.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV-300x214.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV-768x547.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV.png 1239w" sizes="(max-width: 1024px) 100vw, 1024px" />][7]

I ran into a little bit of trouble setting up the NFS shares on OMV. I realised one needs to create ACLs as well as adding the right NFS client subnets and options for access. You can double check how this ends up getting parsed by looking at the /etc/exports file via SSH. Getting Docker running on it was easy, I used the ARM specific container build for Plex. After sorting out the NFS ACLs, I had no trouble starting up the Plex container.

[<img class="alignnone size-large wp-image-739" src="https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV2-1024x432.png" alt="OMV2" width="1024" height="432" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV2-1024x432.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV2-300x126.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV2-768x324.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV2.png 1912w" sizes="(max-width: 1024px) 100vw, 1024px" />][8]

Use host-based container networking, and bind-mount the Plex directories on the NFS shared folder:

[<img class="alignnone size-full wp-image-741" src="https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV3.png" alt="OMV3" width="647" height="698" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV3.png 647w, https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV3-278x300.png 278w" sizes="(max-width: 647px) 100vw, 647px" />][9]

Plex setup found the Plex server right away, so setup was easy. I don&#8217;t have SSL setup on the device, but I didn&#8217;t need to set the &#8220;Allow Fallback to Insecure Connections&#8221; to &#8220;Always&#8221; as some others had reported. Next up is adding media to the Plex library.

[<img class="alignnone size-large wp-image-743" src="https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV4-1024x425.png" alt="OMV4" width="1024" height="425" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV4-1024x425.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV4-300x125.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV4-768x319.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV4.png 1905w" sizes="(max-width: 1024px) 100vw, 1024px" />][10]

Overall, pretty pleased with this board for what little I spent on it.

 [1]: https://www.hardkernel.com/shop/odroid-h2/
 [2]: https://www.hardkernel.com/shop/odroid-hc2-home-cloud-two/
 [3]: https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152413.jpg
 [4]: https://calgaryrhce.ca/wp-content/uploads/2019/02/MVIMG_20190204_152420.jpg
 [5]: https://calgaryrhce.ca/wp-content/uploads/2019/02/IMG_20190204_152404.jpg
 [6]: https://wiki.odroid.com/odroid-xu4/os_images/os_images
 [7]: https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV.png
 [8]: https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV2.png
 [9]: https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV3.png
 [10]: https://calgaryrhce.ca/wp-content/uploads/2019/02/OMV4.png