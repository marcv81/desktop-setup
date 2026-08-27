# Drives

## Information

Block device summary.

    lsblk /dev/nvme0n1

List of partitions.

    sudo parted /dev/nvme0n1 print

Encrypted block device information.

    sudo cryptsetup luksDump /dev/nvme0n1p3

## New internal drive setup

### Initial setup

Create the partition table.

    sudo parted /dev/sda mklabel gpt

Create a partition taking all the space.

    sudo parted /dev/sda mkpart primary 0% 100%

Encrypt the partition.

    sudo cryptsetup luksFormat /dev/sda1 --verify-passphrase

Map the partition to a virtual block device.

    sudo cryptsetup luksOpen /dev/sda1 sda1_crypt

Format the block device.

    sudo mkfs.ext4 /dev/mapper/sda1_crypt

Mount the logical volume.

    sudo mkdir -p /data
    sudo mount /dev/mapper/sda1_crypt /data

Create the user directory.

    sudo mkdir /data/user
    sudo chown user: /data/user/

### Automount

Generate a random decryption key.

    sudo mkdir /etc/crypt
    sudo chmod 700 /etc/crypt
    sudo dd if=/dev/urandom of=/etc/crypt/sda1.key count=1
    sudo chmod 400 /etc/crypt/sda1.key

Add the decryption key to the encrypted partition.

    sudo cryptsetup luksAddKey /dev/sda1 /etc/crypt/sda1.key --key-slot 1

Find the partition UUID.

    sudo cryptsetup luksUUID /dev/sda1

Update `/etc/crypttab`.

    sda1_crypt UUID=4f1f1644-98c9-4ddc-a84a-ba175b8fafda /etc/crypt/sda1.key luks

Update `/etc/fstab`.

    /dev/mapper/sda1_crypt /data ext4 defaults 0 2

## New external drive setup

Create the partition table.

    sudo parted /dev/sdb mklabel gpt

Create a partition taking all the space.

    sudo parted /dev/sdb mkpart primary 0% 100%

Encrypt the partition.

    sudo cryptsetup luksFormat /dev/sdb1 --verify-passphrase

Map the partition to a virtual block device.

    sudo cryptsetup luksOpen /dev/sdb1 sdb1_crypt

Format the block device.

    sudo mkfs.ext4 /dev/mapper/sdb1_crypt
