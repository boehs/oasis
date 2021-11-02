After searching a bit I could not find a simple and good howto to do that.  
The following method should work for any Linux distribution (Ubuntu, Debian, Manjaro, Archlinux, Fedora…). Source and target systems must be on the same processor architecture (though transfer from 32bit to 64bit should work).

What you need:

-   2 live USB keys (or cds)
-   To speed up data transfer: good quality ethernet cables (one cable between the 2 computers is OK), or a usb key/drive with a BIG ext4 partition. You can try over wifi, but it may be slow.

## 1. Boot source and target machines on live USB/CD

Any live USB/CD should be OK.  
On the target computer, you will need a tool to partition your hard drive, like [gparted](http://gparted.org/ "gparted").  
[rsync](http://rsync.samba.org/ "rsync") is also required for data transfer: it’s included in many live systems.

[Ubuntu](http://www.ubuntu.com/ "Ubuntu") live cd is OK, [Manjaro](http://manjaro.org/ "Manjaro") live cd too.

## 2. Partition your target hard drive

Use a tool like [gparted](http://gparted.org/ "gparted") to partition the target hard drive, with the same partitions as your source system (slash, swap, home…).  
I recommend you to assign LABELs to your partitions: for the fstab, it’s easier than UUIDs.

## 3. Mount all partitions on both machines

On both systems, open a root terminal. Then, for each data partition (you can ignore `swap`):

```bash
$ mkdir /mnt/slash
$ mount /dev/sdaX /mnt/slash
```

If you have a home partition:

```bash
$ mkdir /mnt/home
$ mount /dev/sdaY /mnt/home
```

## 4. Transfer the data (network or usb)

This part may be tricky. Choose the method you prefer.

### Network

1.  **Setup the network.** Test the connectivity with ping command.  
    The easier is to plug the PCs on a DHCP network (like your ISP box) so that you get automatic IP addresses. If you linked the 2 pcs with a single cable, you’ll have to setup the IPs with NetworkManager (static ips, or adhoc network).
2.  On source system, as root, create a simple `/etc/rsyncd.conf` file:
    ```toml
    uid = root
    gid = root
    use chroot = no
    
    [all]
        path = /
	```
    
3.  Then start the rsync daemon server: `rsync --daemon`
4.  On target PC, for each partition:
    
    `rsync -avHX SOURCE_IP::all/mnt/slash/ /mnt/slash/`
    
    Don’t forget `/` at the end of paths. `-a` will preserve many file attributes like owner and permissions, `-H` will preserve hardlinks if any, `-X` will preserve extended attributes like setuid. You may also add `-A` if you are using acls. What is good with rsync is that you can stop and restart the transfer whenever you want.
    

### USB

Prepare a USB drive with a BIG `ext4` partition.

1.  Mount the USB partition on source system (`mount /dev/sdbX /mnt/usb`)
2.  For each partition:
    
    `rsync -avHX /mnt/slash/ /mnt/usb/slash/`
    
3.  umount, unplug and remount the USB disk on the target system.
4.  For each partition:
    
    `rsync -avHX /mnt/usb/slash/ /mnt/slash/`
    

## 5. Change fstab on target system

As root, edit `/mnt/slash/etc/fstab`  
For each partition (including swap), replace the first field with the new UUID or LABEL (it’s straightforward with LABELs):  
`UUID=the-long-uuid`, or `LABEL=yourlabel`

2 ways to get the UUIDs / LABELs:

```bash
$ ls -l /dev/disk/by-uuid/
$ blkid /dev/sdaX
```

## 6. Reinstall Grub

We will use a **chroot** (changed root environment) to be able to call the [grub](http://www.gnu.org/software/grub/ "Grub") install inside the migrated system.

First, bind mount some system directories needed by grub, then chroot:

```bash
$ mount --bind /proc /mnt/slash/proc
$ mount --bind /sys /mnt/slash/sys
$ mount --bind /dev /mnt/slash/dev
$ mount --bind /run /mnt/slash/run
$ chroot /mnt/slash
```

Then install grub in the [Master Boot Record](http://en.wikipedia.org/wiki/Master_boot_record "master boot record") of your hard drive, and update grub config file (with the new uuids…):

```bash
$ grub-install /dev/sda
$ update-grub
```

## 7. Reboot target machine

That’s it! Your system should be working on the new computer now.  

---

Use clonezilla

---

This can be the fastest way to move. As to copy your hard drive partitions as disk images are quite fast. If you don't want to re-install every piece of software. Though creating, resizing and moving the disk images can take quite a long time. I would only recommend this if you are not going to upgrade to a new version of Ubuntu. Make sure you understand disk partitions and grub. Most of what I am doing will use the command line. You need to make sure you understand what a command does before you run it. I am not responsible for data loss as a result of the instructions that follow.

Step one create a disk image of your installation.

Fist we need to get some information about the setup. Using `parted -l` and `mount`

```bash
$ sudo parted -l
Model: ATA ST9320423AS (scsi)
Disk /dev/sda: 320GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type      File system     Flags
 1      32.3kB  197MB   197MB   primary   ext4            boot
 2      197MB   10.2GB  10.0GB  primary   linux-swap(v1)
 3      10.2GB  50.2GB  40.0GB  primary   ext4
 4      50.2GB  299GB   249GB   extended
 5      50.2GB  54.4GB  4195MB  logical   ext4
 6      54.4GB  65.9GB  11.5GB  logical   ext4
 7      65.9GB  299GB   233GB   logical   ext4

$ mount
/dev/sda5 on / type ext4 (rw,errors=remount-ro)
/dev/sda7 on /home type ext4 (rw)
/dev/sda1 on /boot type ext4 (rw)
/dev/sda6 on /usr type ext4 (rw)
# I took out the entries that were not need for these instructions

$ cat /etc/fstab 
proc            /proc           proc    nodev,noexec,nosuid 0       0
UUID=ddc8c237-e8ac-4038-a0ed-f7c866d6603b /               ext4    errors=remount-ro 0    1
UUID=aa9881d1-5cc1-4e94-8cd7-8125e18ece2f /boot           ext4    defaults        0      2
UUID=31a6fde1-6b96-4cc3-acfd-88573f52be36 /home           ext4    defaults        0      2
UUID=073146a7-5668-4728-9a6f-1a599f358a8d /usr            ext4    defaults        0      2
UUID=540b96b6-b3c3-4092-b4ad-6b33bcbbe16d none            swap    sw              0      0
```

Your set up might look different. I have a separate partition for `/home`, root (`/`), and `/usr`.

## Creating the Disk Images

I use `dd` as it is simple and quick. Make sure you read and understand how it works. You will need an empty partition that is bigger than the entire partition size that you are copying. This can take quite some time. Creating resizing and copying the partitions can take a few hours depending on their size. You will need to replace the external drive with a part to the storage media you will use for this process.

```bash
sudo dd if=/dev/sda5 of=/media/externaldrive/sda5-root.img
sudo dd if=/dev/sda7 of=/media/externaldrive/sda7-home.img
sudo dd if=/dev/sda6 of=/media/externaldrive/sda6-usr.img
```

Here is an actual example of out put after running this on my set up.

```bash
$ sudo dd if=/dev/sda5 of=/media/home0/sda5-root.img
8193087+0 records in
8193087+0 records out
4194860544 bytes (4.2 GB) copied, 55.3159 s, 75.8 MB/s
```

We can reduce the size of this disk image, using the tools provided by Linux.

```
$ sudo resize2fs -P sda5-root.img
 resize2fs 1.41.11 (14-Mar-2010)
 Estimated minimum size of the filesystem: 605972
$ ls -sh ./sda5-root.img
 4.0G ./sda5-root.img
$ sudo resize2fs -M sda5-root.img
 resize2fs 1.41.11 (14-Mar-2010)
 Please run 'e2fsck -f sda5-root.img' first.
$ sudo e2fsck -fy ./sda5-root.img # y makes it run without asking thousands of questions.
```

`e2fsck` will output lots of errors or fixes necessarily. This is because the information in the file system is no longer correct in terms of where the partition boundaries start and end. This is correct because it is no longer in the partition it was configured for.

```
$ sudo resize2fs -M sda5-root.img
 resize2fs 1.41.11 (14-Mar-2010)
 Resizing the filesystem on sda5-root.img to 605505 (4k) blocks.
 Resizing the filesystem on sda5-root.img to 605505 (4k) blocks.
 The filesystem on sda5-root.img is now 605505 blocks long.
$ ls -sh ./sda5-root.img
 2.4G ./sda5-root.img
```

It essentially removes all the free space in the partition. So for the larger partition, this can be more the 50% of the disk size. Much quicker to copy a smaller file

You now need to boot up your new laptop with a live disk and do what follows here. You need to use a live disk as you can not make changes to a running partition that is currently used by the installed operating system.

You can now copy these disk images into the partitions on the new computer. You should have set up these partitions already. Using the live disk and `gparted` is a quick and easy way to do this. Make sure you have all the partitions your system requires. You can make these partitions larger than the ones you had on your previous system. When we copy the disk images into them, we will resize the file system and it will take up all the free space on the partition.

Now step two: copying the disk images on to the new drive and into the new partitions.

```
sudo dd if=/media/exteranldrive/sda5-root.img of=/dev/sda3 # replace the [sda3] with your partition.  
```

On my machine, this is what the output looked like

```
$ sudo dd if=./sda5-root.img of=/dev/sdb6 
4844040+0 records in  
4844040+0 records out  
2480148480 bytes (2.5 GB) copied, 87.4921 s, 28.3 MB/s  

$ sudo fsck.ext4 -fy /dev/sdb6  
e2fsck 1.41.11 (14-Mar-2010)  
Pass 1: Checking inodes, blocks, and sizes  
Pass 2: Checking directory structure  
Pass 3: Checking directory connectivity  
Pass 4: Checking reference counts  
Pass 5: Checking group summary information  
root1: 50470/504000 files (1.4% non-contiguous), 616736/2060328 blocks  
```

Now we need to edit the fstab file to point to the correct devices. If you have just copied the new disk partition on to your new disk, the fstab file is on that partition so you need to mount it in order to access the file. You will also need to have the root partition mounted in order to install grub if you don't have a separate boot partition.

```
$ sudo mkdir /mnt/tmp  
mount /dev/sdb6 /mnt/tmp  
$ sudo blkid  # to see what the disk uuid is   
/dev/sda5: LABEL="root1" UUID="ddc8c237-e8ac-4038-a0ed-f7c866d6603b" TYPE="ext4"  
/dev/sdb6: LABEL="root1" UUID="ddc8c237-e8ac-4038-a0ed-f7c866d6603b" TYPE="ext4"  
$ gksu gedit /mnt/tmp/etc/fstab  
replace the UUID with the UUID of your partition  
UUID=ddc8c237-e8ac-4038-a0ed-f7c866d6603b /               ext4    errors=remount-ro 0  1
```

Here you can see that the new disk image that I copied across to the other disk has the same UUID as the the original file system. So you could copy your fstab file form your old install across into your new install and have a working system. That will boot. On my set up I can't leave my computer like this or it will boot to whichever device it finds first.

Edit fstab and make sure the uuid match the partitions that you have set up for root and home and whatever other partition you set up.

Last step is to install grub on you new disk.

```
sudo chroot /mnt/tmp # your root partition.   
grub-install /dev/XXX  
```

In my case:

```
grub-install /dev/sdb
update-grub
```

Please read these instructions before beginning. It is no use having all the data on your new laptop and not being able to boot it up.

[https://help.ubuntu.com/community/Grub2](https://help.ubuntu.com/community/Grub2)  
[https://help.ubuntu.com/community/RecoveringUbuntuAfterInstallingWindows](https://help.ubuntu.com/community/RecoveringUbuntuAfterInstallingWindows)

---

Moving a Linux installation from one machine to another is actually relatively easy to do, but there aren’t many articles online that walk through the whole process. Unlike some other operating systems (I’m looking at you Windows) Linux is by default fairly uncoupled from the hardware it is running on. That said, there are still a few gotchas that need to be watched out for, especially when it comes time to configure the bootloader. This post takes you through the whole process and assumes minimal Linux experience: if you’re comfortable with basic shell commands you should be able to follow along.

Since there are a lot of different reasons to want to clone a system we’ll be focusing on actually understanding what each step is doing so that you can adapt what I’ve described to your situation. While I’m assuming you’re using physical machines here, this procedure works just as well with VMs, whether run locally via something like VirtualBox or VMs provided by a cloud provider like Amazon AWS. If you find yourself needing to move from one cloud provider to another, you can adapt the steps in this guide to make that happen, just keep in mind that on a cloud VM it may be difficult to boot into a livecd so you will probably need to instead attach two hard drives to the VM–one with a fresh Ubuntu install that can act as your “livecd” and an empty one will be used as the restore target.

I’ve listed out the commands to clone a system with minimal explanation as a reference below. If you know your way around Linux you may be able to just run through these commands, adapting as needed to fit your situation. If you’d like more detail, keep reading and we’ll go over exactly what each command is doing (and why it’s needed) below.

> 1.  Bind-mount the source drive to a new location so that we don’t end up in an infinite copy loop while copying `/dev/zero`, etc.:
>     
>     mount --bind / /mnt/src
>     
> 2.  `tar` up the source filesystem:
>     
>     tar -C /mnt/src -c . > source-fs.tar
>     
>     and copy the resulting `source-fs.tar` onto a USB drive or network share that you can access from the destination machine.
>     
> 3.  On the dest machine boot from a live-cd (I used the Ubuntu install disc)
>     
> 4.  Partition the drive on the destination machine. The easiest way to do this is to use `gparted` (included on the Ubuntu live-cd). How you partition will differ depending on whether you want to use MBR or EFI mode:
>     
>     -   **MBR mode**: just create one big ext4 partition on your target drive, and use use `gparted`’s ‘Manage Flags’ right click menu to add the `boot` flag
>     -   **EFI mode**: create one 200-500MB vfat/fat32 partition (use `gparted`’s ‘Manage Flags’ right click menu to add `boot` and `esp` flags), and create one ext4 partition in the remaining space.
> 5.  Once booted into the live-cd, mount your destination filesystem. I’m mounting mine at `~/dest`.
>     
>     mount /dev/<some-disk> ~/dest
>     
> 6.  Use `tar` to extract your image onto the destination filesystem, (using `pv` to provide a progress meter since this can take a while):
>     
>     pv < [image-file] | tar -C ~/dest -x
>     
> 7.  `chroot` into the newly extracted filesystem
>     
>     cd ~/dest
>     for i in /dev /dev/pts /proc /sys /run; do sudo mount --bind $i .$i; done
>     mkdir -p ./boot/efi  # skip if using MBR mode
>     sudo mount /dev/<your-efi-partition> ./boot/efi  # skip if using MBR mode
>     sudo chroot .
>     
> 8.  Run `grub-install` from inside the chroot:
>     
>     apt install grub-efi-amd64-bin       # skip if using MBR mode
>     grub-install /dev/<your-boot-drive>  # use the whole drive (e.g. sda, not sda1)
>     

# Step 1: Bind mount the root filesystem

The first command we run is `mount --bind / /mnt/src`. In Linux-land filesystems are accessed by mounting them to a path (usually under `/media` or `/mnt`). Here we’re using something called a bind mount, which allows you to “bind” a mount point to another mount point. In other words, you can access the same folder at two locations. In this instance, we are telling the system to make the `/` folder available at `/mnt/src` as well. If you write a file to `/test-file`, you’ll see that it’s also available at `/mnt/src/test-file`.

Why is this needed you ask? Well, when a Linux system boots it creates some virtual filesystems that many Linux programs rely on. One of the more commonly used ones is the `/dev` folder, which is how Linux addresses the physical hardware installed in your system. The files in the `/dev` folder aren’t real files though, so it doesn’t make sense to copy them to another system–that system will have it’s own `/dev` that reflects it’s own hardware. More importantly for our current purposes, `/dev` also contains some special “files” such as `/dev/zero`, which returns an infinite amount of zeros, and it’ll take more time than any of us have to copy an infinite amount of zeros.

Bind mounting `/` to `/mnt/src` allows us to sidestep this issue: this system’s `/dev` will still exist at `/dev`, but you won’t find a corresponding `/mnt/src/dev/zero` folder, so copying from `/mnt/src` avoids starting an infinitely long copy process.

# Step 2: `tar` up the source file system

Now that we’ve got the filesystem bind-mounted we can start preparing our image. All we really need to do here is save the contents of the root filesystem (excluding special filesystems such as `/dev`) into a tar archive:

```bash
tar -C /mnt/src -c . > source-fs.tar
```

The `-C` flag tells `tar` to change directories to `/mnt/src`, `-c` tells tar to use ‘create’ mode (as in, create a tar archive, not extract one) and the `.` tells it to do so in the current directory (which is now `/mnt/src` thanks to our `-C` flag). We then use shell redirection via the `>` sign to write the output to the file `source-fs.tar`. **Make sure `source-fs.tar` is not on the same drive you are copying from** or you may kick off another infinite loop!

> **NOTE:** In this example I’m just writing the image to a file, but if you wanted you could also stream the filesystem directly to another machine over the network. The most common way to to this is to use `ssh` and a shell pipe like so:
> 
> tar -C /mnt/src -c . | \
>   ssh <some-other-machine> 'tar -C <some-folder-on-the-other-machine> -x'
> 
> This uses a shell pipe to send the output of `tar` into the ssh command, which takes care of setting up an encrypted connection to the other machine, and then runs `tar -C <some-folder-on-the-other-machine> -x` on the other machine, connecting the stdin of `tar` on the remote machine to the stdout of `tar` on the sending machine.

# Step 3: On the dest machine boot from a live-cd

On the destination machine (the machine we want to clone our system _to_), we need to boot into an operating system that is not running off of the system’s primary hard drive, so that we can write our cloned image into the new drive. I usually just grab the latest Ubuntu live-cd from [Ubuntu’s website](https://ubuntu.com/download/desktop) website and write it to a USB via [Etcher](https://www.balena.io/etcher/) or the `dd` command. Ubuntu provides directions on how to prepare an Ubuntu LiveUSB [here](https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-ubuntu).

If you don’t like Ubuntu any Linux livecd should work fine, just make sure it has a partitioning tool like `gparted` (gui) or `fdisk` (cli).

# Step 4: Partition the drive on the destination machine

Here is where things start to get a little tricker. **There are two common ways to boot a Linux system, MBR (an older method) or EFI (a newer method), and each have different partitioning requirements.** If possible you’ll want to use EFI, but if you have an older machine that doesn’t support EFI mode you may need to use MBR. The easiest way to check if a machine supports EFI mode is to boot into the Ubuntu livecd and check if a directory called `/sys/firmware/efi` exists:

$ ls /sys/firmware
acpi  devicetree  dmi  efi  memmap

If there’s no `efi` folder in `/sys/firmware` then you’re on an MBR machine. If there is an `efi` folder present, then you’re on an EFI machine and we’ll need to create an EFI partition as well as a root partition.

From the Ubuntu livecd open a terminal and let’s fire up [gparted](http://todo/) on the drive we’re going to partition:

sudo gparted

Using the selector in the upper left, choose the drive you’re going to be restoring to. On my system this is `/dev/nvme0n1`, but depending on the hardware in you’re machine you may have a different designation such as `/dev/sda`.

Once you have your drive selected, choose `Device -> Create Partition Table` from the Device menu. You’ll be greeted with a scary looking screen like the following:

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAApAAAACsCAYAAADMkpyvAAAAh3pUWHRSYXcgcHJvZmlsZSB0eXBlIGV4aWYAAHjaVY5RDsQgCET/OcUeAQEHOU5jatIb9PiLazem7wMmE32Bzvsa9JkUFrLqDQFwYmEhR4bGC2UuwmXunItna8kkuyaVFRDN2fZDe/o/VdEw3NxR0dEl7XKq/mb+o2nleUZsScal0Xd/9Lecvke1LH024xp3AAAKBmlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPD94cGFja2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4KPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iWE1QIENvcmUgNC40LjAtRXhpdjIiPgogPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4KICA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIgogICAgeG1sbnM6ZXhpZj0iaHR0cDovL25zLmFkb2JlLmNvbS9leGlmLzEuMC8iCiAgICB4bWxuczp0aWZmPSJodHRwOi8vbnMuYWRvYmUuY29tL3RpZmYvMS4wLyIKICAgZXhpZjpQaXhlbFhEaW1lbnNpb249IjY1NiIKICAgZXhpZjpQaXhlbFlEaW1lbnNpb249IjE3MiIKICAgdGlmZjpJbWFnZVdpZHRoPSI2NTYiCiAgIHRpZmY6SW1hZ2VIZWlnaHQ9IjE3MiIKICAgdGlmZjpPcmllbnRhdGlvbj0iMSIvPgogPC9yZGY6UkRGPgo8L3g6eG1wbWV0YT4KICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgCiAgICAgICAgICAgICAgICAgICAgICAgICAgIAo8P3hwYWNrZXQgZW5kPSJ3Ij8+4l+/DAAAAARzQklUCAgICHwIZIgAACAASURBVHja7J13fBzF+Yef2d1rOnXJlnuvsmxLlrsxBDCmhRIgEGJKIKQQICSkQAhJSKOFEBJI/wUSSCChkwLYwRgM7tjGvctVlmWVU7u+u/P7407SqZ+KjQPzfHwf3Z33Zt+p+933nZkVr7zysgSIRqOsW7uGsqNHMM0wYKNQKBQKhUKhUMTQMAxo2H8AQ0qorq5m6X9fA2HiMAw8HgMQqpwUCoVCoVAoFHEklmWTMmw4RjQaYembr6HrNrpmYJoWkUjsIIVCoVAoFAqFIoZA0wSGw0DMm10shw0fjGHoRKOWKhuFQqFQKBQKRacY7ox+aJpGJGKq0lAoFAqFQqFQdC0gU906lm0jpQpZKxQKhUKhUCiSEJAOXWJbSjwqFAqFQqFQfJwIR6IEAiHC4QiWFdt9x9AFTpcLb4obp9PRsYCUUiKl2rJHoVAo+g6baChCKCIxUtx4jHZ2tZA24UCIYETiSPPiNf43cyptk1AoQtQ28Hqd6OJUtSuJOlEoPiaYloWvpg6vpnPpuMEUDcgiN8UFQGUgzMYyH6/tKaXWssnKSkfXtTZp6BMnjLlXCNWRFIr/VaS0iUYsbE1D+1+1VUbxHauivCaC8Lhxad079tQqW5toSKLnjmDcxFF4fIepiuoYWmvxqJExdirFxePxHttNWdSB45StQEmgqoLS4w3UmwZpHr1ZpEW99B8+nglDnRw/UoNtaH24CVz7501KPLawy0fEEhid1YlC8TEhEo1SVVXDleOH8L054ynOctNft0i1IqTaUfo7oDA3lYvGD8KyJWsPHsPldKBpLTuMPmH8aCUgFYreYJv46xqoqq6n2teArzZIgz9CxNZwuvQTKuqkFaSiwsSRkYnLDBIVp66I7NBWGaWmJo8L7v4e37psHL7VK9kbtKirSu7YkogTp3YKla2MUFU1nKt/ehufnj4Q37tL2BJy00L7yAjVvqFc8f0vclFBDsff/W/bY04h8RisqmfQoof45bc+zdneHby4tpoUjw6BeqrGfpaHv3YhRbnlvLl4D2aqA/1En7erX7a2640dVIVH89nO6kSh+DhcrmybqsoavjV9FJcMzUAL1iEjIaQZQVrR2MuMICMhtEiIwn7pDExLYem+Y7jdLhL1YjyEreZAKhQ9usSZISoa0ik693LOmTWRUXnpeAhRXXaALWve5F/L9hNNdXJCopN2mIq6yXz1Dzcx3TjA3759P8uimWQ4TsVRqxNboxHCGWOYOjINjzaBqSN0lmyYzDeTPPbtLTapfR2O7E3ZtjeeStny62SOOYXqzi8nMK84Hc2uYe2qXWiejJi9bfYLlvHvT/R5u5KeMql6Upc+xccNX009V47J5fQcB1aDjy47QSTEJ3I8HBiVy6uHasjOTk8UkKhOpFD0RDzaYapCo7n2BzdzzhBnQtjOS+7wfOab23n5tT3g1fHX+qkPmURNGxvQHR5y+3lxCZtQvZ+ahghRW6IZDjypXrK8Rjw9Sai2Fl/AxLIlEg3D5SI9w4tXlyA1NE00qllqjlVQ60plYK6HmKbqKv3WmDRUN+CP2EQtGykFusMgJc1LRkoSNjkEYBFok18nhrMjW92k173LE0+4mZN1iGUbTAyZ7LGQkS6Q0iZc76fWHyZsgaYbeNK8ZHodca+hSUO1n4ZIzGZbgqY78KR1UBayq7LtpAza3C1YNByvpF7TcHm8ZGW4cHSgO2NjcXfrLPabPs1/ol2hEHbBLIrTNOyq9azcqyG0WkqrIlhS4GwvH53mwcZfVU1VEHRvBoOyHAggWl/DsdooxMtYb3NeHZfXz/FjYUKmBUJDN1xk5aTGPIm2SUOtn7pg+3Z1WScqEKf4iBONRvFic8XQVKx6X/LiLxLiymFpLDlUTSRi4nAYygOpUPTCPULAJ5n6hRtYMMSJDB7izaef5bUPjuCz3OQOHcdIvYRAqoFW62bixVexcMoohg3IxKtFqTn0Fr/86X84aLsZtfBabj+nkFGZGv7yvaz+1/P8fUMtmekG4bowgy+4g++cPoR+GR70aD1lu9bxyjOvsikkmv0s+kgWPfIEiwBzxzPc9tBKyHIS8MGwczpK39FKOJj4fRnMuOEmzhozkLzsNDxaBN+RXax87RVe2uAjI9NJpDObwg6MQAoT2uR3KY8+XN6+rQ8up0FO5KpFF3C6uwK5eSPPHCXJY7/HSz4Do14weMEibllQyJgcB8GKfax57SX+sfI43iydoC+TmTfcxFljBpCbnorXJfFXlLD238/x7Fof6a3LItHL1aZsVxDWNYZe2FG9aM2/FRnM+eJPOb1fNu5wJTtXvMqTL2wmnO5o6yOTEikt6qu7U2dxMVTVx/lPbOdBJ4VzJuPVJMfXrmWvMZAzF13FRTNHkWv4KavW0Gl+dpmUJnUd5sGH0xJMufU33FLsxLf053z1mVIyND/Os+/m758Zgb3jWW5/aAUB0fK8+2ybzGEXcttVpzF5SDp6qJ6qo+/z9C9fpkR3EAplM++aL3VsV1d1kuHFqUSk4iNMIBjmk3leHCE/diTc9P2Pd5RwyaD+TMlIBWBzbQOvHj3O9yaOajrGKSUL8lJ5rTKMYcTmfmiNAlK91Eu9uvEyw/hdxZwzPQ1NRtn5j8d4YlUpYYeTdLdJoGwzHxwJ43XYhCL9KT57BpOG5eCxA9QGBV47gC8Kw674Jt+5ei7jMm181SHcgyZx3pfu4IuTBdV+G+wwZkouOUaEuiofAS2DoYULuOWr55FbE6Zp/wQZpa7iGEfLjlFWFQRdEKoJM+jyztK3WubJtghZWYwvnsjoARk4zQZqwwbZI4q4+OY7+fYZ6VTVm8jObKoO4A+3n98au2NbW6spSfLHmvURBl95B99ddBr5eR7siCR1YD4LP/9N7jo3i+raKCErM56vTFIIUh/SSBswkXM+fyufGRHBF7JblkULY1rboHVeL74wdmMCwkXOwHQIRNDSBjD1vC9y15UjaaiNtg3+Sptgd+tM2gRrTkD+G19WhIBewNypKWh2OWtW1zD3y3fw+TPHk+ex8AcM+uelNc8NlZJwp3nQCRJk84a9RKRGZv4khkTCBKJp5E8agobNgQ2bqdUhnHjelQexU2Zy4+2fZObwNMzKUg5Xm6Rm6YQabML1GtO/2LldTXRYJxFsNbap10f4FY1GmZImsAIN2JFQ0+vySRO5b1cJGyur2FJdzX27Srhk4oQWx1iBBqakQSQSbUpPeSAVip6Er00Le+AgBhoC7KNs2lqLOzUtvopWw+F0xsKUifOsZAWv3383T+2VGC4XWvpsvnDOQAxzL8985yFePmKSc/bXefSmAmacXcxfNqxCy/Ry+G/f5oa/OEjJSMOTUczNP/wMkweMY2IGHAo2OoqO8K/7f8yr5Rqa00NOmqTemsWnu0jfdLubt11pYWslix+8m6f2uBn/mW/yg4tHMOGSc8lf9jcOZnRiU6bkUG0H+dWL27U1N90AX7vyMYljbUzvLC5bMAiHrGbF4/fx2Ko6+i28jfuvn8zoT15I0ZI/siEhX288eDdP7c3ivO/8hBsnZ1E8YwR/+ccRbGeCF052YkOGE4cwkisDu5LFP72bP+2Q9Dv7Nh66cQp5889gyt93s162yrMdpsE9i1u6UWfSCtHgnt33+W/0bYaiuKbNotAjsEvfZ2VwBjcVetHsct64/yc8sbUBz/zb+b+vFMbFmoXfPa/zdrf+XRo2bmB3dBKT+xdQNPAlXq4tpmisgbAO8v6GKhxCx0g474pDEjGsP3lOgYzs5Nkf/4Il1RLd4SE7QxBxzObcTu1KaFYd1smf2GUbeJQXUvERxbJscohgh0OQsH3jeN3g+/Pm8P13VyAR/GDeHArMmMhsvvHSyBGyxYNnlIBUKHoiICUImSh17PYn5bf5QiMlK5U0y49v+GhGOgVCjGHRw39gUaIsyu1Hrm1xtFZn3BXf5PPnTqCfK2F7FMuNy2VDMDFtndSsNFINkGE/cmTX6Vch0WQntmba7F/yHrsvHMGk1OGMyrXYUaGT35lN7eQ3XQ/jC7RvKzLSRWl3fqwYMZrRDoFdt4W33q8no5+T6ndXs/OaAqZ7hjNmIKw71NIml6uG3fuqkZMHkp6ZDlarp3G1WaiRYAM2wRqdMUmVgU3UhJQsJ7XrNrDv+ilMdQ9kaH9YV9YqWByNIsd2r85kxESOPgH5jyuuYNDD9DkTcQmbQ6vXUZp3LkN0gV2znbW7omSmO/CbVsJPLERX7VpC2P8B722/ioLCwRQX9mdxZSH5LjD3rmVlhYEQbooTznvE5cRZtpUNZecyePAkbnrkIU5f+y7/+debfFAVRU4Y0rldLZbVdFQnkq0NErdDjW+KjyoC24wiZbSFgJT1PqzaWoSmgQTz+BEs3Wr1Uw1LaLG+1Cwg1SIahaLbAlLXEJXlVNiS/vpACiZl8c93grjSHBiAbZlEbA13ezP5ZawjSyFic7Qih1j5+maOJegO6d+L32FhTbqcWz+Zjze4lzeeXcau4FDOveFcJjoAAYjGC6NAE7G+bEuQMon03Q40G6Ro6/hLHHAwTUwJaAJNkzClc5tke/mVokNb2z1vd45tHf5OYpWuQMM0o/EPAhFPv8lbZXdsrwwFCE/+XPfKwJZIw4gtbJIS2243092uM3mi8g9IO0IodRanTXQirEOsXH0YOy9+Ot1AFza2bK+qu8qDEzd+1q/cQWjqVEbMnMvC6nw8mOxYuY5qpwBXUcJ5S3G500nlMM//+EGOXHg+F5wxlfHzL2X8zEKe/9H9vELXdpFknajroeKjiqYJjodM8pwW2M0dc3swwgNHfHx31kwcmuDHq9bwzUFZTPEmXMA0SWXYQtO0pj6iPJAKRY8wcNRvYc2eKPkTnUy68itcF3qW1z4opcY0yBgwiqKsOjbsLm9/sYSuwdHDHLIk4410nJVreGVpKSEpMdJyydYDhBFk5PUnRQNr/0peXryKGr2Ogs8ujAkVgHCQgCnBlcPQwS7879cQdKSQntp1+qbTiYPEsHXihVbD0CFQE6bfWUWM1kGGjnGkQpI5pQub2suv6NjWjPR2LvDdOFYe3Mu+6BympE3mrOlp/GpVHf0XzmaCLpCBg+wrp/1V1q0kRgvh1Ym9qQ7Zdb20SttfJxj/6XmM1cGuO0hJpUCXNqYtAS+5uU4ih0y0btaZNHTEoROQfyRW0CR93kwmOMEqWcuq4wZED3LInMOE1KmcM+ff/OKd4632xhTIZPIgJPXvL2dd3RTmjz6Xy0cYENrEWyvrcACe6YnndeLOtLEsN17rEMue+TXLXhjC5d/7AZ8ZM5QZU3J5aWVXdiVTJxqOVLW1j+Kji8Mw2FQfoiBbgtXsYfynL8jd04uZGKgEKbl7ejGv7NjOZHfCRqlCsLHexjCcKoStUPQOgcdTy7I/v8SMe66kIG0k5998N+cnOjhq3uWhO55iU9tLNWgOPL6VvPze2XzrjFxm3PhDnri6gYDtJi0lwts/+wZP7hYEDxyixh5Ndv6V/OQnMygLpDDE2ywHdLGfrbtCnFaYxryvPkR+TRRXaCX3ffcVIqKL9EsM0hOXnSaOA1oO5939KLPqbVIzvTiFRdnyZXxg6lhd2EQ7+ZW2hlPrwNa7X6K+9Z0yyR4r0BrW8MpbC8k/dwDzbnuQ4hujGF43BhFKFi9mo2m02dhadiR0m953Zu+LHEi2DEQ/Fn77IeZJL5mpTjSC7PjXG2zXXTjsckrLLORYL7Nv/jH+R+/mic3drDPhwH1C8m8TimZwxpzRGFjsXrWBSqcLT91qXlq+gDvP6sfML/2UP32mjrAjPeZ1BxAaLl8yeTDwyu0sWVHBaRfk4TBsqt97h3VhA016mdXivAaZkRA1/S/hke+fhaeqgsqAg/5DNZBByo/Xo/u6sCsxx53USQZKQCo+wgLSabCsRnJ5Jrik3TTmf7doCpavHMtfB8AEafPdoslEj+xpEo8hG972gdPraNKMahW2eqlXD18YLlJ9y3jwu4/y1ze3sL+igbAlsSJ+qg/vYvV7e2lwae16f6QEV6rFjicf4L6/r2T7sQYsVxppzihVh/dTZTnRDScpe1/i4afeY9dxk4yRE5g8aRCehuPs27aTspATlyvIyif+yD8/KKXOcpGZ4cDvj+LQBQ5vF+nTTp6abIxQdbwew+sGfzmb3/gDD/yjBHe6uwub9Pbzi8DtDnVga1txbriSPRZ0j8a+Z37G/c+tYXdlGN2jEyzfxVtPP8x9/yojJZlVEa3rtzN7DR1np2Vg4BA+dm/cxYGKemxXBplui5pDm/jP7x/kZ2/WkOrW8XhqePP//spbO4/jtwPU11o9qDNwek9A/q0I4ezpzBtjgLmPlWurcDoFLq/Frqcf5IHnVrGzPISelkW6Eaa6bD+btxwh7DBwJ5kH3Q0li99iV0QirSO8+fpWcFrtnDcm/5xGgLJSP0b2YEYOz4bqfbz33K/583qLtPQu7HIaOLSu6kSocU29PtIvXdMIaE5eqNRj001sC2lbRA7txKr3NX226n1EDu1s+iyB5yp1groDXW/uJ2LhgjNk6+cbKhSKbmCbBIIRwlELy4qJD03XcDgdpHicGESp8wUJSw1PRmp8IUbTj4kEwwTDVmzjbgS6ruP2ekhxCEASDYXwh0xMKy7wNA1Dd+BJdeLSBFgmDf4wYdPCRqDrTtKaNqvuKv1EERGlrmE8X/rdVzndXcG/7v0uT+2x4oLOTZq30ZPVhU3C7Di/Hdkq2ymj7hyLJBIMEQjHbdINXC4XXreOJuJ5a/UbM+DHF7TA6SE3rYN9EDss267qxSbYECIUlVjSjrcJHZfbRUrC4y2lGaUhECZsCjzpjZuQd6POEnyKfZl/M+gn9dx7ePiq4Vhb/8ptP1sJXmcsrbh9jeeSQqBrGrrTRZrHaHFMV3mQVpjamjBRdLwZXpzRjs8rzTB1/gimGfeUagZOt4tUT2N5dmWXlVSdKBQf6cuVLfE3+PlSVpDTU4JgW53/QNNZFvDwxxoPqV5v88MVVAhboeiLaLaOJ8WDp8MArkFaVhppjZ9lS2+bw+3G4W7PKRSfZ+Jyk+HqyHEkQdPxpqXgbTcc2XX6Hcc1Be601OYnqySEODu1qbP8dmhrO7/pzrGAw+0mw91O+Uugnd/onhRyPe2Eb0nG3q7qRcPtTcHdVahYN0hNM0htYWs36iwxPNVn+bcJRweyYPZQdBll6+oNNOg6qa3sy3B30N67kwfNSUZ240R9i0Cgk/PqTtLTnZ2UZ1d2JVknCsVH+XIlwJPi5vc1cDACV6T5cWM19YOmg4AQOs/VenijIQWv140QLfuvEpAKhaLl4NFK/qrh4WOGFSU6ZCZzh+rI8DZWrKvH4fSceJH1YZ1XofiYoWkanhQPS0I67waczHcHmewMkRPfuqfK0tkcdvNe2ENAc5LqdSKEaKMVxdlnnaZC2AqFIq4YTRpqw0SkhjvdQ4quiuTjKSIj1NZFsYSON8ONS3zEz6tQfEyxLZuoaWJaNjK+lZXQBIau4TAMNL1jfag8kAqFIgEdb0ZjyFZ5Hz++LgoH6ZmNexKdxHbwYZ1XofiYIjSB0+nA2ZFPoZNOqDYSVygUCoVCoVB0C+WBVCgUCoVCoVAoAalQKBQKhUKhUAJSoVAoFAqFQnEqCUjLslRJKBQKhUKhUCiSQtTV1ij3o0KhUCgUCoUiadQGkAqFQqFQKBQKJSAVCoVCoVAoFEpAKhQKhUKhUCiUgFQoFAqFQqFQKAGpUCgUCoVCoVACUqFQKBQKhUKhUAJSoVAoFAqFQqEEpEKhUCgUCoVCCUiFQqFQKBQKhRKQCoVCoVAoFAolIBUKhUKhUCgUSkAqFAqFQqFQKBRKQCoUCoVCoVAolIBUKHqEGaTqyB42bzlEg/xfMtwmUFHClnXLWfzWVqpkV98rFKpvKxS9wyJUXcrezevYWhb5n7dbRuoo37+d9Rv3U9fLPmKoxqH4WCGrWfLAN/jLjija4Iv54f3DSNXbO66BLf94hN+9eYyMeTfyzc9NJ1ucTDvbOT8+3v39j/jLDhN98MWMPaOAHB2QHXyvUKi+rVB00mbqWP7IHfx+Qwgyz+Bbj3yBQlere5Ld/+A7P36NctvB1C/8kkkDnYgP2aak7qXas1tWseShbzRdK344ZSTpvegjSkD27naXvc/exb3/PoaNg4IbH+E7Z2d10bhMtv7pa9z/Vg1S68f59/yMa8c3V4N54AW++/1XOGyBPugi7r3/KkYbbe8sDi75DU8uO0BFVQ0NwQi27sSTlsugkROYNv8cFkwfQopoPueWJ77OA0t9kD6Vz33v65wzKCFR+zAv3fM9XjhoovU7j+8/fA3jjObfSGM4n/7Jj/nUUK2HaSX0Df9h1ix5g7fXbWP/sRoClo43awBDx0xixvyzOWNqHt3rK1HW//YrPPJeENlFUx/72Qe59wIDiaTLG6/oNpYt3oMvIvG9s5QPLi/mrPSTqCDbO39a79trU522+/8Cz2lf53c3T8PR7TbWztmSasuxOqza+iavvPYem/YdpSYo0b05DBk7mZnzz+Gc6YNxi+7Y3kUpJGVXR+2/p8e17cMl/3yQX//3IFU1ASI2CMNJSlomeUNGM2HqbM48o4jBnRRw5/noZr+4MK9p3Eq+3k72xb6GbUteY+WuUo555nLrTfPIalE8Mrm+fVL6T3fHyauo/2N36iuHre22u55fG7rfr3rShjvrL90cB1r9Plq6mId//Fe21EuEewyX3/0dLhvd+dVE1m5k9bYwEkFW0Szy2z385Lqyk7MpqZROuK0fzrBgW0R3bSfywTqsnZsxDx/EqvVBJO5mdTrRM7MwhgxHnzgVZ+EMHOPyQTvVIu4Gg0cOwyGOEZYWZUfKsciKFaqsZv2LL7KuysYxagHXnjMaZ3wQPHK0AQkIx1BGDk2sgjDb3nyHI1a8e5a9x5tbLmF0kbtNw6g7sps9hxI6vR2mobqU3dWl7N6wnKWzb+Ker8yjn9b65mYzf/vVcwz+/tXkp/ROEHUvLUnowBIef+RZNlaZCU3bpK7iINsqDrLzsJNxBVcw4lTwGhhDGTfaw5odQZyDxzIyRXy8zt+LNta9tmxzfPnj/PCPG/DZCa2irpx9649zODyMedMH4+6zfCVr14kvX39lGceq/U3lK80wfl85Jb5ySrasYslrBVx+261cPM7bzk3picrHqVI+7V03qtm6dDHLSi2MiVOw/2ecXH035p74fnsy23DfjQOybhN/ffTvbK2XCC2bOTd9lUtHu7q0v2bDarZHJGjZFM+eGLtGf7it5RS06RQRkFblcfyv/J3gf1/DkZONZ8wI3JPGYHxiJnpaOpozZo4dMbHq6zArqokc2UP9O0swa2rwLLiAlEuuQs/tf8oUoGvYCAbraykxbepKS6mXE8gSIP3befu15awPS7Sdqcw/c3TME2cd5fDR2OisDxjJcHfiDfZalqyuae74soa1S9dyZeHpre60E28QM8k/+ywKMi3qynby/rrdVEaiVKz+K3+fVcStM1JadVxJ5Mgb/PYvY/nxl2eQKXrX2JNNS9at54mf/42N1TYSgTN3LEWTR9EvVSNcfYTd23YhZs1iWLfFo86QOVfw2WHxK559hJUvvcv+iETLLeKicyaSKgA0siemAcHkktUGcd6dDzHpUC0pQ4eTc7Jvtdo7f1/eUGrZFF10IVNbVJrAGDQUvddtrBttObKdfz0fu2gII4f8+acxKc+JWVvG7s3bcc0tbtv2u2N7T+06qXWdTdHFF1GYYeKvPsz299ex7VgQs3oLz/3i92T95OucniO6mY/u9otTuHxORbrVBpMZJ3tWX53S3X7bi37Vkzbcgp6MA02/PcLiX/+WN49GkcLF6Eu+yk1zsrte4CF9vL96J1EJWnYxs8Y5Pvx2dSra9GELSLu2htr/e4zwquWkFhfR/7rPoqe6kcE6CDUgqw5jVkRBxu8phQa6E8PpxMgfhbe4EKs+iH/LNipvXoTrtDNJv/FWtLT0D38c6TeSkamCkhqJefQQpRZkGRDdu509ERm/cd7OtmM244ZoyOpSSgMSEKSMHEWe1jzIVKxaxraQBH0wk8b52bGjhtDWt1l1fD4X5HXQe/QMJi24lEuHaoDkkvd+wTd+t4GAbGD3tgNYM/LbVrK0qVrxJH+cOIJvfKJf71ZSJZWWSclrz7PKZyPRyJp+I/fc8gkGJt5amX4azBQ0QFav48lHnmRFdS6n3fA1Pjcju5O7V428wnO5sDD+MbqRg/98l/0R0LLzOfPC8+mf+GPZLCDtind49GtvUlcXxZk1nClnXs7Vn5wcn1dYxeIHE+aK3H8lo3SQocOsePE5Xlu9i9I6E0dqOrkDxnDmoi9x7qhWnV02sOLR2/nN+2FwFvHlx+5gfqrAPvYf7r3zWfY55/L1x7/CdBeYe57hWz96jXLbYMSVP+UnF7v5b+vz96UDXktjzJxzWJhMuLXbbSz5tixrj3A4PpPbKLic2z9/evxCCUgLS+pt6747tvfQrpM7iKQxZvbZnBPPz6WXX8Ly3/yUP6ytxq77gBf/s4vZ101I8EQkk49u9os+Kh/Lt5M3X3qVtzbu4ViDxNt/DEVnXMxl505qvgmTQfa+9TyvryvhUFk5VbUBItIgtd9Ipp7xKT5zwSSyulAs5o6nue2ap2OXi6wF3P3o55iU8JtO+3aCrUtefJm3NuzleFAnLW8s0xdexhVnjm5ug33Rf5IaJ7tbX2bf99se96uetOFWxdOTcSDmr2P9X37BM9sakMIgd+4X+dplY5KKWMiqdazeYyLRyJk+i3HOmCe0ducS/vHCUjaUVBDQMhk22NHhYpNO2xB+Vv3yazy+LgiOSXzukbs4Jz6RPrjmMW57bA0BPMz8yqPcPjfmoW3PpuSuOd2zG8D2recv966i7EgNUXcuo6adzRVXLmRiRvL1f8JjwoFlSzh6YbF2zgAAIABJREFU46fR/T7yPnc93nGjsMr2E9m2nujBvZiVx7Hr67ADIexQNPYKhLDrajErjxM9uJfItvXY5QfwjhtF/+uvR6up4ugNV+B/57+ngAQfwdiRRqzy6w9z2CcBkwPbdzWvArSOsnVbLKQQPXqIoxYgHIwYM6J5Xol9hPfeLSEiBcbw07j28jn000BG97H8vcNYyd1y4s3t1zS/JRyJtnNIGlmZBkLW8cEzT7C03Op53pNNyzzA6rVlWBKEczyXXHtGS/EIYHhJdQtAUrX2Dd7eX0ewtoRli9dRfYKmcshILZXVASJmlIaKvax8/hf87IW9RDq5O1z1+/v57WsbOVAdIGpGCNRUcmjPYWra60rCy4TJozEESPMg+w7HyidUsofDFsjwfvYesQBJ7YEDVNmAlsOkSXmn8PYISbSx7rRlTyqpeiwxc/dbvLzuKMHG+hY6el8WRJ/0sZOAI4/511zERKcAbKo+2MBB8yTko5fpyuo1/P6HD/DUW1s47AsRjYapKd3Gsmd/xr2/eY/KpphzgH2r3mLVlr2UVtYTilrYZpi6sp28+4+f87OXS4ic4L7daOtfl22jtDZMNBKg+vAmljxxH/c9v5twn3aZPhxzT2S/PZltuDU9GQdkhP2v/orfvlOOKQWp+VfxjZtmkZPUmCGpWLeGvVEJWj9mzh6DAwjvfoEHHvwbb+8ooy5sYgYrKdlb1mxLO+29wzYkUsgvHIdDgDT3s2NPo/Miyt6tuwhKEI5xFE5q9AS3Y1OS15zu2N1kf6CUPSUVNESihOvK2PH233jwwRfZ3Y3Gf8I8kNI0qX70AcIfrKffxZegOzUiu7YgrTDC0BC6hrAFWBKpCdBE0x2GBLAl2BJp20hLIkO1yCofQneRMm4MruEjqPzjbwhtWE/2V7+F0D+kiXPCy6gxg9E3HsC0Stl/KAo5FWzbWoUt0hk4QHKsrIGSLdupXziP+kOlhCSgD2DsmNTmiet73+XdwyYIB6PmzGTouABzBi7hlVKLIyuWs/fiaxjv6LgzSDNETdkO3n51TUyICJ1Bgwe0DT3oAzn3xiI2/e45dvi38o//e5OCb+X3LO9JpiUDRzhcbcd/MomJnS5nFqQNGkyGtpsqKcgYNLhrb0BPqy51PAs/NZ9h8gDvvLqU3fURDr+9lK2XjmFae7fKoZ2s+aAeicA78WK+cPlU0qI+jhyIMHqI3m5esiYWMFTfTolZx/6SKuTEHPbv2kdEAlYFe/bWIkencajkEBYgMiYxZdhJCAyYB3n+rut4PvE75zS+8us7OC2ld22sO21ZeAs5e24Om96uxArs5fVf3clbuROYc+YCFp49nRHtLaPttu190cdO8rCSOYEJAzS2HbKwfWUci8BY48Tmo1fpSj8b/vE0KypMpEhhxLwL+MRYwf53XmN5iZ+qtX/jmbVTuW12WgtPksiYzEVXzCUvuIdl/1zG3oYIB99ayvaLR1HYyeQvfehZ3HRlEekCcPRjuN6dvu1nw3N/ZUWFCY4BzLrsU8wbGGLba8+zZHcDB19/keVn38U5nYVcu9MG+3LM7f6VOPlrQw/7VU/acNvLaA/GAauM9ati46xr1Cf5+u3nMTzZCYN2OevWlGBK0PKKmT3KAFnF8hfe4FBEIrQs8s+7iNOHGVRtXcqrKw4Slq3aexJtaMGUQsYam9kWDbJj8x4is6biNA+yZXsdEoFj9BQKGhdntmdTMtec7tidWOYpozjj4jPJz4iw9+1X+e+uOiKH3uDFFQu466yspFaanxBHhwyHOfadrxPdX0K/C87HrjpOeMcWrIYGZNRCRixkxMSOWNgRM/YKmdTWB6ipC2CHzObv48fKiIWMWlgNDYR3bEb6Kul3/nlEd+/g2HfuwA6HP6ShXiNvzGjSBCBDHNpfhlm9la1HLYR3Cp+6dDJeIYns3szOUITDB47GhELaaMYNaOwUEXatWEOFDcIxlrmz+iP0YcyePRgdsCvW8t6uaCcD2fUsuv4mbrnr5zy/uQaJwOg3j4vn92+3ETiGnc/N1xSRLiSBHS/y52XlPZ6QnlRaoWBMNAN4U1sJQpvDr/+cu+68k2/f9XNeP2ThmnIdP/zBV7n19h/wo+sKcJ2omssYz/xzPsGZ51/DZ+fHwkoycJiDFR2UhmbgiFeZGQpieQcxfsoszrl4PqM6GLi0AQVMztMBi8P7DhCyyti1pz4+x8zi4J59RKyj7NsfjA8SUxh7qs2a7lYb62ZbFqkUXXcnNy8cR5YhQErCFTt4+7nHuecb9/LkmopkgnVJ0Is+9qEMKx5SvY0XljChpg50ovLRu3RlYBPvvl8bm9888dPc8eVLWbjgEr749auY7BIgG9j43gdt9mbUUkcy84z5nHnhdVx7Rv9YH/Qf4kCF3cXN32AKioooKiqiqGBIm5vMTvt2cDPvrou14fQ5i/jyxfOYPuNsrv38+Qw3QEb3snVXsE+rsy/H3BPTb09mG263Qns1Dsiwn0A4+VCVXb6O1fstJDoDZsxmhBFbt/DBnigSgbvoKm777ELmzz+LT56d33beapJtSGQVUjzaQCCp37qZEhPso1vYWmmDMBhdPI0c0bFNyVxzumV3YvVkFbDggjM57fRzuf7WK5jkFCAj7N68k9CH5YGUpknZPd9ED4fInD6d8PatyHAAYWgxr6KMexYtgdBtpKaBgJIaP87R+QhN4/iebYzMTIm5IuMeyNhvbKRpIy2b6NFSRJWP7BmzqP5gI8e+920G3v/Ih+KJdIyawFj3W6wN2hzbt48j2ZvZZ4JragFTp8J45yrWB7bzwfYSUg9GYhU9diKjGks/vIMV66uxETjHzWZ6tgB0hsyYydBXD3HA9PH+yq1cU1CUhJgSZORfys03X8KUDred0ck9/XNcs2Evv32/jq0vPMtxR0+HsyTScrpwNpoSaMAvSZgQLYnWHqP0SBmWblMTjTXL7DEzmHvSalAnJzcbjePYdpBAqIOycE7i9Ln9WftWOeH9S3j87nd4dsIczr/sMs7Jz26/M+nDmDo5i3+VVhIp2cMBXwO7jlpo6VmkNvho2LuLEl8du4/ZIFxMLJxwwgRzy9Ejl5lXfIppWQn3kFpOfB5QL9pYT9qycyBzr/s+0z+5l/fffZu331nFtuNh7Ib9vPm739J/6D1cOKiXtvdpHzsJ2A3U1ccuiEJz43GLE5uPXqZrV5ZxLCoBnQETxjf1b5E5ngkDdTYfMDHLj1JhQ5rWQR/sn4NGObYdwB+0+8i/0bZvW5VH47ZC7fKfc8PyNq4tfL56bFI6tqDbbbAvx9yeh7E7vTb0akzoRhvuiO6OAyKV3BybqsoAkdJlPPaAxtfuuZ7CLrdbsylbu4aDpgR9ADNnD8cArOoKqiwJGOSNHN5p5CvpNiT6M33maJ7dtZOobwubDkXJ3L6JIyYIxyhmTs+NC/n2bUrmmmN3w+4OW0bGcEbmaGwts7B8VdTY4Emi+/W5gDz2yAPI2lrSp08ntHUTREIIXWsSjtgSqdsITUNqAqHZNFiScbd+n6zxBQD4dm6h5g8P4tUFMiGUjRUXkXEhaYdqCW3fTOakAireX0f5Lx9mwB13nvzB3jOe/BEG63ZEiZas5xVrJ1EcFEyZRGqqYOpYBxu21rNp2Rtkx+88Rk4c3zQfJbh1FRtqYo0xsvUJbr3miTZhiLr1q9gUKGJm61CClsdp1y+i2L+MPz+/kVoJ/towzpQuqlZkM+/6z7Jm9+9ZX1dOea/Gpc7TEqkDGZSmsbXaxjq6gx01lzAk+9Ra0qk7jOYpFB2N6yKFKdd9j7sGvcTLi1ewvSJM5Y63+esDm9l7yw+5dVZ7bn+DUYVTyFj8Fj7fbja/V80+U5A96yLm7fwbr5btZvN7PkpMiXCMp3CSl5NSMpqX4dPmc3oyE+a70cZ605ad2WOYe8kY5l54GWv/9FN+tbwcO1LCuk1VXDAoq2e294FdH4p+LN/C5mOxhqj1G8Ig14nNR2/TFUJDtJiDlPjLxoO0TiWhpmld98H20u1x3xYYnnTS21wpddJdWuf9sAdtsE/H3KT6eDevDT3JUw/acFckPQ7oOXziluvRn3uE53Y0ECldym8eH8D3vnkeQzsTvXYpa1YfwQT0wTOZNVRv07KiUSvJ9tVVGxLkTJ/JuL/vYluknI3rN5O68yAWAseYmUzPFZ3blMQ1p2d2t86yhWnK5n6a5EWoT0PYdUuX0PD++2QVFhHesQ07EIjNXzRtZMSKh6yt+HszHsY2cY4Y3yQeAbImTMYYOqbFMa1/H/NESiy/n9CuHeQUFtOwagV1by89+aO9yGDCxAHx8Mtm1m2PgD6CKQWZCJHBlMIR6MKm5oONlFiANpCJEzLjA1SALas+oL6LWpf+D1i12d+2cWhuBo4vZNbFn+famZkIJObRJfz5lb1dTkQX2XO55vJJ9MXWZJ2mZYymuDAzVj6RHfzzrys43mlc0sS3731Wvb8Pn8mphZFJ/nk38t2fP8YDt53L6BSBtHy8v3QDNR3UoXNcUeyO3zrI0te3EsTLxCmnUVSQg2YdZOkbWwhKgWNsEVMyOp8f2hyyCRORXX3fVxeiZNtYT9qyJNTgbxmeMrIpmDwUQ8T/3x/sZbivl33sJCODJfz7yf9QYsY9esXT49tbnah89D5dLXcgAxyxBRPHdu7EFz9I1uxiZ5kVu9AOGEhub644woXTEV9uUFtLXQ8bhZadR258wYYr/zP8+FeP8fhjia9H+eaCfifkRq4vx9y+67cnsw13fDvQk3FAeMZw8e23sHCIE4GkYfs/ePyFXZ2GYK3Da1lTagIGQ2fOpFGradn96WfE2vDxrVs4ZvVNGxLZMzltkguBxZF3nuL1vSZSuMg/bVZT+Lojm5K55nTH7g6vuEc2s63aBgTuvAFJb+/XZx5Is7aGw794mCGnnUbk0EGshgaEEDGJKiVSCoQtwbKRuobQBFLEFs8YTk+b9BwpadhhqynsLRt/a0tkYxhcgrQldn09kcOHyJ1WzOGHH2LCtBno6Sdzix+dQZMnkfPKYY7bsSchGMOLmNYvVgu5hdMY/vfd7IsrfC0rn4LB8RYS2MKa+KCsD5zP566eSXbCIGsfXc4Tf1+Hzw6yZfVm/LPmtO+iFpnM+exlLN/yJJsDJocXP8N/z7iHCwd2es9P3icW8cnl3+f5fdFeXjg7S8vJpE9eRP7qp9gasKla+3u+W7qG2UWjyPFC1a76FgNDeMtTfP+hZVRJQc5Z3+bhGwtOjbCiuYf//N97hMaMZWi/TDx6OmmO2OAmLbPjVaquiUyfnMry9+ppaAgg3MVMHudhhLOAjDfewlcfAOFgdHFh549LFG68nphnxK5ew8svjeYz581hZHpH33eSmF3P3lX/ZcmO1pPH+lFwRhGDHD1oYz1py/IIr913L0u1fKZOGE6/NAe2/yib3ttIVALCyYBBrfZ0667tve1jdi07l73Ky+kt98dLHXs6CyalJXlcJ5PS7Xr2rPwvi9Mj1FccYPPaDezzxeY0aTlz+cz5I2MDdV+NFfRx+QB4CphdmMbaVXVEdr7AI78PcuYYwf53/sPWsASRytRZU/AKer6fqZbNwDwH4oCJdWw5z73Yj9MHa9RWGuR/ch7DkxSnwjuJmQUeNm4I4N/wJD/62X4+UTSMNBGiruoopSlz+dJFEzq/OPak//T5mJusak3y2tDjPHWjDXf4+x6MA43ZS5vMoq9dzdF7n2JLQ5Qji//AMwU/5IYpqe30OYuDq9fGdkIxhjBz5uCmNIW3gJkFHjasDxAteZlHHgtzXvFAHEcPE5C9aEMigxmnF/G3D1bR4KuiGhCphZw+IyNuX8c2JXPN6Y7dLYrcf5D1773HkXApa15/g8OxVZwUzZyQ9OblfSYgy377OOnDRyJsi0h5OUKDWEwjJhSFBIRE2gJhSaQgtvJaCGRVVdv7kZoaZNhsEotIYu9lXDjKZmEppSRafgxXegZpw4ZT9offMOSbd51cx9SIQiZnLGapLzYXYci0aU17PGp5RRQPfYF9+01AkD55atP8x8CWtWwOxO7SBs0+j08UD2+5Om6Siy2L17Okyia4dQ2b/bOZm9rBOJF7Op85dynbXzmIGdnDv19azxlfmdH5RcQYyvlXn8U79y2mvLdTcjpJS8tbwM23lPPzXy+mJCDxH9nI0iMb270TrS8tpdaWgKT2aCkNsgDXKRDxtit2snrVUva9u7SNZ2TUtIJOxJ+b/BlTSF2xgnopcIwrosArcI4tYnLaMpbXSYRjNDOLc7rwengYP3Usro3bCNk1bPn3vxg3ZxYj0zv6vpNbfruaja8+TZsacBTy5dmFHV4sOmtjPWnLs3zrWHskjM/ayNv7N7bxuHpGX8DF01MRib6JLm0XrfRRL/uYXcOWxS+ypbUQuDCfM1sIyM6Oy+p4sLWr+eCfT/NBq7w7B8zk2ttvoDguSHuWj647Tp+kK1KZcfW1zNnze1ZVBjjw7vM8+W7j/+lkTbuaq2en99Kr52by/Jlkr1tOlV3D5lf/xGYAYxzXzp3L8NxkBVUW865ZxPsHnmB9dYTyTUv4x6aEYWxMHpdfOIEBnQnSHvafPh9zk81yMteGbvarnrThDr2CpT0YBxKLdOACvnjtdu753TpqzeMse/IFZv70egpau3rN/axZdwwLgTFsFjMTRbTIZM5nr2ZNyZ/Z6ItQtu4VnlzXF21IkFJ4JnNz17CkIja3N2fOWUxrXGDUiU3JXXO6YXcLjbWJl/+QYLQw6DfvWq6clpJ0P+2TEHakogLfO++QPnwY4QP7Y1vv2DFxZzfOW7Rt7PhcRttOmMdoWtjtCEi7qhrbtJoWzdh2yzSkZcfSthtfNuED+8kYMZzqpW8Rqag8uerCOZap+fH5a/pApk0b0Fy42gCmFQ2MDcrCS/7UcXGFH2DL2q2xuwR9AMXFQ9pureAcw4yiePg3tJ21WzoLTRkMP/9y5mbGbvPr1v2LN0u79me7JlzEp4vT+iRk03FagqzCa/j+/Xdy/bnTGTcwixSnjhAGTm8mA0YVMHfhQqb108mZsZD5w9NwpY/gjIXTOWWmS+oDmFA0loEZbgxNoDm95AyfyjnX38kdFwzutDN5Js+hOEPEVt4VFpAhANcEpk3yIhA4J57GzJyuMirod9aXueOqeYzLS8XpGsTg/non35+QW6UO2ljP2rKWdxqLrr2AuflDyU11YggNw5lKztB8TvvULdx716UdrnBPOjzbp32szy7ppPYbSE6qA10IEALN4SI1ayBjpp7GRTfcyUP338bZw5wnOB99l66WM4cv/+DbLDozn8GZLhyGi/SBE5h/5R388LYzyNN7X2beouu480sLKRicjlMXGK50BoweRJrdvRrT887gqz/8DtctKGREbipOw8CZkkneiEnMnDQQxwkec/pyzO1dvz2Zbbiz+ujtOCDImXctn52WhkBiVbzNX17Z02Y/T7NkDWvLLRA6I2bNaHOTYAw4k6/96FssOmsqI3K8OA0dw5VK9oARTJpxOtOHu5vqrFttyDmBc84egUOAMEZw9jnNXr5ObUrympO83SmMmjGf4onDyctKxWXoGM5UckcUce7n7uZHX2oOqyfVAupqa3o9Vh55/DGimzaS0S+XyMH9iLhnMeZhJPZeiLhDsvkv8T/C4WLon5+neRY2HLnpauxAA7LpIZsxz2PLv/H3cS+ktCXOESOpq6jCKJrGkJtvQaFQKBQKxcedKDue/jY/faMC6RjHoge/9+E9feqUtil5ev84ddvm+Otv4B0wkEj5sZg30ErwMloS20rwGCb8bXxvhUJE6+ubvY+RCGZ9fZvjWv6NbzCeeC5bEjlWjrd/f8r/9e/Yym2FQqFQKBQfbyJ7WLOuChuBMWYWM/sLZdOHLSD9O3YgHE50wKqrbw5fx8PLdvxzUwi6SRC2FJKR6uaQc7i6qoVwTPxNUwg8IW1pNYexrbo6NE1Dc7rw79ypOo1CoVAoFB93/bh7Det9NggH42ZO71ao9uNkU3fo9SIa37p1eNLSMGt8sZXSEqQGQsRXXgsBQsZj1bG/Ehn7nvhiGiBcWYl3xKhYoVZXYVtx72HjdhCy6U2LUHbiwprGULZZU4s7LR3f+g2k5uernqNQKBQKxccYZ8ENPPb0DcqmPqTXHsjaDZtxedxYdQ3NXsGE8HLr8LMdDzXbrcLcZsJCmmhVVYvwdIvftBMGbxHKtiVWbR1ut4fa9RtVr1EoFAqFQqHoY3rtgWw4eJDM4YOxfNXYlt2096MUMS9j81+BQIIg/p7Ye2KeyGhVcwg7Wl3d7IFENv6LPc0m4b2Uidv70LxHZDCIkZaB/+AhVcMKhUKhUCgUp5qADNfWowudcDgS268xIXTdVkTGVKMQsc22iYexBWBWNgtIsyo+B7Lxi3j4OvanlWiUbfeItEJhnLpBqKZW1bBCoVAoFArFqSYgo5HYZt921Iz5EoVAJHogaSkiG+c9CuJzImOSECtRQPoS5kA2aUbZcj6kbJ4XmeiBbHpqjZREw6aqYYVCoVAoFIpTTUDaEmzTjIWPaX/xTKKIJC4epWi5/aRd2TwH0qrytRSQrUWkbLuopsViGmKC1paqghUKhUKhUChOOQEpDQdWxIyvgrZbrrZu2kQ8Fqhu3EBc0igkofGNWVGRICCrm1Z0x2Vj8/NTE0Rih6FsTcOMRpEOp6phhUKhUCgUilNNQDoyMggHQ2hoSNtqN3Qde/qMjM12FLLNAhoERCsqkbaN0DSsqngIu3kSZMLimbiojG/l014oW+g64WAYR2amqmGFQqFQKBSKU01ApgwbSvDoUby6jh0Ot108ExeNzfMeaVpAIxoVogBp2YSrq3Hn5mJWxhbRNM15hKaFNG1C2e14InW3TsgfImXIUFXDCoVCoVAoFKeagMwsKsR34CBew4m0GmKLaFqJyJheTHjfKCobE4lvJh6tqMBwubCCoQTlSEvh2PhetnqfMA8Sw0EgbJI9vVDVsEKhUCgUCsWpJiD7zy5m/zMv0j8nBduWTaFq0bjfYzxeHROUze+btKNofnZP7X9ex5+djbSa/79JKDYJysbwdeL75nmRUoJwOvHVNzBhVrGqYYVCoVAoFIpTTUBmTZpA2LIJhk10w4GMROIh61iIWormeY+ycd/HJs3YUkwe+9uzbU+QsIJGJi6kafy66X18/qPTSTBkYkpB5sRxqoYVCoVCoVAo+pheP8pQaBojLr2AqkAE4fbEHjvY9LhBu83jDBu/b3qsYRev9o5tnVbj4wxty0a4PVQEowy/5AKEpqkaVigUCoVCoTjVBCTA2EWXU14fIaS5MIWDqA2WBXab51fbLT4nfnc8HGaH389Ov5+KcLjp+0TRmPhd02dbYlkQtcHUnIQ1F8frg4xddIWqXYVCoVAoFIpTVUB68/ox9LwzORaIYqemEbV1IlIjautEpYZpC8wEUdkkHO3Y3/qoyeCbv8SiHZv5zKb3GfD5G6mPmi2OkQli0bTBtAXR+Dkaz2V70ygNRhlxwUJS+ueo2lUoFAqFQqE4AYi62ppeP69FSkmgyscrF1zHMI8TRySMDIVie4rHd3vUmlZgN3/XuJm4NmcG8/7yhxZprrz+i1ir1pGwZziyeeMfbCmav5Mg3G4iTjdHQhEuff0pPFmZLRboKBQKhUKhUCj6hl57IE3TJBqNonlTmPadWzkQCGM6PEQ1JxG72QsZsXWitkbU1ohIPfayNSK2Ru60aW3SzZ4ypen/G49v+n08zagdSyOqOTGdHg4FghTfcxvC4yESiWCa6lnYCoVCoVAoFH1Nj1dhSymxLAvTNLFtG8uyGHjmbPJWrufw8jUM8nix/H6w7diTDeNeRyFk48NnGrd/pH7Tjtgq6kaPoZTUvr+FiK3HPpL4JEPR0hupaegeL4eDEQYsmE/e/JmYZhRd15u2ANJ1XXkjFQqFQqFQKPqIHoewTdPEsqwm8dj4ORqOsvKb9xHac5B+ugM7FATLbhHOFnFZmKjpRi66jLG3fg7N4WD3r56g5KnnE8QqgGgSko1ha3QNzZNChRnFM344c392N4bTQNd1DCP2V9O0ps8KhUKhUCgUig9JQDaKxkYB2SgeG4VkNBhi1d2PENxzgFwcCNNExsPJIkE4NgrJ5vfNJGw1nuB9bHx+NgjDQDoMKu0onnEjmHPfHTg87ibh2CgaGwVko5hUKBQKhUKhUJyCAtKybcxwhE2PPkXpO2vJlQ5cQiBNCynt5kU00K54bCsimz2PQmgIQyckJdUiwqBPzGLq7ddhuJzomqYEpEKhUCgUCsWpKCCh4xB27GVjWbHPh99azaZHn8aNTpo00KVE2hJsq0k9diogG63TdIQmsISgXrMIYVF4x/UMOWNGXCAa6LrWQjyqELZCoVAoFArFKSQg21tE0/JlY9kWtmURqqln6x9f4PBbq0nRXDgtgUvTkbYde8ShlAnPKWy0TCBELHwtNI2IbRLSICDDDF8wl0lfuBx3eiqarqNregvxmOhxbBSSahGNQqFQKBQKxYcsIBtpFpA2tt1SRDZ9L21sy8J/vIrdz/+XA4tXIE0Lh6mhW2CgoQmBiO8qJLGxpcTExtIhatgIh8HIc+cx9ooFePvnoOk6mtDQda1FmDomHHU0TSjPo0LxcSMchPKDyNpyME1aToRRtLkAOJyQ3h8GjACnWxWIQqE4eQISmr2RMdEose3m0LZt261eEmlbVGwv4dj67VRs2k3doaOEahuww7GFNprLwJ2RSvqwQfQrHM/A4nxyJoxA0w00TaBpWotXo7cxUTgqr6NC8fFC1lUh920idPwAlJYgwn4lIDstMJBuLyJvFK6BwxFjihDp6gleCoXiJArIRFoLx0ZxKaVs+tz4N/HVrnHxMHbjS9P+v737jq+qvv84/vqeO7IJCZAQSEISEoIJe4QhoCI4sSCIEwdqq/ir1VbbWuto/bniykt0AAAgAElEQVR+7tXaVkWtigMEAXELOHAgQxkBDEtW2AlZJHec7++PuzNvIiQIn+fjcR9J7j3JOd/PPdzz5nOWEfLVFxJ93/vCpBDiBOOoxlz7FYfXfo2OSyD+1ElEdc1GyYlzDedH06Rq5ybKF8+C8gNE9RqBkTdMOpFCiLYJkCEfUN6wGBwm6wuOjQXI+oKkL0T6HtJpFOIEt20dlWu/hupKkq/4a72fRUqpkM+a4M+Nhj6DflEf5rU+B4PHHPy19ngNw6D4v/djREYTnT8c0nJlfRJCNOmoHiDo6wzW7go2FCB9X4ODY+0AKYQQdRzah95ZROzEGxv8LKodoBr6vPmlCx5X8JiDf64dLGNHnU/l7Keha3cJkEKItg+QjQVLCYNCiCMWmlwOjJoqYtJ61AlTgQCF5zq0SuG5E5YvTAaeO16Co/cZ700bfONS9XYgAWLSelDlOIx2OZFPZiHEMRsghRDiqPzntJ5jHoPDku/bwFdfF+64GD0Aplk7SPpH33jdtEZOOhJCSIAUQpxQtDf8mKZZb4DyBUnf6/UFzOOqHkGHBgXXor4upOeauxIehRASIIUQJyjl2VcdcnMCHUhV/psX1A5Wx209vOPGN9baV76Qw4mEEC0g17g4JlsHB/jujWd4YXExplRDhKwb5fy4cCZzvtvX8Lpxoq4/QZnI1J5+WvBOWV37Oe+dro7IQx9k1YJXmbOiBPNI/c2f+ahdA39HstajvvoJIYQEyF8i8xCbVyxnw76an/+ZrqvZ9+NKVu84fGJtHxoat6uIt/4yjVtfW4ejOdO11vI1+XulrPnkfb7cVN7w7/2c9ecXvb4EYlHgpBjPQ+ujHNfce/hu3ny+2lqFPibio+cEGoejpok60FCcFEIICZBtsSGr2ryQFx/4M/9zzVVcfuW1TLvlbzzx9ipKW/sz2r2Zd596ilk/lJ1Yb0FD4zaiSUhOISUx2rPyhzvdifK+/ILXF0+XLfREmYZ+PhoP30LoY+RRVLSR2277C0VFG8Ori3xwCyGaQY6BPBobsrKlvPDwS6zqOJIJUy8iNdakrHgTm4xoouVwozb+L1NXxt58F2OP1HTi2EqQoXEy8JOuPYE+SjM/Njp5GzduYvr06XTq1Inp06dz9dVXk53dvYHlVUenJEIICZCiedw71rG+KpGRl03l3J7eEvcbxMjgiVz7WTn3deYsKWR7iUm7zIGMmzKFMVnR9V+HranpzVLWLnidNz9ZybZDbiISMznz+j8zMQvARdGMW7hsBoCdghv/xc1D7UHbkoMsfeU/vPP9DvaWlFNDBAnpAzjn8is4M9v79xucv5U1L9zCwxtP5Z57J5FhAZyreO6mJ9g78VFuH5OAQlP+2WPc+IqN6576HcOiffMt4bsZzzF3xTaKD5TjVDEkZQ/kjMkXMrZHnO+iJBQvfJpHZ61mX4ULIy6F/FMv5upJfUk0AF3KD2+/zJxvN7J9XzlOazyDrvw7Nw5rYNwFe5h9x90s6XMHD12checS92FOZ5ayev5rvPXp92wrU8Sn92XMxZcyLi/B06UMp46hb2o974u18fF6a1K+6k3u++4nNu93Ep2Sx6hJU5g0OBlbQytls9a32sv1FKf88Cce/2ks9917PmmGZxm2z7mTOxb15LZHx1HxVlPvY+PLQPky/nPncxQNuJF7r+hFy26mp+vPj/oA30x/hpnLt7LnYAU1RJOcdxrjhthYu3gJq7cexBHdhf7jruX68ScRpwB9mC2fvMTzc75h4/5qrO1SGHTZX7n5tE6e8Th28dXr03ljcSG7qyNIyumGvVJ76u+br1nCD3NeYsZHK9h6SNE+oz9nTrmKCb1860sT82hpeNy0iWeffRaAffv2AfDss88ybdo0srOzUQ3kR0mQQggJkG3M0qEzSUYpq5esYn/3AXSss1WvYcNbD/Pk1x057/LfMzWxgtVzX+KVx2eQ9NC19LU3c/ooJ5tnP8Ij77sZPPFaJmfF4iotJybJ6n+bM869metHdkChiK69QLqSHes3cChtMv9zTQbW6j2sfPdNXnvSSsrD19I3svH55+RlY/tyE5srNBnxCndxEZsqnBws2oJzTAJ2nGwp2gxZ55MbFTzfCravXUdJ10ncMDUTe80+1n46h9ce3Ebl3XdwfjcboIjPPo1LbjibhGgoWbuAl2f9i9cyHuW3g6NRupxNK1ayN3kiN0zNJdpdiSWlHYpdTY876J9B09M52TjrIR79UDH8omlcnAo7vpjJmw8/iuPOu7ggyx5GHcObb6Pj9f6mwxVLr/OuYUIHzc6v3mHmM4/guO1erjgpop5lb2r9aWq5oonSudi+Wcf60gmkJSrQ5Wz8cRe2HhPIslSwoMn3sfFl6IPG7Xbj/hkZRjfUf9QVbCsspDRjCn+4IRNbxSbef/FVnns1i7GXXM7Nl0ZSumwmL8x4mpk9nmJqnhVz27s888JyEi/8Hff0T8As3cXhju293cxKfnj5Pp74IppTL76Ja9KsHFi/kNlFngDpmbeDH9+4lwfeU4y87HdMSYNti19nxv33U3PPfVycbW98Hj/j86d79+488sgjjdapvvwo8VEIIQGyjank0fz66u088+qT3LqyG/1PHsXpo0eSnxzhuYRIxTLeW1hCn6vuYGKBp0OTMXUfP/zhHb7ZcCV9e9f6wG9q+h4rePejnaRNuI/rzu2CJbSZBIAtvjNpacmNdDYUkSk96ZOfhYU88hL3s/pvX/L9Fhd90hqff5+evchmJuuKHIweZKd0/Qb2Rsagitax1TWAHmoHhRsOkzoyj/aqnvl2yaNfL0+Xr3fvdNRd9/DBe6s5e9oAIlFEp/dhoG/yjBi2f3snnxXtwj0427sCG0Sl9qJfXlZg7A2Nu4HTkpuaTlcuZ8HHxaRPuI9rz+iCAeTnJnN4xx0sWPA959xYQHQTdex7krXp+UJY4+0w4FzGn+oZb5/8VJw7b2fex99zwUlDiKaZ608/W5PLpfMH0NN4iRWryhhzajzKuZkNWwxyLsrBTnmT72NEk8swmBueHvzz/uH5j+kzQ2+Pqk1AEZWSS++87ljIJaH4K5a/k8bgMQX0sQI5Jms/f4j164pxn9QVs6yUMh1Dn7yTyMmIBDICf7t8Ke8tPkj2xXdw3dneGvWMZuvCFWzQJlqbmBVLmffBTrpNfJjrzurqWV96JlO1/U/Mn7eMcb8fSkQj8zhKn0z+IoXeG1yjlEUSpBBCAmTbs9Nl5K+5b8hEipZ9yWeL3uXRD+aSff6N3Dy+BxG7f2JbzWH2P/c7rno+sPVzuxT20rpnv7qbmN65azNbaxIZeFJSaHj8GYxOyXQyKiir0E3On7696ZfxGu+t2YRzYDrrC3eQfe44IhZ8w9pik2zbOgr3J9Gvb3LTJ6TY0+lzUjsWFG5mt3sAGRYne5fP5Y13l1K0q5QaWwy2w26MHGervqPu4i385PDU2D8GSzJ5uQnM/n4Lu90FZKnG6xieFozXSCInuz2ONdvY4x5CZjPXH42tyV2mKq4vw/INpi9fRfkpI4netp4fazIY2zseRXmT72PqEViGsBKk9xqHWvu+B0zvV63Rpucc6fYJCaiaQ5RVm+hoQMWTGA/rKiswTRNLzun8qte3vPK/t7Bp+BjOOOs0CjLaYQFcO7eyw9WBgpxEME3Pv1fvV6012jRx79zI1poODMrrhPJNo5LIz01k5sqNFDsL6N7IPI5GeFRKB06c8aRIlAEaJXehEUJIgDymOpH2DvQYPp4ew8/ijPkP8fdZL/J+/3uZoAGjPcN/82fGZxohH/IR7WNR7K/bWWls+p1N3EOiBVtmZRhY8G74mpq/iqVP/1Te/Ox7tlZV80NRMn0uGk5k4Vy+WrOPkdYf2N6hH1d2tYQ7c/8poub293jqmY8wTruc6y7PJF4V8+m//8HycBsuR2y6lt2nI6SOYcy3xePVJihV/1CaXN/CqIeKo9/JfbD++xtWHBpG9tq1HEwtoE+iCut9bPYy/IwmpPYFRW8o0r4A5+sOalAWA7QTt8uNaSrQBharwnSbuE0TZUnlrD8/Tr81n/PRggX887Z5vHfhbfx1QncMT2LEdHt/F09I9b0PpmlimhrtnXfgzjcmpjfYmqaJ2cg87EcpPHo6j4anLkqB6QuRQgghAfIYFEFq7550mPkBxXtNLD3TSbV8xLbdJkknp9Z9E9zebZF342vp3Pj0unM6qdaP2bBuL+7sWruwsWG3QVXlYUxoUXejqfkDpAwaROqcL1jyfhmFcb04JymB6L5pvL78MxaqTXQYdBHdwlnbzL0UFZViT00nyQLOHVvYoXO5euIIesUq0FGkxIZzYZ1wxx3edJaUTLrZP2bDun2Y2SneSwDtYd2GEuzpGXS2QPOu2l3/fFs0Xuc2Vq8vI7Jbpmc5mrn+hFcPRWz/0xga8xifLSmkbOUuug4aSIrRwLhrvY/NW4aWh8dAB9IM7UR662GaJgrPRbZ94dLtBqVNb8DyBkBvLZLyT2dK/ggKXruT//1wEevPySS/cwbp9g9Yu2onjuw0z1jM0N9XnTNIt3/I+rV7cHX3rS/FFK4vwd4tg2Rl4smV9c+jVwsK5Hu/Q+6uoxQKXete32ZgF7YBaMNfIyGEkADZhpwb3+P5z6rJycskOd6OWb6TlR8uotiew5mZVlTcQM46bR4Pvfs0/7CM55QeHbHW7GdnRTKjRuQQZcTRLkazv3Apq3Z3pl/nJqaPG8Q5p83nwTlP8k8mMCo7AaNyH46kIQxMSyEz3caHX8/no+zTSdMHKGvXn2E54Xd9mlxeBUbnAoZ1m8Ob7x4g5dy7SLMojIGDSJv5Fu/qzpw3JaOBlc3k4IoFzO08jJyOsOurObyzrROnXtqPaMDdJY3O+iMWvrOExKGpxFpK2FsZxqbO0sC4u7dsOhUzkHFjU7h37jM8H3kBI1I127+YydwdXTjnqgF1jjts6fIVhDVeTfWu9awqdBDh2Efhp7N5b083xl/Xx3P2cnPXHxVGPXJiUfaejD4lmbvff569lamc8euUoEMSGn8faWIdiqw4Amdh6+AQ6btdISEXadS1gpLvJV/H0NcddO35noXrTbqmdSDSdYC1OyogJpYYbaKjBnDu2K7cN/dxntLnc1rPDtiqN1Jco/0dSCOqP+eOTeG+uU/zXMQkRqRqtn0xi3d2dOHsK/sR2cQ8zBYcBhm4x7U/O9YTDFWgm+6tl9KBn4UQQgJkm9G4bbHElC3lvVcWsL/MiYqKJyVrIFP+OJnTOyogivxLbuMPca/z9mev8sTswxCVSHrBxQwaAVGqIyeffzYrpi9ixqIB9L4kp/HpieSki//MLXGvM+uTF3l8pgOjXQoFl+YzID2RgouvZN2/32LWk8twRyXT94IshuY0Z7dhE8sLYCQzbFRPZm05RMHQdE/nKmkwQ7JmssU9ipFpjfQArRWsmfc8cw84iE7JZ+xvp3CB92xiS7dxXH9VCf+d/yqPfFiFaY0kJr4z3ZNiGl9+FVv/uLu3cDrsZF/wR26NeI035v2T/yuHdun9GH/rpZzX3daCrX0D8x3TxHhVDKkn9SB++Xye+r9K3EYsnbIHcMltF3FGhnc5mr3+hLFcObEorHQbfRb5HzxPYfdzObmLEfb72NQ6FAl1wl3LepDacwu/oA6k9h8DaQY6kKavA+lG67odXtehbSx990OK9pbjMmLolNmfKb85h3SLp2uYPemP/CnuLd7+5DWeeKcS0xpLYudcBnf1nSlvJ3vSrdxin8Gb85/1ri99GX/LZYzzri+Nz6PlHcjNmzcx/cWX6rx+zTVXk5WZGcjTSqFNbxdS7kQjhGjuZqzsUKl8aoi2YW6v55qM4pjm+JFXb3+Cfec/wM0nx3uvQdr276Ne+Qlln88m+baXcDodnmMPQ06mCb2ouNZm6K7e48yWLVt5bcYM4uLiKC8vZ8pll5KZmYnneEi87UmwGAbKMLDZ7Ox58CrajZqI6j9G1nMhRJOkAymEaCo1sv+nHVRRzaaPXuHzqLHcPiSeYy5+effDau3tQpreTqQ3Nfq+er4c3/9vzsjoxqWXXsJLL73MVVddSbdu3QInFeE9eUYptNKBM7CllSCEkAAphDhi3MV88fz9zNlupVPuKK676VdkHJOfHDr0htjaDLrJc9DX4CB5PIfI9HTuuP0v2Gw2f4BWtQZtmiYWpZCDIIUQzSW7sIUQv3h65Scc+uxtkm97EUdNDaY2/ZfyCXTeCLnY+PG8C9tfF/9le/y9R5ThuW6PUspzmSmLgd0ewZ4HpxJ/yiTZhS2ECIt0IIUQx8P/hT0hyW0GNRwDJ4aENtiCjoc8vtOjNyR7TiJSSgXfjMY/jWlqtMstDUghhARIIcQJxmLDbY2g7Kd1RKRkBaVDz0W0lVZo5ctMnmCllOcOLArlf+64yI14xoSh/Md6GoYRiNlBIdJXg0Ob12JGRKFsEbIuCSEkQAohThDxnaBzFqWLZ5N6xV9wOp0YhhG4FqQCw3vvZ8OwegNjUOg6Dk6qqR2APV3H4OtDBl1oXPmPiMRqtVG2eCa2Ljnodp1QsjYJIcL5zJFjIIUQv3iOavSaL9m/fCGuiDjiRo0nIbsvyjACHTlCb0jpvUdLyM+/ZPWNxfdcfWPT2qS0aA3lX8zBUl1Ox4IxGHnD0LZIWZ+EEBIghRAnBl12ADb/wIGt62FHERZnlRzX1+inv8Jtj0J1zSGhWy5Gdn+IS5S6CCEkQAohTjDOanTxVlTZPrSzRurR+Mc/ymqD9snQuRtI51EIIQFSCCGEEEIcLYaUQAghhBBCSIAUQgghhBASIIUQQgghhARIIYQQQgghAVIIIYQQQkiAFEIIIYQQQgKkEEIIIYSQACmEEEIIISRACiGEEEIICZBCCCGEEEICpBBCCCGEkAAphBBCCCGEBEghhBBCCCEBUgghhBBCSIAUQgghhBASIIUQQgghhARIIYQQQgghAVIIIYQQQggJkEIIIYQQQgKkEEIIIYRoU1YpgRBCCNF6tNaUlBykrPQQbrcLrbUUJZzAYrXSrl087RM7YBhKat3GtVZlh0qlmkIIIUQrhcft238iwh5Bl9RUoqJiUEpJYcKoW1VVJbt27qCmppr09Iwm6ya1Prq1lgAphBBCtJID+/fhcNSQk5uH1hqlVJNfgzfsx4vagaS+sfp+Dh63YRj8uL6QiIgIEjt0lFq3Ya3lGEghhBCilRw8eJAuqel1Nuz1BZrjMczUprX2jy/4++Aa+Orke65LahoH9x+QWrdxreUYSCGEEKKVOB01REfH1Hnev+H2bN1RQWFGAdo0j6vdr9o0Q8Yd0hXTGpSqtysGEB0dg9PlkFq3ca0lQAohhBCtxPSHEx0UaKjTBVNKeafwPq9Acxx1x3z5JWR8wamn4foopXC73VLrNq61BEghhBCilfg2zqapAV3P8WmB8BMccE6k+gR2owZqEgg1qk7IkVq3Ta3lGEghhBCilZghuxOVd4MdeAQHH8/XutP8Yh7mPr555VGe/bQYdxPTmqauNe7Q13x1qF3HE6LWzahjOI8jVWsJkEIIIUSrBkhPR8yz8Q591PfcL/Wh9CE2Lv2WdXuqvbtPG5nWc0Big3XQ2hdiPD+HHyB/+bVuTh3D+ntHqNayC1sIIYRoJVpr/7ZaefYPBl7ztYJCngjjb1bv4Nu5b7NgySo27S7DZW9Pl+69OXnCFC7on0ib7ZTVNHsswb+gUajg3w06nDHsXdhHuNZ1HPqMB298ih+yruPJu88gSR1rdTx6tZYAKYQQQrQS0zT9vR7PhlvX2ZQHf9/UcXm6spDX//4As4s7MOSMSVyX24koZynbNqym0mlr45NBao2lWb+rUP5+m+8MaU/Q8dWxtWtdzxzY+ckCVloTiCqcx/tFo7mih+UYq+PRq7UESCGEEKK1IlXwxliHZANvVyw0N3g6Zw39tRo2zHyWOTvTufjev3FhVoS/WzRkxBnejb+L4k8e44HXv2dPuRNLu670HnMF113Yj0QD4CDfTn+Gt1ZsY++BMqqJpEPGYM67+hrOyYn2RAizlDXz/8urHyxja4mbyI7dOfd3dzO5hwVc+1g++xVmfr6abQdM4rsXMGHqVM7oHo0KHl8LOpBBpw/XfSqMAHlka11P7KpZw4KPiul1yV0M/upvvL5gGednDyHOn0FLWPryP5i1bCu79pXhMGLp3KOAsy+5jLNy47xDCWOaoGVXOFj5zPU8sOUcHn14MmlKAyY7Zv2JWz7J581/TW1hQG1+rSVACiGEEK3ErLUbVdezKSfMzKUca/hk8R7iR1zLeZl23CGBKbAvsn3PM7ji9+NJiNYcXP0Oz894kpcz/8lNQ6IwdAXbCgs5lH4ZN1+fha16N8vmvMJLj9hIeXIa/SNr2DzzPu6f72boRTdwaXYcrpJyYjvb0FSxYca9PLIkifOn3sZ1nSr4fuZ/eOGh/9LpiWkMiKh7BF6LQnfdTBNSx9aodX1LVfbdRyzhZP4wKoceCcOY+dj7LDkwhDO9N21RupyfVq/hYOol3PSb7thr9rL6w7d46Z6tVNx3LxdkWMOaJnj5TW2jZ/887F+uYW3JBaQmArqcDet3Yu85+Wd3J5tTazmJRgghhGitAOm/ZIzpPVlB+7/3PGr/3PDDPLiDHVUG6TmZWE13A9NpotL6MLBXNllZORSMv4yx3aop2rATl28aFFGp+fTtlUfvQadz1bVnklK6kpWbXVD5He+8t530C27lf341jN4n5dF/+FBy2rnR5d8y7+MS+l12E5OH5ZLRfQDn/2YCuYe+5at1zpCTMcIdU0N18Z3s4fubzTkL+0jUus7D3M3nH64kftQY8uwmUX3HMDJhAx8t/ClQV19t03rTv3c+fQaN5vI//4lfJW1hwfzvqWrGNIE6aiJ7DSLPUsSyHw5hahMcRRRuMsjt07NFY2lpraUDKYQQQrSSQHAJftTtgflea+yYPOU9xk95N/L1T+lkz3dzeG3uN2zYWYrDFoPtsBuVW+M9RjBwORffBaONjskkGRUcKjdx7dzE5uoOFOQnoV1Ogi8prYu3srW6in3/uJaL/xnoYbldiojSSkwdOFml4eWrZ1ze2+jVvkezYYDnEjPBZwq3Tq3r/O3tn7NwUxqnTstEud24jSxGn5bG+wsX8uP4K8i1aox6amsaXenXK565q4sodvUnywhjGhVaRx3bj+G9DJ77ZiWHRo4g7qdC1ldncVbvdpimu1nr48+ptQRIIYQQopV4rrPnDS2m95SIhoKNqfEcSFg/o31nkuwmG7ZswzGqB/b6Jtr5Lo8//gHGmKncODWL9qqYD595nKXac2yb/6QMrb0/e4KDBdOz61J7T0TRJnVyhNZgJDDyt3cyMSuwQ1NhEJkQB+Zuf6gM/O1G40w9d4rx1kUpTz2M0Dq2Vq1rh/KiRZ+zrWY3r944mVdDhnCQRasvokdfa7211bgBA7TpfT6MaYI7kKaJmygGjurH9H98yfJDQ8ldvZoD6UPpk2CGdWzokaq1BEghhBCilWjTFwp0SIfMF3RC85n3os6BrXutLXgeIwra8c3ns1l05q2MTap7VJp7+2a26Vx+M/kUeka5MIggJdbAd30/E9N/RomvS+jPUVqjkruRZvuAdWv24OzWieBzjC3J6aRa32frLjeJQ5JrBQpn4Cxo02y4Axl8X2p0rWEqlNLe2zVrtOFtxXnDVqvWOji4O9ayaMlBcibfxbUFcUGvlPH1fx7k489WckmvgbRT9dV2D+t/LCEiLYOOylOXJqdx161jdL/RDI95mMVfrqNs2U7SBg8m2XQ1fmzoEa61BEghhBCilZi+jp6pMX2BJiTYeDbuvlDT2G5Vt7Yz6JKpDC98ipfvuoet54ymX3oiEbqC/T/9yJ7O47gkLZ3O+n0+nrWY+KFdaWctYU9l4ALSISdK68ClXHxfzZiBjBudzL1vP8KzeiIju8djqdqHI3kYg9IHcu7pydw/9zGeURM5JScBm+MAOytTGDk8i2gVR7sYzf7V37BqxDj6JkXUDZE6cBfq0PEGAo3ndYXSJmjDezKJ2aq1DnZ4zRKWVmYz+bRepLZ3BeWzrkQPz2L+W4tZUTGIU2JBY3Lwu3nMSx5KdiLs/no2s7d2YvSU/kS63b5LdTc+TT11dFl7MPa0ztw+/1l2V6Zy9rQUMF2Nd3mPcK0lQAohhBCtRJs6qCvm3YWpg+8IEhTmwgg1ztgB/Pa+u+nx9hwWffoaS0oOY1pjSOyazYBzXJhp47jxmoNMn/syD79fhWmNJKZ9CjnJMd7AaPrDhak1SmsM038FaVxOCydddjt/jHuVmR8/z2NvOrC068KQK/rRJyWa/Mv+yq2xrzJr8Us8PuswRHWg27ApDBimidAJjLpgHMue+5gZiwaRNzmtydARGLOvG+a5DSFaYyrvsnnr1tq19lnx+Qpqsi+kX3sXbrcr5LWOA4eQNeMNvvi2hJGjPbW1WstY/c5/mLPfSXRKPmfcdCWTci2YLhdKNT2NVqF1zL8wHYsL0k8/m14L/s3a7PMYluzpVDZrXfyZtVZlh0q1/JMWQgghjr6FH3/IOb86H6fD4dnF69+1CoHdiqG7Wf0nNzTwN5VSWK02LBbDe1KE55hFt9vE5XJitVqxWq1Br3lCVE1NDRaLBbvdc/RkdXU1WmtsNhtWqxWtobr6MAA2mw2LxeI/6cI0NQ5HjXfeViwWC77bl5imxuXy7MK2WCzYbDaUUhw+fDiwzE0VSnnvjOL9m0opDIsFQxnY7HbemzeH0WPPbPVaA0RFRYXUK5jVasVms3nCfc1m5txxF0v6/Z1HL83CqhSmqXG7XbhcnuBpUbt45867G50GqLeOVnMzM25/nL2THuW3BREN7no/WrWWDqQQQgjRSkwdOC4vuDuG1nWOv/N1iPy35GuARuN01+Bs4HWXw4nLUf+rbtPFYWdoF81Z48BZ46j7XOkA9zQAAAJrSURBVIPzdjQ4b7fpwl3r74cENBU6VhU0gVb+oxI9T5kmGN4OWZi7sI90rQEOV1Y1+FpwrS0q0N2tOVyNs76Ap3TT0wTVUSknJTuKqTKr2LpwBoujzuKvg2PA7Whk/Tg6tZYAKYQQQrQS31m3nt3HnjDj331M8B1SdMgxa8dvQQLZRpum90QPhcb0ZCvDCD5IM3B8Zpi7sNuy1jooZOsGbkIYzjTBDL2br6bfz9vbrCT1PJVpvx9PV+0I7waHR7jWEiCFEEKIVstLvrNaQ09iqRNojvfgWItZ6zqESil/UfzhSCvPrmjTDKsD2ea1NtKY/NArTAaqDx+ufx7hTBPM0o1JD7zMJO259qfT6QjrkkZHo9YSIIUQQohWYrPZKD1UQlRkVOgLyrM308S3K9VEKfzX6lPeizo352LXv5RArVAoQ/lDnvJcxdqzi1Vp/15eXy1KSg4SYbcf87U23S6qq1zBs23RNKHTu6muqqo9nDaptQRIIYQQopV07ZrO+rVrKRg6DKfTGdQJ8mzFDe/PhmH1hpigAKCPn55k7XDmOQuYkM6YfzoVuDOz1WqjcPUq0rplSq3buNZyFrYQQgjRSky3m6+/+hK73U7PvHwSEzt6umDe7hDQ6PFsiuOjAxk8xtrjrm+MWmtKDh5gXeEanE4Hw04e1WSHUGp9dGstAVIIIYRozQ26Ntm0sYitmzeFXNpGNEwpRWRUFBmZWXTPzkEpQ2rdxrWWACmEEEIIIZrFkBIIIYQQQggJkEIIIYQQQgKkEEIIIYSQACmEEEIIISRACiGEEEIICZBCCCGEEEJIgBRCCCGEEC31//vIDwsb0MPuAAAAAElFTkSuQmCC)

Make sure you have the right drive selected here, because, as the window above indicates, as soon as you hit apply `gparted` will proceed to erase everything on that drive.

Because the MBR approach is how MS-DOS historically loaded itself, some tools (including gparted) refer to MBR partition layouts as `msdos`. If your system is an MBR system, then leave that unchanged, otherwise select `gpt` from the list since GPT is the hard-drive layout that works with EFI. For the rest of this step, we will proceed with an EFI based install. If you’re doing an MBR install then you can skip the create EFI partition portion.

In the next screen we’ll need two create two partitions, one ~500MB EFI partition (this can be smaller if you need to save space, but things may break if you make it less than 200MB) and a second partition filling up the remainder of the drive. This second partition is the partition we will restore our clone into.

Let’s start by creating the EFI partition. Use the menus to choose `Partition -> New`, and in the screen that follows set the size to 500MB, and set the file system to `fat32` which is the filesystem type EFI requires. Repeat the process for the second partition, but this time do not enter a size and choose ext4 for the filesystem type.

When you’re finished your partition layout should look similar to the below:

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAY8AAABHCAIAAABwNFzdAAAABmJLR0QA/wD/AP+gvaeTAAAgAElEQVR4nO2ddVwU6R/HvzMb7NIdy1JLp5QSiqLYiYFdKLaegWfrKXqK3XoWqNhxduv5k7NFFGlFpKSkWWJrnt8fSyyxCFjAzfu1rzt5nnnmeeb7eWqemXm+WNiLp0BCQkLS4sEQQr+6DCT1kJGepsNi/+pStEpI07U9xJpSAYDssFospDTNhjRd2wP/1QUgISEhaRRUQAjIUahlQkrTbEjTtT0QoiJyztxSIaVpNqTp2h5IPLcidW2hkNI0G9J0bY+KudWvLgZJbRAC8QShCUm+PD9x/A1r1DQv1n96MbIZpiNp4VRqKq7Y6Lv9UGl23KuI1BJUFSKMOzlvwuzgSP5XY8kfIACEYQBYtVAN/SStRxQkvHwRm1WGvrkAjfrVlbIF/Jpgulb6+++1F0lNqZU9FwAqfrpz3u4nBXwCcCpDQY1t4th50NDe1iqUJvSCwoRLmzZ/GrjTli1bEYLJquqwWGpyGALUcGxzet02iOQtTKU0BXfXTvvrrUDiKJrjrIPLu0pYD1UnaYQlUUnCvZMnb7x8n1EooClo6Fl0HjFluIMK9vWUFdSVsgVQj+laK9IE+s+1F0lNqZV9GACIuEVckdmwleMcGMKygoyYh5dC1oR/XrrZz0Gu8bW4IguoMibO7r3oz94VgY2J/S8jNgIm8aeEVNbDl4+2o1f8hcnryGG4ooT1UN0kUrMpfPrXnwcjNLoOnTpGT54oSv+QgMvKYc2QoFHZ/RQaMl2rQ6pA2H+qvdTWlFqrnuMKuhbm5gwAsLZ31hfOW3H3n4hxDm709LtbN5yJyOYKKAq6Nt3HTR/hoIoDoILws4cuPI1PyS7m05Q7TA707wQAwvhjs4YeAwC664Kji9wyzy1aEmq/dudYEwpqMBYAiPyIS0dP3Q1PKsSUDR16jZ3obaOCAwDKfR6053x4SnZecTkwVA2dB/hO7mfW5E60JYMAJG9jKoIkpZFjmVtYMiQPECXXtK1EEuGXsIvHL4RGpeQRisYdvCdO7GVSbS5RSnRMiZrnwqmDrKgAAI4ungCAeGF7ZmxK6rNlk48+DgBEyvlFv9+3XLV3tOzDo4cvPU/IKacq6jiPWT6vqwZWV8qOMvVnWqFdclYelweyWlZd+7vQov/3JDIpjy/LcujvN32QpcK3CdmQ6VonUgQCIKoVR+8OTAu4m191mbhGv9V7JltTG5a+tVCvpuJV9oqajir+WzH3wmQYdEzE5wsR0BXNu4+bN0BFFvKjrhw5vfOo0Z55rnIYKkgIC8vUHv7bVEtZEZfCUgRIA6AaDVw0p6saBricOk3i5OJ7lAZi+e/PrNtwE/MYM2esHqT878yp9et5AetGmtABFafExhboj5o7nUMrzwy7fOLYFipr5wwH5k+y3fcHIcAwhADDABBgGFb31qW2NIgQioRCAADAMBzHMahpWwn5ymNPrtv6RMPbd5GfWvG7i4eDNh3T3DHDofKmDVfX1sbz3/4bnm3aXoNWlSHNwt6S/iQqOn+onhoGqCg+7jPdYqjB52t/HHmtOnxOgIMKUZBepq4E9UtZVn+mjOKU2NgCwzELZnJo3IRbwScPneD0GDV23mhGQdj5I6d2nzfb6Stukz/EdK0SKQKBpOK4mU/A9t5CBCBKv7dr1wNKJzcjXKoKLeh+vT4apykVQKKeAwAS8nl84JfmZ8Q8PHk7hW7R05KJEMjqt3MWJzIalfx08cP4z0IXUypCCHBZfTsHa2Px4hYSIgRAU9bW09MWd4yIqFx7QRVPlaXFEtxXV29/NhiyeVpvXRzA2kK7NHXRtauv+893lUMIAcbUtWpnY0wBayuV7IjloeGJAvsm1fKWhrgLqrB9vU2rhjT8V7vGD98ljqCwB6/fOtoYk7Rt1VkRUfzy2v38dn4Bw1wVMACjKdnhsy88iZ1s71hZ8bV6TJ+WsvPolt9eGTp17tqjRxdbbQYGwLR1sqQcCXtb2LObEsb7EPsRNxtrTi+KKkJydlZWpoYMAKOK0taRkiiSkqk9QoAxdSxsrYwpYK6a8ez1Zb323V3sqACmKDp0U1xspshSt2nPMRtvumowhJB4tK7q5lsuUgSqulqEEMgo67CVAXgJZ/f8k2cx9s/hFkzpKjjSGs7w19MITakIAUFUH8J/tdd39F4AAIwqr+803H+SlzogxM969ffJK8/ff87n0eRp5SLcjFfRQlBlPySRpTioMguJYxqMFaZ/TOKpOVtrVnxpjWtZm6uef/MxQ+hijNXICNPU0sS5hcVE236nppY0VOuRq8a1E69bYXQVXRwhooZtxWojhIQZn5LLS7/smzZ6f8WZREJMJr+UQIqVc2saq/P0QJdh71+FPnxwZdONiybD5i8cYiEn7+Bugx9+GV7k6SmbHBPPM+plq0hR7j7Q9kXIWv+PHbv37N2tg6EiBeoRWnqmktphymoqGK+wiIcQBQBXVlWG2JISAqHve6tSbbrKAMAw8f8r/9PCkSJQ7RaHSmNO77mS7zBzUS8WpXHSt1YQAmrlrB4wQABAtRq+fEw7Jo0pr6qmocjAAQAhIvXa9h23cK+JsyZylLD0+3t3vUSSt4yST6TEI1fNZ1RVx3wllqjoXasCq09eIyMMo1CAIIg2/3VFDWlwWU0jDodRI1LSehL/JhDgKp1mLBvMqX6ey1CRw2qaC6OrmXccbN6xT+/LgavOHb7uGDjCUN6xUzvK/qdhBR1N30Xm6rm2UwXA9fou2eEYGXr7+o19i6/eHLFs5WBjel0ppWaaX10wAAqFAkhAiAhAGACFSsEI4keMOlWmkwhobdQjkH6NhoBKo88euFvuNtfPTaVC3MZJ3zqpXGVHlb0JLqdlbGJc0SQqKxEvOTEVWfgN87CRxwAxtBXwGpOlGnMrKp0OpdxSEUIUyZNUHN1QLK7DMaDfiYvOFpno4AAgyoyOy6MbGGnjqOKGUTLHyj9/lqV+BTWlqeeCa9gWAQJEEAghXFufTb2dlCHS7MSm1jm8DjJsOwu1szczskTIAJdz6OYmt+Xhv1GF4ens9s46FRNdGS3bHhNsO7uGLFtz+2Fcf44t1JZSaqY1tashXd36872oMl1bQEIgPUmL8WLOH77Pd58/zkmhUtsmSt+qEK+yg2SLgOp/VEFl6Wmj2w8u/qvqpqdAycsuQZLDOZJMgrOM9Gm3nl65ZdLDAOUUKjq5m0gc02AsyDoN6MUKuLTzAMOnMxslh567lKbbz9eRWTejWjOzNsrXpZE0C6agIIdyol+8zdBx0G7ft+vlDdd27MKHeJqr03g5qVwtTw8zZuVMQ/Dh+sFH5WZWRlpKMkTx5ze3HqTTzXobUhECoFt299Racf1gVgm79zQdDIEoK/x+DMHWV2eIcqLTSkBeQR7qk9JMSqY1tasc1SRmzPVVue9muu962p9GQwJVGlOQfP34vVyjgb7q+Wkp+QCAM9VY6vJfkb71gsSr7NUr3RWBtTsB3HDATN+8o1ePb7pdSlAZcso6JlqyErcBEvcDmFyH0b4x+86e3/5KxNSyH27saiJxTMOxQDfxWfQ7/cSZK3vWF4GSgYP372MHmoifG9bKSNxlffdK3tL4qjSStlXvNLTv6yMPTjxwsh1jajVmmb/CqYuPjm27WAZMVQPX0e09UOVdJBLS5OUKn18/di2nSIAxlXWMncctGeGlIW7dFH2vPjY3D0Yb93dnYQiQoDD5xdVbJ7KLBRQ5DSPH8dP76VMQgjpSmulKybSGdjXXDipXPr//uFNlutaIdIGIKruJst+Gp/B5wgt//n5BnIruPPvAwk6MBqVv1SCMz+Px+bxfXQ6S2hQVFaqra/4aafjvQ5Zs+zJk4/xOSq1xSP6VpiP5MYg1pUo+1iFpYfxkafg5SWmlUP7xbsgjZs9lLoqtebWDrNVtD3J/qxbMz5ZGlB56+M9LqVQN887T5g4wpLTiekHW6rYHAsDKy8t45eW/uiQktSkp4aqqqZPSNAPSdG0Psabip5zkKNRiIaVpNqTp2hpUQCAj03o/t2uzlJSUkNI0D9J0bQ+xptRH/9z91SUhISEh+Tqk99MWSnx8vLm5+a8uRauENF3bQ6zpf3oLbxISklYE2VuRkJC0Dlrz/lD/DXTe6zTp+AyzjB9UEhKSXws5tyIhIWkdkL0VCQlJ66CN9laoNCPqWdinkurnncKogxP6j93zhvfV2LYFKom/vH6GT59unt16+QSGlkg9ro5NSEh+HM1qcU3srVDeVX+v3tOOREnWasGTdb27zf37C9G0c0nLovCftYN7eHbu3LmzZ7deg0ZNXbLj4tscUdNOIow7tXL5kRcF1aXE5TT09PU05fGvxn4HSh6s7Dtid6QQAOVd8+/pG/yxRvnLPl5eNrTH/Ct5P75rEMYeD9j1RmXI8u37dm9a6mMr9Y3JujYh8iLOb1k4cUjvbt16Dhrvv+POp4pPWQQZz0LWzhzZv0e3HgNGzdl4KZZLdnG/GJR3ZX43jyo8xx6ME9c44suLI0snDOrh1XPI5NWnIqrklRYuSXPT8h6t7uXps/VVrZFRlHRyevdus89nEgA1WxwqerRxzMBe3bp07tylW49+wybMXRsUmlrPh1PNWGVHJbHHV65n7Q3ow2qKX9RGI+IWFIqsJ2yZ4cIUlOalRdw+tX/Bi6TAQws6yH/D/iW40eB1+wY3L7apCGLCoyh2C02pAKWRr+OU2o3Tr7AUUfzxf+eCgs49SS3HHb9Xdg1AZEa8y9TyXOrTyabJSmN0AY9qPXTeCF1GUfyto0c2rlHgHJlsSil9c2rvHW7HEf7jtFHS/eCgHSso7JML27eJHZRaK6i0tIxiOurPJT3UMQDA6Kp6FAAQJp5etfJMqef0P2app17bf2jpWrkjmwfp4NLCJU/Z7LREQU6uQJR148CFwQ4TOJXVDuU+OHQ6lkfo5eUj0K7V4oSF2Rlc8/Hbp7Wn8UvyPkfeOXlq1YL8zcf929ccXZvzTJBm6sh+t2P1MYNdk6zqqaPCzGfH9x27F/7xC6Fi7jF6zm+DLGivNg9fEjXgQNBkUwqA4OXGYUvTJ57ZPlgDB1R4c/GwXTJLLq71kqs+B6akb2tjwwQAhw7uHMGEWVduhs3q4ElPvbJqadCr9EI+VUnfccAM/0muGjgAkfc8eNuxh1Gf0gv5dDWPeX/94QUAoqi9I7vsBQB6l9U31nVNP+o35Z7LnuPTLCnwtVgiJ+zknoNXnifkgZqJ26Dps0c7quEAQHx5tHv9seeJ6dmF5cDUMHUfPmf+MOvqTlTwInDwohuF4knmkh63K8Pn9EpdfHFLfxWUfnPXvhcaQ9f4J6zbkdcM0zcZPo9HpJ2Z0fUMAFAtpx/7awz7c702rGsTL3nnsfMqHB05mJW9frQ94VM5mMrJdpgbfJRKowIAuDlQ3z/9IzrqM9HeuI0uKrQKELeYC1omdmamEo0IBO8u/x2v7r3r96E2NAAbevKY5eevxvefZiIl3JLyPdKivJw80GBrpf59/LH3H57iLdIEseePv5DXY5cW5BcSABQgEmu2R8CU9K2trWUAwKGDi3zqoDXR0RlEe06NWtWc3oqiN2ilH2vuyjXbzfYv6aRac8JT/u7g76vua4/+bf18jaJXIdt3LdujHbLYxtGGficmrhCZqmKi5MiYQmFOdDx/sAYDBPGR8cjC1076R10UpqwMJuTxRQCYqu3A6StHqsujnPDTOw+v22d6ZlUXeQwK4p48Tdf1XbHATkFYjOur4pACQDEdtXF5b00McHktmbpnlR7Lizn8+5KLWPepK6Ybwafbhw8uXsTbtX+yJR2g+FNERL7RlFULzelln5+eOrB35V79E4tdKp210RxnHbswmRsa6Hdaa9UeXwu84F7AzDtWGzf6GCsrYQAY22f7uRE4Lni2oRl2byaYTv/V632McMAYajq4VBtKtwlR8jns4p04hQ6zKlSq6KoAgCjKzRMydNnqrXHXvjYEUZRfRMV5OV+KaOqK9AoxiPTomHxZWydzsXMuWTtna8rd6JhcoWz94YSlZlXf8A1piaL8Qsx48iz982tOXPnkMZ5DAZT3IORaofuceUoHN2TnlwNIdxdGCLjZMTfvvBNqd7Fj1R4Am/e+FabiNuePsb/N3bzF3mRdH7XqCFQUevbql/YL9kz0VMIATBdkvRgZ/PDdApd2zpZw6G10+QAPmdyId5my8ljU2w/CTrbYxzeRXKNeDmq1CoaEfD4fE3Bzkt/eOnApUabdIDsmACbH6dCJAwAAZgqJ/0y+FZ0s6mJNBQDA5DnObo6VPbwAAEBGVc+IU+mors6imrRYVPz49KVk4/HBi4Ya4ACO7dilnyadPvt05GpPOQAATNbA3sXJkgKODhoZr2bcffZe6GJfaUaagpqGbG5eNhh2tdXSUBalf8miGvlYaWtUzkFx/KdPQTCaMsuIU+0CRboNa9oEAACK7y4b+ufjMkRl9Vi+vKd2zcILUm9sO/befPxez1a5x2gbgs/DFeWj900atplQMHIbPGPeRFdNCpGfWwDKaiqVosmoqStCcm6+SEo4As2qE35DWn5hYRlNUaP9iBF2N46dfzVssatM4tVzr9QG7PHUe3wK3hcUEqBQtxUIHq3p2WUNIEQghDE4QwLG2de5cWv226EMi7HLp76duSfwkvUG7apQUWpCYllZRuBgr0BxABIJMZncEnDp4Ga2+2xYrKCTydvwRMuRY5hnHoSnENb0t28y2a6u7Fql5z9e399rPQAARlMycp+0dv5AbRxAkP742F8n/xeVmsOjKdJKRRQbfnPLLxVRSnwCT8vDvrLRUtj2dhrBz+NSRJ6WNdskrs3WwQoLi2qtMpYnf8pWNzKUxwAVfkoq0LHXp3/3Qn4DTbOhXMe5fx0YkfH+33NB6+cEUg8s61I5l+YlXVv7+950j5XbR3BavGfNto6cx+8nPH4HETct4t7R7ftWrqAf2D9OHwBB/cOItPDGHPOVtAS3qBhkNWUpms4je5xYefbuBBut81czHCcPMaOXv5EDbmFxvQ9lqO1n7J/VgQai8oK0yDshwSvnwIYDc9vXWKv+hnfZqQaDl8x8OWX3pgvj1KsvBQGu1mPJlrHmVWM5LquuiOGKLm6cAzefx3NLX0SxXaZ1Z745fj8soxf1+Uct17lGtZfrafaTt8xwkaUxFdW1tVUY4o6DSDzzx+oLeP+5y+eaq2CpV9ev+beB4jUsx/eYC2BUKqXCCWIFglfbRq+4lVvOE1Hm97kMgER8PhExtdcJuvOCk+v7KP/6GUhDNqyvdLicFsdCi2NhZ8vM8Nlw4aGfx1AtHECQen31gr0ZHqu2zXWvPS0m+WVQ5NmOgxdOi36+KvRJ2hhDFTUVyM/LJwAoAAD8vJwiUFFToUgJl9Qfb37aEm4pyMoyMWA4DfM28LtwcI/yU2rP9V7qOJYny0Rcbkll4hpgspqGHI4MAICpVTudwugZf99+O7N9J8mh/ptqGq7dZ+Fsp9SQkGeVTxspesaGtPyENIJlUIWehhwOgOt17szJfnz/3MM3Ks7OuuodXDkJ/964+ihWy8PTtE6fiSnoWlpZWpgasiq7KgDgJ8YlEnbek3o7WXA4phZsBemNH6MzZIBbLOXtoQZjKfrmxvSsiLfpFXeHorS3kV9kTMz1G/EAlGbru3PHeCua3uB1R4KDgzb6GNLMRm8JCg4+uLBLy/CVK9WGDVsMAMMwDIRCIQBAacSh5btTXFZsneuuTnZVLY0qZz84y9pKpTQy/L0AAADK3oVFi1jWVmpUKeGSUjY/LVFSzEUyTCYGgOv3G+aS/+BmjIH3UHsGAMZgykAptxHv9AlKuTxEpdNqtZlv/E4Q1+gxd/r/JgU+rribwJQ8fPqfWHj6jwDK+L62WvTy7E9FrD69bOQwwPW6djMLPnTyi97ovRwKjnXsbHTo4BlCb/Rss0YWgmbAYcPFq8fuqHtxlClfMhq4aoqemTH9wv0TF628jVFWgXLHblaNjcUUOo4aYjA3ZPVm5uSeRijx9uETnwyHz3eXq5NJPTBUNPn5WVTjIQ76bKYgNe+LnGk7Wz12y7lTkmrDujYxS7seEiVjaawpSxSlhF0++gSsZ7rr4EBk3Qm+lGM3uY9W3scPeQDiB+YGanWfZJD8HIjPj04+LDYw01XES1JfXz72kGc2uRMbB4qd9xCz28e2bmdP6a6Wdm3fPZ7D7AHmFMDrD0eFoYFTAqM7Bhyc6yxLa1ra6sKgstJyYDIYAACYcpfxU6MUijr108MBAGhMJpUo5pYhqLs4ggqSI9+9ownLi7I/vrx+7noue0hP21oN55u/asY1e/3md//tLmHF33KOM7dtUNoffHPXyuASkNc09pzepSfIYQA4q3sfh6C4PM+uJhQAYHXpanEwXtS7t3Fj39qimI5eMT9n58ndSy5wCRpTUVXfkiVlfoUpeU5fEPHnwaCVoUJZtoufZVerRscCw9pv0wbG3kMn1/rng6qJ69jAOWMsG9kWUeGnxAKWvaEMAJGZ+Klcr6fBD3kprblItWEdm3iyudnv7925dDCbSzBU9W06zd06eZAeDiBIjH3PKy7eO+t51UkNxx04OtW8RV3ofwmiLD/p+YULxz8XCulKuhZu0zZN9TGiAACVM2ptQPm2/YdX3S6TM3T3XT9vIAuXGo5qeApqWtpqUFlZGcjIyogbJt1s0PzF1ZEMJgNlcksQKNW4AKqSprb8gxD/2cFAocsqaxladJm9bay3vSzUhNyNr4VStaUcuQdDUyF342t7kLvxkZCQtCbI/a1aOuRciYREDDm3IiEhaR2QvRUJCUnrgOytSEhIWgfU/Pz8X10GknooKioipWkepOnaHmJNybkVCQlJ64DsrUhISFoH5BsMLZ1xy/c26fiQP2f9oJKQkPxayLkVCQlJ64DsrUhISFoHbbS3QmVZcWERKaUSPrjiQuaMnXkkkv/V2P8ydS1DQvIjaFaLa+K6FSq4s3pycMnQ1QGjLGQrtz8QvNw+bmPuuIMB/b7Hxmyo6PH233b+m88nAKcyFDTYZo5dvYf1sVFtyif+wg9/B2745L3HVr+ylLisGktXV0MO+2rsd6D030C/EPWV+/wsKQV3V0+9YbFp2yhDCoAgK+zyyfP3wxNzeAwN0w4DfSf1Mf1eeX4X6lqmGtGXx7tWbH+sNG5PoLdOGx3l/nMQBZGXDgVfD0suoqiZug6c5NvPrM62Jqjg9qrJ+yIEFX9S2D6bdo8zrWyO5Um3tgUElQ0/FNC7nt0mUXna80vnrv379mNWsZCioGFg7thr1PjuHIZki0PFT/cs+utxdlG5EKh0OWVtI2v3fiOHuLFqb3zSnFX20g/nAndqBS7y0v4he4QQJYXFhPmI1RMdGYKy/Izoh38f/SM8dcXW6Q7f0rBx/b5LN/ZtXmxTEbx/F0exmsmhApTFRiQoWvvoUgAAyqL+DnpY0mHQDB9NlBp6+tTBDRTW/pn2rWFjKFT45kjAwXf8lrNTF8m3Q6Tf2Pzn2ZLO01bONhDGXT1wJGAnY8ey7rW2WESlZeU4Z/CqOeJdrjG6irg6E9ykp1dPn7ryMp2H29Z3elQSdWLVuouf1V17D59hoSFTlpeRFJtcAjSo1eJExTnZJSY+AeMdaPzS/MzY/128uGlVwR97ZtTamr1ZHro4tqyYg5vP6a0fZVZPSxNmh50PPvsoMimHUDZxHTLZr7cp7e1ev3VxvTbvGM2hAAje7J68LmvkwYC+ajigovtrJx+m/xa0xENyNxtFXUsLCwYA2Dq0NxDMWXL7fsQkB3da+u1N6069ySoWUBR07XpNnDHKSQ0HIApen/nr7OPYlKxiAU3FZermhR4AIIoLmuYdBAA0t99PLe2YeWbBgkeOgXvGi0eFBmOJvIiLR47ffp1UACqGzr0nTB5ip4IDAJH77MiOs6+TM78U84Chxmk/0G/aAPPqTlQQvss34H6ReNfRdT7/VIYvG/l5zpHVPZQdpuzcVeEwxtmW8vHV5vjYDMLe8IdOVOqRg1n4bMeCLR891m7xtWKAMO3KyoWXVGdt9/dQqWsZDxkAEKXf2bEr0va3OYyg9VE/srAkPxMiNfRBHKPLCj8vGzoAZ8bE+Kmb7vyb2W1wTWczqIRbAhpGVsacGvtNEZkPDge/Vuu/aMan7QfqexmXF3Nmz98peiPXrx1pWtXr9KlMnVyzPQIosC3MzekAYOvgJPd5/Jb4uCzC3uCbPXThrD4LxmivCNxygLNpjkut6R8vJmTNxlDNIVOWTVMrfnv+wOH1RzT3zrGws6A9jE8oRhxlTJQW+75YlBefIOirJgPCj7EJyHRUfY4JK6AwmDKYkM8XAdCVLXtN8PdWlUV5kZcOndwezDng7yaHQeGHl68ydUbOn24lL+Liuso4pAFQOINXzuumjgEup1F3q8IGYvnvT65Zex3rPG7+BH1IeXgyZG0Af/2m0aY0AG5ydEyB/lj/Gcb08sxXfx8PDgxi75vjWOlejGY7afeRMSXPds2/pOG/YZQpXvho6+KHZitXDjRUUsSghm+r4vwCoYwOS/XH3gjWL4eTq69fx/nb9p912zJG4c5fZzKcZi7qqIIDiOqzjCD58s5TJX1X+9qLzvzQwpL8XIiyklKQVZCv6C5kDI1Z8CIpTQQ1eyuCW8Cl4vy83GKaqkL15sM4a0DAoUE4LgjbVe/Zee/uP8xU8PDzNm2KY1xCUJLz/v7DGKGmu5V27WG8me9bKTn7+Q9btmLvPmujJV4q1eGo+NmVO7kO0zeMdFfEADjTv4RPO/MkZrqjtb0pnIiK4/V0pedHx2Qz5bC4qEShiyWWFBlbYtDVVqVWwZCQLxBggpLctKh/jt9Mplv3tmIAYLIGDi4GAABgLJ/8eN4/cWkiN3PxNcgZ2DvbmUp66KKrsPQNKldY6njokhaLuC/+vplmOHzn7P5sHMDOmlWWMu/S5Vfev7uLhxYm28axnQV8Zc4AAAcfSURBVCkF7GzUst4s/l/YR6FjlSNkmryKGjOvIAf0OllqqCmJMnO/UPUHmmvW3gZYkH7/r7MfTYYHuv/Q7dqlyeHkrOw2aar7/G179xbJh6U5zvDvKCFADcuAMPX6vivgvdabQ8M+/MCykvx0KGwrM+aNJzcee03z0JUR5Gfm8JBIIKp9mICHy8vFBc+dvBfJ6zv3nTh1hJM6BeAr7uaI3LS0UoqRuUkjFzqEz7YM995S4aFLxqDfYh+bOimb/XaojOmw+eOiFwftvmmxvNoNmSj9U3J5edZu32G7xQFIJMTo+aWg7NDe+PDliPcCF6OoyGQz76GMy/+++0yY06Iis1lOTrVXbQUvdo4duhMAAKMp6rcftWRaL00cQJD54tyxi0/i0vP4VAVqGUGx+AEeuj4nJPE1XGwqO3aKjo2l2pnXHz6L3E1reejS1NGCoqLaDod4aSk5avp6chig4pSUQi0b3VqLPfzUu9vWBGW6Llg7yODHrgNJkwOBEqbsOmmi4+zt97PsZy/sWHuwqILIfXT8UonX8v4GNPHMi6TtgMm7+M4bsGXPrpmjtmMAMrIMAaFoXWf8lHWdtc91FohK0qMfnT0QHLiBtmWTTyM28EYIAHDJs4kSzy1b96Ld0k2jTesM0lT7iRsnOdBAxCvMiP3f+dOBS2HFlin2Ndaqv8VDF7vvHN83/of3XPNRlSwhrtJlzuphJtUeupiqChiu4OhseOzB648lZeGxOo7jOzMiz4W+zepKfZ2k4TS1jj8Zms3oPyY6MakMBVVNTWWZCg9dyZc3b76G9Zwyb4qJMvb5zs6tz0E6P81Dl0SI4O1fMzY8yOfxRZSVo24BIEIgIGL8R1yg20/ft8xLCQMQpN/bvCooy9U/YEp7qX3Ed0OaHACAuInRSTwGEz48fpnl1adqvKhhGVQUHhpemBa+bNRVAAAgREJ0fPa4T4uOznchF9xbP7ias++GoNGFOfk8uiL/4eq5Z1UspPVDFDmWXd+Z4+Nfb3r+MmOoQW0XoHVOraqtRSfiEpOFYFdVVUTlRYVFpUJUXwNkqusbGNABADhm1ppFcYtv/BPla1+jmn3Tlze4ZreZk17N338+UQCm4itiGepRr3/KILQ8a08bWG6uhmcehF7Nj1S299ZWlXU2OPbi/h38vYbrBKO6pZDXMTOrfb8rSE5IQlYzR3Vrp4ABIasjL71kGF1GBkq4Ut4bajCWomtiQLseHZVFmOviACDKiIrNpRuZ6FLquZ2sBc1y5Lp1ajuX/mO0eEV/XSzn/qaAt06rFnqpY0wVBQwAyqJPrD+c5rhg/c/oqhqSAxW9Pro/VGnEn0tpQUtP7L/dbnVfFl7XMphixzm7LMsr/iRSrvy59YPbyoUDLcgvttoOuIySprYg88GWKx9VPVc5yzc0khONd+Mg066zq+LT0It3+1v2q3178TWEZSV8RKXVTvWtHrrUukyZ8HTe7hcVL2Ngiq4Del5Y8/emrfjw7pYaNF5OSrG2V1cLWQxwVsdOxmdOXMxlDQk0oOBYezf9kJDLiDVkEqeRhaCyDVhw/c7Zh2qdDRTx3KxS6YdSWMaGtOuhF66b9TFEX4qUOnQya2wsJt9hSD/28vOb9zJGe+qj5IcnL6ToDZrevrb/jXqRUVbnF36hGvaz1WUxBOkFubJG1pYsVoXViS8Pz9zMsxrjpZGflJgPIH4azFb5Yb6cpcnBLIs4eShUwXvjIGNDfMbI5/4nD913XtlTE69rGQt13aorF5UpUjC6shZLldGSXhIjaTaoNDspLTM7OebZ7RuhWbrDl4+3YwKgome7/XfHdVi8dUo7JpHx7OITLttYWwEvTY+4dfYJjzPapTHv22Fy7cdN8Yjefnjp8sSBvToYazAQNz5Julu9otTYmBiqkFeck/Tm3pV7+Tr9PC2/u4cu9a5+Y0KjDleuacjaTQxYrnD09IPDgWdKQU7d0H2CuyfIYgC4dmcvm1MJBe4djSgAoO3W0TTko6hbV8NGe+jiDJk/LffQxSNrr5cgKkNeRddMS8pAgCm6j58evSPk9MbnQqaO0xizjmaNjgUZ8zGrVsgEhVzctroQlA2dhq3wG2rayB4FFackF2pb69EBiC/JyeW6nuzqyxMlf/jI53KDlryuuiQ9n827xpn8ON9W9cohzLpy9AH0WOXNoQEAu++k/veXnD79yv03F/k6lrFowMcsSWtHGHli+eYwGW1DS8fRa5Z1t1Wr7B6qXXUR5QWpr69dO5dZLKIpaJs6j181fkBjPAEDAK7Wad5mNcvzF+/dDgrNKyUockoaOvYdzOvcVlAU1DXk/r3wx9IzQKExFTX0TN0nrRnWx4ZZ6zgsLy/vWy6X5AeRkJBgYmIC5B4MTafKdCRtBrGm5BcUJCQkrQNytbSlQ86VSEjEkHMrEhKS1gHZW5GQkLQO/g/Xa6pawNoFMAAAAABJRU5ErkJggg==)

Go ahead and use the `Edit -> Apply all Operations` menu to write the new partition table. Once that’s completed we have to set some partition flags to make the drives properly bootable. To do this, right click on the first fat32 partition and choose ‘Manage Flags’. Click the checkmark next to `boot` (which may also automatically check the ‘esp’ flag) and hit Close.

Keep track of the device names (they will show in the Partition column with names that start with `/dev/`) as you will need them for the next step.

# Step 5: Mount the destination filesystem

At this point our target system is prepared and we are ready to restore the image onto this machine. Before we can do anything with the new hard drive layout we need to mount it.

Boot back into the Ubuntu livecd if you’re not already in it, and open up a terminal window. We’ll first create a mount point (an empty directory) where we’ll mount the two drives. I’m using `~/efi` and `~/dest`

mkdir ~/efi
mkdir ~/dest

And then mount the drives to them. On my system the drive I was partitioning was `/dev/sdb`, so my EFI and data partitions are `/dev/sdb1` and `/dev/sdb2` respectively. Your system may assign different identifiers, make sure to use the names shown by `gparted` in the `Partition` column:

mount /dev/sdb1 ~/efi
mount /dev/sdb2 ~/dest

# Step 6: Use `tar` to extract your image to the destination filesystem

Now that we have all our mount points set up, we can do the reverse of the image creation process from step 2 to duplicate our source machine’s filesystem onto the new machine. Since this can take a while I like to use a tool called `pv` (pv stands for pipe viewer) to provide a progress meter. You can install `pv` by doing `sudo apt update && sudo apt install pv`.

Once `pv` is installed, we can start the restore process. First, find a way to get the Ubuntu livecd access to the source image we created in Step 2. Most likely this means plugging a USB drive into the machine. Once you have access to the image file run the following command, replacing `[image-file]` with the path to your source tar file:

pv < [image-file] | tar -C ~/dest -x

The above command is saying to take the contents of `[image-file]` and send it to `pv` over stdin. `pv` reads the data from the file, prints out a nice progress meter, and then sends the data it’s reading to `tar` via a shell pipe (the `|` symbol). `-C` then tells `tar` to first change directories to `~/dest` (where we mounted our destination partition in the previous step), and the `-x` tells `tar` to run in extract mode.

This may take a while, but when the process completes you will have completely restored all the files that originally lived on the source machine onto the new machine. Getting the files there is only half the battle however, we still need to tell Linux how to boot into this filesystem, which we’ll do in the next step.

# Step 7: `chroot` into the newly extracted filesystem to install a bootloader

At this point we have all the files we need on the new system, but we need to make the new system bootable. The easiest way to do this is to piggyback off of the Ubuntu livecd’s kernel, and use the `chroot` command to make our current Linux installation (the Ubuntu livecd) pretend like it’s the installation we just copied over to the new machine.

For this to work we have to use our helpful friend `mount --bind` again to do the reverse of what we did in step 1. This time rather than avoiding copying these special filesystems, we instead want to give the `chroot`-ed installation temporary access to the special filesystems that our Ubuntu livecds created so that it can act as a functional Linux installation.

First, change directories to where the new installation is mounted (`~/dest` if you followed the example above):

cd ~/dest

Then we’ll use `mount ---bind` to give the chroot access to the linux special directories:

for i in /dev /dev/pts /proc /sys /run; do sudo mount --bind $i .$i; done

> **NOTE:** We use a `for` loop here to save ourselves some typing, but the above line is just telling the system to run the command `sudo mount --bind <input-dir> ./<input-dir>` for each of the special directories listed between the `in` and the `;` . In other words, the single line above is equivalent to running the following:
> 
> sudo mount --bind /dev ./dev
> sudo mount --bind /dev/pts ./dev/pts
> sudo mount --bind /proc ./proc
> sudo mount --bind /sys ./sys
> sudo mount --bind /run ./run

If installing in EFI mode we also need to give our chroot access to the EFI partition we mounted earlier. `mount --bind` comes to the rescue again here, we simply bind mount the livecd mount point into the `/boot/efi` directory inside the chroot (`/boot/efi` is where grub expects to find the EFI partition).

cd ~/dest
mkdir -p boot/efi 
mount --bind ~/efi boot/efi

Now that we have access to the Linux special folders (and the EFI partition), we can use the `chroot` command to actually use our source installation:

sudo chroot ~/dest

At this point you should have a shell inside the same Linux environment you originally copied. Try running some programs or looking at some files that came from your old machine. GUI programs may not work properly, but other then that you should have a fully functional copy of your old installation. Booting into an Ubuntu livecd and running the above `chroot` commands every time you want to use this machine is not very practical though, so in the next step we’ll install the grub bootloader to make it into a full-fledged bootable Linux installation.

# Step 8: Run `grub-install` from inside the chroot

Grub is the most common Linux bootloader and is what we’ll use here. Grub has an MBR flavor and an EFI flavor. If the machine you cloned from was running Ubuntu it most likely already has grub installed, but may not have the EFI version of grub installed. Run the following to install the EFI version (feel free to skip if you’re doing an MBR clone):

apt install grub-efi-amd64-bin  # skip if using MBR mode

If your source distro is not Ubuntu based make sure to fully install grub via your distro’s package manager first.

Once you have grub fully installed then you just need to run `grub-install` against the drive you installed to. In my case that’s `/dev/sdb`, but this may be different on your machine. If unsure fire up `gparted` as we did in Step 4 and check the names listed in the partition column there.

Next we install grub to our drive, thereby making it bootable. Be careful to install grub to a **drive** and not to a partition. Partitions will usually have a number on the end while a drive will usually end with a letter (e.g. `/dev/sdb`, not `/dev/sdb1`).

grub-install /dev/sdb
update-grub

If all went well you will see messages saying grub was successfully installed. When you see this feel free to reboot and check out your freshly cloned installation.

If you got error messages and are installing in EFI mode it’s possible grub tried to use MBR mode. It might be worth a try running `grub-install` this way to force EFI mode:

grub-install --target=x86_64-efi 

# Wrapping up

That’s it, at this point you should have a fully operational clone of your original system, and hopefully also have a solid understanding of each step in the clone process and why it’s needed. Once you realize that a Linux installation is really just a filesystem and a mechanism for booting it, tools like Docker start to make a bit more sense: a docker image is basically just fancy version of the `tar` image we created here, with some changes to handle docker layers and make the image files easier to distribute.

In fact, just as we were able to “run” the system we installed via `chroot` before we actually made it bootable, you can convert the tar image we created into a docker container quite easily:

docker import [image-file]

99% of the time you’re better off just using a `Dockerfile` and docker’s own tooling to build your images, but if you need a quick and dirty way to “dockerize” an existing server you could do this without even having to shut down the existing server!

Similarly, the `docker export` command can export a tarball like the one we created for any docker image. Once you extract it you could use the same `mount --bind` and `chroot` dance we did above to get a shell inside the “container.” If you wanted to get a bit crazy, you could even use the steps from this guide to restore a tarball exported from a docker image onto a physical machine and run it on bare metal. In real life this won’t work with many/most docker images because (for space conservation reasons) many docker images strip out some of the files needed to support physical booting, so you may be asking for trouble if you try this in real life.