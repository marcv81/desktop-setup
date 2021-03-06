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

## Configuration

Use the `/etc/telegraf/telegraf.conf` from the files directory.

Start and enable the service.

    sudo systemctl start telegraf
    sudo systemctl enable telegraf
