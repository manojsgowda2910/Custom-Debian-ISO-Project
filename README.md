**Custom-Debian-ISO-Project**

Build an Installable Debian 11 Bullseye Image with all possible customization.  
Objective of this project is to build the custom debian ISO image with all cutomization feature made available.

**Debian Live Project:**

It is a framework used to build Debian Live systems and Live Images by themselves.

**Live Build:**

Command line tool that contains the components to build a live system, I.e. It is a set of scripts completely automated to build debian live images.  
Using this tool, all aspects of customized building of Live system is possible.

Live build includes three commands to build image: lb clean, lb config and lb build

- **lb clean:** Responsible for cleaning up after system is built. It removes the build directories and remove other files including stage files.
- **lb config** : populates the configuration directory for live-build. This directory is named 'config' and is created in the current directory where lb config was executed.
- **lb build:** Command reads the configuration from the config/ directory. It then runs the lower level commands needed to build the Live system.

**Installation and Project Setup:**

- Update and Install live build

```ruby
      $ apt-get update
      $ apt-get install live-build
```
      
- Prerequisites package to be installed,

```ruby
      $ apt-get install squashfs-tools live-boot live-config live-boot-initramfs-tools live-config-sysvinit libburn4 libjte1 libisofs6 libisoburn1 xorriso isolinux
```


- Create a folder to debian live project to keep all files in one place and enter to source directory with all root previlages.

 ```ruby
      $ mkdir custom-debian
      $ cd custom-debian
      $ lb clean
  ```
  
**Build Configuration:**  

  Issuing "lb config" without any arguments creates the following subdirectories,  
  - auto
  - config
  - local

 ```ruby
      $ lb config
  ```   
  ```ruby
      [2022-04-29 14:49:05] lb config
      P: Creating config tree for a debian/bullseye/amd64 system
      P: Symlinking hooks...
  ```
  
**Config Automation script**

  It is possible to specify many options based on the requirement, either as some arguments to lb config or by creating 'config' automation script in auto subdirectory.

1. **New config script:**

    Scripts should be newly created as 'config'.

 ```ruby
      $ nano config
 ```

With default template as,
 ```ruby
      #!/bin/sh
      set –e
      lb config noauto \
      "${@}"
 ```
       
Config file needs to become executable.
```ruby
      $ chmod 700 config
      $ lb config
```
lb command options can be referred in 'man lb config' or in live-debian website.

2. **Example Auto Scripts**

  live-build comes with example auto shell scripts to copy and edit.
```ruby
      $ cp /usr/share/doc/live-build/examples/auto/\* auto/
```
Edit auto/config,. For instance:
```ruby
      $ nano auto/config
```
Below example is for my Custom OS requirment, Please refer Debian Live Manual to customize according to user requirment,
```ruby
      #!/bin/sh
      set -e
      
      lb config noauto \
      --mode debian \
      --system live \
      --interactive shell \
      --bootappend-live "boot=live components persistence persistence-encryption=luks console=tty1 console=ttyS0,115200" \
      --bootappend-install "boot=components console=tty1 console=ttyS0,115200" \
      --bootloaders grub-efi \
      --binary-image iso-hybrid \
      --debian-installer live \
      --debian-installer-distribution bullseye \
      --distribution bullseye \
      --debian-installer-gui true \
      --architectures amd64 \
      --mirror-bootstrap http://ftp.tw.debian.org/debian/ \
      --mirror-chroot http://ftp.tw.debian.org/debian/ \
      --mirror-binary http://ftp.tw.debian.org/debian/ \
      --mirror-binary-security http://security.debian.org/ \
      --mirror-chroot-security http://security.debian.org/ \
      --archive-areas 'main contrib non-free' \
      --backports true \
      --security true \
      --updates true \
      --source false \
      --linux-packages linux-image-5.15.59 \
      --linux-flavours amd64 \
      --apt-recommends false \
      --binary-filesystem ext4 \
      --firmware-binary true \
      --firmware-chroot true \
      --initramfs live-boot \
      --iso-publisher manoj \
      --iso-volume manoj-0.0.1 \
      "${@}"

```
Above config file is implemnted to create OS as in below major configuration details,  
 - Debian 11 Bullseye
 - Custom Linux Kernel 5.15.59
 - Full Disk Encryption
 - UEFI support (grub-efi)
 - amd64 (64 bit architecture)
 - Interactive shell (To install packages during build time)
 - Serial Console support
 - Custom systemd Service
 - Package Installation 
 - Auto installation
 - Files on the ISO filesystem

It will create the below shown folder structure in config folder,

  <img width="680" alt="image" src="https://user-images.githubusercontent.com/102230689/193446874-49fd5c05-6966-43a1-891f-71b0f461e675.png">

**Build - Live and Installer image**  

Run live build
```ruby
      $ lb build
```
The build process is divided into stages, with various customizations applied in sequence in each.  
      - Bootstrap stage  
      - Chroot stage  
      - Binary stage  
      - Source stage 

- **Bootstrap Stage:** This is the initial phase of populating the chroot directory with packages to make a barebones Debian system.  
- **Chroot stage:** In this stage preseeds are applied before any packages are installed, packages are installed before any locally included files are copied, and hooks are run later, after all of the materials are in place. Most customization of content occurs in this stage.  
- **Binary stage:** Builds a bootable image, using the contents of the chroot directory to construct the root filesystem for the Live system, and including the installer and any other additional material on the target medium outside of the Live system's filesystem.  
- **Source stage:** Source puts it into a bootable ISO image. 

**Interactive Shell:**
Install required packages to the ISO filesystem during image bulding time. Once the Interactive shell appears pass the installation command,  
For example,
```ruby
$ apt-get install systemd grub-efi extlinux syslinux mtools console-setup python3 python3-pip network-manager ethtool speedtest-cli cryptsetup-initramfs fdisk initramfs-tools rapidjson-dev ntp openssh-server iptables squashFS luks tpm2-brmd tmp2-tools netfilter-persistent auditd ntp watchdog AppArmor openssh-server sudo Python 3.10 rsync
$ pip3 install pyusb pyserial pyftdi
$ exit
```
With this, you will exit the chroot environment and lb build will finish it’s job by downloading whatever extra it needs and eventually squashing your build into a file system, and then containing it inside an iso file.
ISO file will be created in the Live-build source root directory.

**Test the Image**  
Burn the image to USB using either dd command or Balena Etcher and Installer on any Machine or use Virtual machine to test.
Once the custom build image works, then is Good!!!! we have built our own custom OS that can be installed offline.

**CONSIDERATIONS**  

**Legacy and UEFI Boot:**  
      GRUB supports booting x86 systems via either the traditional BIOS method or more modern UEFI.  
      There are two packages, grub-pc and grub-efi.  
       - If we want to prepare an image with efi support, grub-efi package is to be installed and get rid of the grub-pc package.  
       - If we want a classic boot image(BIOS boot), Install grub-pc package and get rid of the grub-efi package. Debian 11 will not let you install both.  
      If we don’t include a boot loader in the packages now, you’ll see that the debian installer from your resulting build not be able to install a bootloader.           This is why we include this package here now.
      
**Kernel Upgrade:**  
You can build and include your own custom kernels. The live-build system does not support kernels not built as .deb packages. The proper and recommended way to deploy your own kernel packages is to follow the instructions in the kernel-handbook. Remember to modify the ABI and flavour suffixes appropriately.
Kernel package naming convention to be as required by Live-Build standard.  
for example, linux-image-{ARCHITECTURE}  
      - Build the kernel by giving EXTRAVERSION as amd64 in Makefile.  
      - Place the Kernel package in includes.installer folder, so that this package will be installed and available in the ISO filesystem.  
      - Update the grub during build time using Hook script as below  
```ruby      
      Add hook script code here
```
**Sources.list update**  
By default generated image will have sources.list updated with debian security repository.  
- Shell script to write sources.list content
- systemd service to run the shell script from boot time. During First boot after OS installation service should be killed by removing the service file using above mentioned shell script.
- Create a custom debian package to run service file.

**Auto Installation**  
Full OS installation can be automated with this concept by using preseed.cfg file.  
Preseeding provides a way to set answers to questions asked during the installation process, without having to manually enter the answers while the installation is running. This makes it possible to fully automate most types of installation and even offers some features not available during normal installations."
      This can be achieved by placing presedd.cfg file in includes.installer folder.
Example preseed configuration file is provided for reference.  
```ruby      
     cp -r path/to/preseed.cfg config/includes.installer
```
