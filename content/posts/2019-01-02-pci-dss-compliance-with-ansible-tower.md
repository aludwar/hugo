---
title: PCI-DSS Compliance with Ansible Tower
author: Andrew Ludwar
type: post
date: 2019-01-02T17:14:25+00:00
url: /blog/2019/01/02/pci-dss-compliance-with-ansible-tower/
xyz_smap:
  - 1
categories:
  - cloud
  - devops
  - security
tags:
  - configuration management
  - devops
  - open source
  - Security

---
Compliance scanning and remediation with Ansible is a common question that comes up. How does Ansible do this? What are its capabilities? Within the Ansible Galaxy community, there&#8217;s been some significant investment in developing [ansible roles for security and compliance][1]. I&#8217;ll show you how to download this Ansible role and make use of it within Ansible Tower.

First off, you can view the available security roles. Here we&#8217;ll use the rhel7-role-pci-dss role:

[<img class="alignnone size-large wp-image-710" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/Ansible-security-compliance-1024x453.png" alt="Ansible security compliance" width="1024" height="453" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/Ansible-security-compliance-1024x453.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/01/Ansible-security-compliance-300x133.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/01/Ansible-security-compliance-768x340.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/01/Ansible-security-compliance.png 1890w" sizes="(max-width: 1024px) 100vw, 1024px" />][2]

On the Tower host, let&#8217;s download and install this role using ansible-galaxy:

[<img class="alignnone size-large wp-image-709" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-1-1024x135.png" alt="Download and install" width="1024" height="135" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-1-1024x135.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-1-300x40.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-1-768x101.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-1.png 1253w" sizes="(max-width: 1024px) 100vw, 1024px" />][3]

Create a playbook that makes use of this role, [here&#8217;s an example][4] you can use and then modify to your liking:

[<img class="alignnone size-large wp-image-708" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-2-1024x414.png" alt="github repo" width="1024" height="414" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-2-1024x414.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-2-300x121.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-2-768x311.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-2.png 1055w" sizes="(max-width: 1024px) 100vw, 1024px" />][5]

I&#8217;ve created a job template in Ansible Tower to then run this playbook via a github project integration in Tower. A user can then just click launch:

[<img class="alignnone size-large wp-image-707" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-3-1024x425.png" alt="Launch via Tower" width="1024" height="425" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-3-1024x425.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-3-300x125.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-3-768x319.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-3.png 1898w" sizes="(max-width: 1024px) 100vw, 1024px" />][6]

Finally, we see the PCI-DSS compliance role run on the example host, and apply all the remediations contained in the role:

[<img class="alignnone size-large wp-image-706" src="https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-4-1024x503.png" alt="Compliance result" width="1024" height="503" srcset="https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-4-1024x503.png 1024w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-4-300x147.png 300w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-4-768x377.png 768w, https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-4.png 1903w" sizes="(max-width: 1024px) 100vw, 1024px" />][7]

 [1]: https://galaxy.ansible.com/Ansible-Security-Compliance
 [2]: https://calgaryrhce.ca/wp-content/uploads/2019/01/Ansible-security-compliance.png
 [3]: https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-1.png
 [4]: https://github.com/aludwar/ansible/blob/master/pci-compliance.yml
 [5]: https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-2.png
 [6]: https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-3.png
 [7]: https://calgaryrhce.ca/wp-content/uploads/2019/01/PCI-4.png