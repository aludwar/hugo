---
title: Managing Configuration Management w/ Puppet
author: Andrew Ludwar
type: post
date: 2014-11-23T21:39:55+00:00
url: /blog/2014/11/23/managing-configuration-management-w-puppet/
categories:
  - devops
  - open source
tags:
  - configuration management
  - devops
  - puppet

---
Over the course of the past year or so, Puppet has taken off in popularity amongst developers and administrators. It&#8217;s becoming the de-facto tool for repeatable state and drift management, and part of the reason for the uptake in popularity of the DevOps culture. As with most techology that grows quickly and organically, it often looks much different a year into implementation than it did day one.  Folks come away with many lessons learned and new ideas that they would implement if they had the chance to do it all over again.

I&#8217;ve been fortunate to be part of two puppet implementations so far in my career and thankfully have been working alongside some intelligent and insightful folks in the process. In one implementation in particular, we started a Puppet Task Force that served to educate the many developers who were operating in silos with the intent of helping them integrate their code and development structure with other teams. No easy task. I&#8217;ve come across this challenge several times now and continously find myself leaning on this one puppet article & slidedeck by Craig Dunn to help us all understand how to architect puppet from a high-level view. It&#8217;s been very well received each time I share it, and its content is very insightful and useful.

Craig does a much better job articulating the concepts than I do, so I&#8217;ll just link you to him. If you&#8217;re struggling with managing your puppet modules either in sheer size, overlap, or integration with other developers, I urge you to take a look at Craig&#8217;s work.

[Designing Puppet &#8211; Roles and Profiles
  
][1] [Designing Puppet &#8211; Roles/Profiles Pattern (slidedeck)][2]

 [1]: http://www.craigdunn.org/2012/05/239/
 [2]: http://www.slideshare.net/PuppetLabs/roles-talk