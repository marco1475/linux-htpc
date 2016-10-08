This Wiki contains all information (instructions, command snippets, and files) necessary to set up a Linux HTPC.

0. [Prerequisites](linux-htpc/prerequisites.md)

9. [Common Tasks](linux-htpc/common_tasks.md)

#Installing the Operating System

##Getting to the Virtual Console

0. Ensure that USB devices have a higher boot priority than the internal hard drive.
    - When you see the ASRock boot screen, hit `F2` to enter the *UEFI Setup*.
    - On the *Boot* tab ensure the following `Boot Option Priorities`:
        1. `USB: <name of flash drive>`
        2. `UEFI: <name of flash drive>`
        3. `Linux Boot Manager` (if available)
        4. `AHCI P0: <name of internal disk>`
    - Note that putting `UEFI:` before `USB:` will spare you having to select the UEFI boot method from the *Easy2Boot CSM Menu*, but it will also prevent you from swapping bootable images (i.e. going back to the *Easy2Boot Main Menu*).
1. Plug in the bootable flash drive and restart the machine.
2. In the *Easy2Boot Main Menu* select the Arch Linux image, hit `Enter` and confirm overwriting the MBR.
3. In the *Easy2Boot CSM Menu* select entry `4 Clover 64-bit UEFI Boot Menu (USB 2.0 ports only)` and hit `Enter`.
    - If your bootable flash drive is plugged into a non-USB 2.0 port you will have to:
        1. Reboot the machine.
        2. When you see the ASRock screen, hit `F11` to enter the *Boot Menu*.
        3. Select `UEFI: <name of flash drive>` as the boot option.
    - Trying to boot the Arch UEFI boot image in MBR mode will result in the following Syslinux error message:  
        
        > Unknown keyword in configuration file: PATH  
        > boot/syslinux/whichsys.c32: not a COM32R image
        
4. In the *Clover 64-bit UEFI Boot Menu* select `Boot UEFI external from EASY2BOOT`.
5. In the *Arch Linux Boot Menu* select `Arch Linux archiso x86_64 UEFI USB`.
    - If you have a 4K screen the default Linux kernel mode setting might not be able to set the correct resolution, resulting in a blank monitor screen ("There is no signal coming from your computer.").
    - To override Linux kernel mode setting:
        1. With `Arch Linux archiso x86_64 UEFI USB` selected hit `e`.
        2. Add `nomodesetting` to the end of the command-line setting that appeared on the screen and hit `Enter`.
6. You are now logged in as `root` in the virtual console of the Arch Linux installation.

##Installation Preparation

1. Verify the boot mode:
    - If the following command lists entries in the directory you have correctly booted using UEFI.  
            
            ls /sys/firmware/efi/efivars
            
2. Verify the internet connection:
    - Ping `archlinux.org` where you'll be downloading packages from; if you get no timeouts you are connected to the internet.
            
            ping -c 3 archlinux.org
            
3. Update the system clock:
    - Synchronize the CPU's clock over the internet via NTP.
            
            timedatectl set-ntp true
            
    - After a while ensure the NTP synchronization was successful by running `timedatectl status` and look for `NTP synchronized: yes` in the output.

##Partitioning the System Disk

**Note that partitions should start on a clean 1 MiB boundary so that block size of the file system aligns with the block size of the SSD.**

1. Start the `gdisk` partitioning utility:
        
        gdisk /dev/sda
        
    - If there are any partitions present remove them by creating a new, empty GUID partition table (`o`).
2. Create a new EFI system partition (EPS).
    1. Press `n` and hit `Enter` to create a new partition.
    2. Press `Enter` to accept the default partition number (in this case 1).
    3. Press `Enter` again to accept the default first sector (in this case 2048).
    4. Enter `+1G` to set the last sector to be 1 GiB past the first sector.
    5. Enter `ef00` to set the partition type to `EFI System`.
3. Create the root partition.
    1. Press `n` and hit `Enter` to create a new partition.
    2. Press `Enter` to accept the default partition number (in this case 2).
    3. Press `Enter` again to accept the default first sector (in this case 2099200).
    4. Enter `+60G` to set the last sector to be 60 GiB past the first sector.
    5. Press `Enter` to accept the default file system to be `Linux filesystem`.
4. Press `p` to print out the partition table:
        
        Number Start (sector) End (sector)       Size Code Name
             1           2048      2099199 1024.0 MiB EF00 EFI System
             2        2099200    127928319   60.0 GiB 8300 Linux filesystem
        
5. Press `w` to write the partition table to disk.

##Formatting the System Disk

1. The EFI System Partition must be formatted as `FAT32`:
        
        mkfs.fat -F32 /dev/sda1
        
2. The root partition will be formatted as `ext4`:
        
        mkfs.ext4 /dev/sda2
        
3. Write down the UUIDs of the newly-formatted partitions:
        
        lsblk -no name,uuid
        
##Encrypting the System Disk

1. Encrypt the `root` partition using `dm-crypt`:
        
        cryptsetup -s 512 luksFormat /dev/sda2
        
    1. Type `YES` when prompted for confirmation that all data on `/dev/sda2` will be overwritten.
    2. Enter the passphrase that will unlock the partition (hint: full+server+partition) and verify it.
2. Open the encrypted partition:
        
        cryptsetup open --type luks /dev/sda2 cryptroot
        
    - The encrypted partition can now be accessed through`/dev/mapper/cryptroot`.
3. Format the encrypted partition:
        
        mkfs.ext4 /dev/mapper/cryptroot
        
4. Write down the UUIDs of the newly-encrypted and formatted partitions.
        
        lsblk -no name,uuid
        
    - Note that the encrypted partition has a different UUID locked vs. unlocked.
5. Close the partition:
        
        cryptsetup close cryptroot
        
