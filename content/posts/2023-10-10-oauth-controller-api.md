---
title: "Using Automation Controller API with OAUTH"
date: 2023-10-10T11:18:06-06:00
author: Andrew Ludwar
type: post
categories:
  - devops
  - open source
tags:
  - automation
  - ansible
  - open source
---
I wanted to spend some time outlining how to query the Automation Controller API using OAUTH tokens. A few customers have reached out with these questions and I noticed the documentation on it doesn't immediately suggest the answer. So it's time for a Solution Architect to step in! These are the common questions we'll answer in this post:

1. What's the proper use of OAUTH tokens when querying the Controller API?
2. How do I use/reference this credential in a playbook?

Notably, there's some confusion when searching docs.ansible.com or access.redhat.com and the OAUTH content that comes up. Let me divide this up into something that (hopefully) adds more clarity. Generally speaking, there's three areas of Automation Controller where you might see the term OAUTH or token:

1. [Administration Guide - Token-Based Authentication][1]
2. [Administration Guide - Social Authentication][2]
3. [User Guide - Credentials][3]

Suffice to say, we're going to be talking about #1 here and this is also the answer to question #1. You've probably seen the Credentials link in Controller GUI (#3) and saw references to Personal Access Tokens or API tokens but these are scoped to how Controller accesses other platforms or vice/versa. Or similarly in enterprise authentication when integrating with other identity providers (#2). What we're going to go through here is the access token that can be assigned to a user of the Ansible Automation Platform, similar to their username & password. Then, using this token to authenticate to the Controller API effectively replacing logging into either the GUI or API via username & password. We're essentially going to switch from manually going through the GUI or API, to doing this programatically... and we'll show the example in an Ansible playbook.

Secondly, to answer question #2, if you go to your Controller API and look at /api/v2/credential_types, you will see that there is an injector field for each type. This injector field is the name of the variable you'd use to reference it in a playbook. Notably, of the several built-in credentials, only 3 have injectors: Insights, Red Hat Virtualization, and Red Hat Ansible Automation Platform. This means that the other credentials are usable in controller but not accessible by the playbook. Now, you could go create a custom credential type for use in playbooks, but for now let's just use the defaults to see if this addresses your need before going down the path of creating custom credentials.


The specific section of the doc you should be referencing is [this][4]. While there are tokens that can be assigned & scoped to an application, the tokens we're concerned about are ones that are assigned & scoped to a user and can be thought of as the same concept of a username & password. These tokens are specific to the user, will have a similar lifetime as the user's password, and will need to be treated and stored similarly as a user's password.

First, I've got a user created in AAP Controller named "aludwar". This is in the Access --> Users section of the Controller GUI:

![Ansible Controller Users](https://calgaryrhce.ca/wp-content/uploads/2023/10/aap-users.png)

In order to [create a token for a user][5], you need to be logged into AAP as that user. This is where the "Token" tab will show up allowing you to create a new token assigned to this user:

![aludwar Token](https://calgaryrhce.ca/wp-content/uploads/2023/10/aludwar-token.png)

When creating the token, you'll be prompted with the token value. Similar to a password, you'll need to save this somewhere safe like a password vault or encrypted Ansible vault file. This is what will take place of the username & password authentication when we later send a query to the Controller API.

![User Token Value](https://calgaryrhce.ca/wp-content/uploads/2023/10/user-token.png)

Now that we have a personal access token created for the user "aludwar", we'll write a playbook using the ansible.builtin.uri module that uses the token to query the Controller API and capture the response. Also, since we're handling secrets data, you should consider using the `no_log: true` attribute on the tasks or play that query the API. This can be used to keep verbose output but hide sensitive information from others, and keep the credential(s) from being leaked. Here's two examples below. Note, I'm doing something incredibly insecure here in hardcoding an example token value. As mentioned previously, the right way to store this would be in an encrypted vault file or a password/secrets manager, but for the sake of simplicity I'm showing a very rudimentary and insecure way of writing the playbook:

    ---
	- name: Query the Automation Controller API w/ User Token
	  hosts: localhost
	  gather_facts: false
	  vars:
	    token: 5qDAV0DUgwe1IcHAuUeEZnzET0Gves
	
	  tasks:
	    - name: Query Automation Controller API - Example 1
	      ansible.builtin.uri:
	        url: https://aap.ludwar.ca/api/v2/projects
	        method: GET
	        return_content: true
	        headers:
	          Content-Type: application/json
	          Authorization: Bearer 5qDAV0DUgwe1IcHAuUeEZnzET0Gves
	        validate_certs: false
	      no_log: true
	      register: projects
	
	    - name: Display query results
	      ansible.builtin.debug:
	        var: projects
	
	    - name: Query Automation Controller API - Example 2
	      ansible.builtin.uri:
	        url: https://aap.ludwar.ca/api/v2/projects
	        body_format: json
	        method: GET
	        return_content: true
	        headers:
	          Authorization: "Bearer {{ token }}"
	        validate_certs: false
	      no_log: true
	      register: projects2
	
	    - name: Display query results
	      ansible.builtin.debug:
	        var: projects2

And from here, the playbook output will look like this:

    TASK [Query Automation Controller API - Example 2] ***************************************************************************************************************************************************************************************************************************************************************************
	ok: [localhost]
	
	TASK [Display query results] *************************************************************************************************************************************************************************************************************************************************************************************************
	ok: [localhost] => {
	    "projects2": {
	        "access_control_expose_headers": "X-API-Request-Id",
	        "allow": "GET, POST, HEAD, OPTIONS",
	        "cache_control": "no-cache, no-store, must-revalidate",
	        "changed": false,
	        "connection": "close",
	        "content": "{\"count\":3,\"next\":null,\"previous\":null,\"results\":[{\"id\":9,\"type\":\"project\",\"url\":\"/api/v2/projects/9/\",\"related\":{\"created_by\":\"/api/v2/users/1/\",\"modified_by\":\"/api/v2/users/1/\",\"credential\":\"/api/v2/credentials/9/\",\"last_job\":\"/api/v2/project_updates/248/\",\"teams\":\"/api/v2/projects/9/teams/\",\"playbooks\":\"/api/v2/projects/9/playbooks/\",\"inventory_files\":\"/api/v2/projects/9/inventories/\",\"update\":\"/api/v2/projects/9/update/\",\"project_updates\":\"/api/v2/projects/9/project_updates/\",\"scm_inventory_sources\":\"/api/v2/projects/9/scm_inventory_sources/\",\"schedules\":\"/api/v2/projects/9/schedules
	    ...
	       },
	        "msg": "OK (8953 bytes)",
	        "pragma": "no-cache",
	        "redirected": true,
	        "server": "nginx",
	        "status": 200,
	        "strict_transport_security": "max-age=63072000",
	        "url": "https://aap.ludwar.ca/api/v2/projects/",
	        "vary": "Accept, Accept-Language, Origin, Cookie",
	        "x_api_node": "aap.ludwar.ca",
	        "x_api_product_name": "Red Hat Ansible Automation Platform",
	        "x_api_product_version": "4.4.0",
	        "x_api_request_id": "490ed5122b71462eaeacca2e780a6f97",
	        "x_api_time": "0.141s",
	        "x_api_total_time": "0.298s",
	        "x_content_type_options": "nosniff",
	        "x_frame_options": "DENY"
	    }
	}
	
	PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************************************************************
	localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   



**Additional Resources:**

[https://opensource.com/article/21/9/ansible-rest-apis][6]

[https://access.redhat.com/solutions/3125551][7]

[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/uri_module.html][8]




[1]: https://docs.ansible.com/automation-controller/latest/html/administration/oauth2_token_auth.html
[2]: https://docs.ansible.com/automation-controller/latest/html/administration/social_auth.html#google-oauth2-settings
[3]: https://docs.ansible.com/automation-controller/4.0.0/html/userguide/credentials.html
[4]: https://docs.ansible.com/automation-controller/latest/html/administration/oauth2_token_auth.html#using-oauth-2-token-system-for-personal-access-tokens-pat
[5]: https://docs.ansible.com/automation-controller/4.4/html/userguide/applications_auth.html#ug-tokens-auth-create

[6]: https://opensource.com/article/21/9/ansible-rest-apis
[7]: https://access.redhat.com/solutions/3125551
[8]: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/uri_module.html
