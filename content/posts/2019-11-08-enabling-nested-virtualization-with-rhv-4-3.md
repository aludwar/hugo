---
title: Enabling nested virtualization with RHV 4.3
author: Andrew Ludwar
type: post
date: 2019-11-08T16:08:08+00:00
url: /blog/2019/11/08/enabling-nested-virtualization-with-rhv-4-3/
xyz_smap_tw_future_to_publish:
  - 'a:3:{s:18:"post_tw_permission";s:1:"0";s:19:"xyz_tw_img_permissn";s:1:"1";s:14:"xyz_tw_message";s:26:"{POST_TITLE} - {PERMALINK}";}'
xyz_smap:
  - 1
categories:
  - enterprise
  - open source
tags:
  - open source
  - virtualization

---
To enable nested virtualization within a Red Hat Virtualization 4.3 environment, you will need to do two things. First, make sure the &#8220;vdsm-hook-nestedvt&#8221; package is present on the host hypervisor. You&#8217;ll need to restart vdsmd as well after installing.

Second, you&#8217;ll need to enable the Pass-Through Host CPU option in the virtual machine settings (under the host section). With both of these done, you should get the virtualization extensions passed into the guest OSs on that hypervisor.

Keep in mind nested virtualization is not supported for production workloads.

References:
  
[1] https://access.redhat.com/solutions/3543721
  
[2] https://access.redhat.com/documentation/en-us/red\_hat\_virtualization/4.3/html-single/administration\_guide/index#Kernel\_Settings_Explained