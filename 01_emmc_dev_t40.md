# emmc develop in t40 device 
[toc]

## 1 Introduction
This document records eMMC configuration in t40 device, details of partation and modification.

## 2. Configuration
Details about modification of emmc configuration in uboot and kernel.

### 2.1 uboot compiling  command
Modify compiling command for mmc device in uboot
```shell
make isvp_t40xp_msc0
```

### 2.2 config bootcommand
```c
/*
Image Part Size						| Address Byte				| emmc blk#cnt by 512 Byte	| name		|
emmc start offset 17K (17 * 1024)	| [0x0		, 0x4400	)	| [0x0	, 0x22	) # 0x22	| reserved	|
UBOOT_PART_SIZE=$((256 * 1024))		| [0x4400	, 0x44400	)	| [0x22	, 0x222	) # 0x200	| uboot		|
KERNEL_PART_SIZE=$((2560 * 1024))	| [0x44400	, 0x2c4400	)	| [0x222, 0x1622) # 0x1400	| kernel	|
ROOTFS_PART_SIZE=$((2048 * 1024))	| [0x2c4400	, 0x4c4400	)	| [0x1622,0x2622) # 0x1000	| rootfs	|
SYSTEM_PART_SIZE=size system.jffs2	| [0x4c4400	, size		)	| [0x2622,--	) # --		| system	|
Total size 16M
*/
// read kernel (uImage) into memory from block 0x222 offset cnt 0x1400 by boot command
#define CONFIG_BOOTCOMMAND "mmc read 0x80600000 0x222 0x1400; bootm 0x80600000"
```

### 2.3 emmc device config in kernel
Modify emmc pin definition in device tree
```git
@@ -83,14 +83,14 @@
 	max-frequency = <25000000>;
 	bus-width = <4>;
 	voltage-ranges = <1800 3300>;
-	cd-inverted;
+	non-removable;
 
 	/* special property */
+	ingenic,rst-gpios = <&gpb 8 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
 	ingenic,wp-gpios = <0>;
-	ingenic,cd-gpios = <&gpc 6 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
-	ingenic,rst-gpios = <0>;
+	ingenic,pwr-gpios = <0>;
+	ingenic,cd-gpios = <&gpb 28 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
 	pinctrl-0 = <&msc0_pb>;
-	#mmc-pwrseq = <&mmc0_pwrseq>;
 };
```

### 2.4 emmc partition in uboot
there is no emmc partition tool in current uboot, port fdisk tool from Uboot(samsung) into our uboot
port file cmd_mmc_fdisk.c from Uboot(samsung) into command/
Do some partition definition

```sh
fdisk usage:
fdisk -p 0	# show partition info
fdisk -c 0	# create partition
# more details refer source file cmd_mmc_fdisk.c
# partition 
# 0 - 10 MiB	| reserved
# 256 MiB		| SYSTEM_PART_SIZE
# 10 MiB		| USER_DATA_PART_SIZE
# 10 MiB		| CACHE_PART_SIZE
# rest			| sd storage
```

add command fdisk into bootcommand

```c
#define CONFIG_BOOTCOMMAND "fdisk -c 0; mmc read 0x80600000 0x222 0x1400; bootm 0x80600000"
```

### 2.5 config bootargs for mount emmc patition p2
```c
#define CONFIG_BOOTARGS BOOTARGS_COMMON " init=/linuxrc root=/dev/mmcblk0p2 rw rootdelay=1"
```

## 3. filesystem

### 3.1 current scheme
```
rootfs -- squashfs (read only)
system -- jffs2	   (write and read) usually for sfc_nor
```

### 3.2  ext4 filesystem

```sh
unsquanshfs root
# package requirement
# sudo apt install android-sdk-ext4-utils
make_ext4fs -l 25m rootfs.ext4 squashfs_root/
```

### 3.3 init 
```
linuxrc -> busybox -> inittab -> rcS -> app_init.sh
```

---

## Burning procedure
### 1. usb cloner configuration
#### 1.1 POLICY
| image name	| address	| file					|
| ---			| ---		| ---					|
| uboot			| 0x4400	| u-boot-with-spl.bin	|
| kernel		| 0x44400	| uImage				|
| rootfs		| 0xa00000	| rootfs.ext4			|

#### 1.2 MMC 
```sh
if "rootfs update or burn in fisrt time"
then
[x]	Erase All
else
[x] Partial Erase
0x0 ~ 0x2c4400
# don't load rootfs
fi
```

### 2. boot at the fisrt time atfer burn image
#### 2.1 parition emmc in uboot
```
fdisk -c 0
# or put this command into bootcommand
```

