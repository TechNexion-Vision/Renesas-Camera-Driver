# Renesas-Camera-Driver

[![Technexion](https://github.com/user-attachments/assets/c78973b8-4967-4d72-8082-5d6b9ef5dc99)](https://www.technexion.com/products/embedded-vision/)

[![Producer: Technexion](https://img.shields.io/badge/Producer-Technexion-blue.svg)](https://www.technexion.com)
[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)

## Introduction

[TechNexion Embedded Vision Solutions](https://www.technexion.com/products/embedded-vision/) provide embedded system developers access to high-performance, industrial-grade camera solutions to accelerate their time to market for embedded vision projects.

---

## Supported Renesas EVK

- [RZ/V2H-EVK](https://www.renesas.com/us/en/products/microcontrollers-microprocessors/rz-mpus/rzv2h-evk-rzv2h-quad-core-vision-ai-mpu-evaluation-kit)

## Support System Version

- [RZ/V2H AI SDK v3.00](https://www.renesas.com/us/en/document/sws/rzv2h-ai-sdk-v300-source-code?r=25470141)

## Support Camera Modules

#### MIPI Cameras
- TEVS-AR0144-C
- TEVS-AR0145-M
- TEVS-AR0234-C
- TEVS-AR0521-C
- TEVS-AR0522-C
- TEVS-AR0522-M
- TEVS-AR0821-C
- TEVS-AR0822-C
- TEVS-AR1335-C

[More Camera Products Details...](https://www.technexion.com/products/embedded-vision)

---

## Install TN Camera on RZ/V2H-EVK

#### Adaptor for **RZ/V2H-EVK**

TEVS-RPI22 Adaptor for TEVS camera

> Connect TEVS camera and TEVS-RPI22 adaptor to **RZ/V2H-EVK - "CN7/CN8/CN10/CN9"** directly.

<a href="https://www.technexion.com/products/embedded-vision/mipi-csi2/evk/tevs-ar0144-c-s33-ir-rpi22/" target="_blank">
 <img src="https://www.technexion.com/wp-content/uploads/2023/11/tevs-ar0144-c-s33-ir-rpi22.png" width="400" height="400" />
</a>

[**Instructional Video**](https://youtu.be/W9p1uqYS8lg?list=PLdUUBf0zT0W8feLAI_IhhhCr6w073zJ-L&t=47)

---

#### Method 1 - Pre-built Image for TEVS Camera on RZ/V2H-EVK

We provide a pre-built image to simplify using TEVS camera on RZ/V2H-EVK.

1. Download the image from the following link.

   [RZ/V2H + Yocto-AI-v5.0 + TEVS](https://download.technexion.com/demo_software/EVK/Renesas/RZ-V2H/DiskImage/rzv2h-evk-ver1_yocto-ai-v5_tevs_demo_20250115.zip)

2. Install necessary tool.

   ```shell
   $ sudo apt-get update
   $ sudo apt-get install -y unzip bmap-tools
   ```

3. Extract the pre-built image.

   ```shell
   $ unzip rzv2h-evk-ver1_yocto-ai-v5_tevs_demo_20250115.zip
   ```

4. Flash SD Card with the image.

   - Enter the pre-built image directory
        ```shell
        $ cd ./rzv2h-evk-ver1_yocto-ai-v5_tevs_demo_20250115
        ```
   - Use `bmaptool` to flash the image to the SD card
        ```shell
        # ${device} is your device path name, such as /dev/sdb
        $ umount ${device}?
        $ sudo bmaptool copy --bmap rzv2h-evk-ver1_yocto-ai-v5_tevs_demo_20250115.rootfs.wic.bmap \
        rzv2h-evk-ver1_yocto-ai-v5_tevs_demo_20250115.rootfs.wic.gz ${device}
        ```

---

#### Method 2 - Build the RZ/V2H AI SDK Source Code with Technexion patch

Please refernece "[How to build RZ/V2H AI SDK Source Code](https://renesas-rz.github.io/rzv_ai_sdk/5.00/howto_build_aisdk_v2h.html#step1)" and follow the steps below:

1. Download the SDK source code.
   - [RZ/V2H AI SDK Source Code](https://www.renesas.com/us/en/document/sws/rzv2h-ai-sdk-v500-source-code)

2. Install necessary tool.

   ```shell
   $ sudo apt-get update
   $ sudo apt-get install -y gawk wget git-core diffstat unzip bmap-tools texinfo \
   gcc-multilib build-essential chrpath socat cpio python python3 python3-pip \
   python3-pexpect xz-utils debianutils iputils-ping libsdl1.2-dev xterm p7zip-full \
   libyaml-dev libssl-dev
   ```

3. Extract the SDK source code.

   - Unpack the source code archive
        ```shell
        $ unzip RTK0EF0180F05000SJ_linux-src.zip
        ```
   - Create a folder for the Yocto recipe package
        ```shell
        $ mkdir -p ./yocto_ai_v5.00
        ```
   - Extract the Yocto recipe package to the specified directory
        ```shell
        $ tar zxvf rzv2h_ai-sdk_yocto_recipe_v5.00.tar.gz -C ./yocto_ai_v5.00
        ```

4. Get TEVS camera driver patch.

   - Clone Technexion renessas camera driver repository
        ```shell
        $ git clone -b ai-sdk-v5.00 https://github.com/TechNexion-Vision/Renesas-Camera-Driver.git
        ```
   - Copy the driver patch to the Yocto recipe directory
        ```shell
        $ cp ./Renesas-Camera-Driver/TechNexion_TEVS.patch ./yocto_ai_v5.00
        ```

5. Apply TEVS camera driver patch.

   ```shell
   $ patch -p1 -i ./TechNexion_TEVS.patch
   ```

6. Initialize a build in poky.

   ```shell
   $ TEMPLATECONF=${PWD}/meta-renesas/meta-rzv2h/docs/template/conf/ source poky/oe-init-build-env
   ```

7. Add layers to `bblayers.conf`.

    ```shell
    $ bitbake-layers add-layer ../meta-rz-features/meta-rz-graphics
    $ bitbake-layers add-layer ../meta-rz-features/meta-rz-drpai
    $ bitbake-layers add-layer ../meta-rz-features/meta-rz-opencva
    $ bitbake-layers add-layer ../meta-rz-features/meta-rz-codecs
    $ bitbake-layers add-layer ../meta-tn
    ```

8.  Apply the patch files.

    ```shell
    $ patch -p1 < ../0001-tesseract.patch
    $ patch -p1 < ../0002-sd-image-size-16gb.patch
    ```

9. Build the kerenl files.

    ```shell
    $ MACHINE=rzv2h-evk-ver1 bitbake core-image-weston
    ```

10. Flash SD Card with the image.

    - Enter the Yocto build images directory
        ```shell
        $ cd ./tmp/deploy/images/rzv2h-evk-ver1
        ```
    - Use `bmaptool` to flash the image to the SD card
        ```shell
        # ${device} is your device path name, such as /dev/sdb
        $ umount ${device}?
        $ sudo bmaptool copy --bmap core-image-weston-rzv2h-evk-ver1-*.rootfs.wic.bmap \
        core-image-weston-rzv2h-evk-ver1-*.rootfs.wic.gz ${device}
        ```

---

## Stream on Camera by GStreamer

Once EVK has finished booting, you will see some archives in the home directory.

```
root@rzv2h-evk-ver1:~# ls -l

-rwxr--r--  1 root root 3599 Mar  9  2018 gstreamer-init-tevs-4cam.sh
-rwxr--r--  1 root root 1424 Mar  9  2018 gstreamer-init-tevs.sh
-rwxr--r--  1 root root  845 Mar  9  2018 v4l2-init.sh
```

There are two main scripts to help you easily set up and stream from TEVS camera.

- gstreamer-init-tevs.sh

    ```
    Usage: gstreamer-init-tevs.sh [media_node] [resolution]
        [media_node]: 0 or /dev/media0 is a sample
        [resolution]: 640x480 is a sample
    ```

    This script requires two arguments to set up the camera:
    - **media_node** - The media device node.
    - **resolution** - The supported resolution depend on the camera.

    Example command:
    ```
    root@rzv2h-evk-ver1:~# sh gstreamer-init-tevs.sh 0 640x480
    ```

- gstreamer-init-tevs-4cam.sh

    ```
    Usage: gstreamer-init-tevs-4cam.sh [resolution]
        [resolution]: 640x480 is a sample
    ```

    This script requires one argument to set up four cameras at once.
    - **resolution** - The supported resolution depend on the camera.

    Example command:
    ```
    root@rzv2h-evk-ver1:~# sh gstreamer-init-tevs-4cam.sh 640x480
    ```
