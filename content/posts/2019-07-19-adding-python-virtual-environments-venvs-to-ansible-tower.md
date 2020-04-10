---
title: Adding python virtual environments (venvs) to Ansible Tower
author: Andrew Ludwar
type: post
date: 2019-07-19T18:06:55+00:00
url: /blog/2019/07/19/adding-python-virtual-environments-venvs-to-ansible-tower/
xyz_smap:
  - 1
categories:
  - cloud
  - devops
  - open source
tags:
  - ansible
  - Ansible Tower
  - configuration management
  - devops
  - linux
  - open source

---
As you&#8217;re developing playbooks you may find that you need to add additional python capability in order to use some modules. These are easy enough to install on a regular linux client with pip, but how does this work for Ansible Tower? Fortunately this is not too different for Ansible Tower, we&#8217;re just going to instruct Tower to add another source directory for our new venvs, and then assign these in Tower at an Organization, Project, or Job Template level. Let me show you how this works.

Ansible Tower uses two different python virtual environments by default, one for ansible named &#8216;ansible&#8217;, and one for Tower named &#8216;awx&#8217;. This allows Tower to run in a stable environment, while allowing users to add or update modules to the Ansible python environment as needed to run playbooks. We can install new python modules within the &#8216;ansible&#8217; venv, however it&#8217;s strongly recommended to create a brand new directory and install them there.

1. Create a new directory for the new python virtual environments:

<pre class=""># mkdir /opt/venvs</pre>

2. Give the directory appropriate write and execution permissions:

<pre class=""># chmod 0755 /opt/venvs</pre>

3. Modify Tower&#8217;s configuration for which directory&#8217;s to look for custom venvs using CUSTOM\_VENV\_PATHS variable. Be sure to change your username/password/Tower URL in the below command:

<pre class=""># curl -X PATCH 'https://username:password@tower.example.com/api/v2/settings/system/' -d '{"CUSTOM_VENV_PATHS": ["/opt/venvs/"]}'  -H 'Content-Type:application/json'</pre>

4. Now that the dir and Tower are setup, create a virtual environment in that location. I&#8217;m going to install &#8216;ansible[azure]&#8217; for use in interacting with Azure Resource Manager modules:

<pre class=""># virtualenv /opt/venvz/azure</pre>

5. Install some base dependences in the new venv needed to properly run playbooks:

<pre class=""># cd /opt/venvs/azure
# source bin/activate
# pip install psutil
# pip install ansible[azure]</pre>

You may need to install gcc on the Tower host depending on the modules you&#8217;re bringing in.

Now that we&#8217;ve got Ansible&#8217;s Azure modules on the Tower server, we need to decide at what level will we apply the custom virtual environment. I&#8217;m going to apply the least needed privilege philosophy and change an existing Job Template I have to use this python virtual environment. We could change it at an organization level, or a project level, but I don&#8217;t want to interfere with anything at that broad a level. So I&#8217;m going to update a Job Template.

I&#8217;m running Ansible Tower 3.5, but noticed I didn&#8217;t get an option to specify the Ansible Environment field in my Job Template edit GUI. I may chase this down later, but for now I&#8217;ll just use the Tower API to update it. I take the job template URL, which has the job template ID:

6. Update Job Template 14:

<pre class=""># curl -k -X PATCH 'https://username:password@tower.example.com/api/v2/job_templates/14/'  -d '{"CUSTOM_VIRTUALENV": ["/opt/venvs/azure"]}'  -H 'Content-Type:application/json'</pre>

Now this field has shown up for me in the Tower GUI, and we can see that instead of the default /var/lib/awx/venv/ansible environment, it&#8217;s using the new /opt/venvs/azure environment:

[<img class="alignnone size-full wp-image-796" src="https://calgaryrhce.ca/wp-content/uploads/2019/07/Tower111e.png" alt="Tower Ansible Environment" width="900" height="398" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/07/Tower111e.png 900w, https://calgaryrhce.ca/wp-content/uploads/2019/07/Tower111e-300x133.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/07/Tower111e-768x340.png 768w" sizes="(max-width: 900px) 100vw, 900px" />][1]

And as the job runs, we&#8217;ll now see the environment it run under in the job activity details:

[<img class="alignnone size-full wp-image-797" src="https://calgaryrhce.ca/wp-content/uploads/2019/07/Tower112e.png" alt="Ansible Tower Environment Job Run" width="900" height="460" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/07/Tower112e.png 900w, https://calgaryrhce.ca/wp-content/uploads/2019/07/Tower112e-300x153.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/07/Tower112e-768x393.png 768w" sizes="(max-width: 900px) 100vw, 900px" />][2]

Now we can see that the Azure Resource Manager modules I&#8217;m using in this playbook are working. The playbook can now successfully create resources in Azure.

Additional References:
  
[1] <https://access.redhat.com/solutions/3062141>
  
[2] <https://docs.ansible.com/ansible-tower/latest/html/upgrade-migration-guide/virtualenv.html>

 [1]: https://calgaryrhce.ca/wp-content/uploads/2019/07/Tower111e.png
 [2]: https://calgaryrhce.ca/wp-content/uploads/2019/07/Tower112e.png