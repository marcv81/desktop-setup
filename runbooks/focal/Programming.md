# Programming

## Python

    sudo apt install black python3-dev python3-virtualenv
    sudo ln -s /usr/bin/python3 /usr/bin/python

## C/C++

    sudo apt install build-essential clang-format cmake autoconf

## Go

Download binaries from https://golang.org/dl/.

    sudo tar -C /usr/local -xzf go1.14.6.linux-amd64.tar.gz
    echo 'export "PATH=/usr/local/go/bin:$PATH"' >>.profile

## Rust

    curl https://sh.rustup.rs -sSf | sh

Restart the session to update the PATH.

## Docker

Create `/etc/apt/sources.list.d/docker.list` with the following contents.

    deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable
    # deb-src [arch=amd64] https://download.docker.com/linux/ubuntu focal stable

Install the Docker repo key.

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

Install Docker.

    sudo apt update
    sudo apt install docker-ce

Add the user to the docker group.

    sudo usermod -a -G docker user

Restart the session to apply the group change.

## AVR flashing

    sudo apt install dfu-programmer avrdude

Create `/etc/udev/rules.d/51-dfu.rules`.

    SUBSYSTEMS=="usb", ATTRS{idVendor}=="03eb", ATTRS{idProduct}=="2ff4", MODE:="0666"

Restart udev.

    sudo udevadm control --reload-rules
