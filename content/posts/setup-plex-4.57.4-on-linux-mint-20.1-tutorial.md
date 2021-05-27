---
title: "Setup Plex 4.57.4 on Linux Mint 20.1 Tutorial"
date: 2021-05-26T23:40:01-04:00
---

I'll cut to the chase, here's how to get a working Plex Media Server running on Linux Mint 20.1 Cinammon.

# Download Plex Media Server

Download Plex Media Server from https://www.plex.tv/media-server-downloads/

Pick `Linux` > `"Ubuntu (16.04+) / Debian (8+) - Intel/AMD 64-bit"`

# Install Plex Media Server

Double-click the installer and install the package. It's very straightforward. 

After installing you should see Plex Media Server in your start menu -- a very Windows 10 like experience, right?

# Mount your external hard drives so Plex can see your media.

Here's the tricky part.

## Unmount external drives

Let's assume your external drives are plugged in and mounted. Unmount them. 

On the bottom right there should be a button with an eject icon. Click it and unmount the drive.

## Create new folder in a place Plex can see it

Go into the `/media` directory, you should see your username folder. In my case `sergio`

{{< highlight bash >}}
cd /media
ls
sergio
{{< /highlight >}}

Create a folder in this `/media` folder to mount your external drive to.

{{< highlight bash >}}
cd /media
mkdir disk1
{{< /highlight >}}

## Find the UUID of your external drive and mount it in disk1 folder.

I named it `disk1` you can name it `chocolate` if you want, it doesn't affect the end result.

Run the `blkid` command to print out the drive's UUID and copy it.

{{< highlight bash >}}
/media sudo blkid
/dev/sdc2: LABEL="Seagate Backup Plus Drive" UUID="C80EDA3F0EDA266A" TYPE="ntfs" PTTYPE="atari" PARTLABEL="Basic data partition" PARTUUID="5ada7d05-65b9-467f-b867-fcdb2716cccc"
{{< /highlight >}}

You'll probably see more than one row. Make sure you copy the `UUID` value, not the `PARTUUID` value.

## Tell fstab to mount the drive to the `disk1` folder.

Run `sudo nano /etc/fstab` and edit it to look like this:

{{< highlight bash >}}
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sdb2 during installation
UUID=da774204-77ee-4122-ba39-507185994b06 /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/sdb1 during installation
UUID=5A3F-31E9  /boot/efi       vfat    umask=0077      0       1
/swapfile                                 none            swap    sw              0       0

UUID=C80EDA3F0EDA266A /media/disk1 auto defaults 0 0
{{< /highlight >}}

## Mount the drive using your fstab configuration!

{{< highlight bash >}}
sudo mount -a
{{< /highlight >}}

That's it! You should be able to access the files in those folders through the normal File Explorer.

But more importantly, you'll see that Plex can see those files just fine and now your libraries will work!

