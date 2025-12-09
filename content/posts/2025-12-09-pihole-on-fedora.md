---
title: "How to install PiHole on Fedora"
date: 2025-12-09T09:38:59-07:00
author: Andrew Ludwar
type: post
categories:
  - open source
tags:
  - open source
---
**How to install PiHole on Fedora Server**

1. [Download the latest copy of Fedora Server][1]. If installing on a Raspberry Pi, make sure you select the ARM archtiecture.

   You can also use Fedora Media Writer for both downloading and writing the image to the storage device used in your Raspberry Pi (sd card, SSD, etc.)

2. Install and open [Raspberry Pi Imager][2]. I like using this one as it helps you make EEPROM changes as well.

   Select the Raspberry Pi model you're using, the operating system/boot device, and the storage device to write it to. Click next and write to disk.

   ![Raspberry Pi Imager](https://calgaryrhce.ca/wp-content/uploads/2025/12/rpi-imager-1.png)

3. With your OS image written to storage device, attach the storage device to the Raspberry Pi. (ie insert the sdcard, or plugin USB to SATA cable of SSD). Boot the Raspberry Pi. You should now having a working default Fedora Server OS installed on your Raspberry Pi.

4. Connect to your Pi from either SSH or keyboard/mouse and Pi browser. Use the [one-step automated install of PiHole][3] to install PiHole on the OS. The installer will finish after a few minutes and output the password/URL for you to login to PiHole on your device. It's a good idea to apply any OS updates and reboot.

5. Login to PiHole and make any necessary configurations. Personally, I like to add a second blocklist, [Hagezi Multi Pro][4], as well as deny some domains I feel are problematic. (TikTok, Roblox, etc.)

   ![DNS Block List](https://calgaryrhce.ca/wp-content/uploads/2025/12/pihole-dns-list.png)

6. Setup DNS over HTTPS (DoH) for additional security with [cloudflared][5]. Granted, this doesn't hide your DNS traffic from your ISP, it does help prevent MitM DNS attacks.

   ![CloudFlared DoH](https://calgaryrhce.ca/wp-content/uploads/2025/12/cloudflared-1.png)

7. Enjoy a more performant, more secure, and ad-free home network!



[1]: https://www.fedoraproject.org/server/download
[2]: https://www.raspberrypi.com/news/raspberry-pi-imager-imaging-utility/
[3]: https://docs.pi-hole.net/main/basic-install/
[4]: https://github.com/hagezi/dns-blocklists#pro
[5]: https://docs.pi-hole.net/guides/dns/cloudflared/
