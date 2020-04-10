---
title: Demystifying automation and Puppet
author: Andrew Ludwar
type: post
date: 2015-07-05T19:46:06+00:00
url: /blog/2015/07/05/demystifying-automation-and-puppet/
xyz_smap:
  - 1
dsq_thread_id:
  - 5506717602
categories:
  - devops
  - enterprise
  - open source
tags:
  - automation
  - private hybric cloud
  - puppet
  - RHUG

---
Automation has grown from its traditional roots of simple scripting into a critical cog of present day IT infrastructure. Today we have full fledged software suites based on automation. We have myriads of start-ups who base their entire business model on it. Every CIO is including it in their execution strategy, and you can&#8217;t seem to find a conference that doesn&#8217;t have a trendy buzzword filled keynote dedicated to the topic. Oh how far we&#8217;ve come! What IT person or company isn&#8217;t looking at automation these days?

But with all the trendiness and hype comes a lot of opinion and confusion. Marketing and business development departments muddy the technical details. Software developers consume it differently than sys-admins. Automation requires more business process change than technology change, management wants it implemented yesterday, and the technology feature-set is evolving at a massive rate. How&#8217;s one expected to keep up with it all? Hopefully this post can help you do that. (I realise the hypocrisy in presenting an opinion on other opinions 😉 )

One of the leaders in the IT automation market is Puppet Labs and their configuration management language, Puppet. I&#8217;ve put together a slide-deck and most recently presented at our local Red Hat User Group meeting ([meetup.com][1], [LinkedIn][2]) on the subject. Specifically, _&#8220;What is Puppet? And challenges/gotchas experienced with real-world implementations&#8221;_. I&#8217;ll take you through where I believe Puppet fits in in the enterprise, what it will and will not do for you, and some things to consider when adopting it yourself. This knowledge has been gathered from my past experience adopting it for previous and current employers. Included in this presentation is a slide on making the transition from traditional IT to private cloud server provisioning. I&#8217;ll dig into this deeper in another post.

For now, <a href="http://calgaryrhce.ca/wp-content/uploads/2015/07/RHUG-Puppet-Path-To-Hybrid-Cloud.pdf" target="_blank">here you go</a>. I&#8217;d welcome any feedback or comments you may have!<figure id="attachment_202" aria-describedby="caption-attachment-202" style="width: 436px" class="wp-caption alignnone">

[<img class=" wp-image-202" src="http://calgaryrhce.ca/wp-content/uploads/2015/07/RHUG-TitleSlide.001-1024x768.jpg" alt="RHUG Puppet Path To Private Hybrid Cloud" width="436" height="327" srcset="https://calgaryrhce.ca/wp-content/uploads/2015/07/RHUG-TitleSlide.001.jpg 1024w, https://calgaryrhce.ca/wp-content/uploads/2015/07/RHUG-TitleSlide.001-300x225.jpg 300w" sizes="(max-width: 436px) 100vw, 436px" />][3]<figcaption id="caption-attachment-202" class="wp-caption-text">RHUG Puppet Path To Private Hybrid Cloud</figcaption></figure>

 [1]: http://www.meetup.com/Calgary-Red-Hat-Users-Group-RHUG-Meetup/
 [2]: https://www.linkedin.com/grp/home?gid=6591615
 [3]: http://calgaryrhce.ca/wp-content/uploads/2015/07/RHUG-Puppet-Path-To-Hybrid-Cloud.pdf