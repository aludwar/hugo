---
title: "Introduction to Automation Metrics"
author: Andrew Ludwar
date: 2023-09-21T15:28:04-06:00
type: post
categories:
  - devops
  - open source
tags:
  - automation
  - ansible
  - open source
---
Another topic that's come up a lot over the past year or more has been around metrics. I'm sure by now you've heard of the [Four Keys][1] developed by Google. Now, these metrics get a **lot** of coverage in the spotlight, but should everyone use them? Not everyone is at Google's level. More commonly it sparks a conversation around:

* *"what is a good metric?"* 
* *"what makes up a good metric?"*
* *"what should I focus on when deciding on my own automation/devops metric(s)?"*
* *"what are the best practices for metrics?"*

Now, there's already lots of coverage around this topic so I'm not going to reinvent the wheel. Rather the points I'll suggest for those starting an automation/devops journey are:

* Don't spend a massive amount of time trying to come up with the perfect metric
* Your metrics are going to evolve as you evolve, so it's more important to start somewhere than it is to start perfect
* Begin using a metric that relates to your individual organizations strategic goals

Your metric(s) should be representative of tracking a value item or showcasing an outcome that's meaningful to you, your team, and your organization. Common metric categories are often related to:

* Adoption: 
    - Metrics related to progress: precentage of devices under management, number of use-cases delivered, percentage of common processes that have had automation aligned to it
* Evangelism:
    - Metrics related to onboarding: number of users/teams brought into the practice, workshops/training sessions completed, percentage of internal courses completed
* Outcomes:
    - Metrics related to deliverables: number of self-service IT requests completed, percentage of new projects leveraging the new methods, contrasting time savings comparing old methods to new and number of times initiated
* Value:
    - Metrics related to monetary impact: FTE hours saved, ratio of sysadmins to infrastructure size, response times to incidents (MTTR)


All of these are off the top of my head without searching while I write this. I'm sure in a room full of brainstorming people there would be a number much greater. The important piece is to pick a few that in aggregate will help tell the story that you want to tell. Also important is not using metrics as a crutch as the only way value is being understood and measured. These metrics are complimentary to your story, not the whole story. And the story is likely to evolve, and mature, and change. This is a sign of healthy, iterative improvement.

With respect to Ansible, I wanted to show an example of gathering a first metric, absent of a formal observability tool. Where the best place is to get your metrics from is another conversation altogether, but suffice to say there's likely going to be options and overlap and that's okay as well. 

I'll start with the first one mentioned, the percentage of devices under management. As one begins an automation practice, there's often an initial onboarding phase where we're connecting to all devices in our infrastructure, sorting through access patterns, networking, authentication, etc. which can take some time to sort though. Especially if other groups have responsibility over other devices and may need convincing to allow them to be onboarded. 

In this example, I show a basic method of performing a connectivity and authentication test against three kinds of devices:

1. Linux Servers
2. Secure Servers
3. Windows Servers

The reason for separating these out this way is you may have different authentication methods to them, or different infrastructure environments that they live in. You also may have thousands of them and want to structure or throttle the connection and reporting on them in different ways. Here's what I did:

* Leveraged the basic ping module for each device type:
    - [ansible.builtin.ping][2] and [ansible.windows.win_ping][3]
    - this module performs this basic connectivity and authentication test out of the box, keeping my playbook complexity very low
* Leveraged two [Ansible magic variables][4]:
    - ansible_play_hosts: Lists the hosts in the current play run, excluding failed/unreachable hosts. So basically a measure of successful hosts in a play
    - ansible_play_hosts_all: Lists all of the hosts targeted in the play
    - These two special variables made it easy to discover and store in a fact, and perform basic math on to get a % of successfully reachable hosts
* Leveraged a jinja2 template to write a connecvity report file
* Leveraged the [ansible.builtin.blockinfile][5] module to append to the report file


Each device type got it's own playbook and jinja template, and I write the file to an NFS server, here's how the resulting report looks:

    Hosts Under Management Metric(s):
    
    Reachable Linux Hosts: 7 / 9  (77.78%)
    
    Reachable Secure Hosts: 2 / 2  (100.0%)
    
    Reachable Windows Hosts: 2 / 2  (100.0%)


Now this is pretty basic, but again it's an example of how to quickly populate a kind of metric that can help you reinforce the value of an automation practice. This result could easily be expanded to publishing the results into a webservice, time series database, or a README file in a git repo potentially accompanied by an auto-updating badge.

Here's my github repo with all the code examples: 

[https://github.com/aludwar/ansible/tree/master/metrics][6]


Happy automating!


[1]: https://github.com/dora-team/fourkeys
[2]: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/ping_module.html
[3]: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_ping_module.html
[4]: https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html
[5]: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/blockinfile_module.html
[6]: https://github.com/aludwar/ansible/tree/master/metrics
