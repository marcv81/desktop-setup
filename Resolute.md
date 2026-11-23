# Resolute

## Installation

Do not install from a thumb drive, it's too slow. I used a USB SSD.

Use the "default selection" of apps. Select "third-party software" and "additional media formats".

Let the installer partition the disk automatically. Select disk encryption with a password. It creates the following partitions.
- a 1 GB EFI partition.
- a 2 GB ext4 `/boot` partition.
- a LUKS block device containing an ext4 `/` partition.

## Core tools

Install the following core tools. This is to bootstrap; we use Nix and Home Manager for the remaining tools.
 
    sudo apt install curl git vim
    sudo apt remove --purge nano

## Grub

Change the following lines in `/etc/default/grub`.

    GRUB_DISABLE_OS_PROBER=true
    GRUB_GFXMODE=2560x1080

There is a race condition between Plymouth and Gnome. Plymouth retains ownership of the display until after Gnome starts. Gnome stalls, GDM sees an unclaimed display, and the GDM login displays instead of the Gnome desktop. Commenting out the `GRUB_CMDLINE_LINUX_DEFAULT` line (or just removing "splash") fixes the problem as it disables Plymouth. This is a bug, I expect the workaround not to be necessary at some point in the future.

Update Grub.

    sudo update-grub

## Gnome

Install the default Gnome session.

    sudo apt install gnome-session

Log out. Select the "Gnome" session when logging back in.

Install Gnome Tweaks.

    sudo apt install gnome-tweaks

Configure Gnome.
- In `Tweaks > Appearance` select the Yaru icons and cursor.
- In `Tweaks > Mouse & Touchpad` turn on "Middle Click Paste".
- In `Settings > Accessibility > Seeing > Cursor Size` select a larger cursor.
- In `Settings > Appearance` select a background image and use the dark style.
- In `Settings > Multitasking` enable "Window Resize" to snap/resize windows by dragging them to the edges of the screen.
- In `Settings > Power > Power Saving` turn off "Automatic Screen Blank".

In the Gnome terminal, select the "Gnome" color theme.

Install the Wayland clipboard manager.

    sudo apt install wl-clipboard

## Data SSD

Restore `/etc/crypt/sda1.key` from a backup. Ensure only root can read it.

Add the following to `/etc/crypttab`.

    sda1_crypt UUID=4f1f1644-98c9-4ddc-a84a-ba175b8fafda /etc/crypt/sda1.key luks

Apply the changes.

    sudo systemctl restart systemd-cryptsetup@sda1_crypt.service

Add the following to `/etc/fstab`.

    /dev/mapper/sda1_crypt /data ext4 defaults 0 2

Create the target directory and apply the changes.

    sudo mkdir /data
    sudo mount -a

Create a symlink in the user's home.

    ln -s /data/user/ ~/data

## AMD R9700

To limit the GPU power draw, create `/etc/udev/rules.d/30-amdgpu-power.rules` with the following contents.

    ACTION=="add|change", SUBSYSTEM=="hwmon", ATTRS{vendor}=="0x1002", ATTRS{device}=="0x7551", ATTR{power1_cap}="210000000"

Add `amdgpu.runpm=0` to `GRUB_CMDLINE_LINUX_DEFAULT` in `/etc/default/grub` to prevent the GPU from going to sleep. Run `sudo update-grub` and restart.

## MX Master 4

Install Solaar.

    sudo apt update
    sudo apt install solaar

Restart the computer.

## SSH key

Restore the SSH key from a backup.

## APT

Edit `/etc/apt/sources.list.d/ubuntu.sources`. Replace `sg.archive.ubuntu.com` with `archive.ubuntu.com`. The local mirror is unreliable.

By default APT does not install "suggested" packages. But it does not remove a package if another still "suggests" it. This makes `apt install` and `apt autoremove` asymmetric.

Create `/etc/apt/apt.conf.d/99_nosuggests` with the following contents.

    APT::AutoRemove::SuggestsImportant "false";

Edit `/etc/logrotate.d/apt` and set rotate to 60 to keep `history.log` for 5 years.

## Docker

Install the Docker repo key.

    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

Create `/etc/apt/sources.list.d/docker.sources` with the following contents.

    Types: deb
    URIs: https://download.docker.com/linux/ubuntu
    Suites: resolute
    Components: stable
    Architectures: amd64
    Signed-By: /etc/apt/keyrings/docker.asc

Install Docker.

    sudo apt update
    sudo apt install docker-ce

Add the user to the docker group.

    sudo usermod -a -G docker user

Reboot to apply the group change.

## Apps

### Dropbox

Download and install Dropbox from https://www.dropbox.com/install-linux. The app did not show an icon in the notifications area until after a reboot.

### KeepassXC

Install KeepassXC.

    sudo apt install keepassxc

Edit `/usr/share/applications/org.keepassxc.KeePassXC.desktop` and set `StartupNotify=false`. Otherwise Gnome waits for a notification that never comes.

In `Settings > Apps > KeepassXC` disable autostart.

### Google Earth

Install Flatpak, and add the Flathub remote.

    sudo apt update
    sudo apt install flatpak
    flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

Restart the computer, then install Google Earth.

    flatpak install flathub com.google.EarthPro

### LibreOffice

Install LibreOffice.

    sudo apt update
    sudo apt install libreoffice

### GIMP

Install GIMP.

    sudo apt update
    sudo apt install gimp

## Thermald

Disable thermald.

    sudo systemctl stop thermald
    sudo systemctl disable thermald
