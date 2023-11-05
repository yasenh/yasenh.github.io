---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Install CUDA and Nvidia Driver in Linux"
subtitle: ""
summary: "Setup environment in 10 minutes"
authors:
- admin
tags:
- CUDA
categories: ["Misc"]
date: 2020-09-07T23:13:09-04:00
lastmod: 2020-09-07T23:13:09-04:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: true

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---





## Introduction

The purpose of this post is to log and share my experience when installing the CUDA and Nvidia driver. The Nvidia driver may be easily messed up if you accidentally upgrade or downgrade your CUDA version. 

Recently I updated my PyTorch version with following commands:
```bash
conda install pytorch torchvision cudatoolkit=10.2 -c pytorch
```

Then I am not able to login my Linux system anymore, it gets stuck in a login loop which indicates there maybe something wrong with the Nvidia driver. The issues was resolved by reinstalling the CUDA and Nvidia driver again.




## Prerequisite
- CUDA-Capable GPU 
- Supported version of Linux
- GCC installed

More details can be found from the [Nvidia official document](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#pre-installation-actions).



## Install CUDA and Nvidia driver

1. Download CUDA toolkit [https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads)

1. Switch to virtual console by pressing "Ctrl+Alt+F1"

1. Disabling Nouveau

    ```bash
    # The Nouveau drivers are loaded if the following command prints anything:
    $ lsmod | grep nouveau

    # Create a file at /etc/modprobe.d/blacklist-nouveau.confs:
    blacklist nouveau
    options nouveau modeset=0

    # Regenerate the kernel initramfs:
    $ sudo update-initramfs -u
    ```

1. Disable  X server and run installation file from there. **Important Note:** user should add `--run-nvidia-xconfig` option to tell the driver installation to run nvidia-xconfig to update the system X configuration file, so that the NVIDIA X driver is used. **Otherwise the system may gets stuck in a login loop after reboot!**

    ```bash
    $ sudo service lightdm stop
    # Remember to run Nvidia xconfig
    $ sudo ./cuda_10.2.89_440.33.01_linux.run --run-nvidia-xconfig
    # User may need to remove old driver, follow instructions from run file.
    ```

1. Post-installation actions: environment setup. The `PATH` variable needs to include /usr/local/cuda/bin. The `LD_LIBRARY_PATH` variable needs to contain /usr/local/cuda/lib64 on a 64-bit system, and /usr/local/cuda-7.5/lib on a 32-bit system
   

    ```bash
    $ sudo vim /etc/profile.d/myenvvars.sh
    export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}

    $ sudo vim /etc/ld.so.conf.d/mylib.conf
    /usr/local/cuda/lib64
    ```

1.  Verify the driver version
    
    ```bash
    $ cat /proc/driver/nvidia/version
    # CUDA Toolkit version
    $ nvcc -V
    ```

1. Install Third-party Libraries

    ```bash
    $ sudo apt-get install g++ freeglut3-dev build-essential libx11-dev \
        libxmu-dev libxi-dev libglu1-mesa libglu1-mesa-dev
    ```

1. Install cuDNN if needed

    ```bash
    $ cd folder/extracted/contents
    $ sudo cp include/cudnn.h /usr/local/cuda/include
    $ sudo cp -a lib64/libcudnn* /usr/local/cuda/lib64
    ```

1. Check CUDA/cuDNN versions

    ```bash
    # Check CUDA version
    $ nvcc --version
    # Check cuDNN version
    $ cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
    ```

## References
1. [cuda-installation-guide-linux](http://docs.nvidia.com/cuda/cuda-installation-guide-linux)
2. [cuda-quick-start-guide](https://docs.nvidia.com/cuda/cuda-quick-start-guide/index.html#linux)
3. [CUDA_Installation_Guide_Linux.pdf](https://docs.nvidia.com/pdf/CUDA_Installation_Guide_Linux.pdf)
