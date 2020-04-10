---
title: Using ansible to manage RHV/oVirt
author: Andrew Ludwar
type: post
date: 2018-10-23T21:09:32+00:00
url: /blog/2018/10/23/using-ansible-to-manage-rhv-ovirt/
xyz_smap:
  - 1
categories:
  - devops
  - open source
tags:
  - open source
  - private cloud
  - server provisioning
  - virtualization

---
A common task most virtualization administrators will perform is [installing the guest agent(s) among their guest VMs][1]. Ansible has a nice module that allows an administrator to automate these common tasks, such as attaching/detaching a CDROM device to a VM, rebooting several VMs, etc. Combining this with ansible&#8217;s management of Windows devices and it makes it fairly trivial to automate a mass [installation of guest agents in windows][2].

To get started with this, [upload the relevant tools ISO to RHV][3]:

<pre class=""># ovirt-iso-uploader upload rhv-tools-setup.iso --iso-domain=ISO
Please provide the REST API password for the admin@internal oVirt Engine user (CTRL+D to abort): 
Uploading, please wait...
INFO: Start uploading rhv-tools-setup.iso 
Uploading: [########################################] 100%
INFO: rhv-tools-setup.iso uploaded successfully</pre>

Once this ISO is present, we can [use the ovirt_vms ansible module][4] to manipulate the VMs in RHV, saving lots of manual clicking. I wrote a couple [basic playbooks and inventory file][5] for reference, just change the authentication items username/password, RHV-M hostname, and inventory to suit your environment.  From here, you can extend this and use a WinRM specific playbook to auto-install the tools by running the specific EXE file mounted on the CDROM device.

<pre class="">$ ansible-playbook attach-win-guest-agent.yml -i inventory -u root

PLAY [rhevm] ******************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [rhevm.ludwar.ca]

TASK [Authenticate to RHV] ****************************************************************************************************************************************************************************************
ok: [rhevm.ludwar.ca]

TASK [Attach ISO to Windows VMs] **********************************************************************************************************************************************************************************
ok: [rhevm.ludwar.ca] =&gt; (item=ad.ludwar.ca)
changed: [rhevm.ludwar.ca] =&gt; (item=windows1.ludwar.ca)
[DEPRECATION WARNING]: The 'ovirt_vms' module is being renamed 'ovirt_vm'. This feature will be removed in version 2.8. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.

TASK [Revoke SSO token after use] *********************************************************************************************************************************************************************************
ok: [rhevm.ludwar.ca]

PLAY RECAP ********************************************************************************************************************************************************************************************************
rhevm.ludwar.ca : ok=4 changed=1 unreachable=0 failed=0</pre>

Here we have the CDROM mounted among all the Windows guests, easy to install the tooling:

[<img class="alignnone size-large wp-image-696" src="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-tools-1024x429.png" alt="RHV guest agent tools install" width="1024" height="429" srcset="https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-tools-1024x429.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-tools-300x126.png 300w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-tools-768x322.png 768w, https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-tools.png 1906w" sizes="(max-width: 1024px) 100vw, 1024px" />][6]

And detach from the environment once we&#8217;re done:

<pre class="">$ ansible-playbook detach-win-guest-agent.yml -i inventory -u root

PLAY [rhevm] ******************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [rhevm.ludwar.ca]

TASK [Authenticate to RHV] ****************************************************************************************************************************************************************************************
ok: [rhevm.ludwar.ca]

TASK [Detach ISO to Windows VMs] **********************************************************************************************************************************************************************************
changed: [rhevm.ludwar.ca] =&gt; (item=ad.ludwar.ca)
changed: [rhevm.ludwar.ca] =&gt; (item=windows1.ludwar.ca)
[DEPRECATION WARNING]: The 'ovirt_vms' module is being renamed 'ovirt_vm'. This feature will be removed in version 2.8. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.

TASK [Revoke SSO token after use] *********************************************************************************************************************************************************************************
ok: [rhevm.ludwar.ca]

PLAY RECAP ********************************************************************************************************************************************************************************************************
rhevm.ludwar.ca : ok=4 changed=1 unreachable=0 failed=0</pre>

&nbsp;

 [1]: https://access.redhat.com/solutions/261763
 [2]: https://access.redhat.com/documentation/en-us/red_hat_virtualization/4.2/html/virtual_machine_management_guide/sect-updating_virtual_machine_guest_agents_and_drivers#Updating_the_Guest_Agents_and_Drivers_on_Windows
 [3]: https://access.redhat.com/solutions/331133
 [4]: https://docs.ansible.com/ansible/2.6/modules/ovirt_vms_module.html
 [5]: https://github.com/aludwar/ansible/tree/master/rhv
 [6]: https://calgaryrhce.ca/wp-content/uploads/2018/10/RHV-tools.png