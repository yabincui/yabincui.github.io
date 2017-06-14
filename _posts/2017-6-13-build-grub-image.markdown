---
layout: post
title: "Build grub image"
date: 2017-06-13 22:11:00 -0700
categories: linux
---
# build a bootable image with grub

## build steps
build grub image
book linux kernel, how to change from rootfs to rootfs on device.

The steps to do things are as below:
### 1. create a disk image with partitions
	
	$ dd if=/dev/zero of=a.img bs=1M count=100
	$ fdisk a.img
	    entry cmd： n
            Partition type
                p   primary (0 primary, 0 extended, 4 free)
               e   extended (container for logical partitions)
               Select (default p): p
               partition (1-4, default 1): <enter>
               First sector (2048-204799, default 2048): <enter> 
               Last sector, +sectors or +size{K,M,G,T,P} (2048-204799, default 204799):  <enter>
                Created a new partition 1 of type 'Linux' and of size 99 MiB.
           enter cmd： w
           The partition table has been altered.
           Syncing disks.
	$ sudo kpartx -av a.img
	add map loop1p1 (252:0): 0 202752 linear 7:1 2048
	$ sudo mkfs.ext4 /dev/mapper/loop1p1
	$ mkdir tmp
	$ sudo mount /dev/mapper/loop1p1 tmp


### 2. install grub in partition 1 of the disk image

	$ sudo apt-get install libdevmapper-dev
	$ git clone https://git.savannah.gnu.org/git/grub.git
	$ cd grub
	$ ./autogen.sh
	$ ./configure --prefix=`pwd`/../grub-install
	$ make install
	$ cd ../grub-install/sbin
	$ sudo ./grub-install --root-directory=../tmp /dev/loop0
  
  深度探索linux操作系统中也有一章讲grub。
  
### 3. build kernel and initramfs

### 4. build busybox

### 5. build rootfs

### 6. run the image using qemu

	$ qemu -hda a.img

I used some scripts to build grub image, at [here](https://github.com/yabincui/linux_kernel_exp).
