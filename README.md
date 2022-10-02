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
 - Debian 11 Bullseye and Linux Kernel 5.15.59
 - Full Disk Encryption
 - UEFI support

It will create the below shown folder structure in config folder,

  <img width="680" alt="image" src="https://user-images.githubusercontent.com/102230689/193446874-49fd5c05-6966-43a1-891f-71b0f461e675.png">

**Build - Live and Installer image** 
Run live build
```ruby
      $ lb build
```
This command runs as four stages,
- Bootstrap stage
- Chroot stage
- Binary stage
- Source stage
