# ams_jetcis_kernel
kernel and device tree

Links:
https://developer.nvidia.com/embedded/linux-tegra-r3261


# Building the kernel

## prep
You need an Ubuntu 18.04 or 20.04 machine with x86 architecture.
WSL also works.

```
sudo apt-get update
sudo apt-get install make
sudo apt-get install make-guile
sudo apt-get install libncurses5-dev
sudo apt-get install gcc
sudo apt-get install python
```
Get Linaro
```
mkdir $HOME/l4t-gcc
cd $HOME/l4t-gcc
wget http://releases.linaro.org/components/toolchain/binaries/7.3-2018.05/aarch64-linux-gnu/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu.tar.xz
tar xf gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu.tar.xz
```
Get L4T sources
```
wget https://developer.nvidia.com/embedded/l4t/r32_release_v6.1/sources/t210/public_sources.tbz2
tar -xvf public_sources.tbz2
cd Linux_for_Tegra/source/public
JETSON_NANO_KERNEL_SOURCE=$(pwd)
tar -xf kernel_src.tbz2
```
TBD: patch the files
```
patch -p0  < kernel_ams_30092020.patch
patch -p0 < hardware_mira130_18012022.patch
patch -p0 < hardware_mira220_18012022.patch
patch -p0 < hardware_mira050_18012022.patch
patch -p0 < hardware_mira030_18012022.patch

```
issue: the hardware patch change the files in hardware_ORIG
Solution: (go inside the hardware folder and use -p1)
```
cd hardware_cgss130
patch -p1 < hardware_mira130_18012022.patch
cd hardware_cgss220
patch -p1 < hardware_mira220_18012022.patch
```


Compile
```
cd $JETSON_NANO_KERNEL_SOURCE
TOOLCHAIN_PREFIX=$HOME/l4t-gcc/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-
TEGRA_KERNEL_OUT=$JETSON_NANO_KERNEL_SOURCE/build
KERNEL_MODULES_OUT=$JETSON_NANO_KERNEL_SOURCE/modules
make -C kernel/kernel-4.9/ ARCH=arm64 O=$TEGRA_KERNEL_OUT LOCALVERSION=-tegra CROSS_COMPILE=${TOOLCHAIN_PREFIX} tegra_defconfig
make -C kernel/kernel-4.9/ ARCH=arm64 O=$TEGRA_KERNEL_OUT LOCALVERSION=-tegra CROSS_COMPILE=${TOOLCHAIN_PREFIX} -j8 --output-sync=target zImage
make -C kernel/kernel-4.9/ ARCH=arm64 O=$TEGRA_KERNEL_OUT LOCALVERSION=-tegra CROSS_COMPILE=${TOOLCHAIN_PREFIX} -j8 --output-sync=target modules
make -C kernel/kernel-4.9/ ARCH=arm64 O=$TEGRA_KERNEL_OUT LOCALVERSION=-tegra CROSS_COMPILE=${TOOLCHAIN_PREFIX} -j8 --output-sync=target dtbs
make -C kernel/kernel-4.9/ ARCH=arm64 O=$TEGRA_KERNEL_OUT LOCALVERSION=-tegra INSTALL_MOD_PATH=$KERNEL_MODULES_OUT modules_install
```

the newly built kernels and devicetree can be copied and used directly in the application
```
$JETSON_NANO_KERNEL_SOURCE/build/arch/arm64/boot/Image
$JETSON_NANO_KERNEL_SOURCE/build/arch/arm64/boot/dts/tegra210-p3448-0000-p3449-0000-a02.dtb
```
