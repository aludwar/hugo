---
title: "Ansible Automation Platform Notifications for package reporting"
date: 2022-10-26T13:18:38-06:00
type: post
categories:
  - devops
  - open source
tags:
  - automation
  - ansible
  - open source
---

I was recently asked to outline the options available to do package reporting after a patch cycle for RHEL. There was preference to try to do this in Ansible Automation Platform, consumed via an emailed report, so I'll focus the bulk of the effort there. The specifics were to have view into what packages were installed, updated, removed, etc. as part of a patch cycle. From there, the intent is to list this out and respond to anything that may have failed or done something unexpected during the update. After outlining this I realised that this is a really common question that didn't have much blog coverage on it. So here we go!


There's a couple of ways to get this info depending on the granularity you're after and the problem you're trying to solve for. I'll order them in what I see folks most commonly do, that's also the least amount of re-work.


1. Red Hat Satellite has errata reporting already built-in that will go to this granularity level of reporting packages applied, their status, on a patch cycle. It's located in:

* Monitor --> Report Templates --> Host - Applied Errata

Usually folks will clone the default report template provided here, narrow down the fields/columns that they're after, apply a host filter, etc.  Satellite can mail this to you on a schedule.

If you want to auto-generate the report every time a patch cycle is run, the best way to automate that is with the [Hammer CLI][1].


2. Ansible Tower / Ansible Controller has email notifications built-in and with its idempotency already tracked in job output, you don't have to do any extra playbook work necessarily, unless you want some extra metadata in your email body that Ansible Tower/Controller doesn't provide. Sending all this info can get pretty verbose with a lot of hosts & package updates, I find folks don't find email as the best way to consume this information, Tower/Controller GUI Job status output is more nicely formatted and already built for you, but to each their own.

Customers typically tend to default to "only show me actionable stuff" in any Ansible notification, so if for example a patch run that didn't update something is shown instead of all the things that did update successfully.

Here's a [playbook example][2] that I mocked up and used below. The job output:

[<img class="size-large wp-image-822" src="https://calgaryrhce.ca/wp-content/uploads/2022/10/note1.png" alt="" width="1024" height="570" srcset="https://calgaryrhce.ca/wp-content/uploads/2022/10/note1.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2022/10/note1.png 300w, https://calgaryrhce.ca/wp-content/uploads/2022/10/note1.png 768w, https://calgaryrhce.ca/wp-content/uploads/2022/10/note1.png 1361w" sizes="(max-width: 1024px) 100vw, 1024px" />][3]

(You can see that this could get pretty verbose if we wanted to report across multiple hosts)


Next, I created an email Notification that's associated with that Job, the notification parameters can be customized, but as I mentioned above the data available here is more specific to the job than the content results from tasks & hosts.

Ref: [Ansible Notifications Docs][4]

Ref: [Ansible Notifications Supported Parameters][5]

In the success message body below, I configure job metadata that summarizes all hosts and their results, then one that just shows job status counts, ie what changed/failed and folks can click on the job ID link to Tower GUI to dig deeper. I'm using my ISP SMTP relay here:


[<img class="size-large wp-image-822" src="https://calgaryrhce.ca/wp-content/uploads/2022/10/note2.png" alt="" width="1526" height="570" srcset="https://calgaryrhce.ca/wp-content/uploads/2022/10/note2.png 1526w, https://calgaryrhce.ca/wp-content/uploads/2022/10/note2.png 570w, https://calgaryrhce.ca/wp-content/uploads/2022/10/note2.png 768w, https://calgaryrhce.ca/wp-content/uploads/2022/10/note2.png 1361w" sizes="(max-width: 1526px) 100vw, 1526px" />][6]

And here's what it looks like once it shows up:

[<img class="size-large wp-image-822" src="https://calgaryrhce.ca/wp-content/uploads/2022/10/note2.png" alt="" width="1526" height="570" srcset="https://calgaryrhce.ca/wp-content/uploads/2022/10/note2.png 1526w, https://calgaryrhce.ca/wp-content/uploads/2022/10/note2.png 570w, https://calgaryrhce.ca/wp-content/uploads/2022/10/note2.png 768w, https://calgaryrhce.ca/wp-content/uploads/2022/10/note2.png 1361w" sizes="(max-width: 1526px) 100vw, 1526px" />][7]


If you prefer different information here, using the community.general.mail module in a task can get you there.


3. Red Hat Insights already does all of the above here for you, auditing all systems, tracking errata, identifying vulnerabilities/CVEs that are present on systems, drift analysis, notifications & webhooks, and high-level executive reporting. There's also an API to do some of these pieces in a programmatic fashion.

For as granular as an email/report as we're after here though, it might be quicker to use one of the above options. Insights does not yet have customized reporting like Satellite does, it's still in RFE status. And it's a little more dashboard-y/GUI driven answering more high-level questions than what we're afterhere.



[1]: https://access.redhat.com/documentation/en-us/red_hat_satellite/6.11/html-single/managing_hosts/index#Generating_Host_Monitoring_Reports_managing-hosts
[2]: https://github.com/aludwar/ansible/blob/master/dnf-report.yml
[3]: https://calgaryrhce.ca/wp-content/uploads/2022/10/note1.png
[4]: https://docs.ansible.com/automation-controller/latest/html/userguide/notifications.html
[5]: https://docs.ansible.com/automation-controller/latest/html/userguide/notification_parameters_supported.html#ir-notifications-reference
[6]: https://calgaryrhce.ca/wp-content/uploads/2022/10/note2.png
[7]: https://calgaryrhce.ca/wp-content/uploads/2022/10/note3.png
