# USB installer

## Ubuntu

Install and use the Startup Disk Creator.

    sudo apt install usb-creator-gtk

## Windows 10

Example: the flash drive is /dev/sdb.

    sudo fdisk /dev/sdb

Type `g` to create a GPT partition table.

Type `n` and accept the defaults to create a new partition.

Type `t` then `11` to change the partition type to "Microsoft basic data".

Format the partition as NTFS. This only works if the UEFI supports NTFS.

    sudo mkfs.ntfs /dev/sdb1 -L Win10 -Q

Mount the ISO and the USB drive. Copy the files from the ISO to the USB drive.

    rsync -avrP win10-iso/ /media/user/Win10/
    sync

## SHA256 verification

Mount the ISO and the USB drive.

Generate the SHA256 sum of all their files.

    pushd win10-iso/; find . -type f | sort | while read F; do sha256sum "$F"; done >~/win10-iso.sha256sum; popd
    pushd /media/user/Win10/; find . -type f | sort | while read F; do sha256sum "$F"; done >~/win10-usb.sha256sum; popd

Compare the SHA256 sums.

    diff ~/win10-iso.sha256sum ~/win10-usb.sha256sum
