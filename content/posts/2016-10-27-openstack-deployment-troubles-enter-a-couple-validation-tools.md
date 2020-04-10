---
title: OpenStack deployment troubles? Enter a couple validation tools
author: Andrew Ludwar
type: post
date: 2016-10-28T00:34:47+00:00
url: /blog/2016/10/27/openstack-deployment-troubles-enter-a-couple-validation-tools/
dsq_thread_id:
  - 5260570894
categories:
  - cloud
  - open source
  - OpenStack
tags:
  - open cloud
  - open source
  - openstack
  - private cloud
  - public cloud

---
Whether you&#8217;re on your first or hundredth OpenStack deployment, any administrator will tell you that getting the initial deployment configuration correct is crucial. This gets easier with experience, but what can one do in the meantime to supplement that knowledge? Enter two tools: clapper, and the network isolation template generator.

Firstly: the [network template generator][1]. If you&#8217;ve read through any of the heat templates, or perused some of the upstream docs, you&#8217;ll know that configuring the networks you want in your OpenStack environment is a big part of the architecture. It&#8217;s also not easy to make changes to that once the environment is deployed. Typically, these configuration parameters are held in the network-environment.yaml template, as well as the nic-configs/<hostname>.yaml templates. To save on confusion and syntax mistakes in manual editing, Ben Nemec has created a GUI tool that creates these templates for you after running through a network creation wizard. With it, you can configure bonding, LACP, VLANs, bridges, etc. all on a per-node basis to satisfy your OpenStack environment needs. Ben publishes a [video guide][2] on the tool, as well as a [blog][3] on it&#8217;s use.

Secondly: [clapper][4]. It&#8217;s a suite of ansible playbooks that run both prior to and post deployment to validate the configuration of the environment. Within the tool are prepackaged validations (ansible playbooks) submitted by community members to check for things like adequate file descriptors, open file limits, network gateways, disk space, ip ranges, network templates, detecting rogue DHCP servers, and many more. You can git clone this tool to your undercloud server and run validations at any stage of your deployment. It&#8217;s handy in helping you implement best practices that come straight from the experience of the OpenStack community.

Got another tool to share? I&#8217;d love to hear from you. One that&#8217;s been picking up traction lately for ease of cloud deployments is the [quickstart cloud installer.][5] When I get some time, I&#8217;ll try it out and post a review.

 [1]: https://github.com/cybertron/tripleo-scripts#net-iso-genpy
 [2]: https://www.youtube.com/watch?v=k2ZBkkHdeEM&feature=youtu.be
 [3]: http://blog.nemebean.com/content/tripleo-network-isolation-template-generator
 [4]: https://github.com/rthallisey/clapper
 [5]: http://rhelblog.redhat.com/2016/09/14/your-cloud-installed-before-lunch-with-quickstart-cloud-installer-1-0/