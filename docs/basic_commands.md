## Basic Linux information
Since our server is using Debian, it is important to know some basics of Linux usage and how to navigate the operating system. We will start with the basics.

## Partitions
At the time of initial configuration, the pgdata server has one single 2TB hard drive (hdd) in place which contains several partitions. Since modern computers come with a unified extensible firmware interface (UEFI) in place, we have to partition the drive using a GUID partition table (GPT). This is standard for modern computers and not something most users would be concerned with. On the drive, there is a few partitions that are important to know.

#### EFI partitions
This partition houses the bootloader binaries (.efi) files. On a computer that has multiple operating systems, this EFI partition servers all of them. Generally this partition is left untouched as deleting or altering files from it could cause the system to not boot. The EFI partition is very small and should be the first partition on the drive.

#### Swap partition
In Linux history, swap was a critical tool to ensure smooth system operation. The purpose of this partition is to allow the system to flush unused or least recently used processes to be cached into a memory on the hard drive instead of main memory. As the memory fills, instead of forcing processes to close or end, they can simply be cached into 'swap' space and restored when needed. This is sort of like an overflow for Linux memory management. The downsides of using swap is that is significantly slower than the system RAM. As modernization happened, storage media became larger and faster however, so did RAM. It is rare that a system needs swap space anymore but it is installed as a fail-safe. This partition is currently 32GB as is default from Dell.

#### Root partition
The root partition is the operating system partition. It is mounted as the kernel is booted and houses all the executable binaries and user files for the entire system. It fills the rest of the harddrive. It is named root as it the beginning of the Linux file hierarchy. The first level of directories and files.

## File system
Linux can use a multitude of filesystems. If you are used to using Windows operating system, you would probably be familiar with NTFS, FAT16, FAT32, ExFat as well as a few others. If familiar with MacOS, one would expect to see HFS+ While these are generally usable in Linux, it is not advisable to attempt to use it as the operating system's main filesystem.

Linux usually uses a version of EXT filesystem by default however, it can use BTRFS, RieserFS, or a handful of others. There are benefits and pitfalls of each that can be researched quite easily.

#### EFI filesystem
The efi partition is formatted with the fat32 filesystem as it needs to be readable by all operating systems. It is a standard across most machines

#### Swap filesystem
The swap partition does not contain a user accessible filesystem. It is mounted as swap and the system uses it as swap. It is not meant for user intervention as memory address and pages will reside here.

#### Root filesystem
Our root file system is using EXT4 and fills the entire partition. The main benefits of EXT4 the use of extents to map file offsets, journaling, multiblock allocation and timestamps. For more information on any filesystem, there is plenty of documentation available around the internet.

## Grub
Grand Unified Bootloader (GRUB) is the bootloader that the server uses to start the operating system. As all computers have some sort of bootloader, it may be a new concept to see an interface or any evidence that it exist at all. GRUB's binary is installed inside the EFI partition and is executed by the UEFI framework. Once it starts, the user is presented with a menu with a few options. The first is the Debian operating system followed by some advanced options for Debian. These advanced options are used in case a system update made the OS unstable or broken. Here you have a few options of previous kernels and configurations in an attempt to restore to a working environment.

## Kernel
The Linux kernel is the operating system itself. It executes the processes and is the connection between the user software and hardware. It resides inside of the root filesystem and is executed by the bootloader (GRUB). It is executed in unison with initial ram filesystem image (initramfs) that allows a few things to be setup before the mounting and using of the main root filesystem. These are mainly hardware initializations or configurations and filesystem mount points. The kernel mounts the root filesystem as the it boots and prepares the system for use.

## FSTAB
The filesystem table (FSTAB) is a table that specifies which partition is to be mounted where and which options to enable. It is located on the root filesystem in the ***etc*** directory. It is what the kernel uses to mount the partitions during the boot process. The example FSTAB on the server during initial configuration is:
```
UUID=70713bf7-fe9e-429e-bbdc         /           ext4    remount-ro      0       1

UUID=345A-779F                       /boot/efi   vfat    umask=0077      0       1

UUID=5cc95679-a822-491d-b6d0         none        swap    sw              0       0
```
Here we can see the UUID (partition identifier), the mount point `/ is the root` filesystem type, and a few options for the mount point.

Filesystems are mounted as directories instead of drives like you would be familiar with in Windows or MacOS. With Linux, you can mount any filesystem in just about any directory inside of the root partition `/` and they serve as normal directories. This is fundamental in Linux/Unix operating systems. It allows for some neat isolation techniques and security implementation. Any changes to this file can be dangerous as it could cause the system to not boot or boot incorrectly.

## File tree
Linux file tree, all stem from the root `/` directory. This is the file hierarchy standard (FHS) and is the same across all Linux distributions
```
├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── lib64
├── lost+found
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var
```
Each of these directories serve a purpose but we really need to only worry about a few.

#### bin and sbin
This contains essential user binaries needed for operation. These are not the user installed application and programs but instead, tools and utilities for maintaining the system. Things that are needed for a the basic Linux environment

#### boot
Boot contains the files and binaries to boot the system. Since we use grub, you will find configuration files and libraries needed for grub in this directory. If you refer to the ***FSTAB*** above, you will notice that the EFI partition is mounted inside of  `/boot` to ensure that grub can read and execute loading other operating systems like windows or other linux distributions.

#### dev
Dev is the location of devices. This mainly include hardware devices. As Linux shows these as files, they are not files in the traditional sense. An example of a device here is `/dev/sda` This is merely a representation or identifier for the first SATA drive in the machine. In our case, the only SATA drive. This nomenclature passes through to USBs drives `/dev/sdb` and other devices that can be used on the system. These files usually do not need any sort of interaction.

#### etc
Etc is the location for system wide configuration files. It houses the server software and package manager configuration along with many more. These configurations are generally plain text and require elevated permissions in order to manipulate. There is a good chance that you will have to edit files in this directory.

#### home
The home folder stores a `home` directory for each user (except root) that contains all of their user files and configurations. This is the location of your personal documents, downloads, pictures, media, and anything else you would keep on a computers

#### lib
Lib is the location of the essential libraries need for the binaries that reside in `/bin` and `/sbin`. These are similar to .dll files on windows except that they are shared among many programs. User specific libraries for user programs are found in `/usr/lib`

#### lost+found
This is sort of self-explanatory. If for some reason the file system crashes (not necessarily the system itself), a check of the filesystem will be automatically performed. If corrupt or partial files are found, they will be placed here to be recovered manually. Most Linux filesystems have a lost+found directory.

#### media
Media is the standard for removable media. Flash drives, external HDDs, and other removable media is usually mounted here so that the system can use it. Applications that mount filesystems automatically generally use `/run/media` instead of `/media` as they are a run-time state

#### mnt
This is the equivalent to `/media` except for the root user. There are no actual limitations on this so it can be used for a mount point for anything by anyone with the correct permissions

#### opt
Opt stands for optional. It is used for software that does not follow the HFS and needs a place to dump its libraries and binaries. Generally proprietary software that was not originally created for Linux but ported afterwards.

#### proc
Proc is similar to `/dev` as the files in here are just representation of devices or process information related to the Kernel and Processor. This directory is best left alone

#### root
The home directory of the root user. Similar to those in `/home` for standard users, except other users can not read anything inside this directory.

#### run
This is place for random files and libraries to be stored during run time by user applications. This can be used instead of `/tmp` since it does not get deleted regularly or upon boots

#### srv
Srv stands for server and houses all data being served across some form of server application. HTTP, SSH, FTP and more are generally served from this directory however, that is not always true on every system. On this specific system, the webserver (http) is served from `/var/www/`.

#### sys
This is again, similar to that of `/dev` and `/proc`. This contains other hardware represented as files. Not something that needs to be altered but useful for learning statuses and possibly some settings of certain devices.

#### tmp
Tmp is a directory (often mounted as a temp filesystem) that is used to store files that only need to be accessed by applications during the time in which they are being executed. It gets wiped upon boot so nothing in this directory is persistent.

#### usr
This is the place for user specific binaries and libraries. Anything you install on the system will usually place its files, libraries, and documentation here. Similar to a Program Files directory on windows. It also serves as some form of isolation from the rest of the system just in case something goes wrong.

#### var
These are variable data files. Generally log files and other application data files are stored here as `/usr` is often read-only during normal operation.

## General information

### Systemd
Systemd is the initialization system that manages services on the system. It is what starts process during the boot sequence. It mounts the filesystem, starts the network, and presents users with the login interface. It is, at the basic level, the Linux system manager. Users can create service scripts that systemd can execute during certain times or at a certian interval. A good example of a systemd service is the SSH service which starts a shell server so that users can access the machine remotely. This service is named `sshd` or secure shell daemon. By starting this service, the ssh server will start and allow users to connect. There are many services that are started and enabled on the system but most are needed for normal operation. 

### Desktop environment and Display Manager
The desktop environment is a a set of tools and applications that uses the same styling and interfaces. It is the main GUI that a user sees when interacting with the machine. Linux has a multitude of desktop environments (DE) available for any number of user needs. The workstation currently has XFCE installed as its DE and it uses the GTK3 tool kit as its graphical backend. This DE was chosen for its minimalistic approach. It is helpful to have a GUI for less experienced users so it is best to have one that does not require an abundant amount of hardware resources such as RAM.

XFCE comes complete with most needed software such as a file manager (thunar) and text editor (mousepad). It also has plenty of settings to customize it to be efficient for most any task. Debian also installs a few software packages that may be needed for other tasks and those have remained installed in case they are needed later.

The Display Manager (DM) is the software that starts the graphical session. It is a utility that allows the user to login and starts the DE of his/her choice. The DM being used currently is LightDM. It is the default DM that comes packages with XFCE and is highly customizable and easy to use. This runs as a service at the end of the boot process.

### Shell
The default shell for Linux is the borne again shell (bash). It is the primary interface in which commands are passed through to the operating system. It serves as both a command interface and an extremely powerful scripting language. To access the shell, it is necessary to use a Terminal Emulator. XFCE provides a terminal emulator (xfce4-terminal) which allows for the execution of commands and scripts written in the bash language.

ZSH shell has also been installed on the server so that more advanced users have the power and features that they need.

### Locale
Locale in Linux is a set of configurations that tell the system and applications (that are locale aware) what language identifier to use. These are generated upon installation and rarely need altering. The locale generated for this machine is for UTF-8 in the english language.
```
en_US.UTF-8 UTF-8
```

### Timezone
Timezone is similar to locale in that it is configured for the physical location of the machine. It is currently set to America/Chicago as that is a major city in the Central Time Zone (CST)(CDST). This usually does not need modification however, the configuration files are in `/etc`.

### Net Interfaces and Devices
Linux uses interfaces in much the same way that Windows uses them. While our server has several options for networking, we purchased a high powered GBit network interface controller (NIC). This is sufficient for our purposes. Below is the example of the interface and device as Debian has it configured.
```
2: enp179s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether b4:96:91:47:8d:af brd ff:ff:ff:ff:ff:ff
    inet ***.***.**.***/22 brd ***.***.**.*** scope global dynamic enp179s0
       valid_lft 58988sec preferred_lft 58988sec
    inet6 ****::****:****:****:****/64 scope link
       valid_lft forever preferred_lft forever
```
Any asteriks are hiding IP addresses.

### Package Manager
A package manager in Linux is used to install and maintain software. Unlike windows, most software is only installed through this package manager. It is useful to keep the system stable and free of malicious software. There are exceptions to this principle and methods to install software outside of the package manager. This is not advised for this machine since security is an important characteristic that does not need to be compromised.

Debian uses APT package manager. It is versatile and quite powerful. For usage and examples, look in the corresponding sections.

### Users
Linux has several users by default. Some are pseudo users that can not be accessed and are only used for specific software or services.There are a few users that are actual accounts.

The first is the `superuser`, or `root` user, it has total access to everything on the system. It is only used for administration tasks and never as a standard user. There are examples on how to elevate control to the root user in order to maintain the systems

The next user is the students user account. This is shared among all students. The account name is `capstone`. It does not have the ability to elevate its control to administration permissions however, it can be used to switch to the root user. It should be noted that this is indeed a shared account so documents and information for this user is not private to other persons with access to the account.

There are several private accounts for certain students or users that are used for other tasks within the system.
