---
title: Getting the most performance out of Openstack, and NFV
author: Andrew Ludwar
type: post
date: 2017-10-14T17:33:55+00:00
url: /blog/2017/10/14/getting-the-most-performance-out-of-openstack-and-nfv/
xyz_smap:
  - 1
dsq_thread_id:
  - 6215585902
categories:
  - cloud
  - devops
  - open source
  - OpenStack
  - performance tuning
tags:
  - devops
  - open cloud
  - open source
  - openstack
  - performance tuning
  - private cloud

---
By now, most IT staff has either had a chance to deploy OpenStack, or at least are familiar with what it is and what benefits it offers. We&#8217;ve moved past the installer race, the fragmentation of &#8220;as-a-service&#8221; components, and endless TCO calculators. The ecosystem and community has reached a maturity stage where the media and industry alike are calling OpenStack &#8220;<a href="https://major.io/2017/02/24/openstack-isnt-dead-its-boring-thats-a-good-thing/" target="_blank">boring</a>&#8220;. **Don&#8217;t be fooled, all of these are good things.** They&#8217;re a product of fast-paced innovation seeing a settling of opinions and decision making. OpenStack has reached a maturity point that Linux also hit earlier on in its journey. As linux was (and still is) an incredibly capable and cost-effective option for an OS, OpenStack is also an incredibly capable and cost-effective option for private or public cloud. But now what? What&#8217;s next? The answer is deeper and more efficient capability, addressing more edge use-cases, and general polish and integration of all the development that&#8217;s happened to date. The low-hanging-fruit development is over, we&#8217;re deeper into the less exciting and less headline grabbing work; maximizing the assets as much as we can.

There&#8217;s been a lot of blog traffic regarding tips and tricks, technologies to consider for different use-cases, etc. Given the recent explosion in NFV development and adoption, I&#8217;ll highlight the ones that I think provide the most benefit in the area to try and cut down on the amount of reading needing to be done. Here we go:

  1. <a href="http://redhatstackblog.redhat.com/2017/01/18/9-tips-to-properly-configure-your-openstack-instance/" target="_blank">Getting the most out of your instances.</a> This is a great summary on the options available to maximize function and performance of OpenStack instances.
  2. <a href="http://chrisj.cloud/?q=node/1" target="_blank">Helpful deployment and operations management tips</a>. These are great suggestions on working smartly with new deployments, and maintaining them throughout the lifecycle.
  3. <a href="https://jaormx.github.io/2017/run-ansible-playbook-on-tripleo-nodes/" target="_blank">Operations must-have: dynamic ansible inventory for TripleO</a>. If you&#8217;re doing ad-hoc maintenance on your cloud outside of tripleo, you&#8217;d be silly to not use ansible and its dynamic inventory.

NFV specific material:

  1. What OSP technologies should I look at to help scale and tune for network intensive workloads? 
      * <a href="http://verticalindustriesblog.redhat.com/scaling-virtual-machine-network-performance-for-network-intensive-workloads/" target="_blank">Scaling virtual machine network performance for network intensive workloads</a>
      * <a href="http://redhatstackblog.redhat.com/2016/02/10/boosting-the-nfv-datapath-with-rhel-openstack-platform/" target="_blank">Boosting the NFV datapath</a>
  2. Now that we know what technologies are available, if I have &#8220;x&#8221; workload, which options should I tune? 
      * <a href="https://access.redhat.com/documentation/en-us/reference_architectures/2017/html/deploying_mobile_networks_using_network_functions_virtualization/performance_and_optimization" target="_blank">Reference Architecture: Mobile Networks and NFV performance and optimization</a>
  3. OVS-DPDK with multi-queue, and further NFV performance tuning: 
      * <a href="https://access.redhat.com/solutions/3215141" target="_blank">How to enable nova multi queue support with and without OVS-DPDK in Red Hat OpenStack Platform 10</a>
      * <a href="https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/10/html/network_functions_virtualization_configuration_guide/" target="_blank">OSP Newton NFV Configuration Guide</a>
  4. Tuning for zero packet loss in Red Hat OpenStack Platform, 3 part series: 
      * <a href="http://redhatstackblog.redhat.com/2017/07/11/tuning-for-zero-packet-loss-in-red-hat-openstack-platform-part-1/" target="_blank">Tuning for Zero Packet Loss in Red Hat OpenStack Platform &#8211; Part 1</a>
      * <a href="http://redhatstackblog.redhat.com/2017/07/13/tuning-for-zero-packet-loss-in-red-hat-openstack-platform-part-2/" target="_blank">Tuning for Zero Packet Loss in Red Hat OpenStack Platform &#8211; Part 2</a>
      * <a href="http://redhatstackblog.redhat.com/2017/07/18/tuning-for-zero-packet-loss-in-red-hat-openstack-platform-part-3/" target="_blank">Tuning for Zero Packet Loss in Red Hat OpenStack Platform &#8211; Part 3</a>

If you have any questions, or come across anything else that you think should be added &#8211; let me know! Happy management and tuning.