# oliverhi06.github.io
This may or may not be too comprehensive but I am too afraid of completely ruining my installation to not document it this thoroughly. 
### 1: First steps:
1. Download the ISO from the Arch page(I opted to direct download because i don't have a torrent client on my computer) 
2. Make a VM. If you're using VMWare, use other Linux kernel 6.x 64x and allocate system resources as you see fit. I gave my VM 40Gb and 4 cores to work with. I figure that's gracious plenty.
3. Go into options for the VM and put into UEFI mode under advanced settings. It is not set to that by default, for some reason.
4. Start the VM!
5. Because this is in a VM, the network should *just work*. To check this, I pinged Google's DNS server
`ping 8.8.8.8`
6. I then used timedatectl to set my time zone to Chicago:
`timedatectl set-timezone America/Chicago`
### 2: Disk Partitioning
I'm using fdisk because that's what Clark (better at this than me) was using when he taught be how to do this. This was the hardest part for me, I really couldn't figure it out on my own.
1. Open the drive you want to install to 
`fdisk /dev/sda`
2. Specify GUID Partition Table 
`g`
3. make a new partition. This will be the boot partition.
`n`
4. use the default partition number (1) and default starting address
`(enter twice)`
5. Add 1 GB
`+1G`
6. Make a new partition. This will be the swap partition.
`n`
7. Use the default partition number (2) and the default starting address
`(enter twice)`
8. Add 4 Gb
`+4G`
9. Make a new Partition. This will be the root partition where we store **everything** else.
`n`
10. Use the default partition number, starting address, and ending address
`(enter three times)`

We now have to specify the disk types. I am told that it still works if we don't do this but it's probably good form to do this. 
11. Start by setting partition type.
`t`
12. Specify the partition you want to change
`1`
13. Pick the type of partition you want it to be.
`1 (UEFI)`
14. Repeat this process for the other two.
`t`
`2`
`19 (SWAP)`
`t`
`3`
`23 (Linux x86-64 root)` 
15. Write the changes to the disk.
`w`
Congrats! You've set up your partitions. 
### 3. Disk formatting
1. Now we have to format the disks into something usable by the file system.
`mkfs.ext4 /dev/sda3`
`mkfs.fat -F 32 /dev/sda1`
2.  The swap partition has its own set of commands.
`mkswap /dev/sda2`
3. Then we mount them so that we can write to them.
`mount /dev/sda3 /mnt`
`mount --mkdir /dev/sda1 /mnt/boot`
4. See step two.
`swapon /dev/sda2`
### 4. Misc. Installation stuff
1. Use pacstrap to install essential packages to /mtn
`pacstrap -K /mtn base linux linux-firmware sudo networkmanager nano` 
2. Make a Fstab file so that the machine can identify file systems on startup
`genfstab -U /mnt >> /mnt/etc/fstab`
3. Double check that the file is actually there. (It wasn't at first.)
`cat /mnt/etc/fstab`
4. Change root, allowing you to pretend to be booted into the system that we so diligently made.
`arch-chroot /mnt`
5. Configure system clock
`ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`
`hwclock -systohc`
6. Create a hostname (computer name)
`echo archlinux > etc/hostname`
7. Make a root password
`passwd`
### 5. GRUB Configuration
1. Install the GRUB package with pacman
`pacman -S GRUB`
2. use the GRUB install command 
`grub-install --targetx86_64-efi --efi-directory=/dev/sda1 --bootloader-id=GRUB`

That's it! You can now safely reboot into arch linux.
### 6. Post-install, Pre-GUI
##### Creating a user account
1. make new user:
`useradd Oliver`
`passwd Oliver`
2. edit `/etc/sudoers` to uncomment the line that lets members of the wheel group use sudo 
	I'm pretty sure there are other, less risky ways to do this, but I don't know what they are. I made a snapshot beforehand just in case.
3. add user to wheel group 
`usermod -G wheel Oliver`
From here, we can `logout` of root and use the user account that we've been working on

##### Getting connected to the internet
After trying to update packages with `sudo pacman -Syu`, I learned that I have not been connected to the internet since the reboot. In order to fix this there are a couple things that we need to do.
1. Start the network manager
`systemctl start NetworkManager.service`
2. Profit. 
	When I started writing this section I really thought there would be more to it than just starting the service. VMware and NetworkManager are very well configured by default.
##### Installing Plasma
1. In order to install Plasma, we first need a display manager. I'm using SDDM because that's the only one I'm familiar with
`sudo pacman -S sddm`
2. Now we can install Plasma. I'm using the default options for everything.
`sudo pacman -S plasma-meta`
3. Start sddm with systemctl
`sudo systemctl sddm.service`
We have a GUI!
(I have to go back to the terminal and install a terminal emulator-- I didn't install one earlier. I'm going with konsole)
### 7. Wrap-up
Everything is a lot easier from here. 

 We need to create a user account for codi, which is just recreating the process described earlier but using the name `codi` instead of `Oliver`
##### Installing zsh
This isn't too tricky. Install `zsh` using pacman, and run `zsh`. You'll be presented with a menu that explains itself pretty well. I opted to use the defaults for everything, as I am not experienced enough to have opinions on these things.
##### Installing ssh
This is even easier than installing zsh. You just need to run 
`sudo pacman -S openssh`
and you're done.
##### Color coding
This took me some time. It took me a while to realize that the color coding is handled by bash itself and not the console. I tried using some themes built into konsole, to no avail. I eventually figured it out, and decided to use [This guy's](https://yalneb.blogspot.com/2018/01/fancy-bash-promt.html)Bash styling. I had to create a file in my home directory named `.bashrc` and pasted the data in to it using nano.
##### Aliases
I added an alias "please" for sudo so that I can be polite when I am trying to install something, and added `c='clear'` because it was recommended online. I added both of these lines to my `.bashrc` file.
`alias please='sudo'`
`alias c='clear'`
