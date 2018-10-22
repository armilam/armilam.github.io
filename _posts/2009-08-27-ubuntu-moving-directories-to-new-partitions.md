---
layout:             post
title:              "Ubuntu: Moving directories to new partitions"
date:               2009-08-27 00:47:00 -0400
tags:               
---

A couple of months ago my laptop's hard drive crashed and I didn't have a Windows recovery disk, so I reformatted and installed Ubuntu 9.04. The default configuration (which I assumed would be reasonable) gave my root partition only 4GiB of space, and it ran out quickly.

My first fix was to move /home to a new partition with plenty of space. That worked for a while, but as I kept installing new programs my /usr directly grew quickly and I began to run out of space once more.

It turns out that moving /home not only helps with hard drive space, but it's also useful if your operating system breaks or you want to change distributions. You have /home on a separate partition, so you can do whatever you need to do to your root partition and keep all of your files.

It took some research and some trial-and-error before I got everything right, so here is the procedure for those of you that find yourself needing to do the same thing. These directions are for moving /usr to a new partition, but can be used for any directory.

## Create your new partition

I used gparted for this. Look [here](http://www.fsckin.com/2007/10/21/partitioning-or-resizing-drives-in-ubuntu-using-gparted/) for instructions on how to use it.

## Mount the new partition

Create a new directory for the partition. In this example, I use /mnt/newusr. Also, replace '/dev/sdb3' with the name of your own partition.

```
$sudo mkdir /mnt/newusr
$sudo mount -t ext3 /dev/sdb3 /mnt/newusr
```

## Copy the files

The files in the directory you want to move need to be copied to the new partition.

```
$cd /usr
$sudo find . -depth -print0 | sudo cpio --null --sparse -pvd /mnt/newusr
```

The find command prints out the name and path of every file in the current directory. This output is directed to cpio, which copies all these files to your new partition. The '--null' option means that cpio reads the list of filenames from find where each filename is terminated with the null character, not a new line. The '--sparse' option is there for files with large blocks of zeros. This way, an exact copy of the files in the directory is made without truncating sparse files (they could be important!).

Once this step is done, make absolute sure that every file was copied and that permissions were copied as well. I used `$ls -alR > ls.txt` on the original and new copies of /usr and used a graphical diff tool (Beyond Compare) to see that all the files were there and their permissions were the same. There may be a better way, but this worked for me.

## Move the old /usr and create a new /usr

Rename the old /usr directory so that we can put the new /usr in its place. We're not deleting it just in case there are problems later and we want to restore.

```
$sudo mv /usr /oldusr
$/oldusr/bin/sudo mkdir /usr
```

The reason that on the second line I used '/oldusr/bin/sudo' is because when we moved /usr, linux no longer has any idea where sudo is located; it's still looking in (the non-existent) /usr/bin! We need to tell it that it's in /oldusr/bin now.

## Mount the new /usr

Mount the new partition in /usr and make sure the new /usr has the correct permissions. Keep in mind that you should change '/dev/sdb3' to the name of your own new partition.

```
$/oldusr/bin/sudo mount /dev/sdb3 /usr
$sudo chmod 755 /usr
```

## Edit fstab to mount /usr at bootup

Add the following line to the end of your /etc/fstab file to ensure that /usr is mounted correctly every time your boot your computer.

```
/dev/sdb3 /usr ext3 nodev 0 2
```

If you're mounting just about any directory other than /usr (like /home), you'll want to add nosuid as shown below.

```
/dev/sdb3 /usr ext3 nodev,nosuid 0 2
```

I used nosuid when I first moved /usr to a new partition and a lot of things weren't working correctly on my machine. When I tried using sudo, I always got this message:

> sudo: must be setuid root

It turns out that /usr must NOT be mounted with nosuid for obvious reasons.

## Make sure everything works

I would try using a bunch of programs, reboot, then use the computer as normal for a few days. If things just don't seem to work right, you can't fix it, and you want to restore your machine, it's as simple as removing the fstab entry, unmounting your new /usr, and moving /oldusr back to /usr.

```
$sudo umount /usr
$/oldusr/bin/sudo mv /oldusr /usr
```

## Delete /oldusr

Once you're confident that everything works as it should, go ahead and delete /oldusr to reclaim that hard drive space.

```
$sudo rm -r /oldusr
```

## That's it

It may seem like a lot of steps, but if you don't forget anything it's usually a relatively simple process. Let me know if I've left anything out or made any mistakes so I can correct this posting before doing any damage!
