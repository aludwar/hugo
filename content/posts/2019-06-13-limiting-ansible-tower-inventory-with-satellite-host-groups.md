---
title: Limiting Ansible Tower inventory with Satellite host groups
author: Andrew Ludwar
type: post
date: 2019-06-13T15:44:07+00:00
url: /blog/2019/06/13/limiting-ansible-tower-inventory-with-satellite-host-groups/
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
  - open source
  - Satellite 6

---
When thinking about ingesting systems into Ansible Tower&#8217;s inventory, a common use-case is to intelligently filter hosts into Ansible Tower such that we&#8217;re selectively adding only the hosts we want into Tower inventory. When pairing with Red Hat Satellite, we can use Satellite&#8217;s host grouping feature combined with Ansible Tower&#8217;s Satellite 6 dynamic inventory script to apply this filtering.

In Satellite, I&#8217;ve created two host groups, &#8220;rhv\_hypervisors&#8221; and &#8220;prod\_web_servers&#8221;:

[<img class="alignnone size-large wp-image-771" src="https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroups-1024x400.png" alt="SatelliteHostGroups" width="1024" height="400" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroups-1024x400.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroups-300x117.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroups-768x300.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroups.png 1915w" sizes="(max-width: 1024px) 100vw, 1024px" />][1]

I&#8217;ve got 2 hosts in the rhv\_hypervisors group, and 1 in the prod\_web_servers group:

[<img class="alignnone size-large wp-image-769" src="https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroupMembership-1024x399.png" alt="SatelliteHostGroupMembership" width="1024" height="399" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroupMembership-1024x399.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroupMembership-300x117.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroupMembership-768x299.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroupMembership.png 1905w" sizes="(max-width: 1024px) 100vw, 1024px" />][2]

In my case, I only want to import the &#8220;prod\_web\_servers&#8221; hosts into Ansible Tower inventory, and not the rest of the Satellite inventory. So, I setup the Ansible Tower inventory source to be Satellite 6, however I will also include the &#8220;host\_filters&#8221; attribute in the source variables section. Here I&#8217;ll use the &#8220;hostgroup\_title&#8221; key/value to limit the hosts that will get imported to Satellite. I could use any other host variable that Satellite advertises to Tower however, such as &#8220;foreman\_location\_calgary&#8221; perhaps if I wanted to ingest only the hosts in that Satellite location:

[<img class="alignnone size-large wp-image-772" src="https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostVariables-1024x528.png" alt="AnsibleTowerHostVariables" width="1024" height="528" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostVariables-1024x528.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostVariables-300x155.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostVariables-768x396.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostVariables.png 1906w" sizes="(max-width: 1024px) 100vw, 1024px" />][3]

[<img class="alignnone size-large wp-image-770" src="https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerInventoryHostFilter-1024x556.png" alt="AnsibleTowerInventoryHostFilter" width="1024" height="556" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerInventoryHostFilter-1024x556.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerInventoryHostFilter-300x163.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerInventoryHostFilter-768x417.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerInventoryHostFilter.png 1897w" sizes="(max-width: 1024px) 100vw, 1024px" />][4]

With the host filter in place, I will save the inventory source, and run a sync. After the sync, the result is just the one prod web server in &#8220;prod\_web\_servers&#8221; host group has been added to the Ansible Tower inventory:

[<img class="alignnone size-large wp-image-768" src="https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostList-1024x459.png" alt="AnsibleTowerHostList" width="1024" height="459" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostList-1024x459.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostList-300x135.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostList-768x344.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostList.png 1902w" sizes="(max-width: 1024px) 100vw, 1024px" />][5]

&nbsp;

This will help keep Ansible Tower inventory purposeful, and can help reduce the managed nodes counting against the Ansible Tower license.

To make the classification of Satellite hosts a little easier, we can also automate the membership of hosts and host groups with the hammer CLI:

<pre class="">[root@satellite ~]# hammer host update --hostgroup prod_web_servers --name web1.dev.ludwar.ca
Host updated.
[root@satellite ~]#</pre>

&nbsp;

References:
  
[1] [https://access.redhat.com/documentation/en-us/red\_hat\_satellite/6.2/html/architecture\_guide/chap-red\_hat\_satellite-architecture\_guide-host\_grouping\_concepts][6]
  
[2] [https://access.redhat.com/documentation/en-us/red\_hat\_satellite/6.5/html-single/managing\_hosts/index#sect-Red\_Hat\_Satellite-Managing\_Hosts-Changing\_the\_Group\_of\_a_Host][7]
  
[3] <https://access.redhat.com/solutions/2777811>

 [1]: https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroups.png
 [2]: https://calgaryrhce.ca/wp-content/uploads/2019/06/SatelliteHostGroupMembership.png
 [3]: https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostVariables.png
 [4]: https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerInventoryHostFilter.png
 [5]: https://calgaryrhce.ca/wp-content/uploads/2019/06/AnsibleTowerHostList.png
 [6]: https://access.redhat.com/documentation/en-us/red_hat_satellite/6.2/html/architecture_guide/chap-red_hat_satellite-architecture_guide-host_grouping_concepts
 [7]: https://access.redhat.com/documentation/en-us/red_hat_satellite/6.5/html-single/managing_hosts/index#sect-Red_Hat_Satellite-Managing_Hosts-Changing_the_Group_of_a_Host