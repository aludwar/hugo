---
title: 32-bit application support in RHEL7
author: Andrew Ludwar
type: post
date: 2019-05-07T16:17:02+00:00
url: /blog/2019/05/07/32-bit-application-support-in-rhel7/
xyz_smap:
  - 1
categories:
  - enterprise
  - open source
tags:
  - enterprise
  - linux
  - linux kernel
  - open source
  - rhel5
  - rhel6
  - rhel7

---
I recently needed to check the supportability of 32-bit applications in Red Hat Enterprise Linux. RHEL can support 32-bit applications in a 64-bit environment in the following scenarios:

  * <https://access.redhat.com/solutions/509373>

Additionally, with the multilib toolchain, one can install the 32-bit packages by specifying the architecture in the yum install command:

    # yum install glibc.i686

&nbsp;

More details can be found here:

  *  <https://access.redhat.com/solutions/55888>