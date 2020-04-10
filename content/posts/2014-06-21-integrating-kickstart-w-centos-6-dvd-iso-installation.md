---
title: Integrating kickstart w/ CentOS 6 DVD ISO installation
author: Andrew Ludwar
type: post
date: 2014-06-21T20:56:38+00:00
url: /blog/2014/06/21/integrating-kickstart-w-centos-6-dvd-iso-installation/
dsq_thread_id:
  - 5088000466
categories:
  - devops
tags:
  - custom iso
  - devops
  - kickstart
  - open source

---
In our DevOps environment, we&#8217;ve got a lot of developers who regularly build VMs.  Sometimes they&#8217;re built locally on their workstations, or sometimes in the data center when they&#8217;re ready to formally move code.  Admittedly, the IT VM provisioning process can be slow at times and we often see them get frustrated and take matters into their own hands.  While this self-build method gets them their VM faster, they don&#8217;t always get IT&#8217;s supported &#8220;base image&#8221;.  This can lead into integration and support problems later on when they&#8217;re trying to move this code into production that was developed against a non-standard OS.  What to do?

[OpenStack,][1] or even [OpenShift,][2] is the long-term solution to this problem.  However, until our OpenStack environment is stood up, we will need something else to address it right now.  What if we could provide them an ISO that has our standard build integrated into it?  Well!  We are going to do just that.  Read on&#8230;

Lets make our own ISO file with our standard kickstart image build into it. We&#8217;ll change the ISO boot menu options to only allow one install selection, and let the developers get to work! You will need the CentOS install images, and a workstation running an RPM anaconda based distribution.  I used CentOS 6.5 for both.

**Set up your build environment:**

You will need the createrepo, isomd5sum, and genisoimage packages:

<pre>sudo yum install createrepo isomd5sum genisoimage</pre>

Setup a mount point for the DVD ISO, and mount it:

<pre>sudo mkdir -p /export/media/centos_build/DVD1
sudo mount -o loop CentOS-6.5-x86_64-bin-DVD1.iso /export/media/centos_build/DVD1
</pre>

Next, make a working directory for what will be the new ISO image:

<pre>sudo mkdir -p /export/media/centos_build/kickstart/{isolinux,images,ks,CentOS}
sudo chown -R &lt;user&gt;:&lt;user's group&gt; /export/media/centos_build
</pre>

Copy the mounted DVD ISO files into the working directory:

<pre>cp -Rp DVD1/isolinux/* kickstart/isolinux
cp DVD1/.discinfo kickstart/
cp -Rp DVD1/images/* kickstart/images/
chmod 664 kickstart/isolinux/isolinux.*
chmod 664 kickstart/ks/ks.cfg (This will be the location of your kickstart file. We'll create this below)
cp DVD1/Packages/*.rpm kickstart/CentOS
</pre>

You&#8217;ll need a comps.xml file. This file describes the package groups included in the ISO. You can grab one from a CentOS repo:

<pre>cd kickstart/
wget http://mirror.centos.org/centos-6/6.5/os/x86_64/repodata/b4e0b9342ef85d3059ff095fa7f140f654c2cb492837de689a58c581207d9632-c6-x86_64-comps.xml
mv b4e0b9342ef85d3059ff095fa7f140f654c2cb492837de689a58c581207d9632-c6-x86_64-comps.xml comps.xml
</pre>

Create the kickstart file. You can use system-config-kickstart to build a new one, or copy one from an already installed server&#8217;s /root/anaconda-ks.cfg file. If you don&#8217;t have system-config-kickstart, use yum to install it. The [RedHat knowledgebase][3] provides a list & explanation of all the [available kickstart options][4]. You should create the kickstart file as /export/media/centos_build/kickstart/ks/ks.cfg:

<pre>yum install system-config-kickstart
system-config-kickstart OR scp root@:/root/anaconda-ks.cfg /export/media/centos_build/kickstart/ks/ks.cfg
</pre>

Now, add any packages you&#8217;d like to include on the ISO. If you&#8217;ve got in-house RPMs, or are sourcing them from somewhere else (epel, fedora, cfengine, puppetlabs, etc.), place the .RPM file in the kickstart/CentOS/ directory. In my case, I&#8217;m going to include two in-house developed CFEngine packages. The first package installs the CFEngine agent, and the second package bootstraps CFEngine. My goal is to have this ISO & kickstart file to automatically provision the VM into the centralized configuration management tool &#8211;> CFEngine. CFEngine will then take care of adding all the admin accounts, setting up NTP, SSH, etc. (This is the real advantage with creating this custom ISO!)

<pre>cp ~/cfengine.rpm /export/media/centos_build/kickstart/CentOS/
cp ~/cfengine-provision.rpm /export/media/centos_build/kickstart/CentOS/</pre>

In your kickstart file, be sure to have these package names listed in your %packages section. This is also a good time to add/remove any other packages you want. My %packages section will look like this:

<pre>%packages
@Core
openssh
openssh-clients
openssh-server
ntp
wget
cfengine
cfengine-provision
</pre>

Now that we have our kickstart file finished, lets modify the ISO boot menu to use the kickstart file by default:

<pre>vi kickstart/isolinux/isolinux.cfg</pre>

My file looks like this. Be sure not to typo your kickstart file directory, otherwise it won&#8217;t work:

<pre>label linux
menu label ^Install w/ base image & CFEngine
menu default
kernel vmlinuz
append initrd=initrd.img ks=cdrom:/ks/ks.cfg
label vesa
menu label Install w/ ^base image & CFEngine (basic video driver)
kernel vmlinuz
append initrd=initrd.img xdriver=vesa nomodeset ks=cdrom:/ks/ks.cfg
</pre>

Now, if you&#8217;ve added packages into /export/media/centos_build/kickstart/CentOS/ directory, we will need to update the comps.xml file to reflect your changes. First, make a backup copy of comps.xml. Open the file for editing with your favourite XML editor. I&#8217;d suggest using an editor that can collapse groups of XML code as the comps.xml file is quite long. I use [sublime text][5] for this:

<pre>cd kickstart
vi comps.xml (or in my case using sublime)
</pre>

After the last <group>, but before the first <category> tag, add your new group entry. Mine looks like this:

[<img class="alignnone wp-image-63 size-full" src="http://calgaryrhce.ca/wp-content/uploads/2014/06/sublime.png" alt="sublime" width="819" height="336" srcset="https://calgaryrhce.ca/wp-content/uploads/2014/06/sublime.png 819w, https://calgaryrhce.ca/wp-content/uploads/2014/06/sublime-300x123.png 300w" sizes="(max-width: 819px) 100vw, 819px" />][6]

You&#8217;ll need to add a new category as well. I made my entry at the end of all the <category> tags, but before the </comps> tag. This category is going to reference the previous group you just created, so be careful not to typo or use different case.  The field you need to get accurate here is groupid. Mine looks like this:

[<img class="alignnone wp-image-65 size-full" src="http://calgaryrhce.ca/wp-content/uploads/2014/06/sublime21.png" alt="sublime2" width="919" height="273" srcset="https://calgaryrhce.ca/wp-content/uploads/2014/06/sublime21.png 919w, https://calgaryrhce.ca/wp-content/uploads/2014/06/sublime21-300x89.png 300w" sizes="(max-width: 919px) 100vw, 919px" />][7]

Use an XML syntax checker to make sure you didn&#8217;t make any errors. xmllint is a good one:

<pre>xmllint comps.xml
</pre>

**Building the ISO:**
  
We&#8217;ll now need to re-initialize the repo list:

<pre>cd /export/media/centos_build/kickstart/
declare -x discinfo=$(head -1 .discinfo)
createrepo -u "media://$discinfo" -g comps.xml .
</pre>

And build the ISO:

<pre>cd /export/media/centos_build
mkisofs -r -N -L -d -J -T -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -V CentOS6.5 -boot-load-size 4 -boot-info-table -o CentOS6.5.iso kickstart/
implantisomd5 CentOS6.5iso
</pre>

Now my boot menu looks like this and will automatically use my ks.cfg file for installation:

[<img class="alignnone wp-image-66 size-full" src="http://calgaryrhce.ca/wp-content/uploads/2014/06/bootmenu.png" alt="bootmenu" width="634" height="493" srcset="https://calgaryrhce.ca/wp-content/uploads/2014/06/bootmenu.png 634w, https://calgaryrhce.ca/wp-content/uploads/2014/06/bootmenu-300x233.png 300w" sizes="(max-width: 634px) 100vw, 634px" />][8]

And although I didn&#8217;t provide this option in the boot menu, if you boot the graphical installer, you should see a new section for the packages you specified:

[<img class="alignnone wp-image-67 size-full" src="http://calgaryrhce.ca/wp-content/uploads/2014/06/graphicalinstall.png" alt="graphicalinstall" width="1016" height="759" srcset="https://calgaryrhce.ca/wp-content/uploads/2014/06/graphicalinstall.png 1016w, https://calgaryrhce.ca/wp-content/uploads/2014/06/graphicalinstall-300x224.png 300w" sizes="(max-width: 1016px) 100vw, 1016px" />][9]

And that&#8217;s it!  Put the ISO up on a file store or web server, and be satisfied that the developers are using a supported base OS install!

 [1]: https://www.openstack.org/
 [2]: https://www.openshift.com/
 [3]: http://access.redhat.com
 [4]: https://access.redhat.com/knowledge/docs/en-US/Red_Hat_Enterprise_Linux/6/html/Installation_Guide/s1-kickstart2-options.html
 [5]: http://www.sublimetext.com/
 [6]: http://calgaryrhce.ca/wp-content/uploads/2014/06/sublime.png
 [7]: http://calgaryrhce.ca/wp-content/uploads/2014/06/sublime21.png
 [8]: http://calgaryrhce.ca/wp-content/uploads/2014/06/bootmenu.png
 [9]: http://calgaryrhce.ca/wp-content/uploads/2014/06/graphicalinstall.png