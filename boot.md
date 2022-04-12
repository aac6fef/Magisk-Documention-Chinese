# 我爱搞基，搞基make me happy

<!-- vim-markdown-toc GFM -->

* [Android 启动过程](#android-启动过程)
	* [一些术语](#一些术语)
		* [一些讨论](#一些讨论)
	* [一些历史](#一些历史)
	* [Piecing Things Together](#piecing-things-together)

<!-- vim-markdown-toc -->
# Android 启动过程

## 一些术语

- rootdir 根目录 : 所有的文件/文件夹/文件系统都存储或者挂载在根目录(/)。这个文件系统可能是 **rootfs** 或者是 **system** 分区。
- initramfs : 具体的介绍见上文，此处介绍 Android 的 **initramfs**. 这玩意实际上是 Android 启动映像的一个分区，Linux 将它用作 **rootfs**。人们通常用 **ramdisk** 这个术语来称呼 **initramfs**
- **recovery** 和 **boot** 分区：这两个分区实际上非常相像，他们都是包含 **ramdisk** 和 Linux Kernel（以及一些其他的奇妙东西）的 Android 启动映像。他们唯一的区别就是在使用 **boot** 分区启动时，手机会启动到正常的 Android 系统，但是在使用 **recoevry** 分区启动时，会启动一个用于修复系统和升级设备的最小 Linux 环境。
- SAR : [ System-as-root ](https://source.android.com/devices/bootloader/partitions/system-as-root) 的简称。顾名思义，SAR 意味着用 **system** 代替 **rootfs**
- A/B 分区和 A-only 分区： 支持[无缝系统更新](https://source.android.com/devices/tech/ota/ab)的设备上会有两个槽位的只读分区，我们把这些设备称为支持 A/B 分区的设备。而不支持[无缝系统更新](https://source.android.com/devices/tech/ota/ab)（或者说只有一个槽位的只读分区）的设备，被称作使用 A-only 分区的设备。
  2SI：两阶段初始化。是 Android 10 及以上的设备使用的启动方式。下文会详细说明。

 还有一些能更加精确地定义设备上 Android 版本的方式：
- LV：Launch Version，出厂版本。即在设备上市时设备预安装的版本.
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
  - 一些 LV<29 的 Android Go 设备？
- 方法C——2SI ramdisk SAR：这种启动类型最初出现在 Pixel 3 上的 Android 10 DP1（注：DP = Developer Preview）。系统的内核使用 initramfs 作为根目录，然后执行根目录中的 /init。 这个 init 负责挂载 system 分区为新的根目录，最终执行 /system/bin/init 来启动。
	- `(LV >= 29)`的设备
	- 除了使用方法B的 `(LV < 28, RV >= 29)`, 的设备
	- Google:  `(RV >= 29)` 的 Pixel 3 和 3a
### 一些讨论

从 Google 的官方文档看 , Google 对 SAR 的定义只考虑了内核如何启动设备（上表中的 Initial rootdir），也就是说，从 Google 的角度来看，只有使用方法 B 的设备才是 Google 官方认为的 SAR 设备.

然而,对于 Magisk 来说，真正的区别在于设备在完全启动时最终使用的根目录是什么（上表中的 Final rootdir），也就是说，就 Magisk 而言，方法 B 和方法 C 都是 SAR 的一种形式，只是实现方式不同。本文档后面提到的每一个 SAR 例子都会参考 Magisk 的定义，除非另有特别说明。

方法C有一点复杂, 从外行的角度来说 : 要么你的设备足够新，出厂 Android 版本是 10 或者更新的版本 , 或者你的设备过去使用方法a,而现在在运行 Android 10+ .

- 运行 Android 10+ 的任何使用方法 A 的设备将自动使用方法 C
- **使用方法B的设备将会停留在方法B**, 唯一的例外是 Pixel 3 和 3a，谷歌对设备进行了改造以适应新方法。

SAR 是 [Project Treble](https://source.android.com/devices/architecture#hidl) 的非常重要的一个部分。因为 rootdir 应该与平台绑定，且谷歌强制所有 OEM 厂商遵守每年更新的要求。这也是为什么方法 B 和 C 带有 ` LV >= API`级别标准的原因。

## 一些历史

谷歌在发布第一代 Pixel 时，还推出了 [A/B (无缝) 系统更新](https://source.android.com/devices/tech/ota/ab). 由于对 [存储大小](https://source.android.com/devices/tech/ota/ab/ab_faqs)的考虑,与 A-only 相比有几个不同之处，最相关的一个是删除`recovery`分区和将恢复 `ramdisk` 合并到 `boot`.

让我们回到 Google 第一次设计 A/B 的时候. 如果使用 SAR (当时只存在启动方法B), 内核不需要使用  `initramfs` 来启动  Android (因为根目录是 `system` ）这意味着我们可以采用一种聪明的做法，就是我们可以将恢复 `ramdisk`  (包含最小 Linux 环境 ) 放入 `boot` 中, 并且移除 `recovery`, 然后让内核根据 bootloader 提供的信息，决定每次启动的根目录  (ramdisk 或 `system`) 。

随着时间的推移， Android 版本从 7.1 逐渐更新到 10 , Google 引入了 [动态分区](https://source.android.com/devices/tech/ota/dynamic_partitions/implement)。 这对 SAR 来说是个坏消息 , 因为  Linux 内核不能直接理解新的分区格式 , 这导致了设备无法直接将 `system` 挂载为根目录 . 他们提出了方法 C: 先将 `initramfs `作为根目录，然后让 userspace 处理剩下的启动步骤，包括决定要启动到 Android 或 Recovery，Google 官方称之为 `USES_RECOVERY_AS_BOOT`。

一些使用 A/B 和 2SI 的现代设备带有 recovery_a/_b 分区，这是 Google 官方支持的标准。这些设备将只使用 boot ramdisk 启动到 Android，因为 recovery 存储在一个单独的分区。

有了以上的信息，现在我们可以把所有的 Android 设备分为这几种不同的类型。 
种类 | 启动方式 | 分区类型 | 是否使用2SI |  `boot`中的 ramdisk
:---: | :---: | :---: | :---: | :---:
**I** | A | A-only | 否 | `boot` ramdisk
**II** | B | A/B | 两者均可 | `recovery` ramdisk
**III** | B | A-only | 两者均可 | ***N/A***
**IV** | C | Any | 是 | 混合 ramdisk

这些类型按时间顺序排列，以它们首次出现的时间为准。

- **种类 I**: legacy ramdisk 启动 
- **种类 II**: 老的 A/B 设备，Pixel 1 是第一款此类型设备，是第一款使用了 A/B 和 SAR 设备。 
- **种类 III**: 2018年末-2019年末的设备大多都是 A-Only，** 就 Magisk 而言，是有史以来最糟的一种设备 **
- **种类 IV**: 所有使用方法 C 的设备都属于类型四，使用 A/b 分区的类型四设备的 ramdisk 可以根据 Bootloader 的信息启动到 Android 或 Recovery，A-Only 的类型四设备的 ramdisk 只能启动到 Android

关于第三类设备的进一步细节。Magisk总是安装在启动镜像的ramdisk中。对于所有其他类型的设备，因为它们的 `启动` 分区包含了RAMDisk，Magisk可以很容易地通过Magisk应用程序修补启动镜像或在第三方 recovery 的 flash zip 来安装。但是，对于采取第三类启动方式的设备，他们会限制将Magisk安装到 `recovery` 分区。Magisk在正常启动时不会发挥作用；相反，第三类设备的所有者必须始终重新启动到`recovery`分区，以保持Magisk的访问。

一些采取第三类启动方式的设备的引导程序仍然会接受并提供手动添加到 `boot`镜像中的 `initramfs`到内核中（例如一些小米手机），但许多设备不接受（例如三星S10、Note 10）。这完全取决于OEM如何实现它的bootloader。
