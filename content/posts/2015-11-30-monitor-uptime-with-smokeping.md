---
title: Monitor uptime with SmokePing
author: Andrew Ludwar
type: post
date: 2015-12-01T00:45:57+00:00
url: /blog/2015/11/30/monitor-uptime-with-smokeping/
xyz_smap:
  - 1
dsq_thread_id:
  - 5076879445
categories:
  - open source
tags:
  - fedora
  - monitoring
  - smoke ping
format: quote

---
If you&#8217;re a linux enthusiast and you don&#8217;t currently read the [LinuxJournal][1], I suggest you have a look. They&#8217;re a great publication on everything linux, and they regularly provide useful content ranging from reviews, code snippets, how-tos, and tips & tricks. They [publish digitally][2] now over a [variety of formats][3], which makes reading very convenient on any device you may have.

One of the articles I found particularly useful was on setting up [SmokePing][4], an uptime and latency tracking project based on FastCGI, and RRDTool. [Shawn Powers][5] highlights its details and [how-to set it up][6]. You can go directly to the project&#8217;s site for the code and installation documentation, however if you&#8217;re a Fedora user, SmokePing is already packaged up in the [fedora repo][7].  I&#8217;ve had some availability trouble with a couple addon domains lately, so I&#8217;m going to track a couple public URLs for my blogsite. I&#8217;m starting with a fresh install of Fedora 23:

Install smokeping:

<pre> dnf install smokeping</pre>

You&#8217;ll need sendmail for notifications, or just to satisfy binary checks in smokeping&#8217;s config:

<pre>dnf install sendmail</pre>

Add your IPs/hostnames to the config smokeping will check (optionally add owner/notification info):

<pre>vi /etc/smokeping/config
...
owner    = Andrew
contact  = email.address@domain.com
...
*** Alerts ***
to = email.address@domain.com
from = root@monitor-f23
...
menu = Top
title = RHCE Blog Site Monitor
remark = Tracking uptime since the beginning of time.

+ RHCEBlogSite
menu= RHCE Blog Site
title= RHCE Blog Site

++ calgaryrhce
host = calgaryrhce.ca

++ rhce
host = rhce.ca

++ torontorhce
host = torontorhce.ca
</pre>

Smokeping adds an httpd conf file module in its install, which I&#8217;m going to modify to allow from all IPs:

<pre>vi /etc/httpd/conf.d/smokeping.conf
...
&lt;Directory "/usr/share/smokeping" &gt;
  Require all granted                   // adding in this line
  # Require ip 2.5.6.8                  // comment out this
  # Require host example.org


&lt;Directory "/var/lib/smokeping" &gt;
  Require all granted                   // adding in this line
  # Require ip 2.5.6.8                  // comment out this
  # Require host example.org

</pre>

Now enable and start both smokeping and httpd:

<pre>systemctl enable httpd
systemctl enable smokeping

systemctl start httpd
systemctl start smokeping
</pre>

Now, browsing to my Fedora 23 host&#8217;s SmokePing URL,    **_http://monitor-f23/smokeping/sm.cgi_**    shows me this: (I&#8217;ve let it run for a few days now, giving it time to gather enough data for interesting graphing)<figure id="attachment_251" aria-describedby="caption-attachment-251" style="width: 900px" class="wp-caption alignnone">

[<img class="wp-image-251 size-full" src="https://calgaryrhce.ca/wp-content/uploads/2015/11/RHCE-Blogsite-smokeping.png" alt="RHCE Blog Site" width="900" height="725" srcset="https://calgaryrhce.ca/wp-content/uploads/2015/11/RHCE-Blogsite-smokeping.png 900w, https://calgaryrhce.ca/wp-content/uploads/2015/11/RHCE-Blogsite-smokeping-300x242.png 300w" sizes="(max-width: 900px) 100vw, 900px" />][8]<figcaption id="caption-attachment-251" class="wp-caption-text">RHCE Blog Site</figcaption></figure> 

If I click one of the graphs, I get a more detailed view of the data:<figure id="attachment_252" aria-describedby="caption-attachment-252" style="width: 895px" class="wp-caption alignnone">

[<img class="wp-image-252 size-full" src="https://calgaryrhce.ca/wp-content/uploads/2015/11/RHCE-Blogsite-smokeping-detailed.png" alt="RHCE Blog Site Detailed" width="895" height="1001" srcset="https://calgaryrhce.ca/wp-content/uploads/2015/11/RHCE-Blogsite-smokeping-detailed.png 895w, https://calgaryrhce.ca/wp-content/uploads/2015/11/RHCE-Blogsite-smokeping-detailed-268x300.png 268w" sizes="(max-width: 895px) 100vw, 895px" />][9]<figcaption id="caption-attachment-252" class="wp-caption-text">RHCE Blog Site Detailed</figcaption></figure> 

&nbsp;

Happy monitoring! My complete config files for smokeping and apache are located at my [github][10].

 [1]: http://www.linuxjournal.com/
 [2]: http://www.linuxjournal.com/content/linux-journal-goes-100-digital
 [3]: http://www.linuxjournal.com/digital-faq
 [4]: http://oss.oetiker.ch/smokeping/
 [5]: http://www.linuxjournal.com/users/shawn-powers
 [6]: http://www.linuxjournal.com/content/fight-good-fight-smokeping
 [7]: https://fedoraproject.org/wiki/Repositories#The_fedora_repository
 [8]: https://calgaryrhce.ca/wp-content/uploads/2015/11/RHCE-Blogsite-smokeping.png
 [9]: https://calgaryrhce.ca/wp-content/uploads/2015/11/RHCE-Blogsite-smokeping-detailed.png
 [10]: https://github.com/aludwar/configs/tree/master/smokeping