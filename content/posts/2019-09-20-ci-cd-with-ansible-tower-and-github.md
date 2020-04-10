---
title: CI/CD with Ansible Tower and GitHub
author: Andrew Ludwar
type: post
date: 2019-09-20T15:54:42+00:00
url: /blog/2019/09/20/ci-cd-with-ansible-tower-and-github/
xyz_smap_tw_future_to_publish:
  - 'a:3:{s:18:"post_tw_permission";s:1:"0";s:19:"xyz_tw_img_permissn";s:1:"1";s:14:"xyz_tw_message";s:26:"{POST_TITLE} - {PERMALINK}";}'
xyz_smap:
  - 1
categories:
  - automation
  - cloud
  - devops
  - open source
tags:
  - ansible
  - Ansible Tower
  - CICD
  - github

---
Every so often I come across an interesting use-case or a really creative way to integrate different technology that I haven&#8217;t seen before. I follow a few folks on social media and blogs that do this stuff on a regular basis, one of them being Keith Tenzer. He&#8217;s put together a really interesting [demo of Ansible Tower][1] connected with GitHub and a webhook that showcases how CI/CD can be accomplished using Ansible without the need for a formal CI/CD pipeline toolset like Jenkins. He sparked an idea for me to put together a similar environment as I think this makes for a great demo and showcase of what Ansible Tower is capable of.

I&#8217;ve adapted his work to use Azure instead of OpenStack, but the concepts remain the same. He&#8217;s got some detail in his blog about how to setup the environment and connect services together, so I&#8217;ll let his work there speak for itself on how to get it setup. The changes I&#8217;ve made can be found in a couple of playbooks that provision infrastructure in Azure:

  * <https://github.com/aludwar/ansible/tree/master/azure> 
      * create-app-vm.yml
      * deploy-app.yml
      * teardown-app.yml

Then I use a webservice called [ultrahook.com][2] to bridge the GitHub events to my homelab of Ansible Tower. You may not need this if your Ansible Tower server lives in the cloud. The high level architecture and workflow looks like this:

[<img class="size-large wp-image-822" src="https://calgaryrhce.ca/wp-content/uploads/2019/09/TowerCICD-1024x570.png" alt="" width="1024" height="570" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/09/TowerCICD-1024x570.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/09/TowerCICD-300x167.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/09/TowerCICD-768x427.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/09/TowerCICD.png 1361w" sizes="(max-width: 1024px) 100vw, 1024px" />][3]

Lastly, I&#8217;ve [recorded a demo of what this looks like][4]. Similar to Keith, showcasing the developer workflow of checking in code, Tower building infrastructure and running the unit testing, then finally updating GitHub with the result. Then highlighting what a maintainer might see when they&#8217;re doing to merge the new code into the project.

 [1]: https://keithtenzer.com/2019/06/24/ci-cd-with-ansible-tower-and-github/
 [2]: http://ultrahook.com
 [3]: https://calgaryrhce.ca/wp-content/uploads/2019/09/TowerCICD.png
 [4]: https://www.youtube.com/watch?v=_AumRi6GTUM