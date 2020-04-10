---
title: Create Azure VM with Ansible
author: Andrew Ludwar
type: post
date: 2019-01-03T21:55:28+00:00
url: /blog/2019/01/03/create-azure-vm-with-ansible/
xyz_smap:
  - 1
categories:
  - cloud
  - devops
  - open source
tags:
  - ansible
  - azure
  - devops
  - public cloud

---
With a little googling this task isn&#8217;t very complex, however, for those wanting to consume this information easily &#8211; this post is for you.

There&#8217;s a lot of cloud provisioning tools out there; if you&#8217;re like me and prefer to leverage your existing knowledge wherever possible you might come to the conclusion that using the same tool to provision your VMs as you do to manage them makes sense. I already know Ansible, why not leverage it to create my infrastructure as well as manage it post-creation.

First, in your Azure account you will want to create a service principal. This is basically an authentication token that Ansible will use to access the Azure API. Using the Azure Active Directory component for identity management, Microsoft has a good [how-to article guiding you through creating this service principal][1]. In Azure the terminology gets a little confusing as this technically gets created as an &#8220;application&#8221;, but really what we&#8217;re doing here is creating a service account identity to use with the Azure API via Ansible modules.

Once you&#8217;ve got this service principal created, I prefer to create a **~/.azure/credentials** file to store the identity information. This keeps the private auth details out of any public Ansible git repositories I may create and share with others. Microsoft publishes another good article on how to install Ansible and [how-to create the Ansible credentials file][2].

I like to use python virtual environments to keep any installed python libraries from overwriting the ones that come with my linux distro (Fedora 29). So here I create a python virtual environment where I can install the Ansible Azure packages:

<pre class="lang:sh decode:true ">$ python3 -m virtualenv azure
Using base prefix '/usr'
New python executable in /home/aludwar/code/azure/bin/python3
Also creating executable in /home/aludwar/code/azure/bin/python
Installing setuptools, pip, wheel...done.

$ source azure/bin/activate

(azure) $ which python
/home/aludwar/code/azure/bin/python

(azure) $ pip install 'ansible[azure]'
Collecting ansible[azure]
  Downloading https://files.pythonhosted.org/packages/56/fb/b661ae256c5e4a5c42859860f59f9a1a0b82fbc481306b30e3c5159d519d/ansible-2.7.5.tar.gz (11.8MB)
    100% |████████████████████████████████| 11.8MB 2.6MB/s 
Collecting jinja2 (from ansible[azure])
  Downloading https://files.pythonhosted.org/packages/7f/ff/ae64bacdfc95f27a016a7bed8e8686763ba4d277a78ca76f32659220a731/Jinja2-2.10-py2.py3-none-any.whl (126kB)
    100% |████████████████████████████████| 133kB 14.3MB/s 
...
&lt;snip&gt;
...

  Stored in directory: /home/aludwar/.cache/pip/wheels/f2/9a/90/de94f8556265ddc9d9c8b271b0f63e57b26fb1d67a45564511
Successfully built ansible PyYAML tabulate pycparser
Installing collected packages: MarkupSafe, jinja2, PyYAML, pyasn1, pycparser, cffi, idna, asn1crypto, six, cryptography, pynacl, bcrypt, paramiko, pyparsing, packaging, chardet, certifi, urllib3, pyOpenSSL, requests, applicationinsights, azure-nspkg, azure-cli-nspkg, pygments, argcomplete, humanfriendly, colorama, entrypoints, jeepney, secretstorage, keyring, isodate, oauthlib, requests-oauthlib, msrest, python-dateutil, PyJWT, adal, msrestazure, jmespath, wheel, tabulate, knack, azure-cli-core, azure-common, azure-mgmt-nspkg, azure-mgmt-batch, azure-mgmt-compute, azure-mgmt-containerinstance, azure-mgmt-containerregistry, azure-mgmt-containerservice, azure-mgmt-dns, azure-mgmt-keyvault, azure-mgmt-marketplaceordering, azure-mgmt-monitor, azure-mgmt-network, azure-mgmt-rdbms, azure-mgmt-resource, azure-mgmt-sql, azure-mgmt-storage, azure-mgmt-trafficmanager, azure-mgmt-web, azure-storage, azure-keyvault, azure-graphrbac, ansible
  Found existing installation: wheel 0.32.3
    Uninstalling wheel-0.32.3:
      Successfully uninstalled wheel-0.32.3
Successfully installed MarkupSafe-1.1.0 PyJWT-1.7.1 PyYAML-3.13 adal-1.2.0 ansible-2.7.5 applicationinsights-0.11.7 argcomplete-1.9.4 asn1crypto-0.24.0 azure-cli-core-2.0.35 azure-cli-nspkg-3.0.2 azure-common-1.1.11 azure-graphrbac-0.40.0 azure-keyvault-1.0.0a1 azure-mgmt-batch-4.1.0 azure-mgmt-compute-2.1.0 azure-mgmt-containerinstance-0.4.0 azure-mgmt-containerregistry-2.0.0 azure-mgmt-containerservice-3.0.1 azure-mgmt-dns-1.2.0 azure-mgmt-keyvault-0.40.0 azure-mgmt-marketplaceordering-0.1.0 azure-mgmt-monitor-0.5.2 azure-mgmt-network-1.7.1 azure-mgmt-nspkg-2.0.0 azure-mgmt-rdbms-1.2.0 azure-mgmt-resource-1.2.2 azure-mgmt-sql-0.7.1 azure-mgmt-storage-1.5.0 azure-mgmt-trafficmanager-0.50.0 azure-mgmt-web-0.32.0 azure-nspkg-2.0.0 azure-storage-0.35.1 bcrypt-3.1.5 certifi-2018.11.29 cffi-1.11.5 chardet-3.0.4 colorama-0.4.1 cryptography-2.4.2 entrypoints-0.2.3 humanfriendly-4.17 idna-2.8 isodate-0.6.0 jeepney-0.4 jinja2-2.10 jmespath-0.9.3 keyring-17.1.1 knack-0.3.3 msrest-0.4.29 msrestazure-0.4.31 oauthlib-2.1.0 packaging-18.0 paramiko-2.4.2 pyOpenSSL-18.0.0 pyasn1-0.4.5 pycparser-2.19 pygments-2.3.1 pynacl-1.3.0 pyparsing-2.3.0 python-dateutil-2.7.5 requests-2.21.0 requests-oauthlib-1.0.0 secretstorage-3.1.0 six-1.12.0 tabulate-0.8.2 urllib3-1.24.1 wheel-0.30.0</pre>

Now with the Azure dependencies for Ansible installed, and my **~/.azure/credentials** file created, I can start writing a playbook to create a new virtual machine. I&#8217;ve previously created some virtual networks, security groups, and public IP addresses, etc. so I&#8217;m going to reference those in my playbook. If you don&#8217;t have these, you can create them in the playbook as well. The [Ansible docs guide for Azure gives a good example playbook][3] to do this.

Here&#8217;s my playbook:

<pre class="lang:yaml decode:true">(azure) $ cat create-vm-azure.yml 
---
- name: Create Azure VM
  hosts: localhost
  connection: local
  tasks:

  - name: Create VM
    azure_rm_virtualmachine:
      profile: default
      resource_group: rgDefault
      name: tower
      vm_size: Standard_B1s
      admin_username: aludwar
      ssh_password_enabled: false
      ssh_public_keys:
        - path: /home/aludwar/.ssh/authorized_keys
          key_data: &lt;public SSH key&gt;
      network_interfaces: nicTest
      image:
        offer: CentOS
        publisher: OpenLogic
        sku: '7.5'
        version: latest
</pre>

So with that created, let&#8217;s run it:

<pre class="lang:sh decode:true ">(azure) $ ansible-playbook create-vm-azure.yml
 [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'


PLAY [Create Azure VM] ********************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Create VM] **************************************************************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP ********************************************************************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0   

(azure) $</pre>

Alright, that looks to have been successful. Let&#8217;s check the Azure Portal:

[<img class="alignnone size-large wp-image-721" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/azure-1024x396.png" alt="Azure VM" width="1024" height="396" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/azure-1024x396.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/01/azure-300x116.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/01/azure-768x297.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/01/azure.png 1891w" sizes="(max-width: 1024px) 100vw, 1024px" />][4]

Nice. It&#8217;s showing my VM as created and now running. Let&#8217;s test a login:

<pre class="lang:sh decode:true ">(azure) $ ssh 13.88.235.189
Warning: Permanently added '13.88.235.189' (ECDSA) to the list of known hosts.
[aludwar@tower ~]$ hostname
tower
[aludwar@tower ~]$ exit
logout
Connection to 13.88.235.189 closed.
</pre>

There we have it! Note, you&#8217;ll of course need to make sure your security group allows inbound SSH.

&nbsp;

 [1]: https://docs.microsoft.com/en-ca/azure/active-directory/develop/howto-create-service-principal-portal
 [2]: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/ansible-install-configure
 [3]: https://docs.ansible.com/ansible/latest/scenario_guides/guide_azure.html#creating-virtual-machines
 [4]: https://calgaryrhce.ca/wp-content/uploads/2019/01/azure.png