# Telegraf

## Installation

Install Telegraf.

    wget https://dl.influxdata.com/telegraf/releases/telegraf_1.15.3-1_amd64.deb
    sudo apt install ./telegraf_1.15.3-1_amd64.deb

## Quiet logs

Add the following line to `/etc/pam.d/sudo`, just before `@include common-session-noninteractive`.

    session [success=1 default=ignore] pam_succeed_if.so quiet uid = 0 ruser = telegraf

Add the following line to `/etc/sudoers.d/telegraf`.

    Defaults:telegraf !syslog

## Dependencies

### SMART

Install smartctl, but disable smartd.

    sudo apt install smartmontools
    sudo systemctl stop smartd
    sudo systemctl disable smartd

Allow the `telegraf` user (`_telegraf` on Ubuntu 22.04) to run smartctl in `/etc/sudoers.d/telegraf`.

    telegraf ALL=(root) NOPASSWD: /usr/sbin/smartctl

Install nvme-cli.

    sudo apt install nvme-cli

For NVME devices, the unit of `Data_Units_Read` and `Data_Units_Written` is 512000 bytes.

### Turbostat

Install `turbostat-telegraf` from https://github.com/marcv81/turbostat-telegraf.

### Nuvoton NCT6796D-S

Install lm-sensors.

    sudo apt install lm-sensors

Edit `/etc/default/grub` and add `acpi_enforce_resources=lax` to `GRUB_CMDLINE_LINUX_DEFAULT`. Run `sudo update-grub`.

Create `/etc/modules-load.d/nct6775.conf` with the following contents.

    nct6775

Reboot.

For my motherboard and case, we have the following.

- Temperature sensors
  - `tsi0`,`smbusmaster_0`: CPU die
  - `systin`: Motherboard
  - `auxtin*`, `pch_*`: Garbage values
- Fans
  - `fan4`: `CHA1` (bottom)
  - `fan5`: `CHA2` (back)
  - `fan7`: `CHA3` (top)

## Configuration

Use the `/etc/telegraf/telegraf.conf` from the files directory.

Start and enable the service.

    sudo systemctl start telegraf
    sudo systemctl enable telegraf
