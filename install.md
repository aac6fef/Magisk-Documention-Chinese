# 安装

如果你已经在你的设备中安装了 Magisk 本身, **非常推荐**直接使用 Magisk 应用中的 “直接安装”。以下的说明仅仅适用于初始化安装。

## 安装前的一些准备

在开始前:

- 这篇指南假设你会使用 `adb` 和 `fastboot`。
- 如果你打算安装自定义内核，请在 Magisk 安装完成之后再安装。
- 你需要解锁你的设备的 bootloader。具体信息请访问你的设备提供商的官网。

---

下载并安装最新的  [Magisk app](https://github.com/topjohnwu/Magisk/releases/latest).
<br>译者注：如果你发现你无法访问 github 或者下载速度很慢，请考虑从本地的镜像站下载 Magisk。[清华镜像站](https://mirrors.tuna.tsinghua.edu.cn/github-release/topjohnwu/Magisk/LatestRelease/)
<br>在软件主页，你应该看到如下界面:

<p align="center"><img src="images/device_info.png" width="500"/></p>

Ramdisk 的结果代表着你的设备的启动分区是否具有 ramdisk。如果你的设备没有 ramdisk ，请在开始前阅读 [Magisk in Recovery](#magisk-in-recovery) 
> _(不幸的是，有些设备的 bootloader 接受 ramdisk，但检测出来却是不支持 ramdisk. 如果你的设备是这样的, 请遵循 **拥有**  ramdisk 的设备的安装指南。目前没有办法检测到这种情况，所以唯一的方法就是尝试。 幸运的是, 我们目前得知只有小米的设备有可能会出现这种情况, 所以大部分人可以忽略这条信息.)_

如果你在使用三星的设备，你可以跳转到 [三星设备安装指南](#samsung-system-as-root).

如果你的设备支持 ramdisk ，请去获取你的设备的 `boot.img`.<br>
如果你的设备不支持 ramdisk ，请去获取你的设备的 `recovery.img`.<br>
请从官方固件包或者第三方 ROM 中获得你说需要的文件.

现在，我们需要确认你的设备是否有专门的  `vbmeta` 分区.

- 如果你的官方固件中包含 `vbmeta.img`, 这意味着你的设备有专门的 `vbmeta` 分区
- 你也可以把你的手机了连接到电脑，通过以下命令来验证你的设备是否有单独的 `vbmeta` 分区:<br>
  `adb shell ls -l /dev/block/by-name`
- 如果你在执行上文的指令后，在输出中发现了 `vbmeta` 或 `vbmeta_a`、 `vbmeta_b`,这代表这你的设备具有单独的 `vbmeta`分区。
- 否则，你的设备没有专门的 `vbmeta` 分区。

因此，在现在，你应该知道并且准备了：

1. 你的设备是否支持 `ramdisk`
2. 你的设备是否有单独的 `vbmeta` 分区
3. 按照前文准备 `boot.img` 或 `recovery.img` 

让我们继续来 [修改镜像](#修改镜像).
>如果你手机中的数据对你很重要，请提前备份数据
## 修改镜像

- 把你的 boot/recovery 镜像复制到你的设备中。
- 在你的设备中打开 Magisk ，按下 Magisk 卡片中的 **安装**。
- 如果你要修改 `recovery.img`, 选中 **"恢复模式"** 选项
- 如果你的设备 **没有** 单独的 `vbmeta` 分区, 选中 **"修改boot映像中的的vbmeta"** 选项。
- 选中 **"选择并修复镜像"** , 然后在内部存储中选择 boot/recovery 映像
- 开始安装，在安装完成后，将修改后的镜像复制到电脑中:<br>
  `adb pull /sdcard/Download/magisk_patched_[random_strings].img`
- 将 boot/recovery 镜像刷入你的设备.<br>
  对于大部分设备，你可以通过一下命令来刷入修改后的镜像:<br>
  `fastboot flash boot /path/to/magisk_patched.img` 或者 <br>
  `fastboot flash recovery /path/to/magisk_patched.img`
- (可选的) 如果你的设备有专门的 `vbmeta` 分区, 你可以通过一下命令刷入修改后的 `vbmeta` 分区:<br>
  `fastboot flash vbmeta --disable-verity --disable-verification vbmeta.img`
- 重启，然后享受吧!
> 注:如果你发现你的设备在刷入后无法启动，请将没有被修改的镜像刷入你的设备（将上文的命令中的修改后的镜像改为没有修改的镜像即可
## 卸载

最简单的卸载方法是在 Magisk 应用中直接卸载. 如果你在使用第三方 recovery , 把 Magisk APK 重命名为 `uninstall.zip` 然后以和其他刷机包一样的方法刷入.

## Recovery 中的 Magisk

如果你的设备的 boot 分区中没有 ramdisk, Magisk 只能劫持  recovery 分区. 对于这些设备, 你需要在每次你想要使用 Magisk 的时候 **重启到 recovery** .

When Magisk hijacks the recovery, there is a special mechanism to allow you to _actually_ boot into recovery mode. Each device model has its own key combo to boot into recovery, as an example for Galaxy S10 it is (Power + Bixby + Volume Up). A quick search online should easily get you this info. As soon as you press the key combo and the device vibrates with a splash screen, release all buttons to boot into Magisk. If you decide to boot into the actual recovery mode, **long press volume up until you see the recovery screen**.

As a summary, after installing Magisk in recovery **(starting from power off)**:

- **(Power up normally) → (System with NO Magisk)**
- **(Recovery Key Combo) → (Splash screen) → (Release all buttons) → (System with Magisk)**
- **(Recovery Key Combo) → (Splash screen) → (Long press volume up) → (Recovery Mode)**

(注释: 你 **不能** 在这种情况下用第三方 recovry 来安装 Magisk !!)

## Samsung (System-as-root)

> If your Samsung device is NOT launched with Android 9.0 or higher, you are reading the wrong section.

### Before Installing Magisk

- Installing Magisk **WILL** trip KNOX
- Installing Magisk for the first time **REQUIRES** a full data wipe (this is **NOT** counting the data wipe when unlocking bootloader). Backup your data before continue.
- Download Odin (only runs on Windows) that supports your device.

### Unlocking Bootloader

Unlocking the bootloader on modern Samsung devices have some caveats. The newly introduced `VaultKeeper` service will make the bootloader reject any unofficial partitions in some circumstances.

- Allow bootloader unlocking in **Developer options → OEM unlocking**
- Reboot to download mode: power off your device and press the download mode key combo for your device
- Long press volume up to unlock the bootloader. **This will wipe your data and automatically reboot.**
- Go through the initial setup. Skip through all the steps since data will be wiped again in later steps. **Connect the device to Internet during the setup.**
- Enable developer options, and **confirm that the OEM unlocking option exists and is grayed out.** This means the `VaultKeeper` service has unleashed the bootloader.
- Your bootloader now accepts unofficial images in download mode

### Instructions

- Use either [samfirm.js](https://github.com/jesec/samfirm.js), [Frija](https://forum.xda-developers.com/s10-plus/how-to/tool-frija-samsung-firmware-downloader-t3910594), or [Samloader](https://forum.xda-developers.com/s10-plus/how-to/tool-samloader-samfirm-frija-replacement-t4105929) to download the latest firmware zip of your device directly from Samsung servers.
- Unzip the firmware and copy the `AP` tar file to your device. It is normally named as `AP_[device_model_sw_ver].tar.md5`
- Press the **Install** button in the Magisk card
- If your device does **NOT** have boot ramdisk, check the **"Recovery Mode"** option
- Choose **"Select and Patch a File"** in method, and select the `AP` tar file
- Start the installation, and copy the patched tar file to your PC using ADB:<br>
  `adb pull /sdcard/Download/magisk_patched_[random_strings].tar`<br>
  **DO NOT USE MTP** as it is known to corrupt large files.
- Reboot to download mode. Open Odin on your PC, and flash `magisk_patched.tar` as `AP`, together with `BL`, `CP`, and `CSC` (**NOT** `HOME_CSC` because we want to **wipe data**) from the original firmware.
- Your device should reboot automatically once Odin finished flashing. Agree to do a factory reset if asked.
- If your device does **NOT** have boot ramdisk, reboot to recovery now to enable Magisk (reason stated in [Magisk in Recovery](#magisk-in-recovery)).
- Install the Magisk app you've already downloaded and launch the app. It should show a dialog asking for additional setup.
- Let the app do its job and automatically reboot the device. Voila!

### Upgrading the OS

Once you have rooted your Samsung device, you can no longer upgrade your Android OS through OTA. To upgrade your device's OS, you have to manually download the new firmware zip file and go through the same `AP` patching process written in the previous section. **The only difference here is in the Odin flashing step: do NOT use the `CSC` tar, but instead use the `HOME_CSC` tar as we are performing an upgrade, not the initial install**.

### Important Notes

- **Never, ever** try to restore either `boot`, `recovery`, or `vbmeta` partitions back to stock! You can brick your device by doing so, and the only way to recover from this is to do a full Odin restore with data wipe.
- To upgrade your device with a new firmware, **NEVER** directly use the stock `AP` tar file with reasons mentioned above. **Always** patch `AP` in the Magisk app and use that instead.
- Never just flash only `AP`, or else Odin may shrink your `/data` filesystem size. Flash `AP` + `BL` + `CP` + `HOME_CSC` when upgrading.

## Custom Recovery

> **This installation method is deprecated and is maintained with minimum effort. YOU HAVE BEEN WARNED!**

Installing using custom recoveries is only possible if your device has boot ramdisk. Installing Magisk through custom recoveries on modern devices is no longer recommended. If you face any issues, please use the proper [Patch Image](#patching-images) method.

- Download the Magisk APK
- Rename the `.apk` file extension to `.zip`, for example: `Magisk-v24.0.apk` → `Magisk-v24.0.zip`. If you have trouble renaming the file extension (like on Windows), use a file manager on Android or the one included in TWRP to rename the file.
- Flash the zip just like any other ordinary flashable zip.
- Reboot and check whether the Magisk app is installed. If it isn't installed automatically, manually install the APK.

> Warning: the `sepolicy.rule` file of modules may be stored in the `cache` partition. DO NOT WIPE THE `CACHE` PARTITION.
