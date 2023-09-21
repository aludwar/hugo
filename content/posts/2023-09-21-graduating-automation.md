---
title: "Graduating with Automation Practices"
Author: Andrew Ludwar
date: 2023-09-21T09:20:21-06:00
type: post
categories:
  - devops
  - open source
tags:
  - automation
  - ansible
  - open source
---
Lately I've been engaged in a lot of automation discussions. Particularly around governance, standards, scale, workflows, best practices, and the list goes on. My impression is that folks are looking at establishing some of the common foundations to an automation practice and are getting farther down their DevOps & Automation journeys. Often we don't get deep into sorting through these details until some operational concern or pain is felt with the current approach - and that's fine. These are called journeys for a reason and the most important piece is to remember that they're iterative and subject to continual refinement.

One of the common topics of discussion is inventory and host reconcilation. Essentially trying to answer:

* *"How do I best structure what I've got?"*
* *"What's the right approach to graduate from mostly static inventory and host management into something very dynamic and large scale?"*

These approaches are very similar to executive goal planning and execution. Typically we need three things:

1. We need to know where we are (the current state)
2. We need to know where we want to go (the ideal state)
3. We need to define the path to walk to get us from #1 to #2 (the approach/execution plan)

There are challenges with each. For knowing your current state, the largest challenge I see is folks being objective and/or measured about where they are. Essentially, are we being honest with ourselves and our assessment of where we're at? How are we coming to this answer? It's common to overestimate where we are, especially when the maturity around metrics and observability is low. We're optimists after all! We often look on the bright side of life and this trickles into our self-awareness. 

Knowing the ideal state is typically the easiest, we've been flirting with ideas to improve things for a long time. We've read books, articles, gone to conferences, know the latest buzzwords and generally there's little disagreement internally on what the ideal looks like. After all, we've been doing the "if you could wave the magic wand" exercise internally in our heads everytime we run into friction with our day-to-day.

The last part is obviously the hardest part. The execution of digital transformation initiatives in industry does not have a good track record - for a variety of reasons. One of the prevailing fixes is to break these steps down into smaller chunks and introduce more agility into review of the strategy. The same is true for automation practices. It's less important to get the first decision perfect, it's most important just to start somewhere with the expectation that you'll learn and refine the approach as you go.

So where does this leave us? How does this help us answer the above questions and better define our initial approach?

Red Hat has an internal automation community of practice. We publish our "good practices" publicly for everyone to read and benefit from. These practices get defined from our experience helping customers in the field, and these practices get updated and refined as we learn ourselves. One of the good practices regarding inventory and host management is around defining structured inventory: "[Define your inventory as structured directory instead of single file][1]". So this helps us define the ideal state, great! What about the approach to get here?

In case you haven't guessed or noticed the theme here, the answer is it's going to be an iterative approach. There's two common techniques that can help you get started:

**1. Inventory Exporting/Reporting w/ [Red Hat Community of Practice controller_configuration Ansible collection][2]**

    This export tool can help you export your current AAP inventory into a text format like JSON where you can begin to parse it with whatever method you prefer to reconcile similar hosts, groups, types, functions, etc. and start to make some decisions around intelligent grouping of hosts. If you're familiar with data normalization, this is exactly the same process. Common grouping parameters are geographical location, technical function, business function, operating system derivatives (linux or windows, major version, etc.).

**2. Inventory Host Variable Reporting**

    There's a variety of methods available to do this, I've written an [example playbook][3] that illustrates how this can be done. Using a report like this can help you group together common configuration artifacts like NTP server, DNS information, device services or purpose, etc. Using this information combined with the above helps you decide what commonality may exist across hosts. This can help you better define what should be handled in a host_var, group_var, role or default var, etc. Putting this in a text format helps you build a directory folder scaffolding structure for these common configuration artifacts and logic. A resource that can help in this area is the Ansible Variable Precedence order. Using this as a reference, you can decide where host variable classification should be, what might make sense as a sane default with other variable options that have an option to supersede it. For example:

![Ansible Variables Precedence](https://calgaryrhce.ca/wp-content/uploads/2023/09/ansiblevariableprecedence.png)

* You may have NTP or DNS servers specific to a geographical region. These may make the most sense to include in the inventory group_vars. However, there may be some subregions like a DMZ that may have a different entry for this that is better placed in an inventory host_vars level.
* You may have common configurations specific to RHEL7, RHEL8, RHEL9, or Windows Server 2016, 2019, 2022, etc. that might be appropriate for a role default value.
* A particular group of server infrastructure for a specific application might be better placed in a playbook group_vars or host_vars location and that more customized inheritance structure could be applied to other applications or groups.


I hope this added some insight into structuring inventory reconcilation and assist with being iterative in your automation practice.





[1]: https://redhat-cop.github.io/automation-good-practices/#_define_your_inventory_as_structured_directory_instead_of_single_file
[2]: https://github.com/redhat-cop/controller_configuration/blob/devel/EXPORT_README.md
[3]: https://github.com/aludwar/ansible/blob/master/metrics/export_host_variables.yml
