# Focal

## Installation

Select "Minimal installation". Tick "Install third-party software".

Create 3 partitions on nvme0n1.

- a 500 MB EFI partition
- a 1 GB ext4 partition mounted as /boot
- a LUKS block device containing an ext4 partition mounted as /

Install the bootloader on nvme0n1.

## Configuration

### Grub

Add the line `GRUB_DISABLE_OS_PROBER=true` to /etc/default/grub.

Update Grub.

    sudo update-grub

### Gnome

Install the Gnome Shell theme.

    sudo apt install gnome-session

Log out and choose the "Gnome on Xorg" session in GDM.

Select the GDM Gnome Shell theme.

    sudo update-alternatives --config gdm3-theme.gresource

Reboot to apply the GDM theme.

Install Gnome Tweaks.

    sudo apt install gnome-tweaks

In "Tweaks", "Appearance", select the Yaru icons and cursor.

In "Settings", "Universal Access", increase the cursor size.

Set the terminal color theme to "Tango Dark", and disable "Use transparency from system theme".

Set the terminal "Copy" and "Paste" shortcuts to the dedicated keys.

### Bash

Add the following at the end of ~/.bashrc

    export PS1='`if [ $? = 0 ]; then echo 🟢; else echo 🔴; fi` \t \w '

### Thermald

Disable thermald.

    sudo systemctl stop thermald
    sudo systemctl disable thermald

### Data SSD

Restore `/etc/crypt/sda1.key` from a backup. Ensure only root can read it.

Add the following to `/etc/crypttab`.

    sda1_crypt UUID=4f1f1644-98c9-4ddc-a84a-ba175b8fafda /etc/crypt/sda1.key luks

Add the following to `/etc/fstab`.

    /dev/mapper/sda1_crypt /data ext4 defaults 0 2

Create a symlink in the user's home.

    ln -s /data/user/ ~/data

### WiFi

Edit /etc/modprobe.d/iwlwifi.conf and add the following line.

    options iwlwifi 11n_disable=1

Reload the iwlwifi module.

    sudo rmmod iwlmvm
    sudo rmmod iwlwifi
    sudo modprobe iwlwifi

This reduces the bandwidth to around 10 Mbps but prevents the network from dropping out regularly.

### Apt

#### Suggested packages

By default installing a package does not automatically install the packages it suggests. However packages suggested by other installed packages are not automatically removed. This often prevents packages which are no longer required from being automatically removed.

Create `/etc/apt/apt.conf.d/99_nosuggests` with the following contents.

    APT::AutoRemove::SuggestsImportant "false";

#### History

Edit `/etc/logrotate.d/apt` and set rotate to 60 to keep `history.log` for 5 years.

### Clock

Set the RTC clock to use local time.

    timedatectl set-local-rtc 1

### SSH keys

Restore the SSH keys from a backup.

## Software

### Sysadmin

    sudo apt install apt-file curl git net-tools traceroute vim
    sudo apt remove nano

### Apps

Install the following packages.

    sudo apt install gparted keepassxc libreoffice meld transmission vlc

Download and install the packages from the following sources.

- Dropbox: https://www.dropbox.com/install
- Google Earth Pro: https://www.google.com/earth/versions/#download-pro
- Steam: https://store.steampowered.com/about/
- Visual Studio Code: https://code.visualstudio.com/

Update the Google GPG key.

    wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

## Peripherals

### Oculus Quest

Create `/etc/udev/rules.d/50-oculus-quest.rules`.

    SUBSYSTEM=="usb", ATTR{idVendor}=="2833", ATTR{idProduct}=="0186", MODE="0660", GROUP="plugdev"

Restart udev.

    sudo udevadm control --reload-rules

### Huion Kamvas pen display

Install `digimend-dkms_10_all.deb` from https://github.com/DIGImend/digimend-kernel-drivers/releases/tag/v10.

Reboot.

Find the monitor.

    $ xrandr --listmonitors
    Monitors: 2
     0: +*HDMI-0 2560/798x1080/334+0+0  HDMI-0
     1: +HDMI-1-2 1920/256x1080/144+334+1080  HDMI-1-2

Create /usr/local/bin/huion_setup.sh

    #!/usr/bin/env bash
    set -euo pipefail
    xsetwacom --list | grep 'type: STYLUS' | sed 's/.*id: \([[:digit:]]\+\).*/\1/' | xargs -I {} xinput map-to-output {} HDMI-1-2

Run `huion_setup.sh` after connecting the device to map the pen to the display.

### Canon LiDE 300 scanner

Download the official drivers. Extract and install the package.

    tar xvf scangearmp2-3.70-1-deb.tar.gz 
    sudo apt install ./scangearmp2-3.70-1-deb/packages/scangearmp2_3.70-1_amd64.deb

Install simple-scan and remove ippusbxd.

    sudo apt install simple-scanner
    sudo apt purge ippusbxd

Reboot.

## Known issues

### NVIDIA graphics

Run the following to prevent tearing. It may degrade performance.

    nvidia-settings --assign CurrentMetaMode="nvidia-auto-select +0+0 { ForceFullCompositionPipeline = On }"
