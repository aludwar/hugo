---
title: Update cobbler signatures for RHEL8
author: Andrew Ludwar
type: post
date: 2019-07-03T22:52:03+00:00
url: /blog/2019/07/03/update-cobbler-signatures-for-rhel8/
xyz_smap:
  - 1
categories:
  - open source
tags:
  - open source
  - server provisioning

---
Cobbler is a great tool for PXE booting, I&#8217;ve been using it for years in both my personal and professional lives. Occassionally when a new OS comes out, you may find yourself needing to update cobbler&#8217;s distro signatures so it can import a new OS distribution. Thankfully, with their distro signatures being hosted on GitHub, this is really easy to do if your cobbler server isn&#8217;t able to pull the latest signatures on its own.

I know lots of us are booting OS&#8217;s these days from QCOW images in a cloud, but I still find PXE booting very useful. I&#8217;ve built up quite a few custom kickstart files over the years, including some that [provision automatically with Ansible][1], so I still find myself building a quick kickstart file to automate a few of my server builds. Recently, I was building a RHEL8 host via PXE and realised I needed to update my cobbler server to include the new distro:

<pre class="">$ sudo mount rhel-8.0-x86_64-dvd.iso /mnt/
$ sudo cobbler import --name=RHEL-8.0 --arch=x86_64 --path=/mnt
task started: 2019-07-03_162132_import
task started (id=Media import, time=Wed Jul 3 16:21:32 2019)
No signature matched in /export/www/cobbler/ks_mirror/RHEL-8.0-x86_64
!!! TASK FAILED !!!
$ sudo cobbler signature report
Currently loaded signatures:
debian:
	jessie
	squeeze
	stretch
	wheezy
...
redhat:
..
	fedora28
	fedora29
	rhel4
	rhel5
	rhel6
	rhel7
...
11 breeds with 82 total signatures loaded</pre>

Oh yea&#8230; no rhel8 listed&#8230; time to update. Cobbler provides a method to do this, but YMMV depending if the cobbler community has updated their [github.io signature file][2]. I did the update below, but noticed rhel8 still isn&#8217;t in this latest.json file:

<pre class="">$ sudo cobbler signature update
task started: 2019-07-03_162517_sigupdate
task started (id=Updating Signatures, time=Wed Jul 3 16:25:17 2019)
Successfully got file from https://cobbler.github.io/signatures/2.8.x/latest.json
*** TASK COMPLETE ***</pre>

Thankfully, there&#8217;s not a ton of magic to this signature file, and it&#8217;s available on the cobbler [github][3] project page. So I grabbed it directly, restarted cobbler, and I was able to see rhel8 included in the signature listing.

<pre class="">$ sudo wget https://raw.githubusercontent.com/cobbler/cobbler/master/config/cobbler/distro_signatures.json -O distro_signatures.json
--2019-07-03 16:25:48-- https://raw.githubusercontent.com/cobbler/cobbler/master/config/cobbler/distro_signatures.json
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.52.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.52.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 52085 (51K) [text/plain]
Saving to: ‘distro_signatures.json’

distro_signatures.json 100%[=========================================================================================================================================&gt;] 50.86K --.-KB/s in 0.05s

2019-07-03 16:25:48 (978 KB/s) - ‘distro_signatures.json’ saved [52085/52085]

$ sudo systemctl restart cobblerd
$ sudo cobbler signature report
Currently loaded signatures:
debian:
	buster
	jessie
	squeeze
	stretch
	wheezy
...
redhat:
...
	fedora28
	fedora29
	fedora30
	ovz7
	rhel4
	rhel5
	rhel6
	rhel7
	rhel8
11 breeds with 95 total signatures loaded</pre>

Sweet. So I was able to import a RHEL8 ISO DVD into cobbler for use in my PXE booting.

<pre class="">$ sudo cobbler import --name=RHEL-8.0 --arch=x86_64 --path=/mnt
task started: 2019-07-03_162628_import
task started (id=Media import, time=Wed Jul 3 16:26:28 2019)
Found a candidate signature: breed=suse, version=sles15generic
Found a candidate signature: breed=redhat, version=rhel8
Found a matching signature: breed=redhat, version=rhel8
Adding distros from path /export/www/cobbler/ks_mirror/RHEL-8.0-x86_64:
creating new distro: RHEL-8.0-x86_64
trying symlink: /export/www/cobbler/ks_mirror/RHEL-8.0-x86_64 -&gt; /export/www/cobbler/links/RHEL-8.0-x86_64
creating new profile: RHEL-8.0-x86_64
associating repos
checking for rsync repo(s)
checking for rhn repo(s)
checking for yum repo(s)
starting descent into /export/www/cobbler/ks_mirror/RHEL-8.0-x86_64 for RHEL-8.0-x86_64
processing repo at : /export/www/cobbler/ks_mirror/RHEL-8.0-x86_64/AppStream
need to process repo/comps: /export/www/cobbler/ks_mirror/RHEL-8.0-x86_64/AppStream
looking for /export/www/cobbler/ks_mirror/RHEL-8.0-x86_64/AppStream/repodata/*comps*.xml
Keeping repodata as-is :/export/www/cobbler/ks_mirror/RHEL-8.0-x86_64/AppStream/repodata
processing repo at : /export/www/cobbler/ks_mirror/RHEL-8.0-x86_64/BaseOS
need to process repo/comps: /export/www/cobbler/ks_mirror/RHEL-8.0-x86_64/BaseOS
looking for /export/www/cobbler/ks_mirror/RHEL-8.0-x86_64/BaseOS/repodata/*comps*.xml
Keeping repodata as-is :/export/www/cobbler/ks_mirror/RHEL-8.0-x86_64/BaseOS/repodata
*** TASK COMPLETE ***</pre>

Happy PXE booting.

&nbsp;

&nbsp;

 [1]: https://calgaryrhce.ca/blog/2016/02/03/ansible-pull-and-kickstart-for-one-touch-server-provisioning/
 [2]: https://cobbler.github.io/signatures/2.8.x/latest.json
 [3]: https://github.com/cobbler/cobbler/blob/master/config/cobbler/distro_signatures.json