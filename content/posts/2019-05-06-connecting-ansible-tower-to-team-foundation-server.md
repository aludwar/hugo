---
title: Connecting Ansible Tower to Team Foundation Server
author: Andrew Ludwar
type: post
date: 2019-05-06T18:35:32+00:00
url: /blog/2019/05/06/connecting-ansible-tower-to-team-foundation-server/
xyz_smap:
  - 1
categories:
  - devops
  - open source
tags:
  - ansible
  - configuration management
  - devops
  - open source

---
Recently I was investigating how to connect Ansible Tower to Microsoft Team Foundation Server to source a git repo. To document this for later, I discovered that the way to do this is with an SSH key, and by turning on the SSH service on TFS. Here&#8217;s how I got it working:

<https://docs.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops>

<https://docs.microsoft.com/en-us/azure/devops/repos/git/clone?view=azure-devops&tabs=visual-studio>

<https://github.com/ansible/ansible/issues/39858>

&nbsp;