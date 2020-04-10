---
title: Importing VMs into RHV from KVM
author: Andrew Ludwar
type: post
date: 2018-10-23T16:13:33+00:00
url: /blog/2018/10/23/importing-vms-into-rhv-from-kvm/
xyz_smap:
  - 1
categories:
  - cloud
  - enterprise
  - open source
tags:
  - open source
  - private cloud
  - virtualization

---
To import virtual machines from a KVM/libvirt hypervisor into Red Hat Virtualization, is pretty easy. RHV will handle the v2v copying for you in the background, there are just a couple prerequisite steps to take prior to importing. On the RHV hypervisor host you&#8217;re proxying the import connection through, setup passwordless SSH from the vdsm user to the root user on the source hypervisor:

<pre class=""># sudo -u vdsm ssh-keygen
# sudo -u vdsm ssh-copy-id root@kvmhost.example.com
# sudo -u vdsm ssh root@kvmhost.example.com</pre>

From the RHV virtual machines section, click &#8220;Import&#8221;. You&#8217;ll get to enter the URI connection string and authentication details, clicking &#8220;Load&#8221; will make the connection and populate the VMs listing:

[<img class="alignnone size-large wp-image-690" src="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import-1024x431.png" alt="RHV Import" width="1024" height="431" srcset="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import-1024x431.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import-300x126.png 300w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import-768x324.png 768w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import.png 1906w" sizes="(max-width: 1024px) 100vw, 1024px" />][1]

You can chose to directly import the VM, or clone it into the environment:

[<img class="alignnone size-large wp-image-689" src="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import2-1024x412.png" alt="RHV Import" width="1024" height="412" srcset="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import2-1024x412.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import2-300x121.png 300w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import2-768x309.png 768w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import2.png 1910w" sizes="(max-width: 1024px) 100vw, 1024px" />][2]

After a little wait for the copying and conversion, we&#8217;re done!

[<img class="alignnone size-large wp-image-692" src="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-moved-1024x481.png" alt="RHV Moved" width="1024" height="481" srcset="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-moved-1024x481.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-moved-300x141.png 300w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-moved-768x361.png 768w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-moved.png 1908w" sizes="(max-width: 1024px) 100vw, 1024px" />][3]

 [1]: https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import.png
 [2]: https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-import2.png
 [3]: https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-moved.png