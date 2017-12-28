This will create a fully automatic installation of Ubuntu Desktop versions so you dont need a keyboard, mouse or a monitor to get it installed and configured. Tested on 17.10 and all previous releases under it.

Scripts are attached of my basic configuration I used that worked. Just change the passwords under root and user account and if you use /dev/sda it will work straight out of the box

Mount Ubuntu ISO

You will need to mount the ISO files so that you can edit the pertinent files.
```
mkdir -p /mnt/iso
mount -o loop ~/Downloads/ubuntu-16.04.1-desktop-amd64.iso /mnt/iso
```
Copy ISO Files

Now you need to copy the files in the mount location to a different directory so that you can edit them. /opt/ubuntuiso was good for me

```
mkdir -p /opt/ubuntuiso
cp -rT /mnt/iso /opt/ubuntuiso
```

Edit the txt.cfg File

Here we will edit the /opt/ubuntuiso/isolinux/txt.cfg file and customize our boot parameters to get a completely unattended install which will include a preseed file. Use any editor of your choice:

```
#default live
#label live
#  menu label ^Try Ubuntu without installing
#  kernel /casper/vmlinuz.efi
#  append  file=/cdrom/preseed/ubuntu.seed boot=casper initrd=/casper/initrd.lz quiet splash ---
#label live-install
#  menu label ^Install Ubuntu
#  kernel /casper/vmlinuz.efi
#  append  file=/cdrom/preseed/ubuntu.seed boot=casper only-ubiquity initrd=/casper/initrd.lz quiet splash ---
#label check
#  menu label ^Check disc for defects
#  kernel /casper/vmlinuz.efi
#  append  boot=casper integrity-check initrd=/casper/initrd.lz quiet splash ---
#label memtest
#  menu label Test ^memory
#  kernel /install/mt86plus
#label hd 
#  menu label ^Boot from first hard disk
#  localboot 0x80

default live-install
label live-install
  menu label ^Install Ubuntu
  kernel /casper/vmlinuz.efi
  append  file=/cdrom/ks.preseed auto=true priority=critical debian-installer/locale=en_US keyboard-configuration/layoutcode=us ubiquity/reboot=true languagechooser/language-name=English countrychooser/shortlist=US localechooser/supported-locales=en_US.UTF-8 boot=casper automatic-ubiquity initrd=/casper/initrd.lz quiet splash noprompt noshell ---
```

Create seed file to fully automate things. nano/vi/whatever editor create the file ks.preseed and place it in /opt/ubuntuiso

```
# Partitioning
# Old style using d-i command
#d-i partman-auto/disk string /dev/sda
#d-i partman-auto/method string regular
#d-i partman-lvm/device_remove_lvm boolean true
#d-i partman-md/device_remove_md boolean true
#d-i partman-auto/choose_recipe select atomic

# Newer ubiquity command
ubiquity partman-auto/disk string /dev/sda
ubiquity partman-auto/method string regular
ubiquity partman-lvm/device_remove_lvm boolean true
ubiquity partman-md/device_remove_md boolean true
ubiquity partman-auto/choose_recipe select atomic

# This makes partman automatically partition without confirmation
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

# Locale
d-i debian-installer/locale string en_US
d-i console-setup/ask_detect boolean false
d-i console-setup/layoutcode string us

# Network
d-i netcfg/get_hostname string unassigned-hostname
d-i netcfg/get_domain string unassigned-domain
d-i netcfg/choose_interface select auto

# Clock
d-i clock-setup/utc-auto boolean true
d-i clock-setup/utc boolean true
d-i time/zone string US/Pacific
d-i clock-setup/ntp boolean true

# Packages, Mirrors, Image
d-i mirror/country string US
d-i apt-setup/multiverse boolean true
d-i apt-setup/restricted boolean true
d-i apt-setup/universe boolean true

# Users
d-i passwd/user-fullname string User
d-i passwd/username string user
d-i passwd/user-password-crypted password yourEncryptedPasswd
d-i passwd/user-default-groups string adm audio cdrom dip lpadmin sudo plugdev sambashare video
d-i passwd/root-login boolean true
d-i passwd/root-password-crypted password rootEncryptedPasswd
d-i user-setup/allow-password-weak boolean true

# Grub
d-i grub-installer/grub2_instead_of_grub_legacy boolean true
d-i grub-installer/only_debian boolean true
d-i finish-install/reboot_in_progress note
```

Create the iso with the full automation
```
mkisofs -D -r -V "UNATTENDED_UBUNTU" -cache-inodes -J -l -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o /tmp/ubuntu-desktop-unattended-install.iso /opt/ubuntuiso
```


if you are having trouble installing it fully unattended it may be because when you insert your USB stick, that gets assigned as /dev/sda rather than your actual internal hard drive which should be /dev/sda so you can place this in the seed file to fix it and ignore USB sticks altogether

```
# Due notably to potential USB sticks, the location of the MBR can not be
# determined safely in general, so this needs to be specified:

d-i grub-installer/bootdev  string /dev/sda

# To install to the first device (assuming it is not a USB stick):
#d-i grub-installer/bootdev  string default
```
