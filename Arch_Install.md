# Arch Linux install process
## By Mason Davenport

### Below are the steps I have taken to install arch linux as well as a desktop environment
#

## 1. Step One: Download the iso

- 1.1. By navigating to the [download page](https://archlinux.org/download/) you can select your preferred method of downloading. 

- 1.2. I chose [acm.wpi.edu](http://mirrors.acm.wpi.edu/archlinux/iso/2023.10.14/) and selected [archlinux-x86_64.iso](http://mirrors.acm.wpi.edu/archlinux/iso/2023.10.14/archlinux-x86_64.iso)
#

## 2. Step Two: booting the ISO

- 2.1. In your chosen VM create a new machine. I will be utilzing VMware Workstation Pro v17

- 2.2. Open VMware and select "Create a new virtual machine"

- 2.3. Leave the default selected as "Typical" 
    
- 2.4. Select "Installer disk image file" and navigate to the iso you downloaded earlier.

- 2.5. For the version of linux. select "Other Linux 5.x kernel 64-bit"

- 2.6. Chose a name for your vm "Arch Linux Project"

- 2.7. Leave the suggested size at 8gb.

- 2.8. Select "Power on this virtual machine"
#


## 3: Utilizing the VM
 - 3.1. Your virtual machine should have now booted into the iso file. After the machine finishes its boot process. run the command: 

```
cat /sys/firmware/efi/fw_platform_size
```
- 3.2. If neither 64 or 32 is returned, you will need to change the Firmware type by navigating to the vm settings. selecting the "options" tab at the top, selecting the last most option.and switching "Firmware type" to UEFI.
`NOTE: this will need to be done with the vm turned off`

#
## 4: Partition the Disks
 - 4.1. The first step in setting up an arch machine is to partition the drive you will be installing it to.
 - 4.2. identify all the disks installed on the system by running
 ```
 fdisk -l
 ```
- 4.3. we will be using the disk labeled "sda"
- 4.4. start up fdisk on sda by running
```
fdisk /dev/sda
```
 - 4.5. create the partition table using
 ```
 g
 ```
 - 4.6. create the individual partitions by using
 ```
 n
 ```
 - 4.7. the three partions we will be making are 300MB, 1GB, and the remainder of the drive. 

 `NOTE: Leave the partition number, first sector and last sector of partition three as default.`
- 4.8. finally, write and save your changes with: 
```
w
```
`NOTE: it is EXTREMELY important to create and format and mount the partitions in the order listed. this can cause severe issues when done out of order later on.`
#
## 5: Formatting the Partitions
- 5.1. format sda3 as an ext4 partition using:
```
mkfs.ext4 /dev/sda3
```
- 5.2. format sda2 as a swap partition using:
```
mkswap /dev/sda2
```
- 5.3. format sda1 as a FAT32 partition using:
```
mkfs.fat -F32 /dev/sda1
```
`NOTE: it is EXTREMELY important to create and format and mount the partitions in the order listed. this can cause severe issues when done out of order later on.`
#
## 6: Mounting the Partitions
- 6.1. mount the third partition as /mnt using:
```
mount /dev/sda3 /mnt
```
- 6.2. mount the second partition as swap using:
```
swapon /dev/sda2
```
- 6.3. mount the first partition as /mnt/boot using:
```
mount --mkdir /dev/sda1 /mnt/boot
```

`NOTE: it is EXTREMELY important to create and format and mount the partitions in the order listed. this can cause severe issues when done out of order later on.`

## 7: Install and Configure the System
- 7.1. Install all the packages that we will be using for the remainder of the setup with the command:
```
pacstrap -k /mnt base linux linux-firmware nano man-db man-pages texinfo iproute netctl util-linux grub efibootmgr zsh
```
`NOTE: you can install the packages later, but for sake of simplicity a large number is being installed now`
- 7.2. generate the fstab file using:
```
genfstab -U /mnt >> /mnt/etc/fstab
```
- 7.3. change root into the system using:
```
arch-chroot /mnt
```
- 7.4. set the clock to central time using:
```
ln -sf /usr/share/zoneinfo/US/Central /etc/localtime
```
- 7.5. generate "/etc/adjtime" using:
```
hwclock --systohc
```
- 7.6. create the UTF-8 locales using:
```
locale-gen
```
- 7.7. uncomment "en_US.UTF-8 UTF-8" in the file you just created by using nano:
```
nano /etc/locale.gen
```
- 7.8. create the locale.conf file using nano and paste the line "LANG=en_US.UTF-8"
```
nano /etc/locale.conf
```
- 7.9. create a hostname for your machine by creating the hostname file and typing whatever you want the name of the machine to be:
```
nano /etc/hostname
```
- 7.10. enable network services upon rebooting into the new machine with the following:
```
systemctl enable systemd-networkd
```
    systemctl enable systemd-resolved

- 7.11. identify the network interface to use in the following step by using:
```
ip addr
```
`NOTE: there should be two devices listed. in my case there was lo and ens33. lo is the looback address and not what we will be using.`

- 7.12. create the wired network file using:
```
nano etc/systemd/network/20-wired.network
```
input the following in said file:

~~~
[Match]
Name=ens33

[Network]
DHCP=yes
~~~

- 7.13. set the root password using:
```
passwd
```
- 7.14. install grub onto the disk using:
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```
- 7.15. create the config file using:
```
grub-mkconfig -o /boot/grub/grub.cfg
```
- 7.16.  exit from the new system using:
```
exit
```
- 7.17. unmount the partitions using:
```
umount -R /mnt
```
- 7.18. reboot your system using:
```
reboot
```
- 7.19. log into your new system using the password you set back in step 7.13.

`NOTE: if you run into issues upon rebooting with arch not appearing as a boot option, go back and make sure you created, formatted and mounted the drives in the order listed.`

#
##  8: Adding a GUI (Gnome)
- 8.1. install Xorg using:
```
pacman -S xorg xorg-server
```
 - 8.2. Install the Gnome DE using:
```
"pacman -S gnome"
```
- 8.3. start the Gnome GUI using:
```
systemctl start gdm.service
```
- 8.4. after the GUI has started, and ensuring that everything is functional. start the "Console" and run: 
```
systemctl enable gdm.service
```
#
##  9: Create new users users
- 9.1. using the console on the GUI, install the sudo package using:
```
pacman -S sudo
```
- 9.2. uncomment the config file addressing the "sudo group" using:
```
EDITOR=nano visudo
```
- 9.3. create the actual sudo group using:
```
groupadd sudo
```
- 9.4. create the user "codi" with a home directory using:
```
useradd -m codi
```
- 9.5. set a password for the user "codi" using:
```
passwd codi
```
- 9.6. add the user "codi" to the sudo group using:
```
usermod -aG sudo codi
```
- 9.7. set the password to be changed on first login using:
```
passwd -e codi
```
 - 9.8. repeat steps 9.4. through 9.6. for each additional user you would like to add.

#
##  10: Colors!
- 10.1. log into the user you want to establish colors with by using:
```
su - codi
```
- 10.2. first navigate to this [generator](https://bash-prompt-generator.org/). it is an interactive way to create and modify the look and feel of the colors for your terminal. and you can test it in your terminal by simply setting the PS1 variable for example:
```
PS1='\[\e[95;1m\]\u\[\e[0;38;5;51m\]@\[\e[93m\]\h\[\e[0m\]'
```
- 10.3. if you find that this is indeed what you want your terminal to look like. make it permanant by changing the variable inside of the bash.rc file.
```
nano ~/.bashrc
```
- 10.4. for sanity's sake, simply comment out the existing variable assignment for "PS1" by prepending it with a #

```
 #PS1='...'
```
- 10.5. paste the "PS1" variable you create with the generator right below the existing "PS1" that you commented out.
```
 #PS1='...'
 PS1='\[\e[95;1m\]\u\[\e[0;38;5;51m\]@\[\e[93m\]\h\[\e[0m\]'
```
- 10.6. while we are inside of the bash rc file. lets go ahead and create an alias for a command. for example, insert the following:
```
alias bee='echo According to all known laws of aviation, there is no way a bee should be able to fly.
```
- 10.7. test out the changes by saving the bash.rc file. And exiting from from the user, and logging back in.
```
exit
```
    su - codi

#
##  11: Congrats!
- 11.1. if you have followed the above steps, you have sucessfully installed arch linux!
    
