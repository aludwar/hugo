---
title: Google Chrome on RHEL/CentOS 6+
author: Andrew Ludwar
type: post
date: 2014-06-02T03:14:17+00:00
url: /blog/2014/06/01/google-chrome-on-rhelcentos-6/
panels_data:
  - 'a:0:{}'
dsq_thread_id:
  - 5297573401
categories:
  - open source
tags:
  - centos6
  - chrome
  - rhel6
  - web browser

---
So if you&#8217;re like me and like to run the same OS on your desktop that is in your data centre, you might have come across the problem of Google Chrome support being discontinued for Red Hat 6+ and its clones.  Thankfully there&#8217;s a script developed by [Richard Lloyd][1] that will automatically download and install the latest Google Chrome browser for you.  It does this by picking libraries from a more recent linux distribution, and packaging them into /opt/google/chrome/lib.  Thank you Richard!

To get started:

<pre># wget http://chrome.richardlloyd.org.uk/install_chrome.sh
# chmod +x install_chrome.sh
# ./install_chrome.sh</pre>

You can easily update to the next version by running the script again, or uninstall using the -u flag.

 [1]: http://chrome.richardlloyd.org.uk/