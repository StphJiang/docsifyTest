# RTC device driver
[toc]

## 1. Introduction
RTC device need driver for IO, then could be used like device file /dev/rtc0.
There are two commands connect with RTC, 'date' and 'hwclock', former display
the current time or set the system date, reset after system power off, latter
is an administration tool for the time clocks, it can display hradware clock
time by RTC device driver.


## 2. Device info
Specification	: INS5699S
Vendor			: DAPU
Bus				: I2C
Function		: RTC, Alarm, Counter

## 3. Device Tree
Device mount on I2C channel 1, with address 0x32, compatible attr for match.

```dts
&i2c1 {
	pinctrl-0 = <&i2c1_pd>;
	pinctrl-names = "default";
	status = "okay";

    /* real time clock device ins5699s */
    ins5699s: ins5699s@32{
        compatible = "infiray,rtc";
        reg = <0x32>;
    };
};
```

## 4. RTC relation
```sh
/drivers/rtc/rtc-lib.c
# support functions of convertion in rtc, date and time

/drivers/rtc/hctosys.c
# catch rtc data when system start up

/drivers/rtc/systohc.c

/drivers/rtc/class.c
# register a RTC class in kernel, support register/unregister interface for device, and rtc-PM

/drivers/rtc/interface.c
# support rtc releated api to application and driver

/drivers/rtc/rtc-dev.c
# abstract rtc device as a chrdev, and support file_operation

/drivers/rtc/rtc-proc.c
# acquire rtc data from proc file system

/drivers/rtc/rtc-sysfs.c
# operate rtc device by sysfs 

/drivers/rtc/ins5699.c
# lowest rtc device driver
```

## 4. I2c device driver
### 4.1 difference between module_init(), module_platform_driver() and module_i2c_driver()
all this 

### 4.2 I2C framework
```
=======================================================================
                | i2c-chrdev                 i2c-bus
user space      | 	 |		                    |
                | /dev/device-name           /dev/i2c-0
---------------------|--------------------------|-----------------------
                |    |      I2c-Core(I2c Bus)   |
kernel space    | I2c-Dev   --.---|______       |
                |    |       /            \     |
                | I2c-Driver -------------- I2c-Adapter        
------------------------------------------------|-----------------------
                |                           I2c-Controller
				|      _________________________|______
hardware        |     |                  |             |
                | i2c device-0        device-1        ...
=======================================================================
```

### 4.3 I2c Bus Driver
Add "i2c device interface" in menuconfig, create character device /dev/i2c-N for user space.
Then i2c-tools could be ported for testing.

### 4.4 I2c Device Driver for RTC
+ struct i2c_driver
+ module_i2c_driver
+ of_device_id.compatible
+ devm_rtc_device_register -- struct rtc_class_ops

## Reference
[^1]:[Char Drivers](https://www.oreilly.com/library/view/linux-device-drivers/0596005903/ch03.html)
[^2]:[device tree overview](https://docs.kernel.org/devicetree/usage-model.html)
[^3]:[linux driver dev 15 - RTC driver model](https://blog.csdn.net/wangdapao12138/article/details/82120773)
[^4]:[IIC bus driver](https://www.dgrt.cn/a/2185462.html\?action\=onClick)
