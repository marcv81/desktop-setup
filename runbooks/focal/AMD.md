# AMD GPU

Install the drivers.

```
sudo apt update
wget https://repo.radeon.com/amdgpu-install/6.4/ubuntu/jammy/amdgpu-install_6.4.60400-1_all.deb
sudo apt install ./amdgpu-install_6.4.60400-1_all.deb
sudo apt update
```

```
sudo amdgpu-install --list-usecase
sudo amdgpu-install --usecase=graphics
sudo amdgpu-install --usecase=rocm
```

Reduce the power draw to 210W.

```
echo 210000000 | sudo tee /sys/class/hwmon/hwmon2/power1_cap
```
