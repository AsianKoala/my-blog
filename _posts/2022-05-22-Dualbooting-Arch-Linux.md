---
toc: true
layout: post
description: Dualbooting Arch Linux with a Windows 10 System
categories: [Linux]
title: Dual Booting Arch Linux with a Windows 10 System Part 1: Installing Arch
---
# Pre Install

### Downloading the ISO
Head over to [the arch download page](https://archlinux.org/download/) and download the ISO using either a magnet link if you have a torrent client,
or directly from an appropriate mirror site

Once it's been downloaded, burn the image onto a flash drive using [rufus](https://rufus.ie/en/)  
From the ArchWiki:   
>`Note: If the USB drive does not boot properly using the default ISO Image mode, DD Image mode should be used instead. 
To switch this mode on, select GPT from the Partition scheme drop-down menu. After clicking START you will get the mode selection dialog, select DD Image mode.`

### Setting up Windows for Dualbooting


#### Creating the Linux partition
Search for "Disk Management" and hit enter. This should pop up.
![disk management](https://i.imgur.com/BAYKGAs.png)

Right-click on the partition of the disk that you wish to install Arch under. 
In cases with a single-disk system, this would simply be your (C:) partition under disk 0.
Then click "Shrink Volume", and allocate however much space you wish Arch to use.
This can be changed later using GParted, so don't fret too much about it's initial size.
If Windows is refusing to shrink your disk despite having enough free space, your best bet would be using GParted or another partition management software to handle the creation of your Linux partition.
![right click dialog](https://i.imgur.com/tnxbj30.png)

#### Disabling Windows Settings
Before Arch is installed, several Windows settings must be disabled to allow the 
computer to safely and reliably boot into Arch.

##### UEFI Secure Boot
This is motherboard manufacturer-specific. Generally you will need to enter your BIOS, find a setting for "Secure Boot", and disable it.

##### Fast Boot and Hibernation
Open the Control Panel and go to Power Options -> Choose what the power buttons do -> uncheck "Turn on fast startup (recommended)"

![fast startup](https://help.uaudio.com/hc/en-us/article_attachments/206772223/powerbuttons.png)

Next is to disable hibernation. From the Microsoft Docs:

>1. Press the Windows button on the keyboard to open Start menu or Start screen.
>2. Search for cmd. In the search results list, right-click Command Prompt, and then select Run as Administrator.
>3. When you are prompted by User Account Control, select Continue.
>4. At the command prompt, type powercfg.exe /hibernate off, and then press Enter.
>Type exit, and then press Enter to close the Command Prompt window.

![disable hibernation](https://www.hellotech.com/guide/wp-content/uploads/2020/05/how-to-disable-hibernate-mode.jpg)


Once all of that is finished, Arch can now be installed.


# Installing Arch
Reboot your computer and enter the boot menu during startup. The key to enter varies depending on your motherboard manufacturer. For my MSI board, it was F11.

Select the flash drive that has the Arch image and select that to boot from. Then select the first option in the Arch install menu.

![](https://i.imgur.com/U0u1e5O.png)

### Connect to the Internet

Enter iwctl with this:

`# iwctl`

Then find your WiFi device from the device list:

`[iwd]# device list`

In my case, it was wlan60

And get the network name:

`[iwd]# station DEVICE-NAME scan`
`[iwd]# station DEVICE-NAME get-networks`

Finally, connect to the desired network:

`[iwd]# station DEVICE-NAME connect NETWORK-NAME`

This will prompt you for the network password.

If you already know your device and network name, then simply enter this in the command line.

`# iwctl --passphrase NETWORK-PASSWORD station DEVICE-NAME connect NETWORK-NAME`

To verify your connection, use `ping` 

`# ping archlinux.org`

### Update Clock

`# timedatectl set-ntp true`


### Partition Disks
Find the name of the disk you wish to install Arch under.

`# fdisk -l` or `# lsblk`

Then enter enter the partitioning program `cfdisk`

`cfdisk /dev/name_of_disk`

In my case it was 

`cfdisk /dev/nvme1n1` 

![](https://i.imgur.com/oYL8Fgj.png)

Hit the arrow keys to navigate to the table entry labeled "Free Space". 

First create the swap partition. There are various takes on how much swap size you should allow, but for system with 16GB ram 8GB should be perfectly fine. 

Click enter on the "Free Space" entry and allocate 8GB by typing "8G".
Then move up to the newly created partition with the partition type "Linux filesystem", and use the right arrow key to change the type of the partition.
Then hit enter and select the "Linux Swap" option.

![](https://i.imgur.com/9swtrjY.png)

Then scroll down to the "Free Space" entry again and allocate the rest to root partition.

Your screen should look like this.

![](https://i.imgur.com/6AXJ8CH.png)

Before writing the changes to the disk, note down the root partition, swap partition, and "EFI System" partition names.

In the screenshot above, those would be /dev/sda5 and /dev/sda6 respectively.

In my case, they were /dev/nvme0n1np3 and /dev/nvme0n1p4.

You can view disk/partition information after this by using the `fdisk -l` or `lsblk` commands.

Double check that everything is correct. Now move over to the "Write" option and select it.


### Mount Partitions 
Now setup the partitions with the following commands.   
ROOT_PARTITION should be the name of your root partition.  
SWAP_PARTITION should be the name of your swap partition.  
EFI_PARTITION should be the name of your EFI partition.  

```
# mkfs.ext4 /dev/ROOT_PARTITION
# mkswap /dev/SWAP_PARTITION
# swapon /dev/ROOT_PARTITION
# mount /dev/ROOT_PARTITION /mnt 
# mount --mkdir /dev/EFI_PARTITION /mnt/efi
```

### Install Linux and essential packages 

`pacstrap /mnt base linux linux-firmware`

### Configure the system 

```
# genfstab -U /mnt >> /mnt/etc/fstab
# arch-chroot /mnt
```

##### Set the time zone

```
# ln -sf /usr/share/zoneinfo/REGION_HERE/CITY_HERE /etc/localtime
# hwclock --systohc
```

In my case (US Eastern) the command looks like this:

`# ln -sf /usr/share/zoneinfo/US/Eastern /etc/localtime`

While typing out a directory, you can press `Tab` to explore/autocomplete, which is useful in seeing different timezone options.

##### Install an editor

`# pacman -Sy vim` If you're familiar with vim, or  
`# pacman -Sy nano` For a simpler text editor.

##### Set Locale 

Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8. 

Then generate the locales with:

`# locale-gen`

Create the locale.conf file in /etc/   

`# vim /etc/locale.conf`

And add this single line:

`LANG=en_US.UTF-8`   

![](https://i.imgur.com/ok9Dmm1.png)


##### Hostname 

Create the hostname file and edit it to set the name of your machine on the network.

`# vim /etc/hostname`

My hostname file looks like this:

![](https://i.imgur.com/SeRPkKK.png)

Now edit the /etc/hosts file to look like this: 

![](https://i.imgur.com/N6Lk35d.png)

Replace "koawa" with your hostname that you just set.
Make sure to tab once in between the ip's and hostnames.

#### User 

Add a user to the system with this:

`# useradd -G wheel,audio,video -m USERNAME`

Set that user's password with:

`# passwd USERNAME`

And set the root password with:

`# passwd`

### Grub 

```
pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/efi/ --bootloader-id=GRUB
```

Edit the /etc/default/grub file and add/uncomment 
`GRUB_DISABLE_OS_PROBER=false`

Now enter: 
`# grub-mkconfig -o /boot/grub/grub.cfg`


# Network Setup 
This is how I set up my wireless network using `NetworkManager`. For a wired connection, check the [ArchWiki Ethernet Page](https://wiki.archlinux.org/title/Network_configuration/Ethernet)

If you wish to use another wireless network configuration, again, check the [ArchWiki Wireless Page](https://wiki.archlinux.org/title/Network_configuration/Wireless)

Install NetworkManager and reboot your system.
```
# pacman -S networkmanager
# exit 
# reboot 
```


You should see the GRUB menu on boot. If you don't, then reorder the boot priority of your system through the BIOS. Now press enter on "Arch Linux" (or just wait, as it will automatically boot into Arch after some time).

Now enter these commands to setup your network.

```
$ nmcli d 
$ nmcli r wifi on 
$ nmcli d wifi list 
$ nmcli d wifi connect WIFI_SSID_HERE password --YOUR_PASSWORD_HERE
```

And now Arch has been fully installed with networking capabilities. 

To learn about how to make your Arch look similar to mine, shown below, head over to my [dotfiles](https://github.com/AsianKoala/dotfiles) or read some of my other Arch Linux blog posts.

![my arch rice](https://i.imgur.com/kpwqPUQ.png)









