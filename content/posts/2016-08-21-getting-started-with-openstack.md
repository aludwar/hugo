---
title: Getting started with OpenStack
author: Andrew Ludwar
type: post
date: 2016-08-22T01:03:42+00:00
url: /blog/2016/08/21/getting-started-with-openstack/
xyz_smap:
  - 1
dsq_thread_id:
  - 5084177858
categories:
  - cloud
  - devops
  - open source
  - OpenStack
tags:
  - devops
  - open cloud
  - openstack
  - private cloud
  - public cloud

---
[<img class="wp-image-388 alignleft" src="https://calgaryrhce.ca/wp-content/uploads/2016/08/OpenStack-logo.png" alt="OpenStack logo" width="301" height="301" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/08/OpenStack-logo.png 432w, https://calgaryrhce.ca/wp-content/uploads/2016/08/OpenStack-logo-150x150.png 150w, https://calgaryrhce.ca/wp-content/uploads/2016/08/OpenStack-logo-300x300.png 300w" sizes="(max-width: 301px) 100vw, 301px" />][1]

So you&#8217;d like to get started with OpenStack eh? (Sorry, that&#8217;s my Canadian coming through). You may have been thinking about it for a while, evaluating the landscape, the maturity, evaluating your tolerance for new technology, and waiting for the right time to adopt. Being on the verge of adopting a new and disruptive technology is an intimidating place to be. Lots of questions arise: Where to start? What to evaluate? Who to partner with? What gaps to address? What should the architecture look like? What is the project purpose? Is my staff well prepared? What training should we get? Is our infrastructure capable of supporting it? How to handle long term tasks like patching, scaling, or lifecycle management? These aren&#8217;t easy questions to answer, and the answers usually come from experience; which you don&#8217;t have. What do you do? I&#8217;m going to start the first in a series of posts addressing some of these questions and hopefully adding some clarity to OpenStack adoption. First, some of the basics.

### **What is OpenStack?
  
** 

OpenStack is essentially the culmination of commoditized infrastructure, and years of frustration with inflexible, closed, proprietary data center infrastructure components. (That&#8217;s a bold statement, let me expand on that.) As hardware virtualization took off in the mid to late 2000&#8217;s we began to see the potential, and reap the benefit, of increased application deployment speed. No longer being tied to the physical server procurement process opened the industry&#8217;s eyes to server flexibility and utilization. Virtualization was born from the idea of vertical density, ie. how many OS&#8217;s can we cram onto this server to maximize its use? As we became accustomed to &#8220;spinning up a VM&#8221;, these tasks started to feel trivial, frequent, and burdensome. In true sysadmin fashion, we looked to automate some of these tasks, but we didn&#8217;t get very far. Existing virtualization tools were designed with focus on performance, high availability, and ease of management. But not automated management. As our virtual environments became more utilized and integrated, we started to see delays in the process of &#8220;spinning up a VM&#8221;. Many checks and balances were added, and soon the time to provision virtual resources started to creep closer and closer to that of its physical cousin. Sound familiar? (I also wrote a [post][2] on this last year). It seemed like we couldn&#8217;t even buy an adequate API, let alone integrate it with something else.

At the same time, applications started to adapt to the new concept of commoditized hardware. Server infrastructure began to be treated more like consumables. Scaling out started to be as important as scaling up, and the need to do that quickly was paramount.With these two challenges in mind, the idea of programmable infrastructure spawned, and the idiom **_infrastructure as code_** was born. The OpenStack project is essentially that &#8211; programmable infrastructure. Services native to a datacenter make up the project, and each is designed with a fully featured API.

### **Why should I care?**

The drive behind virtualization was cost optimization and increased utilization of existing resources. The drive behind OpenStack is also cost optimization, but not of the traditional capital expenditure kind. OpenStack serves to reduce the amount of time your IT staff waits for data center resources. It also increases the flexibility and agility of your datacenter resources so that they may quickly be created and re-purposed in quite literally a matter of seconds. It enables your business to adjust incredibly quickly to the constant change we see in our industry. And lastly, it enables you to take advantage of new cloud capabilities in application and service architecture. Quickly scale up, scale down, integrate automation tooling, or create applications with higher resilience and additional capability, so that you are ready to adjust to the next business priority or opportunity.

### What&#8217;s the difference between virtualization and OpenStack?

[Others][3] have described the difference better than I could have, so I&#8217;ll link [their work][4]. But I&#8217;ll add this; traditionally, hardware was assigned to an application. Then hardware was assigned to virtual datacenters or clusters. If you wanted to re-purpose hypervisors, add storage, or provision networks you would incur a lengthy migration, balancing, or configuration task. And none of these services came with a capable API. OpenStack takes the virtualization concept of a software-defined OS component and blows it out of the water. Now every piece of infrastructure is software defined, so you can &#8220;spin up&#8221; much more than just a VM.

### How do I get started?

I thought you&#8217;d never ask! 😉 This is a loaded question with many correct answers to it, but we&#8217;ll want to start with sorting out a couple basics. Firstly, you need to decide the workload intent of your OpenStack environment. Specifically, I mean what type of application workloads do you expect to put there, and how many instances or VMs do you want in the environment. Secondly, you&#8217;ll want to decide the performance and redundancy you expect out of those workloads. Deciding on these two things will help give you a good idea of where to start for physical node count, physical node resources, and the node type or role each physical device should have. Also, this will help you decide what you should be using for an OpenStack installer. Since the environment architecture is a wildly variable conversation, I&#8217;m going to link some helpful resources to assist you in your sizing decisions. This decision is worth engaging an OpenStack provider in, depending on your intent of the environment. Some folks choose a small environment with a flagship app to deploy with it, others deploy general use OpenStack resources for mass consumption.

  * [Cloud Resource Calculator][5] &#8211; This is helpful in sizing your hardware with respect to your expected VM count and the size of your instance flavors.
  * [Deterministic capacity planning for OpenStack][6] &#8211; This is helpful to better understand the factors to consider when sizing an environment, and committing to a deployment.
  * [Sizing your OpenStack Cloud &#8211; OpenStack Summit][7] &#8211; Real world feedback video on the topic

&nbsp;

Next topic I will outline some of the options you have for installers, why you might pick one, and then I&#8217;ll choose an environment size to go through and do an example deploy. Have I missed anything important in the above topics? Got anything to add? I&#8217;d love to hear your experience.

 [1]: https://calgaryrhce.ca/wp-content/uploads/2016/08/OpenStack-logo.png
 [2]: https://calgaryrhce.ca/blog/2015/07/05/automating-server-provisioning-the-transition-to-private-hybrid-cloud/
 [3]: http://blog.rackspace.com/author/duan-van-der-westhuizen
 [4]: http://blog.rackspace.com/on-wind-farms-and-nuclear-power-the-differences-between-openstack-and-virtualization
 [5]: https://github.com/noslzzp/cloud-resource-calculator
 [6]: http://www.slideshare.net/SeanCohen/deterministic-capacity-planning-for-openstack-as-elastic-cloud-infrastructure
 [7]: https://www.openstack.org/summit/tokyo-2015/videos/presentation/sizing-your-openstack-cloud