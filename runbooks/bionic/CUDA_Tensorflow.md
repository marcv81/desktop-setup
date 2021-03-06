# CUDA / Tensorflow

The following are tested configurations, others might work.

## NVIDIA driver 418

    sudo add-apt-repository ppa:graphics-drivers
    sudo apt-get update
    sudo apt-get install nvidia-driver-418
    sudo apt-get install libnvidia-gl-418:i386

## TensorFlow 1.13.1

### CUDA 10.0

Download CUDA from the NVIDIA website. Do not download the .deb package; it breaks the driver. Get the .run package. Do *not* install or configure anything else than CUDA. Install in a custom directory, for instance /home/user/Workspace/cuda-10.0.

    sh cuda_10.0.130_410.48_linux.run --extract=/home/user/Downloads
    sh ./cuda-linux.10.0.130-24817639.run

Configure the system to look for CUDA in the right place.

    sudo ln -sfn /home/user/Workspace/cuda-10.0 /usr/local/cuda
    sudo sh -c "echo /usr/local/cuda/lib64 >/etc/ld.so.conf.d/x86_64-cuda.conf"
    sudo ldconfig

Add /usr/local/cuda/bin to PATH in /etc/environment.

### cuDNN 7.4

Download cuDNN from the NVIDIA website (registration required). Unzip and merge into the CUDA install directory.

    sudo ldconfig

## TensorFlow 2.1.0

### CUDA 10.1

Download CUDA from the NVIDIA website. Get the .run package. Do not run the installer.

    sh cuda_10.1.105_418.39_linux.run --extract=/home/user/Downloads
    mv /home/user/Downloads/cuda-toolkit /home/user/Workspace/cuda-10.1

Configure the system to look for CUDA in the right place.

    sudo ln -sfn /home/user/Workspace/cuda-10.1 /usr/local/cuda
    sudo sh -c "echo /usr/local/cuda/lib64 >/etc/ld.so.conf.d/x86_64-cuda.conf"
    sudo ldconfig

### cuDNN 7.6.5

Download cuDNN from the NVIDIA website (registration required). Unzip and merge into the CUDA install directory.

    sudo ldconfig

### TensorRT 6 (optional)

Download TensorRT from the NVIDIA webiste (registration required). Unzip in a custom directory, for instance /home/user/Workspace/TensorRT-6.0.1.5.

Configure the system to look for TensorRT in the right place.

    sudo ln -sfn /home/user/Workspace/TensorRT-6.0.1.5 /usr/local/TensorRT
    sudo sh -c "echo /usr/local/TensorRT/lib >/etc/ld.so.conf.d/x86_64-TensorRT.conf"
    sudo ldconfig
