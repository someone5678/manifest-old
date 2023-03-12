**English** | [中文](README_CN.md)

# Open Source Kernel for Xiaomi 12T Pro / Redmi K50 Ultra (diting)

The Xiaomi 12T Pro was released at the 4th October 2022.
Unfortunately, Xiaomi did not release kernel resources with the release of the device, but fortunately the drivers for this device are scattered all over the place. This project aims to provide the missing resources so that developers and enthusiasts can easily start compiling kernels for Xiaomi 12T Pro / Redmi K50 Ultra.

## How to build the kernel
1. Intitalize your local repository using this manifest:
```
repo init -u https://github.com/Xiaomi-Diting-Kernel-Dump/manifest.git -b diting
```
2. Then to sync up:
```
repo sync
```
3. Run the kernel build for your device. E.g for diting:
```
./build.sh diting
```
4. Find the build artifacts in `device/qcom/diting-kernel/`. This includes the kernel (Image), the kernel modules, dtb and dtbo, kernel headers and some host utilities.

## What is missing in the official sources?
The official kernel sources insist of the following repositories:
- [Xiaomi_Kernel_Opensource](https://github.com/MiCode/Xiaomi_Kernel_OpenSource/commits/ziyi-s-oss)
This repository contains xiaomis fork of the msm-kernel and the camera-kernel and display-drivers techpack repositories.
- [vendor_qcom_opensource_audio-kernel](https://github.com/MiCode/vendor_qcom_opensource_audio-kernel/tree/ziyi-s-oss)
This repository contains xiaomis fork of the qualcomm audio-kernel-ar repository.
- [vendor_qcom_opensource_wlan](https://github.com/MiCode/vendor_qcom_opensource_wlan/tree/ziyi-s-oss)
This repository contains xiaomis forks of the qualcomm qcacld-3.0, qca-wifi-host-cmn, fw-api and sigma-dut repositories.
- [kernel_devicetree](https://github.com/MiCode/kernel_devicetree/tree/ziyi-s-oss)
This repository contains xiaomis forks of the proprietary qualcomm devicetree, camera-devicetree and display-devicetree.

This kernel combines the code of mayfly-s-oss, ziyi-s-oss, and zeus-s-oss.

Starting with the msm-kernel xiaomi has added their own hwid driver, which is used in kernel to differentiate between different xiaomi devices, to the .gitingore file in the repository and hence the whole source code of this driver is missing. 

Xiaomi also did not open source the devicetree of diting, we decompiled the dtb file to take it apart and update it.

Xiaomi does not provide the source code of many used qualcomm external kernel module repositories. Namely these are: [mmrm-driver](https://git.codelinaro.org/clo/la/platform/vendor/opensource/mmrm-driver), [mmrm-driver-test](https://git.codelinaro.org/clo/la/platform/vendor/opensource/mmrm-driver-test), [video-driver](https://git.codelinaro.org/clo/la/platform/vendor/opensource/video-driver), [video-driver-test](https://git.codelinaro.org/clo/la/platform/vendor/opensource/video-driver-test), [cvp-kernel](https://git.codelinaro.org/clo/la/platform/vendor/opensource/cvp-kernel), [dataipa](https://git.codelinaro.org/clo/la/platform/vendor/opensource/dataipa), [eva-kernel](https://git.codelinaro.org/clo/la/platform/vendor/opensource/eva-kernel), [touch-drivers](https://git.codelinaro.org/clo/la/platform/vendor/opensource/touch-drivers), [datarmnet](https://git.codelinaro.org/clo/la/platform/vendor/qcom/opensource/datarmnet) and [datarmnet-ext](https://git.codelinaro.org/clo/la/platform/vendor/qcom/opensource/datarmnet-ext).
These repositories might not have been changed by xiaomi but since their kernel is based on a pre-public caf tag one has to guess the used revision of all these repositories.

Xiaomi does not provide the source code of the following kernel modules which are not publicised by qualcomm: binder_prio.ko, sla.ko. It does not seem like they have a meaningful function.

Xiaomis proviced devicetree is incomplete and does not work because it misses the video, mmrm and audio devicetree techpacks. These techpack devicetrees are not publicised by qualcomm which makes it harder to get this working.

## Fixes for the mentioned issues

### The hwid driver
Xiaomi has used a similar hwid driver on older device generations already, so it was possible to import an older driver and update the defines of the internal codenames for diting.We reverse the process and find that the codename of diting is 10.

### The device tree
Unfortunately, diting's devicetree is not publicly available. Fortunately, we found the display and camera related devicetree of diting in ziyi-s-oss and the devicetree of diting in the commit log of mayfly-s-oss, and updated the dtb file by decompiling the dtb file and looking for the audio devicetree related changes.

### The missing external kernel module repositories
As mentioned above xiaomi might not have touched these modules so I was able to guess revisions of the modules and include the repositories from qualcomm and it seems to work fine.

### The mysterious binder_prio and sla kernel modules
The system seems to run fine without them.

### The missing devicetree techpacks
Fortunately ASUS has properly released these sources for their Snapdragon 8G1 devices ([link](https://dlcdnets.asus.com/pub/ASUS/ZenFone/AI2201/ASUS_AI2201-32.2810.2205.63-kernel-src.tar.gz)) and i was able to use them as a base to apply xiaomis changes on top of. The video and mmrm devicetrees do not have any changes and I have reversed their audio devicetree changes for diting. [commit](https://github.com/xiaomi-sm8450-kernel/android_vendor_qcom_proprietary_audio-devicetree/commit/b44fdf9e6568219cca4a04a5f39bda495088f4dc)

## Other issues
On qualcomm 5.10 devices the devicetree is built from multiple techpack repositories as multiple dtb(o)s which will be merged later. Xiaomi has introduced the `xiaomi,miboard-id` property to mark split devicetrees which are (in)compatible to merge. The public qualcomm script to merge dtbs does not respect this property and merges incompatible split devicetrees together which results in a broken devictree. This can be fixed by respecting the miboard-id in the merge dts script. [commit](https://github.com/xiaomi-sm8450-kernel/android_kernel_platform_build/commit/8488cc7b670607aa652310cf8eac6a92047b77f1)

## Credits

- [xiaomi-sm8450-kernel](https://github.com/xiaomi-sm8450-kernel/manifest/)：A lot of pioneering work has been done
