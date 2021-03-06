# GPU passthrough

## Host

### KVM

Install the core components: KVM, libvirt, and the VM Manager.

    sudo apt install qemu-kvm libvirt-daemon virt-manager

Add the current user to the relevant group and restart the session.

    sudo adduser `id -un` kvm

### IOMMU/VFIO

Add `intel_iommu=on` to GRUB_CMDLINE_LINUX_DEFAULT in /etc/default/grub.

Update Grub.

    sudo update-grub

Restart the computer.

Find the GPU identifiers. Must we pass all the devices in a group through (except the PCIe controller).

    $ for GROUP in $(ls /sys/kernel/iommu_groups); do echo "Group $GROUP"; for DEVICE in $(ls /sys/kernel/iommu_groups/$GROUP/devices); do lspci -nns $DEVICE; done; echo; done
    ...
    Group 1
    00:01.0 PCI bridge [0604]: Intel Corporation Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor PCIe Controller (x16) [8086:1901] (rev 07)
    01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP106 [GeForce GTX 1060 6GB] [10de:1c03] (rev a1)
    01:00.1 Audio device [0403]: NVIDIA Corporation GP106 High Definition Audio Controller [10de:10f1] (rev a1)
    ...

Add `vfio-pci.ids=10de:1c03,10de:10f1` to GRUB_CMDLINE_LINUX_DEFAULT in /etc/default/grub.

Update Grub.

    sudo update-grub

Disable the NVIDIA udev rules.

    sudo mv /lib/udev/rules.d/71-nvidia.rules /lib/udev/rules.d/71-nvidia.rules.disabled

Disable Wayland. Edit /etc/gdm3/custom.conf and uncomment the following line.

    WaylandEnable=false

Connect a monitor to the iGPU and restart the computer.

Verify the GPU uses vfio-pci.

    lspci -k

### Evdev passthrough

Add the user libvirt to the input group.

    sudo usermod -a -G input libvirt-qemu

Install and configure https://github.com/marcv81/hotplug-evdev-passthrough/.

Edit /etc/libvirt/qemu.conf.

Set `security_driver = "none"`.

Create cgroup_device_acl with the recommended defaults. Add the following two devices.

    /dev/input/by-id/virtual-keyboard
    /dev/input/by-id/virtual-mouse

Restart libvirtd.

    sudo systemctl restart libvirtd

### Scream client

Clone the repository and checkout the 3.6 release.

    git clone https://github.com/duncanthrax/scream.git
    git checkout 3.6

Install the required packages.

    sudo apt install build-essential cmake libpulse-dev

Build and install the client.

    mkdir scream/Receivers/unix/build
    cd scream/Receivers/unix/build
    cmake ..
    make
    sudo mkdir /usr/local/bin
    sudo cp scream /usr/local/bin/

### GPU usage on host

Disable the VFIO passthrough and load the Nvidia drivers.

    sudo sh -c "echo 0000:01:00.0 >/sys/bus/pci/drivers/vfio-pci/unbind"
    sudo sh -c "echo 0000:01:00.1 >/sys/bus/pci/drivers/vfio-pci/unbind"
    for MODULE in nvidia nvidia-modeset nvidia-drm nvidia-uvm; do sudo modprobe $MODULE; done
    sudo sh -c "echo 0000:01:00.1 >/sys/bus/pci/drivers/snd_hda_intel/bind"

Restart Xorg so use the GPU for display. This is not required to use CUDA.

    sudo systemctl restart gdm

## Guest

### Downloads

- Windows 10 ISO
- VirtIO drivers: https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso

### Base

Install OVMF.

    sudo apt install ovmf

Create a new VM.

- ISO image: Windows 10
- Cores: 8
- RAM (Mb): 16384
- Disk (Gb): 200

Tick customize configuration before install. In the VM configuration select the following.

- Overview
  - Select UEFI firmware (OVMF_CODE.fd)
- Disk 1
  - Select VirtIO
- Add CDROM 2
  - Connect to VirtIO driver ISO

Click begin the installation but shut down the VM.

Run `virsh edit win10` to edit the VM configuration.

Define the topology in the `cpu` element. This is required for performance.

    <topology sockets='1' cores='4' threads='2'/>

Replace `type='pflash'` by `type='rom'` in the `loader` element. This is required to take snapshots of the VM.

### GPU

In the VM configuration remove the Spice display, the QXL video, and the tablet.

Add the following two host devices.

- GPU: 0000:01:00:0
- GPU audio controller: 0000:01:00:1

Run `virsh edit win10` to edit the VM configuration.

Replace `managed='yes'` by `managed='no'` in both `hostdev` elements. This prevents libvirt from binding/unbinding the vfio-pci module automatically.

Add `vendor_id` and `hidden` to the `features` element as follows. This fixes the error code 43 when installing the GPU drivers.

    <features>
      <hyperv>
        ...
        <vendor_id state='on' value='abcdef'/>
      </hyperv>
      <kvm>
        <hidden state='on'/>
      </kvm>
    </features>

### Evdev passthrough

Run `virsh edit win10` to edit the VM configuration.

Add the following in the `devices` element.

    <input type='mouse' bus='virtio'/>
    <input type='keyboard' bus='virtio'/>

Modify the `domain` element as follows.

    <domain type='kvm' xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>

Add the following at the end of the `domain` element.

    <qemu:commandline>
      <qemu:arg value='-object'/>
      <qemu:arg value='input-linux,id=mouse1,evdev=/dev/input/by-id/virtual-mouse'/>
      <qemu:arg value='-object'/>
      <qemu:arg value='input-linux,id=kbd1,evdev=/dev/input/by-id/virtual-keyboard,grab_all=on,repeat=on'/>
    </qemu:commandline>

### Windows 10

In the VM configuration remove the NIC to skip the Microsoft account registration.

Start the VM and *quickly* switch to the KVM port it is connected to.

In the UEFI shell type `exit` and use the menu to boot from the first CD ROM.

Follow the instructions. Select the custom installation type, then load the VirtIO drivers from the second CD ROM. Shut down the VM when the installation completes.

### Drivers

In the VM configuration add a NAT VirtIO NIC. Start the VM.

Open the device manager.

Use the VirtIO dirvers ISO to install drivers for the following devices.

- Ethernet Controller (RedHat Virtio Ethernet Adapter)
- PCI Device (VirtIO Balloon Driver)
- PCI Keyboard Controller (VirtIO Input Driver)
- PCI Mouse Controller (VirtIO Input Driver)
- PCI Simple Communications Controller (VirtIO Serial Driver)

Use the VirtIO dirvers ISO to update the drivers for the following device.

- System devices: SM Bus Controller (RedHat Q35 SM Bus Controller)

Search automatically for an updated driver for the following device.

- Microsoft Basic Display Adapter (GPU)

Shut down the VM.

### Scream server

Run `virsh edit win10` to edit the VM configuration.

Add the following to the `devices` element.

    <shmem name='scream-ivshmem'>
      <model type='ivshmem-plain'/>
      <size unit='M'>2</size>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x11' function='0x0'/>
    </shmem>

Remove any sound device.

Start the VM.

Download the 0.1-161 VirtIO drivers from https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/upstream-virtio/. In the device manager, under system devices, find PCI standard RAM controller and update its drivers.

Download the 3.6 release from https://github.com/duncanthrax/scream/releases. Extract and run the install batch script as administrator.

Run cmd as administrator and execute `REG ADD HKLM\SYSTEM\CurrentControlSet\Services\Scream\Options /v UseIVSHMEM /t REG_DWORD /d 2`.

Restart the VM.

Start the client on the host.

    scream -m /dev/shm/scream-ivshmem
