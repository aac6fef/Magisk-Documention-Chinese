# Android 启动过程

## 一些术语

- rootdir 根目录 : 所有的文件/文件夹/文件系统都存储或者挂载在根目录(/)。这个文件系统可能是 **rootfs** 或者是 **system** 分区。
- initramfs : 具体的介绍见上文，此处介绍 Android 的 **initramfs**. 这玩意实际上是 Android 启动映像的一个分区，Linux 将它用作 **rootfs**。人们通常用 **ramdisk** 这个术语来称呼 **initramfs**
- **recovery** 和 **boot** 分区：这两个分区实际上非常相像，他们都是包含 **ramdisk** 和 Linux Kernel（以及一些其他的奇妙东西）的 Android 启动映像。他们唯一的区别就是在使用 **boot** 分区启动时，手机会启动到正常的 Android 系统，但是在使用 **recoevry** 分区启动时，会启动一个用于修复系统和升级设备的最小 Linux 环境。
- SAR : [ System-as-root ](https://source.android.com/devices/bootloader/partitions/system-as-root) 的简称。顾名思义，SAR 意味着用 **system** 代替 **rootfs**
- A/B 分区和 A-only 分区： 支持[无缝系统更新](https://source.android.com/devices/tech/ota/ab)的设备上会有两个槽位的只读分区，我们把这些设备称为支持 A/B 分区的设备。而不支持[无缝系统更新](https://source.android.com/devices/tech/ota/ab)（或者说只有一个槽位的只读分区）的设备，被称作使用 A-only 分区的设备。
  2SI：两阶段初始化。是 Android 10 及以上的设备使用的启动方式。下文会详细说明。

 还有一些能更加精确地定义设备上 Android 版本的方式：
- LV：Launch Version，出厂版本。即在设备上市时设备预安装的版本。
  <br>译者注：Launch 在此处的意思是上市、发行（to make a peoduct avalible to the public for the first time)，在原文中有解释。
- RV：Running Version，运行版本。即设备正在运行的 Android 版本。
  人们常常使用 Android API 版本来定义 LV 和 RV。Android 版本，代号和 API Verison 见下表：

| Android Version | API Version |      Codename      |
| :-------------: | :---------: | :----------------: |
|       1.0       |      1      |         /          |
|       1.1       |      2      |         /          |
|       1.5       |      3      |      Cupcake       |
|       1.6       |      4      |       Donut        |
|       2.0       |      5      |       Eclair       |
|      2.0.1      |      6      |       Eclair       |
|       2.1       |      7      |       Eclair       |
|       2.2       |      8      |       Froyo        |
|       2.3       |      9      |    Gingerbread     |
|      2.3.3      |     10      |    Gingerbread     |
|       3.0       |     11      |     Honeycomb      |
|       3.1       |     12      |     Honeycomb      |
|       3.2       |     13      |     Honeycomb      |
|       4.0       |     14      | Ice Cream Sandwich |
|      4.0.3      |     15      | Ice Cream Sandwich |
|       4.1       |     16      |     Jelly Bean     |
|       4.2       |     17      |     Jelly Bean     |
|       4.3       |     18      |     Jelly Bean     |
|       4.4       |     19      |       KitKat       |
|      4.4W       |     20      |         /          |
|       5.0       |     21      |      Lollipop      |
|       5.1       |     22      |      Lollipop      |
|       6.0       |     23      |    Marshmallow     |
|       7.0       |     24      |       Nougat       |
|       7.1       |     25      |       Nougat       |
|       8.0       |     26      |        Oreo        |
|       8.1       |     27      |        Oreo        |
|        9        |     28      |        Pie         |
|       10        |     29      |         Q          |
|       11        |     30      |         R          |
|       12        |     31      |         S          |
|       12L       |     32      |     Android12L     |

注：在 Android 10 之后 Google 取消了 Android 版本代号。
 ### 启动方法
 Android 的启动方式可以大致被分为三个主要方法。以下是简单的判断你的设备的启动方式的方法

 |方法|初始化根目录|最终根目录|
 |:-:|:-:|:-:|
 |A|rootfs|rootfs|
 |B|system|system|
 |C|rootfs|system|

- 方法A——旧式 ramdisk：这是过去所有 Android 设备的启动方式(作者感叹：「过去的好日子啊（大雾」)，设备上的内核将 **initramfs** 作为根目录，然后通过执行 **/init** 来启动。
- 方法B——旧式SAR：这种启动类型初由 pixel 1 采用。内核直接将 **system** 分区挂载为根目录，然后通过执行 **/init** 来启动。
  - LV = 28 的设备
  - Google : Pixel 1 和 Pixel 2. Pixel 3 和 Pixel 3a (只有当RV=28才是如此）
  - 一加：6-7
  - 一些RV<29 的 Android Go 设备？
- 方法C——2SI ramdisk SAR：这种启动类型最初出现在 Pixel 3 上的 Android 10 DP1（注：DP = Developer Preview）。系统的内核使用 initramfs 作为根目录，然后执行根目录中的 /init。 This `init` is responsible to mount the `system` partition and use it as the new rootdir, then finally exec `/system/bin/init` to boot.
	- Devices with `(LV >= 29)`
	- Devices with `(LV < 28, RV >= 29)`, excluding those that were already using Method B
	- Google: Pixel 3 and 3a with `(RV >= 29)`

### Discussion

From documents online, Google's definition of SAR only considers how the kernel boots the device (**Initial rootdir** in the table above), meaning that only devices using **Method B** is *officially* considered an SAR device from Google's standpoint.

However for Magisk, the real difference lies in what the device ends up using when fully booted (**Final rootdir** in the table above), meaning that **as far as Magisk's concern, both Method B and C is a form of SAR**, but just implemented differently. Every instance of SAR later mentioned in this document will refer to **Magisk's definition** unless specifically says otherwise.

The criteria for Method C is a little complicated, in layman's words: either your device is modern enough to launch with Android 10+, or you are running an Android 10+ custom ROM on a device that was using Method A.

- Any Method A device running Android 10+ will automatically be using Method C
- **Method B devices are stuck with Method B**, with the only exception being Pixel 3 and 3a, which Google retrofitted the device to adapt the new method.

SAR is a very important part of [Project Treble](https://source.android.com/devices/architecture#hidl) as rootdir should be tied to the platform. This is also the reason why Method B and C comes with `(LV >= ver)` criterion as Google has enforced all OEMs to comply with updated requirements every year.

## Some History

When Google released the first generation Pixel, it also introduced [A/B (Seamless) System Updates](https://source.android.com/devices/tech/ota/ab). Due to [storage size concerns](https://source.android.com/devices/tech/ota/ab/ab_faqs), there are several differences compared to A-only, the most relevant one being the removal of `recovery` partition and the recovery ramdisk being merged into `boot`.

Let's go back in time when Google is first designing A/B. If using SAR (only Boot Method B exists at that time), the kernel doesn't need `initramfs` to boot Android (because rootdir is in `system`). This mean we can be smart and just stuff the recovery ramdisk (containing the minimalist Linux environment) into `boot`, remove `recovery`, and let the kernel pick whichever rootdir to use (ramdisk or `system`) based on information from the bootloader.

As time passed from Android 7.1 to Android 10, Google introduced [Dynamic Partitions](https://source.android.com/devices/tech/ota/dynamic_partitions/implement). This is bad news for SAR, because the Linux kernel cannot directly understand this new partition format, thus unable to directly mount `system` as rootdir. This is when they came up with Boot Method C: always boot into `initramfs`, and let userspace handle the rest of booting. This includes deciding whether to boot into Android or recovery, or as they officially call: `USES_RECOVERY_AS_BOOT`.

Some modern devices using A/B with 2SI also comes with `recovery_a/_b` partitions. This is officially supported with Google's standard. These devices will then only use the boot ramdisk to boot into Android as recovery is stored on a separate partition.

## Piecing Things Together

With all the knowledge above, now we can categorize all Android devices into these different types:

Type | Boot Method | Partition | 2SI | Ramdisk in `boot`
:---: | :---: | :---: | :---: | :---:
**I** | A | A-only | No | `boot` ramdisk
**II** | B | A/B | Any | `recovery` ramdisk
**III** | B | A-only | Any | ***N/A***
**IV** | C | Any | Yes | Hybrid ramdisk

These types are ordered chronologically by the time they were first available.

- **Type I**: Good old legacy ramdisk boot
- **Type II**: Legacy A/B devices. Pixel 1 is the first device of this type, being both the first A/B and SAR device
- **Type III**: Late 2018 - 2019 devices that are A-only. **The worst type of device to ever exist as far as Magisk is concerned.**
- **Type IV**: All devices using Boot Method C are Type IV. A/B Type IV ramdisk can boot into either Android or recovery based on info from bootloader; A-only Type IV ramdisk can only boot into Android.

Further details on Type III devices: Magisk is always installed in the ramdisk of a boot image. For all other device types, because their `boot` partition have ramdisk included, Magisk can be easily installed by patching boot image through the Magisk app or flash zip in custom recovery. However for Type III devices, they are **limited to install Magisk into the `recovery` partition**. Magisk will not function when booted normally; instead Type III device owners have to always reboot to recovery to maintain Magisk access.

Some Type III devices' bootloader will still accept and provide `initramfs` that was manually added to the `boot` image to the kernel (e.g. some Xiaomi phones), but many device don't (e.g. Samsung S10, Note 10). It solely depends on how the OEM implements its bootloader.
