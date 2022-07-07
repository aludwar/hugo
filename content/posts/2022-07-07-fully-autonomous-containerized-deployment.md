---
title: Fully Autonomous Containerized Deployment
author: Andrew Ludwar
type: post
date: 2022-07-07T11:36:33-06:00
categories:
  - cloud
  - containers
  - devops
  - open source
tags:
  - containers
  - fedora coreos
  - ignition
  - open source
---

After a pandemic hiatus, I'm back with some renewed vigor to blogging. Over the past two years, I've built up several topics that have been stuck in my head and need to get put down to (digital) paper. I think I'm finally at a spot with either spare sanity or needing a purposeful distraction that keeps me from doomscrolling the news. Either way, there should be some new and interesting content coming out of the backlog.

The first topic I'd like to cover is expanding on the changes and maturity we've seen over the last few years in container technology. We've seen A LOT of k8s content but not very much container OS content. Yes it's the attractive thing to learn these days, but k8s is complex! What if we have simpler needs or a desire to start somewhere that avoids adding unnecessary complexity?

What if we had a way to add some k8s orchestration and management like capbilities to containerized app deployment without all the k8s complexity? This is what I'm going to address in this first topic, divided into a multi-part series:

  1. Deploying a self-managing container OS based on Fedora CoreOS
      * Are you aware Fedora CoreOS can be configured to automatically update itself?
      * Did you know we can also setup intelligent health checks and automated rollbacks for any failed updates?
  2. Deploying a secure, self-healing and auto-updating containerized application with podman and systemd
      * Exploring rootless containers with Fedora CoreOS and podman
      * Have you seen systemd's capability to auto-start individual containers or a container pod?
      * What about systemd and podman's ability to update containers automatically?
  3. Adding a GitOps workflow for managing all of this with code
      * Integration with Ansible for automated initial deployment and future release management



First, let's start with a baseline configuration to deploy Fedora CoreOS that will serve as the foundation for this deployment. And if you're wondering, yes the [OS still matters][1]. (All applications, inclusive of containerized applications, [rely on the underlying kernel][2].)

Fedora CoreOS has matured quite a bit over the past 2-3 years since the move from Fedora Atomic. If you've read a [previous article][3] of mine you might already be familiar with the move from Fedora Atomic to Fedora CoreOS. I'll re-use some of what was built before but modified slightly to adapt to the new changes. Particularly around Butane tooling for producing an Ingition configuration file. These technology pieces allow us to define a configuration artifact that Fedora CoreOS will use at deployment time to configure the OS. We can store all OS customization in this configuration for easy re-deployment or scaling up of the OS.

After [downloading the Fedora CoreOS qcow2 image][4], I've built a YAML formatted Butane file and shell script to deploy locally to libvirt. After we get the foundational concepts understood here, I'll later adapt this deployment for cloud in AWS.

The Butane file is YAML, so we'll declare all of our configuration pieces in this file. We'll convert it to an Ignition file, which is JSON formatted, for passing to libvirt at VM creation time. The [Fedora documentation][5] for Butane and Ignition is pretty good. For additional details on configuration examples, head over there. I've done some basic things for my file:

* Specified the root filesystem disk size of 10 GB (Needs to be at least 8 GB)
* Set the hostname, static IP networking with DNS details
* Configured two files that [setup automatic updating][6] of the OS


<pre class="">
variant: fcos
version: 1.4.0
storage:
  disks:
    - device: /dev/vda
      wipe_table: false
      partitions:
        - label: root
          number: 4
          size_mib: 10240
          resize: true
  files:
    - path: /etc/hostname
      mode: 0644
      contents:
        inline: coreos.ludwar.ca
    - path: /etc/NetworkManager/system-connections/enp1s0.nmconnection
      mode: 0600
      overwrite: true
      contents:
        inline: |
          [connection]
          type=ethernet
          interface-name=enp1s0

          [ipv4]
          method=manual
          addresses=192.168.100.50/24
          gateway=192.168.100.1
          dns=192.168.100.1
          dns-search=ludwar.ca
    - path: /etc/zincati/config.d/51-rollout-wariness.toml
      contents: 
        inline: |
          [identity]
          rollout_wariness = 0.5
    - path: /etc/zincati/config.d/55-updates-strategy.toml
      contents: 
        inline: |
          [updates]
          strategy = "periodic"
          [[updates.periodic.window]]
          days = [ "Sat", "Sun" ]
          start_time = "22:30"
          length_minutes = 60
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - "[ public SSH key hash ]"
    - name: aludwar
      password_hash: "[ password hash, salted"
      ssh_authorized_keys:
        - "[ publis SSH key hash ]"
      groups: [ sudo, docker ]
</pre>


From there, we convert the Butane file into an Ignition file (you may need to install the Butane utility on your machine, or just pull the container for it):

<pre class="">$ butane --pretty --strict fedora.coreos.bu > fedora.coreos.ign</pre>

And lastly, a virt-install script that will pass the Ignition configuration, Fedora CoreOS qcow2 image, and other parameters to the VM for creation:

<pre class="">$ cat create_coreos_vm.sh
IGNITION_CONFIG="/home/aludwar/fedora.coreos.ign"
IMAGE="/home/aludwar/fedora-coreos-36.20220618.3.1-qemu.x86_64.qcow2"
VM_NAME="coreos.ludwar.ca"
VCPUS="2"
RAM_MB="2048"
STREAM="stable"
DISK_GB="15"

virt-install --connect="qemu:///system" --name="${VM_NAME}" --vcpus="${VCPUS}" --memory="${RAM_MB}" \
        --os-variant="fedora-coreos-$STREAM" --import --graphics=none \
        --disk="size=${DISK_GB},backing_store=${IMAGE}" \
        --network network=default \
        --qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=${IGNITION_CONFIG}"
</pre>


[1]: https://www.redhat.com/rhdc/managed-files/li-why-the-os-matters-ebook-f29731wg-202108-en.pdf
[2]: https://www.redhat.com/en/blog/architecting-containers-part-1-why-understanding-user-space-vs-kernel-space-matters
[3]: https://calgaryrhce.ca/blog/2020/02/24/moving-from-fedora-atomic-to-fedora-coreos/
[4]: https://getfedora.org/en/coreos/download?tab=metal_virtualized&stream=stable&arch=x86_64
[5]: https://docs.fedoraproject.org/en-US/fedora-coreos/getting-started/
[6]: https://docs.fedoraproject.org/en-US/fedora-coreos/auto-updates/
