---
title: Comparing OpenStack installers
author: Andrew Ludwar
type: post
date: 2016-09-10T22:12:03+00:00
url: /blog/2016/09/10/comparing-openstack-installers/
dsq_thread_id:
  - 5134791471
categories:
  - cloud
  - devops
  - open source
  - OpenStack
tags:
  - devops
  - open cloud
  - open source
  - openstack
  - private cloud
  - public cloud

---
Building on a [previous post][1], I promised I&#8217;d do a comparison of open source OpenStack installers. Choosing an installer requires more thought than most people think, as the choices are vast, and each installer offers different strengths, benefits, and trade-offs. Depending on your environment&#8217;s purpose and intended result, it would be wise to align your needs with the installers&#8217; capabilities and limitations. Here they are:

[table id=1 /]

Now of course, this is just one man&#8217;s opinion, and this comparison may be outdated a week after it&#8217;s posted. But, generally speaking, these are the installers to look at and align your requirements with when you&#8217;re considering an install. For my OpenStack needs, my requirements can be summarized in one statement, while my use-cases can be summarized in two:

  * **Requirement:** My installer needs to have a strong community surrounding it. Both to anticipate and deliver the features my customers want, but also to continue to maximize the value of my OpenStack investment throughout its lifecycle and be supportive and responsive to my needs (which may or may not be the next _cool and shiny_ thing to work on).

  * **Use Case 1:** Brainless, quick and easy installation to get OpenStack up and running in the shortest amount of time possible, with the least amount of resources required.
  * **Use Case 2:** Highly capable, fully featured deployment options, and long-term lifecycle management to manage clouds for years to come.

For use case #1, it doesn&#8217;t get any more brainless than packstack. Packstack is my installer of choice when doing basic proof of concepts or demos. I only need one host, or possibly two if I want to add another compute node. After doing a standard OS installation, a [packstack all-in-one deployment][2] is literally one command and I&#8217;m off and running 20 minutes later. I don&#8217;t need to know jack squat about OpenStack, or what my needs are, or setup and configure any other infrastructure or software.

For use case #2, my installer of choice is TripleO. The complexity is a concern, but that&#8217;s not an uncommon trait for software that&#8217;s highly innovative and one which incorporates changes frequently. I&#8217;m going to need to learn about OpenStack to understand how to manage it, there&#8217;s no avoiding that piece, it might as well start at installation time. Also, TripleO is appealing because it&#8217;s OpenStack on OpenStack (OOO). The project value is multiplied by the existing innovation rate of OpenStack and also by not being tied to any one vendor. TripleO will be around long after the vendor wars are settled, because it&#8217;s lifecycle is directly tied to OpenStack itself.

Because packstack is so easy, I&#8217;ll let you run with that one on your own. For the next part in this series, I&#8217;ll show a virtual home lab environment for an OpenStack installation, and do an install of TripleO. Lastly, and finally, I&#8217;ll show an example 9 node HA deployment with Ceph.

For Red Hat customers, Red Hat publishes a comprehensive <a href="https://access.redhat.com/articles/2477851" target="_blank">installer best practices and supportability matrix</a> to help you decide.

&nbsp;

 [1]: https://calgaryrhce.ca/blog/2016/08/21/getting-started-with-openstack/
 [2]: https://access.redhat.com/articles/1127153