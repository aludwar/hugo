---
title: Creating custom credentials in Ansible Tower
author: Andrew Ludwar
type: post
date: 2019-07-19T22:20:23+00:00
url: /blog/2019/07/19/creating-custom-credentials-in-ansible-tower/
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
  - open source

---
As we go further in our efforts to automate all the things with ansible, you will likely come across a need to manage some cusom credentials that do not come with a predefined type in Ansible Tower. You might be currently setting these in an environment variable, in group\_vars or host\_vars, or perhaps hardcoded into a playbook itself. What if we need to extract these from the eyes of others, or make them easily reused across the organization? Can I use Ansible Tower to manage and abstract custom credentials like it does for machine, github, or cloud account types? The answer is yes.

So let&#8217;s look at moving a custom credential into Tower. I&#8217;m going to use Red Hat Subscription Manager credentials as an example, this is the Customer Portal account username and password that&#8217;s used to register a RHEL system. This credential is obviously private and sensitive in nature and I&#8217;d like to make this easily usable across my organization, but I don&#8217;t want it to be stored in plain-text or easily accessible by the wrong eyes.

I have an example playbook that registers a host using the subscription-manager utility. Currently I store my username and password in an environment variable that&#8217;s private to my workstation and user login session:

<pre class=""># export SMUSERNAME=andrew.ludwar
# export SMPASSWORD=secretpassword

# cat playbook.yml
---
- name: Deploy a VM
  hosts: all
  become: true
  vars:
    # Fetch subscription-manager credentials
    subs_mgr_username: "{{ lookup('env', 'SMUSERNAME') }}"
    subs_mgr_password: "{{ lookup('env', 'SMPASSWORD') }}"

  tasks:
    - name: Register host with subscription-manager
      redhat_subscription:
        state: present
        username: "{{ subs_mgr_username }}"
        password: "{{ subs_mgr_password }}"
        auto_attach: true</pre>

Now this works great, but it&#8217;s limited to my specific workstation and login session. What if I wanted others to be able to register systems using these credentials? Or, what if I wanted these to be easily passed into playbooks run via Tower in a workflow? Enter custom credentials.

In Ansible Tower, we have a concept of Credential Types. We can define a credential type that&#8217;s different from the pre-packaged credentials, and then create a new credential using that framework. In the Ansible Tower GUI, under Administration at the left hand side, click on &#8220;Credential Types&#8221;. You should see a listing similar to this, although yours may be blank:

[<img class="alignnone size-full wp-image-805" src="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT.png" alt="Tower Credential Type" width="900" height="440" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT.png 900w, https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT-300x147.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT-768x375.png 768w" sizes="(max-width: 900px) 100vw, 900px" />][1]

Let&#8217;s define a new credential type, which Ansible Tower will abstract away from those who we don&#8217;t want to see it but will still allow us to use it easily in our playbooks.

I&#8217;ve named this credential for the Red Hat Customer Portal, and I&#8217;ve given it some input configuration and injector configuration to define what this credential is and how we will reference it. We define the credential ID, its data type (string), label, and for the password I&#8217;ve included the secret flag to keep this hidden wherever it gets referenced.

<span style="text-decoration: underline;">Input Configuration:</span>

<pre class="">fields:
- id: rhnusername
  type: string
  label: rhnusername
- id: rhnpassword
  type: string
  label: rhnpassword
  secret: true
required:
- rhn-username
- rhn-password</pre>

The injector configuration is basically taking over for the environmental variable structure, in that we&#8217;ll be using these as extra vars passed into playbooks, and I&#8217;m assigning these a name and a value to be referenced

<span style="text-decoration: underline;">Injector Configuration:</span>

<pre class="">extra_vars:
  rhnpassword: '{{rhnpassword}}'
  rhnusername: '{{rhnusername}}'</pre>

Here&#8217;s what it looks like as I&#8217;ve now saved it:

[<img class="alignnone size-full wp-image-806" src="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT2.png" alt="Tower Credential Type Details" width="900" height="329" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT2.png 900w, https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT2-300x110.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT2-768x281.png 768w" sizes="(max-width: 900px) 100vw, 900px" />][2]

Now that the credential framework is there, we can use this to create a new credential for the Red Hat Customer Portal. Let&#8217;s create a new credential. When clicking on add credential, we should now see a new option for credential type named &#8220;Red Hat Customer Portal&#8221;:

[<img class="alignnone size-full wp-image-809" src="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT3.png" alt="New Credential Type" width="621" height="509" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT3.png 621w, https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT3-300x246.png 300w" sizes="(max-width: 621px) 100vw, 621px" />][3]

So we select our new credential type. Now we should see the two fields that we&#8217;ve defined previously in RHNUSERNAME and RHNPASSWORD, and also that the password field should hide our data once entered:

[<img class="alignnone size-full wp-image-808" src="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT4.png" alt="Credential Fields" width="900" height="240" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT4.png 900w, https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT4-300x80.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT4-768x205.png 768w" sizes="(max-width: 900px) 100vw, 900px" />][4]

After entering our username and password details here and saving, this is ready to be used in playbooks called with Tower.

Remember the playbook we just used that had environment variable look ups? We&#8217;ll change that slightly to use the names defined in the injector configuration. Then we&#8217;ll add this new credential in the Job Template and Tower will pass the credential details to be used in the playbook, and the playbook will capture them by their variable names, just like if we were passing extra variables at runtime. Here&#8217;s the new playbook we&#8217;ll use:

<pre class="">--- 
- name: Deploy a VM
  hosts: all
  become: yes 
  vars:
  # Fetch subscription-manager credentials
    subs_mgr_username: "{{ rhnusername }}"
    subs_mgr_password: "{{ rhnpassword }}"

  tasks:
  - name: Register host with subscription-manager
    redhat_subscription:
    state: present
      username: "{{ subs_mgr_username }}"
      password: "{{ subs_mgr_password }}"
      auto_attach: true</pre>

And in the Job Template, we&#8217;ll simply add another credential which will get passed to and captured by the playbook to be used to register the system:

[<img class="alignnone size-full wp-image-812" src="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT5.png" alt="Custom Credential Job Tempalte" width="900" height="509" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT5.png 900w, https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT5-300x170.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT5-768x434.png 768w" sizes="(max-width: 900px) 100vw, 900px" />][5]

Which gets captured and used in our playbook results:

[<img class="alignnone size-full wp-image-814" src="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT6.png" alt="Tower Custom Credential Job Result" width="900" height="486" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT6.png 900w, https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT6-300x162.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT6-768x415.png 768w" sizes="(max-width: 900px) 100vw, 900px" />][6]

References:
  
[1] https://access.redhat.com/solutions/3331011
  
[2] https://docs.ansible.com/ansible-tower/latest/html/userguide/credential_types.html

&nbsp;

 [1]: https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT.png
 [2]: https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT2.png
 [3]: https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT3.png
 [4]: https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT4.png
 [5]: https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT5.png
 [6]: https://calgaryrhce.ca/wp-content/uploads/2019/07/TowerCT6.png