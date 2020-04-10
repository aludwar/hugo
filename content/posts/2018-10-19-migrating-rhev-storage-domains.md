---
title: Migrating RHEV storage domains
author: Andrew Ludwar
type: post
date: 2018-10-19T21:20:50+00:00
url: /blog/2018/10/19/migrating-rhev-storage-domains/
xyz_smap:
  - 1
categories:
  - cloud
  - open source
  - performance tuning
  - storage
tags:
  - hardware
  - open source
  - private cloud
  - software defined storage
  - SSD
  - storage

---
Recently, I&#8217;ve been doing some troubleshooting in my virtualization environment, specifically with the NFS storage backing it. To isolate an issue I needed to migrate all the VM disks off the main data store to another one. I hadn&#8217;t performed this kind of activity before, but found it to be quite easy. I quickly built an additional NFS server, added it into RHV, and migrated the VM disks to it with just a few clicks and some waiting for the copying to be complete. [This article][1] was basic, but helpful.

I selected all my VM disks on the original NFS data store and clicked &#8220;Move&#8221;. Since I only had two data stores, the other was auto-selected and I just had to hit &#8220;OK&#8221;.

[<img class="alignnone size-large wp-image-673" src="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-move-1024x475.png" alt="RHEV-move" width="1024" height="475" srcset="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-move-1024x475.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-move-300x139.png 300w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-move-768x357.png 768w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-move.png 1906w" sizes="(max-width: 1024px) 100vw, 1024px" />][2]

They slowly started moving over:

[<img class="alignnone size-large wp-image-671" src="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-moving2-1024x482.png" alt="RHEV Copying" width="1024" height="482" srcset="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-moving2-1024x482.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-moving2-300x141.png 300w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-moving2-768x361.png 768w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-moving2.png 1902w" sizes="(max-width: 1024px) 100vw, 1024px" />][3]

After a little while, I saw my previous NFS data store to be depleted, and the new store now containing all the VM disks:

[<img class="alignnone size-large wp-image-674" src="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-domains-1024x252.png" alt="RHEV Domains" width="1024" height="252" srcset="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-domains-1024x252.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-domains-300x74.png 300w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-domains-768x189.png 768w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-domains.png 1906w" sizes="(max-width: 1024px) 100vw, 1024px" />][4]

[<img class="alignnone size-large wp-image-675" src="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-nfs_bak-1024x276.png" alt="RHEV VMs" width="1024" height="276" srcset="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-nfs_bak-1024x276.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-nfs_bak-300x81.png 300w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-nfs_bak-768x207.png 768w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-nfs_bak.png 1911w" sizes="(max-width: 1024px) 100vw, 1024px" />][5]

Next up, I&#8217;ll elaborate on why I&#8217;m doing this troubleshooting :).

 [1]: https://access.redhat.com/solutions/736513
 [2]: https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-move.png
 [3]: https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-moving2.png
 [4]: https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-domains.png
 [5]: https://calgaryrhce.ca/wp-content/uploads/2018/10/RHEV-nfs_bak.png