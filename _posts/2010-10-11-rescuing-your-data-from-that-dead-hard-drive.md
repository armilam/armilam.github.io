---
layout:             post
title:              Rescuing your data from that dead hard drive
date:               2010-10-11 19:36:00 -0400
tags:               
---

Recently I put myself in a position where a good chunk of my memories ended up on a failing external hard drive. Windows refused to recognize that there was anything attached to USB, while Ubuntu acknowledged the drive, but wouldn't mount it. I needed a way to get my photos, videos, and other assorted files back before the drive died forever. Here's how I did it.

## gddrescue
By searching Google, I found that many people use gddrescue to create an image of the drive before messing with the data. This way, once you have an image, you can do all the processing you want on that data without worrying about the drive complaining about how much work it's doing and just quit.

There are two major features of gddrescue that I found incredibly useful: its ability to resume after being interrupted and its ability to recognize I/O errors and continue gracefully without user interaction.

So, using my Ubuntu-equipped laptop, I hooked up the failing drive and let gddrescue do its thing. One day later, I checked on it... 1GB complete.

Now, I'm fairly impatient when it comes to just about anything, so 300 days wasn't going to cut it for me. I asked for help at work. Two people recommended to me that I freeze the drive. That apparently helps many failed drives to work correctly. Something to do with mechanical components. So I looked up some information online about this, and it seems it's not an uncommon practice. Just put the drive in a couple Ziplock bags for moisture protection, and the drive will work well for about 20 minutes.

So I tried this. After 2 days, I got about 10GB of imaged data, riddled with errors as reported by gddrescue. Again, I was impatient, so I took the 10GB worth of image to see if I could do anything with it and see if it would even be worth spending another 60 days recovering the rest of the drive.

## foremost
After another Google search, I found foremost, a simple command-line program for Linux that will try to find files for you. It works with CDs, USB flash drives, hard disks, and images (like I had), and probably more. It works with FAT, ext, NTFS (which I had), and just about any other file system out there. It recovers files of all types on incomplete file systems.

So I ran it on my image. And found thousands of photos, videos, and other files I forgot I had on the drive! It's working! And how great is that since I didn't have the whole partition? But many of the photos were corrupted in some way, which isn't surprising since there were so many disk read errors.

I read up on gddrescue a little more. Turns out, you can re-recover your hard drive on top of your existing image, and it will fill in the holes as it can! So I tried something a little different. I stuck the hard drive in the bottom drawer of my refrigerator where it wouldn't be disturbed, plugged it in from inside the fridge, and reran gddrescue on it. To my surprise, it only took about 10 hours to get back up to 10GB. The cool-but-not-freezing temperature seemed to work better in my case.

I reran foremost on the image, and was able to squeeze a few more photos out of it. Now my next step is to rescue and re-rescue the entire drive to get everything back that I can.

## Instructions
Here's how I did it, in a nice, reproducible instruction set.

First, get Ubuntu, or some other Linux variant. I happen to like Ubuntu. If you don't want to install it, you can download the latest version, write it to a CD, and live-boot. Just choose "Try Ubuntu Before Installing" upon boot with the CD in the CD drive.

Once you're in, install gddrescue and foremost. But first, make sure the universe and multiverse repositories are enabled. Go to System->Administration->Software Sources and check the universe and multiverse repos. Open a Terminal window and enter the following:

```
$ sudo apt-get update
$ sudo apt-get install gddrescue
$ sudo apt-get install foremost
```

Next, refrigerate your drive! Stick it in a Ziplock bag with the USB and power cables sticking out and zip up the bag. If you have an internal hard drive, use a USB IDE/SATA drive connector. They're really very convenient. For added protection, I put my drive into an additional Ziplock bag. Let it cool down for a few hours before continuing.

While you're waiting, run these two commands in that terminal window:

```
$ ls /dev/hd*
$ ls /dev/sd*
```

And take note of the results. It will list out several entries that look similar to this:

```
hda hda1 hda2 hdb hdb1
sda sda1 sda2 sda3 sdb sdb1 sdc sdc1
```

Each one without a number is a physical drive. Each one with a number is a partition on the physical drive.

Plug in your failing drive and rerun those commands. Whatever is new is how we're going to identify the bad drive. For me, it was `sdd1`.

So at this point you have your drive cooled, plugged in, and identified. Now you should decide where you're going to put your backup. I put mine in `~/Desktop/recovery/`. In the Terminal, run this command, replacing `sdd1` with the name of your own drive, and replacing the directory with the directory you chose for your backup.

```
$ sudo ddrescue /dev/sdd1 ~/Desktop/recovery/recovery.img log_file
```

This starts the imaging of your drive. It will take all the data it can from your drive and put it in `recovery.img`. `log_file` is gddrescue's log. It uses this to start from where it stopped. If you need to stop the recovery for some reason, hit Ctrl+C. If either this happens or the power goes out, or the drive just stops but decides to start working again later, you can continue the recovery process. Just add the `-C` argument to the command line:

```
$ sudo ddrescue -C /dev/sdd1 ~/Desktop/recovery/recovery.img log_file
```

And recovery will start from where it left off. If you leave off the `-C` argument, recovery will start from the beginning, but it won't erase your progress. Instead, it will fill in the gaps as well as it can.

Once you're finished making an image of the drive, decide where you want your recovered files to go. I chose `~/Desktop/recovery/recovered/`. Run foremost to recover those files:

```
$ cd ~/Desktop/recovery
$ sudo foremost recovery.img -o recovered
```

As it's working, you can watch your files show up in the folder you specified. It does not retain directory structure; instead, there is a folder for each file type.

And that's it! I wrote a lot, but the process is actually very simple. Good luck!
