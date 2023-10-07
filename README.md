# Mwatch

## Allwinner F1c200s

- ARM926EJ-S

- QFN-88-EP(10x10)

- 64m ddr1 inside

- 408MHz(800MHz Over C)

## Periph

- TF Card

- MPU6050

- ST7789 SPI TFT (172*320)

- Audio Codec

- IO Key * 3

- SDIO Wifi(ESP8266、RTL8723)

- TypeC
  USB Host/Slave
  USB2UART for Debug

## Linux Design

### ARM-GCC Set Env

- 安装交叉编译链#

> 
> wget http://releases.linaro.org/components/toolchain/binaries/7.2-2017.11/arm-linux-gnueabi/gcc-linaro-7.2.1-2017.11-x86_64_arm-linux-gnueabi.tar.xz  
> sudo tar -Jvxf gcc-linaro-7.2.1-2017.11-x86_64_arm-linux-gnueabi.tar.xz -C /opt/  
> sudo vim /etc/bash.bashrc
> 
> 在文件末尾 添加内容  
> PATH="$PATH:/opt/gcc-linaro-7.2.1-2017.11-x86_64_arm-linux-gnueabi/bin"  
> source /etc/bash.bashrc  


### u-boot
> 
> 获取u-boot Code  
> git clone -b u-boot-Mwatch https://gitee.com/byleefei/u-boot.git  
> cd u-boot
> 
> 编译  
> make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- mwatch_defconfig  
> make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j16
> 
> 烧录到SD卡  
> sudo dd if=./u-boot-sunxi-with-spl.bin of=/dev/sdb bs=8K seek=1  
> sudo dd if=./u-boot-sunxi-with-spl.bin of=**/dev/sdX** bs=8K seek=1  
> **/dev/sdX**为sd卡挂载的路径  
> 使用sudo fdisk -l查看


### Kernel

> 获取kernel Code
> 
> git clone -b linux-5.7.1-Mwatch https://gitee.com/byleefei/linux.git  
> cd linux/  
> 
>编译
> make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- mwatch_defconfig  
> make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j16  
>
> 烧录  
> 将zImage 和mwatch.dtb 复制到SD卡第一个分区里
> sudo fdisk -l  
> sudo -s 
> mount /dev/sdb1 /mnt  
> cp -rf arch/arm/boot/zImage /mnt/  
> cp -rf arch/arm/boot/dts/mwatch.dtb /mnt/  
> umount /mnt



### Rootfs

#### buildroot
buildroot-2020.02.6  
编译完成后挂载TF卡到mnt  
root下
> tar -xvf output/images/rootfs.tar -C /mnt  
> sudo chown root /mnt/usr/bin/* -R


error调试 [link](http://fuqiang1986.com/2022/07/22/%E5%85%A8%E5%BF%97%E5%93%AA%E5%90%92d1%E5%BC%80%E5%8F%91%E6%9D%BF%E4%BB%A3%E7%A0%81%E4%B8%8B%E8%BD%BD%E7%BC%96%E8%AF%91%E4%B8%8E%E8%B0%83%E8%AF%95/)

##### error1
> c-stack.c:55:26: error: missing binary operator before token "("  
>    55 | #elif HAVE_LIBSIGSEGV && SIGSTKSZ < 16384

vim ./output/build/host-m4-1.4.18/lib/c-stack.c
注释三行代码
> //#elif HAVE_LIBSIGSEGV && SIGSTKSZ < 16384
> 
> /* libsigsegv 2.6 through 2.8 have a bug where some architectures use
> 
>    more than the Linux default of an 8k alternate stack when deciding
> 
>    if a fault was caused by stack overflow.  */
> 
> //# undef SIGSTKSZ
> 
> //# define SIGSTKSZ 16384


##### error2
> ‘_STAT_VER’ undeclared (first use in this function)
>   100 | #define INT_NEXT_LSTAT(a,b) NEXT_LSTAT64(_STAT_VER,a,b)

vim ./output/build/host-fakeroot-1.20.2/libfakeroot.c
添加三行代码
```
diff --git a/libfakeroot.c b/libfakeroot.c
index 3e80e38..14e56bc 100644
--- a/libfakeroot.c
+++ b/libfakeroot.c
@@ -90,6 +90,10 @@
 #define SEND_GET_XATTR64(a,b,c) send_get_xattr64(a,b)
 #endif
 
+#ifndef _STAT_VER
+#define _STAT_VER 0
+#endif
+
 /*
    These INT_* (which stands for internal) macros should always be used when
    the fakeroot library owns the storage of the stat variable.
```

##### error3  
> /worktmp/Mwatch/buildroot-2020.02.6/output/build/qt5base-5.12.8/include/QtCore/../../src/corelib/global/qendian.h:333:35: error: ‘numeric_limits’ is not a member of ‘std’  
>   333 |     { return QSpecialInteger(std::numeric_limits<T>::min()); }

`vim /worktmp/Mwatch/buildroot-2020.02.6/output/build/qt5base-5.12.8/include/QtCore/../../src/corelib/global/qendian.h`  
加入`#include <limits>`

其他文件遇到这种错误,都加入`#include <limits>`  

> /worktmp/Mwatch/buildroot-2020.02.6/output/build/qt5base-5.12.8/src/corelib/tools/qbytearraymatcher.h  
> ./output/build/qt5base-5.12.8/util/lexgen/generator.cpp  
> ./output/build/qt5base-5.12.8/src/tools/moc/generator.cpp


### APP

- LVGL
- Qt

## Demo

### 离线地图

### 重力球

### ikun大战

### ChatGPT语音交互

### 天气时钟

### 

<!-
