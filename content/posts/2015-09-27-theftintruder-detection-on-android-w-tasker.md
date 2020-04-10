---
title: Theft/Intruder detection on Android w/ Tasker
author: Andrew Ludwar
type: post
date: 2015-09-27T16:29:27+00:00
url: /blog/2015/09/27/theftintruder-detection-on-android-w-tasker/
xyz_smap:
  - 1
dsq_thread_id:
  - 5075804531
categories:
  - open source
tags:
  - Android
  - Security
  - Tasker

---
If you have a smartphone, surely you covet it&#8217;s presence and usage. It probably doesn&#8217;t leave your sight very often, and losing it would be a huge inconvenience to you, perhaps almost disastrous. Thankfully, there&#8217;s a slew of services out there that help you locate your lost or stolen device, remote wipe it, etc. in case this happens. I&#8217;m going to highlight one more tool available to you in your smartphone arsenal that can help you keep tabs on your device. Also, this tool can be used for quite a bit more than security. Since this is a linux blog, of course I&#8217;m going to be talking about the android platform 🙂 .

<a href="https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm&hl=en" target="_blank">Tasker</a> is an automation application that&#8217;s available for Android. Think of it as an automation language for your smartphone. From settings to states and events, Tasker allows you to setup rules and logic to handle common tasks that you may want to automate, or integrate together. If you&#8217;re familiar with <a href="https://ifttt.com/" target="_blank">IFTTT</a> think of it as IFTTT for your phone. One of the recipes I&#8217;ve been meaning to setup for a while (a recipe or profile is the term used to group rules and logic together) is to detect when someone enters my PIN or PATTERN code incorrectly, have a photo of them taken discreetly, and then email it to me with the phone&#8217;s GPS location. This gives me a picture of the potential thief, as well as the location of my phone should I ever lose it. This is something I could provide to authorities to help recover the device, or at least see which friend/child has &#8220;borrowed&#8221; my phone for mischief. 🙂

I have a Samsung Galaxy S4 running <a href="https://wiki.cyanogenmod.org/w/Jflte_Info" target="_blank">CyanogenMod11</a>, which is Android KitKat v 4.4.4. You&#8217;ll need to install <a href="https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm&hl=en" target="_blank">Tasker</a>, add the <a href="https://play.google.com/store/apps/details?id=com.intangibleobject.securesettings.plugin&hl=en" target="_blank">Secure Settings plugin</a> to Android, install <a href="http://www.calgaryrhce.ca/wp-content/uploads/2015/09/sl4a_r6.apk" target="_blank">SL4A (Scripting Layer for Android)</a>, install <a href="http://www.calgaryrhce.ca/wp-content/uploads/2015/09/PythonForAndroid_r5.apk" target="_blank">Python</a>, and have <a href="http://www.calgaryrhce.ca/wp-content/uploads/2015/09/sendemailA.py" target="_blank">this script</a> in your /sdcard/sl4a/scripts/ directory. Tasker has a [wiki][1] and <a href="http://tasker.wikidot.com/automatically-take-and-email-your-photo" target="_blank">further details</a> on how to do this if you have questions. I had some troubles locating these .apk files as the SL4A project has recently moved from Google Code. I&#8217;ve linked them here for your convenience, but you could download from a more \*trusted\* source if you&#8217;d rather 🙂 . Once those items are installed from Google Play, you&#8217;ll want to open Python and follow the prompts to complete installing the modules. You should end up being able to run SL4A and see a few scripts in it&#8217;s list, one of them being sendemailA.py:<figure id="attachment_215" aria-describedby="caption-attachment-215" style="width: 200px" class="wp-caption alignnone">

[<img class=" wp-image-215" src="http://calgaryrhce.ca/wp-content/uploads/2015/09/Screenshot_2015-09-27-09-37-09-576x1024.png" alt="SL4A and Python" width="200" height="356" srcset="https://calgaryrhce.ca/wp-content/uploads/2015/09/Screenshot_2015-09-27-09-37-09-576x1024.png 576w, https://calgaryrhce.ca/wp-content/uploads/2015/09/Screenshot_2015-09-27-09-37-09-169x300.png 169w, https://calgaryrhce.ca/wp-content/uploads/2015/09/Screenshot_2015-09-27-09-37-09.png 1080w" sizes="(max-width: 200px) 100vw, 200px" />][2]<figcaption id="caption-attachment-215" class="wp-caption-text">SL4A and Python</figcaption></figure> 

From here, you&#8217;re ready to open Tasker, and create the profile. The Tasker wiki has pre-built profiles you can either use directly, or slightly modify for your own use. I used both of these profiles to create mine. The first, <a href="http://tasker.wikidot.com/failedloginphoto" target="_blank">failed login photo</a>, and the second, <a href="http://tasker.wikidot.com/automatically-take-and-email-your-photo" target="_blank">automatically take and email your photo</a>. At this point, you&#8217;ll be creating your first Tasker profile, associating Actions with it, and creating/setting a few variables. Don&#8217;t worry, this is pretty easy to do, you don&#8217;t need any programming experience. Here is what my Tasker profile looks like. I&#8217;ve removed my personal values in the variables, these will be substituted for your own:<figure id="attachment_218" aria-describedby="caption-attachment-218" style="width: 201px" class="wp-caption alignnone">

[<img class=" wp-image-218" src="http://calgaryrhce.ca/wp-content/uploads/2015/09/Screenshot_2015-09-27-08-48-34-576x1024.png" alt="Tasker Profile" width="201" height="358" srcset="https://calgaryrhce.ca/wp-content/uploads/2015/09/Screenshot_2015-09-27-08-48-34-576x1024.png 576w, https://calgaryrhce.ca/wp-content/uploads/2015/09/Screenshot_2015-09-27-08-48-34-169x300.png 169w, https://calgaryrhce.ca/wp-content/uploads/2015/09/Screenshot_2015-09-27-08-48-34.png 1080w" sizes="(max-width: 201px) 100vw, 201px" />][3]<figcaption id="caption-attachment-218" class="wp-caption-text">Tasker Profile</figcaption></figure> 

You can hit the play button at the bottom left to test your recipe. This is what my final product looks like&#8230; After 3 failed login attempts, I&#8217;m emailed a photo of the intruder (me in this case!), with a link to my phone&#8217;s GPS location transposed onto Google Maps:<figure id="attachment_220" aria-describedby="caption-attachment-220" style="width: 365px" class="wp-caption alignnone">

[<img class=" wp-image-220" src="http://calgaryrhce.ca/wp-content/uploads/2015/09/Tasker-Email1.png" alt="Tasker Email" width="365" height="258" srcset="https://calgaryrhce.ca/wp-content/uploads/2015/09/Tasker-Email1.png 712w, https://calgaryrhce.ca/wp-content/uploads/2015/09/Tasker-Email1-300x212.png 300w" sizes="(max-width: 365px) 100vw, 365px" />][4]<figcaption id="caption-attachment-220" class="wp-caption-text">Tasker Email</figcaption></figure>

 [1]: http://tasker.wikidot.com/userguide-en
 [2]: http://calgaryrhce.ca/wp-content/uploads/2015/09/Screenshot_2015-09-27-09-37-09.png
 [3]: http://calgaryrhce.ca/wp-content/uploads/2015/09/Screenshot_2015-09-27-08-48-34.png
 [4]: http://calgaryrhce.ca/wp-content/uploads/2015/09/Tasker-Email1.png