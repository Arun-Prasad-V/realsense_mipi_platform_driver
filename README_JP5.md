# Intel® RealSense™ camera driver for GMSL* interface on NVIDIA® Jetson AGX Xavier™ JetPack 5.x.2

# D457 MIPI on NVIDIA® Jetson AGX Xavier™
The Intel® RealSense™ MIPI platform driver enables the user to control and stream RealSense™ 3D MIPI cameras.

The system shall include:
* Jetson™ platform (Currently Supported JetPack versions are: 5.1.2, 5.0.2)
* RealSense™ De-Serialize board (https://store.intelrealsense.com/buy-intel-realsense-des457.html)
* RS MIPI camera (e.g. https://store.intelrealsense.com/buy-intel-realsense-depth-camera-d457.html)

> Note: This MIPI reference driver is based on RealSense™ de-serialize board. For other de-serialize boards, modification might be needed.

# Links

- Jetson AGX Orin™ board setup - AGX Orin™ [JetPack 6.0](./README_JP6.md) setup guide
- Jetson AGX Xavier™ board setup - AGX Xavier™ [JetPack 5.x.2](./README_JP5.md) setup guide
- Jetson AGX Xavier™ board setup - AGX Xavier™ [JetPack 4.6.1](./README_JP4.md) setup guide
- Build Tools manual page [Build Manual page](./README_tools.md)


## NVIDIA® Jetson AGX Xavier™ board setup

Please follow the [instruction](https://docs.nvidia.com/sdk-manager/install-with-sdkm-jetson/index.html) to flash JetPack to the Jetson AGX Xavier™ with NVIDIA® SDK Manager or other methods NVIDIA provides. Make sure the board is ready to use.

## Build kernel, dtb and D457 driver

<details>
<summary>JetPack manual build</summary>

Download Jetson Linux source code tarball from 
- [JetPack 5.1.2 BSP sources](https://developer.nvidia.com/downloads/embedded/l4t/r35_release_v4.1/sources/public_sources.tbz2)
- [JetPack 5.0.2 BSP sources](https://developer.nvidia.com/embedded/l4t/r35_release_v1.0/sources/public_sources.tbz2)

- [JetPack 5.x.2 Toolchain](https://developer.nvidia.com/embedded/jetson-linux/bootlin-toolchain-gcc-93)


## JetPack 5.1.2
```
mkdir -p l4t-gcc/5.1.2
cd ./l4t-gcc/5.1.2
wget https://developer.nvidia.com/embedded/jetson-linux/bootlin-toolchain-gcc-93 -O aarch64--glibc--stable-final.tar.gz
tar xf aarch64--glibc--stable-final.tar.gz
cd ../..
wget https://developer.nvidia.com/downloads/embedded/l4t/r35_release_v4.1/sources/public_sources.tbz2
tar xjf public_sources.tbz2
cd Linux_for_Tegra/source/public
tar xjf kernel_src.tbz2
```
## JetPack 5.0.2
```
mkdir -p l4t-gcc/5.0.2
cd ./l4t-gcc/5.0.2
wget https://developer.nvidia.com/embedded/jetson-linux/bootlin-toolchain-gcc-93 -O aarch64--glibc--stable-final.tar.gz
tar xf aarch64--glibc--stable-final.tar.gz --strip-components 1
cd ../..
wget https://developer.nvidia.com/embedded/l4t/r35_release_v1.0/sources/public_sources.tbz2
tar xjf public_sources.tbz2
cd Linux_for_Tegra/source/public
tar xjf kernel_src.tbz2
```

# Apply D457 patches and build the kernel image, dtb and D457 driver.

```
# install dependencies
sudo apt install build-essential bc flex bison

# apply patches
./apply_patches_ext.sh ./Linux_for_tegra/source 5.0.2

# build kernel, dtb and D457 driver
./build_all.sh 5.0.2 ./Linux_for_tegra/source
```

</details>

---

The developers can set up the source code with NVIDIA's Jetson git repositories by using the provided scripts.
- Prepare build workspace for cross-compile on host
- Apply necessary patches
- Build workspace
- Deploy build results on target Jetson

```
./setup_workspace.sh 5.0.2

./apply_patches.sh apply 5.0.2

./build_all.sh 5.0.2
```

## Install kernel and D457 driver to Jetson AGX Xavier

1. Install the kernel and modules

Building with `build_all.sh`

The necessary files are:

- kernel image `images/<JetPack_version>/arch/arm64/boot/Image`
- dtb `images/5.0.2/arch/arm64/boot/dts/nvidia/tegra194-p2888-0001-p2822-0000.dtb`
- D457 driver `images/5.0.2/drivers/media/i2c/d4xx.ko`
- UVC Video driver `images/5.0.2/drivers/media/usb/uvc/uvcvideo.ko`
- V4L2 Core Video driver `images/5.0.2/drivers/media/v4l2-core/videobuf-core.ko`
- V4L2 VMalloc Video driver `images/5.0.2/drivers/media/v4l2-core/videobuf-vmalloc.ko`

Copy build results from Host to Jetson target `10.0.0.116` user `nvidia`
```
scp ./images/5.0.2/arch/arm64/boot/Image nvidia@10.0.0.116:~/
scp ./images/5.0.2/arch/arm64/boot/dts/nvidia/tegra194-p2888-0001-p2822-0000.dtb nvidia@10.0.0.116:~/
scp ./images/5.0.2/drivers/media/i2c/d4xx.ko nvidia@10.0.0.116:~/
scp ./images/5.0.2/drivers/media/usb/uvc/uvcvideo.ko nvidia@10.0.0.116:~/
scp ./images/5.0.2/drivers/media/v4l2-core/videobuf-core.ko nvidia@10.0.0.116:~/
scp ./images/5.0.2/drivers/media/v4l2-core/videobuf-vmalloc.ko nvidia@10.0.0.116:~/
```

Copy them to the right places on Jetson target:
```
sudo cp Image /boot
sudo cp tegra194-p2888-0001-p2822-0000.dtb /boot/dtb
sudo cp d4xx.ko /lib/modules/$(uname -r)/kernel/drivers/media/i2c/
sudo cp uvcvideo.ko /lib/modules/$(uname -r)/kernel/drivers/media/usb/uvc/
sudo cp videobuf-core.ko /lib/modules/$(uname -r)/kernel/drivers/media/v4l2-core/
sudo cp videobuf-vmalloc.ko /lib/modules/$(uname -r)/kernel/drivers/media/v4l2-core/
sudo depmod
```

2. Edit `/boot/extlinux/extlinux.conf` primary boot option's LINUX/FDT lines to use built kernel image and dtb file:

    ```
    LINUX /boot/Image-5.10.104-tegra
    FDT /usr/lib/linux-image-5.10.104-tegra/tegra194-p2888-0001-p2822-0000.dtb
    ```

3. Make D457 I2C module autoload at boot time:
    ```
    echo "d4xx" | sudo tee /etc/modules-load.d/d4xx.conf
    ```

After rebooting Jetson, the D457 driver should work.

**NOTE**

- It's recommended to save the original kernel image as backup boot option in `/boot/extlinux/extlinux.conf`.