# uboot emmc config 
[toc]

## 1 Introduction
sfc boot (nor flash) is default boot meathod in t40 GANDER board. The aim of this document is record the test boot from eMMC media. Now the SD card drivered as same as eMMC device. Test SD card first will be convinient.

## 2 Step of test
Record the test content step by step.

### 2.1 Compile mmc boot uboot then got error of 'no mmc 0 device'
Compile mmc specific target of uboot.

```
# macro msc0 for SD card - PB
make isvp_t40xp_msc0

# macro msc1 for eMMC - PC
make isvp_t40xp_msc1
```

### 2.2 Modification of hardware of GANDER board then got error of 'no mmc 1 device'
GPIO of enable pin of SD card need to be pull up in highlevel which means VDDMSC0 need 3.3V. This operation need done by hardware in the uboot stage.
There still no mmc 1 device in uboot command line, but it's ok to load firmware by usb boot then boot from eMMC. That means the boot of menufacture and usb boot is ok to detect eMMC device.

### 2.3 Load firmware into SD card

```sh
# insert SD card into host-pc
umount /media/<user>/<media device>

sudo fdisk /dev/sdb
> o
> n
> \r # as default -- p
> \r # as default -- 1
> \r # as default -- 2048
> \r # as default -- total size
> w

sync
# re-insert SD card

# format SD card as VFat file system

# burn firmware into SD card in offset 17KB
dd if=u-boot-with-spl.bin of=/dev/sdb bs=1024 seek=17
```

### 2.4 Boot from SD card then got error of 'can't get kernel image'
Then start uboot from SD card but assert an error shows "can't get kernel image!".
That's because the mmc read operation havn't catch the right offset and the count of block. Thus can't copy correct image into memory.
Do modification of the offset and cnt in isvp_t40.h
```c
#ifdef CONFIG_SPL_MMC_SUPPORT
/* mmc read <mem_addr> <mmc_offset_sector_512byte> <cnt_of_sector_512byte> */
#define CONFIG_BOOTCOMMAND "mmc read 0x80600000 0x222 0x7E00; bootm 0x80600000"
#endif  /* CONFIG_SPL_MMC_SUPPORT */
```

### 2.5 After load kernal image then got error of 'VFS: Cannot open root device'
Error shown,
> VFS: Cannot open root device "mmcblk0p2" or unknow-block(179,2):error -6
That shows no device of "mmcblk0p2".
Then part SD card with partition 2 by cmd 'fdisk'
```c
sudo fdisk /dev/sdb
```
Set SD card partion2 as VFAT file system.
```
# in host-pc
mkfs.vfat /dev/sdb1
```
Then got error of 'devtmpfs: error mounting -2'

That's because the bootcmd shows 'init=/linuxrc'(3 ways as usual: ramdisk, initrd, disk)[^1], so we need load rootfs to 'root' manually.
```c
#define CONFIG_BOOTARGS BOOTARGS_COMMON " init=/linuxrc root=/dev/mmcblk0p2 rw rootdelay=1"
```
load rootfs manually in host-pc,
```sh
sudo dd if=./root-glibc-toolchain720.squashfs of=/dev/sdb2
```
Then it works, enjoy it.

### 2.6 eMMC configuration and test
+ s1. compile msc1
```sh
make isvp_t40xp_msc1
```
We need to change driver file from 'jz_sdhci.c' to 'jz_mmc.c'.

+ s2. emmc driver in uboot

### 2.7 eMMC driver in kernel
```
diff --git a/ISVP-T40-1.2.0-20220609/software/sdk/Ingenic-SDK-T40-1.2.0-20220609/opensource/kernel-4.4.94/arch/mips/boot/dts/ingenic/shark.dts b/ISVP-T40-1.2.0-20220609/software/sdk/Ingenic-SDK-T40-1.2.0-20220609/opensource/kernel-4.4.94/arch/mips/boot/dts/ingenic/shark.dts
index e72ed4135..8641f2f70 100644
--- a/ISVP-T40-1.2.0-20220609/software/sdk/Ingenic-SDK-T40-1.2.0-20220609/opensource/kernel-4.4.94/arch/mips/boot/dts/ingenic/shark.dts
+++ b/ISVP-T40-1.2.0-20220609/software/sdk/Ingenic-SDK-T40-1.2.0-20220609/opensource/kernel-4.4.94/arch/mips/boot/dts/ingenic/shark.dts
@@ -94,7 +94,7 @@
 };
 
 &msc1 {
-	status = "disable";
+	status = "okay";
 	pinctrl-names = "default";
 	/*mmc-hs200-1_8v;*/
 	cap-mmc-highspeed;
@@ -106,7 +106,7 @@
 	/* special property */
 	ingenic,wp-gpios = <0>;
 	ingenic,rst-gpios = <0>;
-	pinctrl-0 = <&msc1_pb>;
+	pinctrl-0 = <&msc1_pc>;
 };
 
 &mac0 {
```
## Reference
[^1]:[uboot start up flow](https://zhuanlan.zhihu.com/p/520575102)
