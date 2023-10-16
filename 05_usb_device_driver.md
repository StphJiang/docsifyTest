# USB bus device driver
[toc]

## 1. Introduction
The universal serial bus(USB) is a connection between a host computer and a number of peripheral devices. It was originally created to replace a wide range of slow and different buses.

## 2. gadget history
### 2.1 gadget framework
	< David Brownell in early 2003. >
+ the term "gadget"
+ supports only monolithic gadget drivers
+ g_zero and g_ether
+ a few usb device controller drivers

### 2.2 gadgetfs
	< in late 2003 >
+ enables userspace gadget drivers
+ MTP/PTP is a common use case (MTP/PTP: Media/Picture Transfer Protocol)

### 2.3 composite framework
	< added in 2008 >
+ enables multi-function (or USB composite) gadget drivers
+ existing gadget drivers slowly moved over to compositable function implementations

### 2.4 functionfs
	< added in 2010 >
+ compositable version of gadgetfs
+ now usersapce gadget functions can be combined with kernel gadget functions in a composite gadget
+ e.g. mass storage(kernel) + MTP(via FunctionFS)

### 2.5 configfs
	< arrives in 3.11 in 2013 >
+ A userspace API for creation of arbitrary usb composite devices using reusable kernel gadget function drivers
+ Supports all major existing gadget functions except FunctionFs and mass storage in 3.11
+ added conversion of FunctionFS and mass storage in 3.13
+ Usage of configfs[^2]

### 2.6 Review of these fileystems
+ GadgetFS: original monolithic kernel driver that provides an interface to implement usersapce gadget drivers
+ FunctionFs: rewrite of GadgetFS to support userspace gadget funcions that can be combined into a USB composite gadget
+ USB Gadget ConfigFS: interface that allows definition of arbitrary and configurations to define an application specific USB composite device from userspace

## 2. Device info

## 3. Device Tree

## Reference
[^1]:[The USB composite framework](lwn.net/Articles/395712)
[^2]:[Linux USB gadget configred through configfs](https://www.kernel.org/doc/Documentation/usb/gadget_configfs.txt)
