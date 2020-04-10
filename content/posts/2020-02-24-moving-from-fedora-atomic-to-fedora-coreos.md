---
title: Moving from Fedora Atomic to Fedora CoreOS
author: Andrew Ludwar
type: post
date: 2020-02-24T23:52:59+00:00
url: /blog/2020/02/24/moving-from-fedora-atomic-to-fedora-coreos/
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
Recently the merging of [Fedora Atomic][1] and [CoreOS Container Linux][2] into what is now known as [Fedora CoreOS][3] has [moved out of preview][4] and is ready for prime time use. I&#8217;ve been running a few small container apps on a Fedora Atomic 28 VM, which is now ripe to be moved into the new replacement Fedora CoreOS. Here&#8217;s how I did this.

I&#8217;ll be deploying on a KVM environment, so I [downloaded the new QCOW2][5] image (You can also get an AMI for AWS). The Fedora documentation has instructions on how to [migrate from Fedora Atomic Host][6], so this is what I followed. Essentially we need to move from using cloud-init and cloud-config scripts to ignition which uses a JSON formatted config to provision a Fedora CoreOS system.

First we create a YAML formatted configuration file, which we&#8217;ll then convert to JSON with the `fcct` tool. I only have some basic configuration items I need on my CoreOS system:

  * Static IP, with hostname & DNS details
  * A couple of users and their passwords & SSH keys

Here is what my config file looks like. You can use the [FCCT spec docs][7] to add in config for storage, files, disks, etc.

<pre class="">$ cat coreos.fcc
variant: fcos
version: 1.0.0
storage:
  files:
    - path: /etc/NetworkManager/system-connections/eth0.nmconnection
      mode: 0600
      overwrite: true
      contents:
        inline: |
          [connection]
          type=ethernet
          interface-name=eth0

          [ipv4]
          method=manual
          addresses=10.0.0.20/24
          gateway=10.0.0.1
          dns=10.0.0.10;10.0.0.1
          dns-search=calgaryrhce.ca
    - path: /etc/hostname
      mode: 0600
      overwrite: true
      contents:
        inline: |
          coreos.calgaryrhce.ca
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - "[ public SSH key hash ]"
    - name: aludwar
      password_hash: "[ password hash, salted ]"
      ssh_authorized_keys:
        - "[ public SSH key hash ]"
      groups: [ sudo, docker ]</pre>

I don&#8217;t have the fcct tool local on my Fedora 30 workstation, so I&#8217;ll use the podman container to convert the YAML to JSON:

<pre class="">$ podman pull quay.io/coreos/fcct:release
$ podman run -i --rm quay.io/coreos/fcct:release --pretty --strict &lt; coreos.fcc &gt; coreos.ign</pre>

Then, with virt-install, I create a new VM referencing the QCOW2 image, and provide the coreos.ign file in the qemu command line parameter:

<pre class="">$ virt-install --connect qemu:///system -n coreos.calgaryrhce.ca --memory 1024 --vcpus 1 --network default --os-variant=fedora31 --import --graphics=none \
--disk=/home/aludwar/VMs/fedora-coreos-31-qemu.qcow2 --qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=/home/aludwar/VMs/coreos.ign"</pre>

At first boot, the VM will use DHCP for its address, but will also have the ignition configuration applied to the host. Reboot the host for the config to be applied:

<pre class="">...

[ 1.573639] systemd[1]: systemd v243.5-1.fc31 running in system mode. (+PAM +AUDIT +SELINUX +IMA -APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN2 -IDN +PCRE2 default-h)
[ 1.576925] systemd[1]: Detected virtualization kvm.
[ 1.577759] systemd[1]: Detected architecture x86-64.
[ 1.578555] systemd[1]: Running in initial RAM disk.

Welcome to Fedora CoreOS 31.20200127.3.0 dracut-049-27.git20181204.fc31.1 (Initramfs)!

[ 1.581667] systemd[1]: No hostname configured.
[ 1.582401] systemd[1]: Set hostname to &lt;localhost&gt;.
[ 1.583229] systemd[1]: Initializing machine ID from KVM UUID.
[ 1.650595] systemd[1]: Started Dispatch Password Requests to Console Directory Watch.

...

[ 3.904515] systemd[1]: Successfully loaded SELinux policy in 389.023ms.
[ 3.943381] systemd[1]: Relabelled /dev, /dev/shm, /run, /sys/fs/cgroup in 23.715ms.
[ 3.946100] systemd[1]: systemd v243.5-1.fc31 running in system mode. (+PAM +AUDIT +SELINUX +IMA -APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN2 -IDN +PCRE2 default-h)
[ 3.949186] systemd[1]: Detected virtualization kvm.
[ 3.949897] systemd[1]: Detected architecture x86-64.

Welcome to Fedora CoreOS 31.20200127.3.0!

[ 3.952521] systemd[1]: Set hostname to &lt;coreos.calgaryrhce.ca&gt;.
[ 4.148591] systemd[1]: initrd-switch-root.service: Succeeded.


[ OK ] Reached target Multi-User System.
Starting Update UTMP about System Runlevel Changes...
[ OK ] Started Update UTMP about System Runlevel Changes.
[ OK ] Started RPM-OSTree System Management Daemon.

Fedora CoreOS 31.20200127.3.0
Kernel 5.4.13-201.fc31.x86_64 on an x86_64 (ttyS0)

eth0: 10.0.0.20 fe80::605c:b122:5e5d:6a5f
coreos login:</pre>

And we&#8217;re done! We now have a container specific OS that will get updates beyond Fedora Atomic & CoreOS Container Linux EOL.

 

 

 [1]: https://www.projectatomic.io/
 [2]: https://coreos.com/os/docs/latest/
 [3]: https://getfedora.org/coreos/
 [4]: https://fedoramagazine.org/fedora-coreos-out-of-preview/
 [5]: https://docs.fedoraproject.org/en-US/fedora-coreos/getting-started/
 [6]: https://docs.fedoraproject.org/en-US/fedora-coreos/migrate-ah/
 [7]: https://docs.fedoraproject.org/en-US/fedora-coreos/fcct-config/