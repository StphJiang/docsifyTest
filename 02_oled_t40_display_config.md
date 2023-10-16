# Oled display in t40 device 
[toc]

## 1. Introduction
Oled device seeya-103 driver development in t40xp, including oled device control, mipi
configuration, moudle developemnt and character device configuration. The oled driver
is same as lcd driver. The former 's pixels called 'self-emissive', which the light can be 
controlled on a pixel-by-pixel basis, while the latter need backlight. Just different light 
control methods in code.

## 2. Add Panel
### 2.1 device tree
Add oled device into device tree, corresponding with seeya driver file by attr 'compatible'.
```patch
@@ -271,6 +271,13 @@
 		#address-cells = <1>;
 		#size-cells = <1>;
 		ranges = <>;
+		panel_seeya103@0 {
+			compatible = "ingenic,seeya103";
+			ingenic,vdd-en-gpio = <&gpa 20 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
+			ingenic,rst-gpio = <&gpa 19 GPIO_ACTIVE_LOW INGENIC_GPIO_NOBIAS>;
+			status = "okay";
+		};
+
```
GPIO-A20: enable pin to enable power of oled display
GPIO-A19: reset pin of SeeYA oled display device

### 2.2 kconfig
Add SeeYA oled panel into linux which can be found in menuconfig
```patch
@@ -108,3 +108,8 @@ config PANEL_MTF070
 	help
 		lcd panel mtf070, for ingenicfb drivers.
 
+config PANEL_SEEYA
+	tristate "SeeYA 103 OLED (1280x1280)"
+	depends on FB_INGENIC_DISPLAYS_V12
+	help
+		lcd panel seeya, for ingenicfb drivers.
```

### 2.3 Makefile
Add reliable rule in Makefile to compile panel-seeya103.c
using 'ojb-y' to compile this module into kernel.
```makefile
obj-$(CONFIG_PANEL_SEEYA)	+= panel-seeya103.o
```

### 2.4 Display config
Select all moudles needed by lcd display configuration in menuconfig
```sh
make menuconfig
```
Choose colorbar test code as lcd display test at first, after 
After all, configuration in isvp_chark_defconfig shown like before,

```defconfig
CONFIG_FB=y
CONFIG_FB_CMDLINE=y
CONFIG_FB_CFB_FILLRECT=y
CONFIG_FB_CFB_COPYAREA=y
CONFIG_FB_CFB_IMAGEBLIT=y

CONFIG_FB_INGENIC=y
CONFIG_FB_VSYNC_SKIP_DISABLE=y
CONFIG_FB_VSYNC_SKIP=9
CONFIG_FB_INGENIC_NR_FRAMES=2
CONFIG_FB_INGENIC_NR_LAYERS=1
CONFIG_FB_INGENIC_MIPI_DSI=y
CONFIG_MIPI_TX_LANE=y
CONFIG_MIPI_4LANE=y
CONFIG_FB_INGENIC_V12=y
CONFIG_INGENIC_FB_COLOR_MODE=y
CONFIG_RGB888=y
CONFIG_FB_INGENIC_DISPLAYS_V12=y
CONFIG_PANEL_SEEYA=y
CONFIG_BACKLIGHT_LCD_SUPPORT=y
CONFIG_LCD_CLASS_DEVICE=y
CONFIG_BACKLIGHT_CLASS_DEVICE=y
CONFIG_BACKLIGHT_GENERIC=y
```

### 2.5 OLED config
#### 2.5.1 add config command
Add mipi cmd  to config registers in oled by mipi command mode.
Notice: cmd get from oled display vender
```c
{0x15, 0x03, 0x80},														// DSC Close
{0x15, 0x53, 0x29},														// BCTRL Enable
{0x39, 0x03, 0x00, {0x51, 0xFF, 0x01}},									// DBV set to 511
{0x15, 0x69, 0x01},														// scaling up x2
{0x15, 0x6B, 0x00},														// MIPI 1 port
{0x39, 0x05, 0x00, {0x80, 0x01, 0x40, 0x40, 0x11}},						// Resolution = 1280x1280
{0x39, 0x08, 0x00, {0x81, 0x02, 0x56, 0x00, 0x08, 0x00, 0x10, 0x00}},	// VBP + VSW, VFP
{0x39, 0x08, 0x00, {0x82, 0x02, 0x56, 0x00, 0x08, 0x00, 0x10, 0x00}},	// IDLE setting
{0x15, 0x35, 0x00},														// TE mode1
{0x15, 0x25, 0x01},														// TC enable

/* CMD2 P1 */
{0x39, 0x03, 0x00, {0xF0, 0xAA, 0x11}},
{0x39, 0x04, 0x00, {0xC0, 0x00, 0x04, 0x00}},
{0x39, 0x0A, 0x00, {0xC2, 0x00, 0xCC, 0x00, 0xCC, 0x00, 0xCC, 0x00, 0x90, 0x82}},	// Brightess
{0x39, 0x0A, 0x00, {0xC2, 0x03, 0xFF, 0x03, 0xFF, 0x03, 0xFF, 0x00, 0x90, 0x82}},	// Brightess

{0x39, 0x03, 0x00, {0xF0, 0xAA, 0x12}},
{0x15, 0xD3, 0x20},
{0x39, 0x03, 0x00, {0xBF, 0x37, 0xBE}},									// Thermal Detect
//{0x15, 0xB1, 0x02},														// MIPI 2 LANE

/* CMD3 P1 */
{0x39, 0x03, 0x00, {0xFF, 0x5A, 0x81}},									// TC
{0x15, 0x65, 0x0B},
{0x39, 0x0E, 0x00, {0xF9, 0x58, 0x5F, 0x66, 0x6D, 0x74, 0x7B, 0x82, 0x89, 0x90, 0x97, 0x9E, 0xA5, 0xAC}},
{0x05, 0x11, 0x00}														// sleep out
{0x05, 0x29, 0x00}														// display on
{0x39, 0x03, 0x00, {0xF0, 0xAA, 0x11}},
{0x15, 0xC0, 0xFF},
```

#### 2.5.2 add porch
Add porch, resolution and pixels clk, config format, mode, polarity and so on.
```c
	.name = "seeya_oled",
	.xres = 1280,
	.yres = 1280,

	.refresh = 60,
	.pixclock = KHZ2PICOS(110162),

	.left_margin = 32,//hbp
	.right_margin = 64,//hfp
	.hsync_len = 32, //hsync

	.upper_margin = 6,//vbp
	.lower_margin = 16,//vfp
	.vsync_len = 2, //vsync

	.sync                   = FB_SYNC_VERT_HIGH_ACT,
	.vmode                  = FB_VMODE_INTERLACED,
	.flag = 0,


	.color_even = TFT_LCD_COLOR_EVEN_RGB,
	.color_odd = TFT_LCD_COLOR_ODD_RGB,
	.mode = TFT_LCD_MODE_PARALLEL_888,

	/* busrt mode */
	.video_config.video_mode = VIDEO_BURST_WITH_SYNC_PULSES,
```

#### 2.5.3 adjust mipi clk
Location: jz_mipi_dsi.c in fb_v12.
Advice from ingenic:
```txt
1. mipi clk怎么给：
MIPI_CLK_LANE= [ (HBP+Hactive+HFP+Hsync) x (VBP+Vactive+VFP+Vsync) ] x(bus_hpw) x fps/ (lane_num)/2
我们没办法直接设置mipi clk，是PLL_Output_Frequency给的
PLL_Output_Frequency =  12 / reg03  x reg04
要保证：PLL_Output_Frequency  > 4 * MIPI_CLK_LANE
2. ratio_clock_XPF对porch整体的微调
```

#### 2.5.4 colorbar test
if colorbar test is ok, panel moudle config done.

## 3. frame buffer 
Character device /dev/fb0 created by fbmem.c
Using system call to operate graphic device fb0 in user space.

### 3.1 init procedure
+ open: open device with wirte and read
+ iotcl: get info by FBIOGET_VSCREENINFO and FBIOGET_FSCREENINFO, catch size, resolution and etc.
+ mmap: mmap framebuffer with write and read

### 3.2 display procedure
+ pthread-1: snap data frame from ISP to memory, one more frame for cache
+ pthread-2: copy data frame from memory to framebuffer, one more frame for cache
+ pthread-3: set which frame to display, cache 1 or cache 2 by ioctl
sync among treads use msg queue and pthread_mutex_t in user space.


## 4. chrdev of lcd
Control lcd display on and off, brightness and etc. Development of character device driver.[^1]

### 4.1 init config
+ alloc_chrdev_region: allocate character device dynamicly, register device into /proc/devices
+ cdev_init: init character device by node number MAJOR and MINOR
+ cdev_add: add chararcter
+ class_create: create device class
+ device_create: create device under /dev


### 4.2 stuct file_operations
Complete struct file_opearions for system call, using ioctl to control brightness, display on/off,
and so on.Using struct mutex for locking and copy_from_user/copy_to_user for exchange data in kernel
space.

## Reference
[^1]: [Char Drivers](https://www.oreilly.com/library/view/linux-device-drivers/0596005903/ch03.html)
