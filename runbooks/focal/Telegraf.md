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

Allow the telegraf user to run smartctl in `/etc/sudoers.d/telegraf`.

    telegraf ALL=(root) NOPASSWD: /usr/sbin/smartctl

Install nvme-cli.

    sudo apt install nvme-cli

For NVME devices, the unit of `Data_Units_Read` and `Data_Units_Written` is 512000 bytes.

### CPU stats

Install `cpu_stats.py` from https://github.com/marcv81/cpu-stats to `/usr/local/bin`.

Allow the telegraf user to run `cpu_stats.py` in `/etc/sudoers.d/telegraf`.

    telegraf ALL=(root) NOPASSWD: /usr/local/bin/cpu_stats.py

### Nuvoton NCT6796D

Install lm-sensors.

    sudo apt install lm-sensors

Edit `/etc/default/grub` and add `acpi_enforce_resources=lax` to `GRUB_CMDLINE_LINUX_DEFAULT`. Run `sudo update-grub`.

Create `/etc/modules-load.d/nct6775.conf` with the following contents.

    nct6775

Reboot.

The following is an educated guess of the meaning of the different voltages on a H370-I motherboard.

| Voltage | Name  | Description      |
|---------|-------|------------------|
| 0       | Vcore | Vcore            |
| 1       | in1   | 5 * in1 = +5V    |
| 2       | AVCC  | AVCC/3VSB        |
| 3       | +3.3V | +3.3V            |
| 4       | in4   | 12 * in4 = +12V  |
| 5       | in5   | ? (~870mV)       |
| 6       | in6   | ? (~870mV)       |
| 7       | 3VSB  | AVCC/3VSB        |
| 8       | Vbat  | CMOS battery     |
| 9       | in9   | VTT              |
| 10      | in10  | ? (~2V)          |
| 11      | in11  | ? (~870mV)       |
| 12      | in12  | 12 * in12 = +12V |
| 13      | in13  | 5 * in13 = +5V   |
| 14      | in14  | ? (~370mV)       |

## Configuration

Use the `/etc/telegraf/telegraf.conf` from the files directory.

Start and enable the service.

    sudo systemctl start telegraf
    sudo systemctl enable telegraf
