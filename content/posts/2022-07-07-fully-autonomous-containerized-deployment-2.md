---
title: Fully Autonomous Containerized Deployment, part 2
author: Andrew Ludwar
type: post
date: 2022-07-14T05:36:31-06:00
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

Building upon the [previous article][1], let's review what we've built so far:

 1. [x] Deployed a self-managing container OS based on Fedora CoreOS
    - [x] Configured Fedora CoreOS to automatically update itself
    - [ ] Setup intelligent health checks and automated rollbacks for any failed updates


I'm going to tackle this first from an upstream approach with Fedora CoreOS, however there's an excellent article written on this same topic for [RHEL for Edge by Brian Smith][2] that covers this incredibly well. I'd encourage everyone to have a read of his article as well!


Right, so let's tackle this last bullet point. For this, we're going to install a tool called greenboot onto the Fedore CoreOS image. The technical term for this is actually called layering, as we're building on top of the base ostree image.

There's some work upstream to make this a little friendlier, but for now we can [use a systemd unit file to install][3] greenboot and have this embedded into our Ignition configuration. Here's my completed example:


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
        inline: coreos.calgaryrhce.ca
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
          dns-search=calgaryrhce.ca
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
systemd:
  units:
    - name: rpm-ostree-install-greenboot.service
      enabled: true
      contents: |
        [Unit]
        Description=Layer greenboot with rpm-ostree
        Wants=network-online.target
        After=network-online.target
        # We run before `zincati.service` to avoid conflicting rpm-ostree
        # transactions.
        Before=zincati.service
        ConditionPathExists=!/var/lib/%N.stamp

        [Service]
        Type=oneshot
        RemainAfterExit=yes
        # `--allow-inactive` ensures that rpm-ostree does not return an error
        # if the package is already installed. This is useful if the package is
        # added to the root image in a future Fedora CoreOS release as it will
        # prevent the service from failing.
        ExecStart=/usr/bin/rpm-ostree install --apply-live --allow-inactive greenboot greenboot-default-health-checks zsh
        ExecStart=/bin/touch /var/lib/%N.stamp

        [Install]
        WantedBy=multi-user.target
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - "[ public SSH key hash ]"
    - name: aludwar
      password_hash: "[ password hash, salted ]"
      ssh_authorized_keys:
        - "[ public SSH key hash ]"
      groups: [ sudo, docker ]
</pre>

The line to pay extra attention to here is the ExecStart, specifying the rpm-ostree install command. This adds zsh as well as both greenboot and some example health checks we can use to make sure a system boots up into a healthy, functioning state.

After modifying our config above and creating a new VM, you will see a second deployment listed when running '$ rpm-ostree status' that should indicate the layered packages you've added. The "--apply-live" flag we used in the rpm-ostree install command should have applied the changes to the image right away and have them persist. You can also reboot the system so it boots into the new modified image with, 'systemctl reboot'. The dot next to the deployment indicates which one is currently active and booted into. Here's my end result:

<pre class="">
[aludwar@coreos ~]$ rpm-ostree status
State: idle
AutomaticUpdatesDriver: Zincati
  DriverState: active; periodically polling for updates (last checked Thu 2022-07-07 21:22:43 UTC)
Deployments:
● fedora:fedora/x86_64/coreos/stable
                   Version: 36.20220618.3.1 (2022-07-05T23:09:41Z)
                BaseCommit: 474557e51b1013d4e737e0fd41f4e3d482546e3615a2480a3b34bb186a8ada94
              GPGSignature: Valid signature by 53DED2CB922D8B8D9E63FD18999F7CBF38AB71F4
           LayeredPackages: greenboot greenboot-default-health-checks zsh

  fedora:fedora/x86_64/coreos/stable
                   Version: 36.20220618.3.1 (2022-07-05T23:09:41Z)
                    Commit: 474557e51b1013d4e737e0fd41f4e3d482546e3615a2480a3b34bb186a8ada94
              GPGSignature: Valid signature by 53DED2CB922D8B8D9E63FD18999F7CBF38AB71F4
</pre>

Now that we've got greenboot installed, let's use one of the example health checks to test a successful boot condition, and a custom script to test an unsuccessful boot triggering a rollback. You can dive deeper into the [greenboot documentation][4] for how exactly this works, but I'll provide a fast track example below.

I'll take one of the health check examples located in /usr/lib/greenboot/check directory. There two categories of checks we can do, ones that MUST NOT FAIL (required) for a successful boot and ones that MAY FAIL (wanted). There are corresponding directories to place these health checks in:

<pre class="">
/etc
└── greenboot
    ├── check
    │   ├── required.d
    │   └── wanted.d
</pre>

I'll take the example '/usr/lib/greenboot/check/required.d/01_repository_dns_check.sh' check and put it the required directory. This check essentially makes sure the networking on the host is valid and the host can resolve external domains it needs to update itself. Here is where I'll also enable the services required:

<pre class="">
$ cp /usr/lib/greenboot/check/required.d/01_repository_dns_check.sh /etc/greenboot/check/required.d/
$ systemctl enable greenboot-task-runner greenboot-healthcheck greenboot-status greenboot-loading-message
</pre>

You can try a test run of the script to see the output before attempting a reboot:

<pre class="">
$ ./etc/greenboot/check/required.d/01_repository_dns_check
All domains have resolved correctly
</pre>

So with that, let's reboot and see what we get. After rebooting, you'll need to SSH into the host to see the status message:

<pre class="">
$ ssh 192.168.100.50
Fedora CoreOS 36.20220618.3.1
Boot Status is GREEN - Health Check SUCCESS
Tracker: https://github.com/coreos/fedora-coreos-tracker
Discuss: https://discussion.fedoraproject.org/tag/coreos

[aludwar@coreos ~]$ 
[aludwar@coreos ~]$ sudo systemctl status greenboot-status
● greenboot-status.service - greenboot MotD Generator
     Loaded: loaded (/usr/lib/systemd/system/greenboot-status.service; enabled; vendor preset: disabled)
     Active: active (exited) since Thu 2022-07-14 16:27:35 UTC; 44s ago
    Process: 973 ExecStart=/usr/libexec/greenboot/greenboot-status (code=exited, status=0/SUCCESS)
   Main PID: 973 (code=exited, status=0/SUCCESS)
        CPU: 22ms

Jul 14 16:27:35 coreos.ludwar.ca systemd[1]: Starting greenboot-status.service - greenboot MotD Generator...
Jul 14 16:27:35 coreos.ludwar.ca greenboot-status[979]: Boot Status is GREEN - Health Check SUCCESS
Jul 14 16:27:35 coreos.ludwar.ca systemd[1]: Finished greenboot-status.service - greenboot MotD Generator.
</pre>


Looks good! Boot status is green. Now, let's test a boot failure by creating a script for a failure condition. I'm going to borrow Brian's test scripts from the [RHEL for Edge article][2], but change one to test for zsh. This script we will place in /etc/greenboot/check/required.d/ directory as zsh.sh:

<pre class="">
#!/bin/bash

if [ -x /usr/bin/zsh ]; then
echo "zsh shell found, check passed!"
exit 0
else
echo "zsh shell not found, check failed!"
exit 1
fi
</pre>

And the bootfail.sh script, which will help highlight the rollback behaviour. We will place this script in the /etc/greenboot/red.d/ directory as bootfail.sh.

<pre class="">
#!/bin/bash

echo "greenboot detected a boot failure" >> /var/roothome/greenboot.log
date >> /var/roothome/greenboot.log
grub2-editenv list | grep boot_counter >> /var/roothome/greenboot.log
echo "----------------" >> /var/roothome/greenboot.log
echo "" >> /var/roothome/greenboot.log
</pre>


Now with that, let's test a rollback by removing zsh from the host. What should happen is the greenboot service will run the zsh.sh script and fail, then reboot. It will repeat this 3 times before marking the boot as 'failed' and will then execute a rollback. Brian's bootfail.sh script here will help us capture that as this process will happen quickly during booting.

<pre class="">
[aludwar@coreos ~]$ sudo rpm-ostree uninstall zsh
...
Removed:
  zsh-5.8.1-1.fc36.x86_64
Changes queued for next boot. Run "systemctl reboot" to start a reboot
[aludwar@coreos ~]$ sudo systemctl reboot

</pre>

After a minute or two, let's go back into the host and see what's occurred. Checking greenboot-status' status, we see the log line of a fallback boot detected and an rpm-ostree rollback executed as a result:

<pre class="">
[aludwar@coreos ~]$ systemctl status greenboot-status
● greenboot-status.service - greenboot MotD Generator
   Loaded: loaded (/usr/lib/systemd/system/greenboot-status.service; enabled; vendor preset: enabled)
   Active: active (exited) since Thu 2022-07-14 11:07:18 EDT; 25s ago
  Process: 1152 ExecStart=/usr/libexec/greenboot/greenboot-status (code=exited, status=0/SUCCESS)
 Main PID: 1152 (code=exited, status=0/SUCCESS)
    Tasks: 0 (limit: 23432)
   Memory: 0B
   CGroup: /system.slice/greenboot-status.service

Jul 14 11:07:18 coreos.test.ludwar.ca systemd[1]: Starting greenboot MotD Generator...
Jul 14 11:07:18 coreos.test.ludwar.ca greenboot-status[1159]: Boot Status is GREEN - Health Check SUCCESS
Jul 14 11:07:18 coreos.test.ludwar.ca greenboot-status[1159]: FALLBACK BOOT DETECTED! Default rpm-ostree deployment has been rolled back.
Jul 14 11:07:18 coreos.test.ludwar.ca systemd[1]: Started greenboot MotD Generator.
</pre>


Checking the bootfail.sh script output, we see 3 attempts to boot and check, which failed and decremented the boot_counter:

<pre class="">
[root@coreos ~]# cat /var/roothome/greenboot.log 
greenboot detected a boot failure
Thu Jul 14 11:06:40 EDT 2022
boot_counter=2
----------------

greenboot detected a boot failure
Thu Jul 14 11:06:52 EDT 2022
boot_counter=1
----------------

greenboot detected a boot failure
Thu Jul 14 11:07:05 EDT 2022
boot_counter=0
----------------

</pre>

And when we check the rpm-ostree status, we see a new entry for an image that had zsh removed, but the active image is the previous working image which had zsh installed:

<pre class="">
[aludwar@coreos ~]$ rpm-ostree status
State: idle
AutomaticUpdatesDriver: Zincati
  DriverState: active; periodically polling for updates (last checked Thu 2022-07-14 16:20:51 UTC)
Deployments:
● fedora:fedora/x86_64/coreos/stable
                   Version: 36.20220618.3.1 (2022-07-05T23:09:41Z)
                BaseCommit: 474557e51b1013d4e737e0fd41f4e3d482546e3615a2480a3b34bb186a8ada94
              GPGSignature: Valid signature by 53DED2CB922D8B8D9E63FD18999F7CBF38AB71F4
           LayeredPackages: greenboot greenboot-default-health-checks zsh

  fedora:fedora/x86_64/coreos/stable
                   Version: 36.20220618.3.1 (2022-07-05T23:09:41Z)
                BaseCommit: 474557e51b1013d4e737e0fd41f4e3d482546e3615a2480a3b34bb186a8ada94
              GPGSignature: Valid signature by 53DED2CB922D8B8D9E63FD18999F7CBF38AB71F4
           LayeredPackages: greenboot greenboot-default-health-checks
</pre>



Perfect. So at this point we have setup and confirmed intelligent health checks and automated rollbacks for failed image updates!

<u>Additional Resources:</u>

Another excellent article for customizing Fedora CoreOS specific to an application is [this article from developers.redhat.com][5].




[1]: https://calgaryrhce.ca/posts/2022-07-07-fully-autonomous-containerized-deployment/
[2]: https://www.redhat.com/en/blog/automating-rhel-edge-image-rollback-greenboot
[3]: https://docs.fedoraproject.org/en-US/fedora-coreos/os-extensions/
[4]: https://github.com/fedora-iot/greenboot
[5]: https://developers.redhat.com/blog/2020/03/12/how-to-customize-fedora-coreos-for-dedicated-workloads-with-ostree#how_package_management_works
