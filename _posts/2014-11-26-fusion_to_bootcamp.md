---
layout: post
title: From VMWare Fusion to Bootcamp
comments: True

---

Every story cannot be a success story, sometimes mistakes need to be mentioned as well.

Earlier in the fall I setup a client with Windows 7 on a [VMWare Fusion](http://www.vmware.com/products/fusion) machine. The client was using this Windows instance to run a specialized [HRV monitoring software](http://www.megaemg.com/products/hrv-scanner/) with external Bluetooth gadgets to get data into the machine. I actually started by installing [Virtualbox](https://www.virtualbox.org/), but when I found myself writing LaunchDaemons which disabled Bluetooth on Mac OS X completely and handed the whole device over to Virtualbox I realized that maybe going with Fusion would not be to expensive. Luckily, moving from virtualbox to Fusion was simple.

But Fusion was by no means perfect either, especially the Bluetooth connection needed constant reboots and resets.

In the end, we decided to go with bootcamp, and I was presented with a a dilemma: spend another couple of hours setting up th windows machine again, or find a way to move the vmware setup directly to bootcamp.

I always seem to be willing to spend hours learning a technically sound way of doing something I could easily bruteforce in the same timeframe, in the hopes of it saving time down the road. This time I was wrong.

After some research and a lot of testing, I came to the conclusion that there is no (free) way of reliably moving a functioning Windows installation from a virtual environment (Fusion) to Bootcamp. I spent hours trying to clone the system using [Clonezilla](http://clonezilla.org/), but even after a successfull reboot the new system refused to boot up. No amount of playing around with [ReFit](http://refit.sourceforge.net/) helped, and I ended up using [Windows Transfer](http://windows.microsoft.com/en-us/windows/transfer-files-settings-from-another-computer#1TC=windows-7) and managed to actually get the whole system up and running in a couple of hours.

Has anyone else managed to clone a functional system from Fusion (or Parallels or Virtualbox) to Bootcamp? I am happy to be proven wrong...