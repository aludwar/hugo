---
title: Ansible-pull and kickstart, for one-touch server provisioning
author: Andrew Ludwar
type: post
date: 2016-02-03T20:55:56+00:00
url: /blog/2016/02/03/ansible-pull-and-kickstart-for-one-touch-server-provisioning/
xyz_smap:
  - 1
dsq_thread_id:
  - 5075920016
categories:
  - devops
  - open source
tags:
  - configuration management
  - devops
  - kickstart
  - server provisioning

---
Recently I&#8217;ve been learning and using Ansible as my configuration management tool. It came recommended by several colleagues, recently had an [O&#8217;Reilly book][1] published, and went through an [aquisition][2]. Safe to say its momentum and adoption is on a high&#8230; and so far, I&#8217;m loving using it. I find it vastly easier to setup and use than Puppet, Chef, CFengine, or SaltStack. I bought the O&#8217;Reilly book, and had a playbook configuring a server in about 10 minutes&#8230; udderly quick. (Sorry, bad pun if you looked at the book cover). Most of my config management experience is with CFengine and Puppet, yet I found some interesting contrasts so far:

  * I found Ansible far easier to consume and get running right away. Literally after 10 minutes I had a basic playbook pushing /etc/motd to my server. There was no setup of a Puppet master or CFengine master server required, and I didn&#8217;t need to download any modules. It uses already encrypted SSH for traffic, so no SSL work was required either.
  * The syntax reads very easy, and the learning curve to understanding and writing a module (or a play in this case) was very short. I remember spending a few days with Puppet and CFengine before I surface-level understood modules and classes. Ansible was 10 minutes. I&#8217;ll use a phrase coined by a colleague, and one that also happens to be in the book&#8230;. Ansible reads like executable documentation.
  * No agent required, everything runs over SSH, so that alleviates management and maintenance of a local agent.
  * A formal Ansible server wasn&#8217;t required, although a remote client to be managed needs to talk to something that has the Ansible code. (This **could** be a formal server, or alternatively a git repo, or any server with repo access. **How** the client gets its code seems to be very flexible)

Pretty impressive so far. Then I started to think&#8230; well running a play or a series of plays (called a playbook &#8211; which are essentially configuration tasks or a group of tasks) looks to be a push methodology, and performed as a one-off task. What about drift management? (Puppet and CFengine excel at this). And how can I get a VM created, bootstrapped, and configured by ansible in a one-touch provisioning method?

Enter ansible-pull. Just like it reads, instead of initiating a push from the server to the remote client, a new client can request its config with ansible-pull. In fact, if you load-balance your config source (server, repo) this can theoretically [scale infinitely][3]. So, I investigated how to set this up, and this is what I came up with. Ansible-pull initiated by kickstart, a git repo with my ansible code, and a systemd unit file to turn the first-boot ansible-pull into a service that automatically configures my server, I just need to enable it with systemctl.

First, you&#8217;ll need some ansible code, and then a place to put it (Creating code is outside the scope of this post, but I&#8217;ll share mine for reference). I&#8217;ve got a gitlab server at home, so I&#8217;ve synced my code into a new repository named &#8220;ansible&#8221;. Having your code at the root of the repo helps make it easy for ansible-pull.<figure id="attachment_290" aria-describedby="caption-attachment-290" style="width: 993px" class="wp-caption alignnone">

[<img class=" wp-image-290" src="https://calgaryrhce.ca/wp-content/uploads/2016/02/gitlab-screenshot-1024x405.png" alt="GitLab Screenshot" width="993" height="393" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/02/gitlab-screenshot-1024x405.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2016/02/gitlab-screenshot-300x119.png 300w, https://calgaryrhce.ca/wp-content/uploads/2016/02/gitlab-screenshot.png 1868w" sizes="(max-width: 993px) 100vw, 993px" />][4]<figcaption id="caption-attachment-290" class="wp-caption-text">GitLab Screenshot</figcaption></figure> 

Ansible has a concept of an inventory file, so what I&#8217;ve done here is copied my site.yml file into localhost.localdomain.yml as I&#8217;ve told ansible-pull to look for that. In my home lab, VMs come up unnamed to begin with, then once they&#8217;re networked and configured I rename them into a more permanent home. This can easily be changed, but this is what I&#8217;ve done thus far. To make it easy to fetch the code, in GitLab I&#8217;ve created the user ansible, uploaded a public SSH key, and given the user access rights to the repository. Then, my CentOS 7 kickstart file %post section looks like this:

<pre class="lang:python decode:true ">%post --log=/root/ks-post.log
yum install epel-release -y
yum install ansible git -y
yum update -y

useradd -p '&lt;hashed and salted ansible user password&gt;' ansible
echo "ansible ALL=(root) NOPASSWD:ALL" | tee -a /etc/sudoers.d/ansible
echo "Defaults:ansible !requiretty" | tee -a /etc/sudoers.d/ansible
chmod 0440 /etc/sudoers.d/ansible
wget http://&lt;ks-server&gt;/ansible.tar && tar -xvf ansible.tar -C /home/ansible
echo localhost.localdomain &gt;&gt; /home/ansible/hosts
chown -R ansible:ansible /home/ansible/.ssh

wget http://&lt;ks-server&gt;/ansible-config-me.sh -O /usr/local/bin/ansible-config-me.sh
wget http://&lt;ks-server&gt;/ansible-config-me.service -O /etc/systemd/system/ansible-config-me.service
chmod 0755 /usr/local/bin/ansible-config-me.sh
chmod 0644 /etc/systemd/system/ansible-config-me.service

systemctl daemon-reload
systemctl enable ansible-config-me.service</pre>

Above:

  * I make sure ansible and git are installed and the latest versions with the yum commands
  * I add the user, and configure it to use sudo without a password and tty
  * Then I unroll it&#8217;s .ssh/ directory w/ SSH keys and pre-populated known_hosts file, and also populate the ansible inventory file
  * Then I grab the script I&#8217;ve created that runs ansible-pull, and will be the executable of my systemd unit file
  * I grab the custom systemd unit file
  * Do a daemon-reload of systemd to trigger reading in my new unit file
  * Enable the new ansible service I&#8217;ve created

Now once kickstart is completed, the ansible-pull script will get run via systemd auto-starting the ansible-config-me service. My unit file and script look like this:

<pre>ansible-config-me.service:
[Unit]
Description=Run ansible-pull at first boot to apply environment configuration
After=network.target

[Service]
ExecStart=/usr/local/bin/ansible-config-me.sh
Type=oneshot

[Install]
WantedBy=multi-user.target</pre>

In the [Service] section, Type=oneshot tells systemd that the process will be short-lived. This is is useful for for scripts that do a single job, and then exit.

<pre>ansible-config-me.sh:
#!/bin/bash

runuser -l ansible -c 'ansible-pull -C master -d /home/ansible/deploy -i /home/ansible/hosts -U git@gitlab.ludwar.ca:aludwar/ansible.git --key-file /home/ansible/.ssh/id_rsa --accept-host-key --purge &gt;&gt; /home/ansible/run.log'

systemctl disable ansible-config-me.service</pre>

To take advantage of the pre-populated SSH keys and access of the ansible user, I run ansible-pull as user ansible, and tell it to pull the master branch, download the code and run it from the deploy directory, use the hosts inventory file which contains the new hosts&#8217; hostname,  give it my git repo, SSH key, and then set the purge flag to delete the local deploy directory and code within it after it&#8217;s done. Since I also just want this script run once, I disable the service in systemd.

There you have it! A one-touch provisioning process. And as for the drift management piece, I read most folks are cron&#8217;ing this ansible-pull. However, an official drift management feature is coming in the near-future for Ansible Tower.

Oh, and for interest sake, this is the ansible playbook task I&#8217;m running:

<pre># This playbook contains common plays that will be run on all nodes.

- name: Install chrony
 yum: name=chrony state=present
 tags: chrony

- name: Configure chrony.conf file and restart
 template: src=chrony.conf.j2 dest=/etc/chrony.conf
 tags: chrony
 notify: restart chrony

- name: Enable chrony at boot
 service: name=chronyd state=started enabled=yes
 tags: chrony

- name: Update motd
 template: src=etc.motd dest=/etc/motd
 tags: motd

- name: Add --long-hostname to getty
 lineinfile: dest=/etc/systemd/system/getty.target.wants/getty@tty1.service regexp="^ExecStart=" line="ExecStart=-/sbin/agetty --long-hostname --noclear %I $TERM" state=present

- name: Deploy hardened SSH client config file (/etc/ssh/ssh_config)
 template: src=etc.ssh.ssh_config dest=/etc/ssh/ssh_config
 tags: ssh
 notify: restart sshd

- name: Deploy hardened SSH server config file (/etc/ssh/sshd_config)
 template: src=etc.ssh.sshd_config dest=/etc/ssh/sshd_config
 tags: ssh
 notify: restart sshd

- name: Add local user aludwar, enable sudo, unroll home directory - Load/copy script
 template: src=set-aludwar.sh dest=/tmp/set-aludwar.sh mode="u+rwx"

- name: Add local user aludwar, enable sudo, unroll home directory - Run script
 command: bash /tmp/set-aludwar.sh

- name: Flush and harden firewall(iptables) rules - Load/copy script
 template: src=set-iptables.sh dest=/tmp/set-iptables.sh mode="u+rwx"

- name: Flush and harden firewall(iptables) rules - Run script
 command: bash /tmp/set-iptables.sh</pre>

&nbsp;

Additional items to work on for an even sleeker provisioning process:

  * Tweaking my virsh-install script to provide a hostname, and have the OS configure it with ansible
  * Finding an appropriately safe, yet public place to pull this ansible code from, so I can use it with cloud providers
  * Removing the local ansible user and using LDAP instead for increased security and control

 [1]: http://www.amazon.ca/Ansible-Up-Running-Lorin-Hochstein/dp/1491915323
 [2]: https://www.redhat.com/en/about/press-releases/red-hat-acquire-it-automation-and-devops-leader-ansible
 [3]: http://docs.ansible.com/ansible/playbooks_intro.html#ansible-pull
 [4]: https://calgaryrhce.ca/wp-content/uploads/2016/02/gitlab-screenshot.png