---
title: Making email a little more secure
author: Andrew Ludwar
type: post
date: 2016-09-12T03:52:31+00:00
url: /blog/2016/09/11/making-email-a-little-more-secure/
dsq_thread_id:
  - 5351616742
categories:
  - open source
  - security
tags:
  - open source
  - Security

---
Every so often I try to make an effort to increase the security surrounding the technology I use. Its usually after I read a [notable][1] [CVE bulletin][2], or hear of the [latest hack][3]. I&#8217;ve been wanting a more secure solution to webmail for the longest time, but knew I didn&#8217;t have many options if I enjoyed using webmail clients. They&#8217;re just so darn convenient. After enabling [two-factor authentication][4] on everything I could, I still was looking for a better solution for encrypted email. I like the project [mailpile][5], but they&#8217;re not as far along yet in features for my needs. (Consider donating if you value secure email!) I ended up going back to a local mail client (Thunderbird), which I&#8217;ve connected to my webmail accounts, and downloaded the [Enigmail add-on][6] for OpenPGP encryption and digital signing.

Once I installed the add-on, it was pretty easy to get started. There&#8217;s a setup wizard that will either create you a new PGP/GPG2 key, or you can select to use an existing key already:

[<img class="alignnone size-full wp-image-443" src="https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-wizard.png" alt="enigmail-wizard" width="766" height="688" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-wizard.png 766w, https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-wizard-300x269.png 300w" sizes="(max-width: 766px) 100vw, 766px" />][7]

I selected advanced, and picked an existing GPG2 key I have already. It imported it in one step, and I was ready to go. Next, a test email to myself to test enabling a digital signature (I had to manually accept my own key as &#8220;trusted&#8221; once I received the mail):

[<img class="alignnone wp-image-446 size-full" src="https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-signature-1.png" width="801" height="444" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-signature-1.png 801w, https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-signature-1-300x166.png 300w, https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-signature-1-768x426.png 768w" sizes="(max-width: 801px) 100vw, 801px" />][8]

And writing an encrypted mail was just as easy:

[<img class="alignnone wp-image-445 size-full" src="https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-encrypted-mail-1.png" width="962" height="470" srcset="https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-encrypted-mail-1.png 962w, https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-encrypted-mail-1-300x147.png 300w, https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-encrypted-mail-1-768x375.png 768w" sizes="(max-width: 962px) 100vw, 962px" />][9]

The Enigmail plugin makes it easy to add others to your GPG circle of trust, and GPG2 encrypted email is a now click away.

 [1]: http://heartbleed.com/
 [2]: https://cve.mitre.org/cgi-bin/cvename.cgi?name=cve-2014-0160
 [3]: http://motherboard.vice.com/read/another-day-another-hack-117-million-linkedin-emails-and-password
 [4]: https://www.google.ca/landing/2step/
 [5]: https://www.mailpile.is/
 [6]: https://addons.mozilla.org/en-US/thunderbird/addon/enigmail/
 [7]: https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-wizard.png
 [8]: https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-signature-1.png
 [9]: https://calgaryrhce.ca/wp-content/uploads/2016/09/enigmail-encrypted-mail-1.png