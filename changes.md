# Magisk的更新日志

### v24.3

- [General] 停止使用`getrandom`系统调用.
- [Zygisk] 更新API到v3，为`AppSpecializeArgs`添加新字段。
- [App] 改进app重新打包的安装工作流程

### v24.2

- [MagiskSU] 修复缓冲区溢出
- [MagiskSU] 修正所有者管理的多用户超级用户设置
- [MagiskSU] 修正使用`su -c <cmd>`时的命令记录。
- [MagiskSU] 防止su请求无限期阻塞
- [MagiskBoot] 用一些魔法来支持 "lz4_legacy "归档文件
- [MagiskBoot] 修正`lz4_lg`压缩法
- [DenyList] 允许瞄准以系统UID身份运行的进程
- [Zygisk] 解决三星的 "早期zygote"。
- [Zygisk] 改进Zygisk加载机制
- [Zygisk]修复app程序UID跟踪
- [Zygisk] 修复在zygote中设置的不正确的`umask`。
- [App]修复BusyBox执行测试
- [app] 改进代理加载机制
- [app] 主要的app升级流程改进
- [通用] 改进命令行错误处理和信息传递

### v24.1

- [app] 稳定性改进

### v24.0

- [常规] MagiskHide从Magisk中移除
- [通用] 支持Android 12
- [通用] 支持纯64位的的设备
- [一般] 更新BusyBox到1.34.1
- [Zygisk] 引入新功能。Zygisk
- [Zygisk] 引入DenyList功能，在用户选择的进程中恢复Magisk功能。
- [MagiskBoot] 支持修补32位内核zImages
- [MagiskBoot] 支持引导图像头V4
- [MagiskBoot] 支持从dtb bootargs中打出`skip_initramfs`的补丁。
- [MagiskBoot] 增加新的环境变量`PATCHVBMETAFLAG`，以配置决定是否要修补vbmeta标志。
- [MagiskInit] 支持从`/system/etc`加载fstab（Pixel 6需要）。
- [MagiskInit] 支持`/proc/bootconfig`来加载启动配置。
- [MagiskInit] 更好的支持某些Meizu设备
- [MagiskInit] 更好地支持一些OnePlus/Oppo/Realme设备
- [MagiskInit] 在一些索尼设备上支持`init.real`。
- [MagiskInit] 检测到DSU时跳过加载Magisk
- [MagiskPolicy] 从system_ext加载`*_compat_cil_file`。
- [MagiskSU] 如果内核支持，就使用隔离的devpts
- [MagiskSU] 如果设置了孤立的挂载命名空间，则修复root shell。
- [resetprop] 删除的属性现在会从内存中抹去，而不是仅仅取消链接。
- [App] 为所有ABI构建一个单一的APK
- [app] 转换为使用标准的底部导航栏
- [app] 取消从集中的Magisk-Modules-Repo下载模块的做法
- [app] 支持用户配置启动镜像vbmeta补丁
- [app] 恢复在一些A/B设备的另一个插槽上安装Magisk的功能
- [app] 允许模块为app内更新+安装指定一个更新URL

### v23.0

- [app] 更新snet扩展。这修正了SafetyNet API的错误。
- [app] 修复代理app中导致APK安装失败的错误
- [App] 隐藏作为代理时日志中恼人的错误
- [app] 修复app被隐藏时修补ODIN tar文件的问题
- [通用] 删除所有Android 5.0之前的支持
- [通用] 更新BusyBox以使用正确的libc
- [通用] 修复C++的未定义行为
- [通用] 几个`sepolicy.rule`复制/安装的修复
- [MagiskPolicy] 删除不必要的sepolicy规则
- [MagiskHide] 更新软件包和进程名称的验证逻辑
- [MagiskHide] 一些防止zygote锁死的变化

### v22.1

- [App] 防止多个安装会话并行运行
- [App] 防止在检查PXA启动镜像的启动签名时出现OutOfMemory崩溃。
- [通用] 正确的cgroup迁移实现
- [通用] 从头开始重写日志编写器，应该可以解决任何崩溃和死锁问题
- [General] 许多脚本更新，修复了退步问题
- [MagiskHide] 防止信号到达时可能出现的死锁。
- [MagiskHide] 必要时部分匹配进程名称
- [MagiskBoot] 在启动图像中保留和修补AVB 2.0结构/头文件
- [MagiskBoot] 适当剥离数据加密标志
- [MagiskBoot] 防止可能的整数溢出
- [MagiskInit] 修复`sepolicy.rule`安装策略
- [resetprop] 在更新前一定要删除现有的`ro.`props。这将修复因修改设备指纹属性而可能引起的启动循环。

### v22.0

- [General] Magisk和Magisk Manager现在已经合并到同一个软件包中了!
- [app] "Magisk Manager "这个词在其他地方不再使用了。我们把它称为Magiskapp程序。
- [app] 支持在安卓5.0以上系统中用高级技术隐藏Magiskapp（代理APK加载）（以前是9.0以上）。
- [app] 不允许在低于安卓5.0的设备上重新打包Magiskapp。
- [App] 检测并警告多个无效的状态，并提供如何解决的说明
- [MagiskHide] 修复停止MagiskHide时不生效的问题
- [MagiskBoot] 修复解压`lz4_lg`压缩启动镜像时的错误
- [MagiskInit] 支持Galaxy S21系列
- [MagiskSU] 修复导致`libsqlite.so`无法加载的不正确的APEX路径

### v21.4

- [MagiskSU] 修复了破坏许多root应用程序的`su -c`行为。
- [General] 正确处理通过套接字的读/写（"断管 "问题）。

### v21.3

- [MagiskInit] 避免挂载`f2fs`用户数据，因为这可能导致内核崩溃。这将解决很多启动循环的问题
- [MagiskBoot] 修复`DHTB`头和华硕`blob`图像格式的一个小头校验错误。
- [MagiskHide] 如果挂载的名字空间是分离的，允许隐藏孤立的进程。

### v21.2

- [MagiskInit] 在传统SAR设备上挂载`system_root`后检测2SI
- [General] 确保`post-fs-data`脚本不能阻塞超过35秒
- [一般] 修复`magisk --install-module`命令
- [通用] 读取文件时修剪Windows换行符
- [一般] 直接将日志记录到文件中，以防止 "logcat "的怪异现象。
- [MagiskBoot] 修复头文件v3图像的转储/加载。

### v21.1

- [MagiskBoot] 支持引导头v3（Pixel 5和4a 5G）。
- [MagiskBoot] 区分`lz4_lg`和`lz4_legacy`（Pixel 5和4a 5G）。
- [MagiskBoot] 支持供应商的启动镜像（用于开发，与Magisk安装无关）。
- [MagiskInit] 支持内核cmdline `androidboot.fstab_suffix`。
- [MagiskInit] 在传统的SAR上支持内核初始化dm-verity。
- [通用] 大幅扩大sepolicy.rule的兼容性
- 一般] 在执行启动脚本时，将Magisk二进制文件添加到`PATH`。
- [一般] 更新`--remove-modules`命令的执行。
- [通用] 使Magisk在安卓11系统上出厂重置后能正常生存。
- [MagiskSU] 在连接`libsqlite.so`时，将APEX包`com.android.i18n`添加到`LD_LIBRARY_PATH`。
- [MagiskHide] 支持隐藏安装在二级用户中的应用程序（例如工作档案）。
- [MagiskHide] 使zygote检测更强大

### v21.0

- [General] 支持Android 11 🎉
- 通用] 增加安全模式检测。当设备启动到安全模式时，禁用所有模块。
- [General] 将 "post-fs-data "模式的超时时间从10秒增加到40秒。
- [MagiskInit] 从头开始重写2SI支持.
- [MagiskInit] 支持没有`/sbin`文件夹的情况（Android 11）。
- [MagiskInit] 将fstab从device-tree转储到rootfs，并强迫`init`对2SI设备使用它。
- [MagiskInit] 剔除2SI的AVB，因为它可能导致启动循环。
- [模块] 从头开始重写模块安装逻辑
- [MagiskSU] 对于Android 8.0+，使用了一个全新的策略设置。这减少了安卓沙盒中的妥协，为root用户提供了更多的策略隔离和更好的安全性。
- [MagiskSU] 隔离的挂载命名空间现在将首先继承父进程，然后将自己与世界隔离。
- [MagiskSU] 更新与Magisk Manager的通信协议，以便与强化的SELinux设置一起工作。
- [MagiskPolicy] 优化所有匹配规则。这将大大减少策略二进制文件的大小，节省内存，并提高一般内核性能。
- [MagiskPolicy] 支持声明新的类型和属性。
- [MagiskPolicy] 使策略声明更接近于股票的`*.te`格式。请查看更新的文档或`magiskpolicy --help`了解更多细节。
- [MagiskBoot] 支持压缩的`extra`blobs
- [MagiskBoot] 将引导图像转为原始大小的零。
- [MagiskHide] 操纵额外的供应商属性

### v20.4

- [MagiskInit] 修复A-only 2SI设备中潜在的启动循环。
- [MagiskInit] 正确支持Tegra分区的命名
- [General] 动态加载 libsqlite.so，这样就不需要在Android 10+上使用包装脚本了。
- 通用] 在某些设备上用回退方法检测API级别。
- 通用] 解决x86内核readlinkat系统调用中可能存在的错误。
- [BusyBox] 启用SELinux功能。添加chcon/runcon等，以及许多小程序的'-Z'选项。
- [BusyBox] 引入独立模式。更多细节见发行说明
- [MagiskHide] 默认禁用MagiskHide。
- [MagiskHide] 增加更多潜在的可检测系统属性
- [MagiskHide] 在跨区域ROM上启用MagiskHide时，增加小米设备启动循环的解决方法。
- [MagiskBoot] 支持修补特殊的Motorolla DTB格式
- [MagiskPolicy] 支持 "genfscon "sepolicy规则
- [Scripts] 支持基于NAND的启动镜像（/dev/block中的字符节点）。
- [Scripts] 更好地支持addon.d（v1和v2）。
- [Scripts] 支持Android 10+的Lineage Recovery。

### v20.3

- [MagiskBoot] 修复`lz4_legacy`解压问题


### v20.2

- [MagiskSU] 正确处理守护程序和应用程序之间的通信（root请求提示）。
- [MagiskInit] 修复kmsg中的日志记录
- [MagiskBoot] 支持修补dtb/dtbo分区格式
- [General] 支持模块中的pre-init sepolicy补丁
- [Scripts] 更新Magisk股票镜像的备份格式

### v20.1

- [MagiskSU] 支持组件名称无关的通信（用于存根APK）。
- [MagiskBoot] 在启动镜像头中设置适当的`header_size`（修复三星设备上的vbmeta错误）。
- [MagiskHide] 多次扫描zygote
- [MagiskInit] 支持没有/sbin/recovery二进制的恢复图像。这将解决一些A/B设备在闪过Magisk后无法启动到恢复的问题。
- [General] 移动acct以防止daemon被杀死。
- [General] 确保"--remove-modules "在移除后会执行uninstall.sh。

### v20.0

- [MagiskBoot] 支持在DTB fstab中注入/修改`mnt_point`值
- [MagiskBoot] 支持对QCDT进行修补。
- [MagiskBoot] 支持给DTBH打补丁
- [MagiskBoot] 支持给PXA-DT打补丁
- [MagiskInit] [2SI] 支持非A/B设置（Android 10）。
- [MagiskHide] 修正拒绝带":"的进程名称的错误。
- [MagicMount] 修复了一个导致/产品镜像无法创建的错误。


### v19.4

- [MagiskInit] [SAR] 以系统为根启动设备，系统安装为/。
- [MagiskInit] [2SI] 支持A/B设备的2阶段启动（Pixel 3 Android 10）。
- [MagiskInit] [initramfs] 延迟创建sbin覆盖层到fs-data之后。
- [MagiskInit] [SARCompat] 旧的system-as-root实现已被废弃，未来不会再有变化。
- [MagiskInit] 为新的系统即根目录覆盖添加overlay.d支持。
- [MagiskSU] 解除对root shells中所有信号的封锁（修复Android上的bash）。
- [MagicMount] 支持替换/product中的文件
- [MagiskHide] 支持Android 10的Zygote blastula池
- [MagiskHide] 所有随机字符串现在也有随机长度了
- [MagiskBoot] 允许对ramdisk.cpio不进行重新压缩
- [MagiskBoot] 支持一些奇怪的华为启动镜像
- [General] 增加新的`--remove-modules`命令，以便在ADB shell中不需要root就可以删除模块。
- [通用] 支持Android 10新的APEX库（项目主线）

### v19.3

- [MagiskHide] 大幅改进进程监控的实现，希望不再导致100%的CPU和守护程序崩溃
- [MagiskInit] 等待分区准备好提前挂载，应该可以解决少数设备的启动循环问题。
- [MagiskInit] 支持EMUI 9.1中使用的EROFS。
- [MagiskSU] 正确实现挂载命名空间的隔离
- [MagiskBoot] 正确计算头文件V2的校验和


### v19.2

- [General] Fix uninstaller
- [General] Fix bootloops on some devices with tmpfs mounting to /data
- [MagiskInit] Add Kirin hi6250 support
- [MagiskSU] Stop claiming device focus for su logging/notify if feasible.
  This fix issues with users locking Magisk Manager with app lock, and prevent
  video apps get messed up when an app is requesting root in the background.

### v19.1

- [General] Support recovery based Magisk
- [General] Support Android Q Beta 2
- [MagiskInit] New sbin overlay setup process for better compatibility
- [MagiskInit] Allow long pressing volume up to boot to recovery in recovery mode
- [MagicMount] Use proper system_root mirror
- [MagicMount] Use self created device nodes for mirrors
- [MagicMount] Do not allow adding new files/folders in partition root folder (e.g. /system or /vendor)

### v19.0

- [General] Remove usage of magisk.img
- [General] Add 64 bit magisk binary for native 64 bit support
- [General] Support A only system-as-root devices that released with Android 9.0
- [General] Support non EXT4 system and vendor partitions
- [MagiskHide] Use Zygote ptracing for monitoring new processes
- [MagiskHide] Targets are now per-application component
- [MagiskInit] Support Android Q (no logical partition support yet!)
- [MagiskPolicy] Support Android Q new split sepolicy setup
- [MagiskInit] Move sbin overlay creation from main daemon post-fs-data to early-init
- [General] Service scripts now run in parallel
- [MagiskInit] Directly inject magisk services to init.rc
- [General] Use lzma2 compressed ramdisk in extreme conditions
- [MagicMount] Clone attributes from original file if exists
- [MagiskSU] Use `ACTION_REBOOT` intent to workaround some OEM broadcast restrictions
- [General] Use `skip_mount` instead of `auto_mount`: from opt-in to opt-out

### v18.1

- [General] Support EMUI 9.0
- [General] Support Kirin 960 devices
- [General] Support down to Android 4.2
- [General] Major code base modernization under-the-hood

### v18.0

- [General] Migrate all code base to C++
- [General] Modify database natively instead of going through Magisk Manager
- [General] Deprecate path /sbin/.core, please start using /sbin/.magisk
- [General] Boot scripts are moved from `<magisk_img>/.core/<stage>.d` to `/data/adb/<stage>.d`
- [General] Remove native systemless hosts (Magisk Manager is updated with a built-in systemless hosts module)
- [General] Allow module post-fs-data.sh scripts to disable/remove modules
- [MagiskHide] Use component names instead of process names as targets
- [MagiskHide] Add procfs protection on SDK 24+ (Nougat)
- [MagiskHide] Remove the folder /.backup to prevent detection
- [MagiskHide] Hide list is now stored in database instead of raw textfile in images
- [MagiskHide] Add "--status" option to CLI
- [MagiskHide] Stop unmounting non-custom related mount points
- [MagiskSU] Add `FLAG_INCLUDE_STOPPED_PACKAGES` in broadcasts to force wake Magisk Manager
- [MagiskSU] Fix a bug causing SIGWINCH not properly detected
- [MagiskPolicy] Support new av rules: type_change, type_member
- [MagiskPolicy] Remove all AUDITDENY rules after patching sepolicy to log all denies for debugging
- [MagiskBoot] Properly support extra_cmdline in boot headers
- [MagiskBoot] Try to repair broken v1 boot image headers
- [MagiskBoot] Add new CPIO command: "exists"

### v17.3

- [MagiskBoot] Support boot image header v1 (Pixel 3)
- [MagiskSU] No more linked lists for caching `su_info`
- [MagiskSU] Parse command-lines in client side and send only options to daemon
- [MagiskSU] Early ACK to prevent client freezes and early denies
- [Daemon] Prevent bootloops in situations where /data is mounted twice
- [Daemon] Prevent logcat failures when /system/bin is magic mounting, could cause MagiskHide to fail
- [Scripts] Switch hexpatch to remove Samsung Defex to a more general pattern
- [Scripts] Update data encryption detection for better custom recovery support

### v17.2

- [ResetProp] Update to AOSP upstream to support serialized system properties
- [MagiskInit] Randomize Magisk service names to prevent detection (e.g. FGO)
- [MagiskSU] New communication scheme to communicate with Magisk Manager

### v17.0/17.1

- [General] Bring back install to inactive slot for OTAs on A/B devices
- [Script] Remove system based root in addon.d
- [Script] Add proper addon.d-v2 for preserving Magisk on custom ROMs on A/B devices
- [Script] Enable KEEPVERITY when the device is using system_root_image
- [Script] Add hexpatch to remove Samsung defex in new Oreo kernels
- [Daemon] Support non ext4 filesystems for mirrors (system/vendor)
- [MagiskSU] Make pts sockets always run in dev_pts secontext, providing all terminal emulator root shell the same power as adb shells
- [MagiskHide] Kill all processes with same UID of the target to workaround OOS embryo optimization
- [MagiskInit] Move all sepolicy patches pre-init to prevent Pixel 2 (XL) boot service breakdown

### v16.7

- [Scripts] Fix boot image patching errors on Android P (workaround the strengthened seccomp)
- [MagiskHide] Support hardlink based ns proc mnt (old kernel support)
- [Daemon] Fix permission of /dev/null after logcat commands, fix ADB on EMUI
- [Daemon] Log fatal errors only on debug builds
- [MagiskInit] Detect early mount partname from fstab in device tree

### v16.6

- [General] Add wrapper script to overcome weird `LD_XXX` flags set in apps
- [General] Prevent bootloop when flashing Magisk after full wipe on FBE devices
- [Scripts] Support patching DTB placed in extra sections in boot images (Samsung S9/S9+)
- [Scripts] Add support for addon.d-v2 (untested)
- [Scripts] Fix custom recovery console output in addon.d
- [Scripts] Fallback to parsing sysfs for detecting block devices
- [Daemon] Check whether a valid Magisk Manager is installed on boot, if not, install stub APK embedded in magiskinit
- [Daemon] Check whether Magisk Manager is repackaged (hidden), and prevent malware from hijacking com.topjohnwu.magisk
- [Daemon] Introduce new daemon: magisklogd, a dedicated daemon to handle all logcat related monitoring
- [Daemon] Replace old invincible mode with handshake between magiskd and magisklogd, one will respwan the other if disconnected
- [Daemon] Support GSI adbd bind mounting
- [MagiskInit] Support detecting block names in upper case (Samsung)
- [MagiskBoot] Check DTB headers to prevent false detections within kernel binary
- [MagiskHide] Compare mount namespace with PPID to make sure the namespace is actually separated, fix root loss
- [MagiskSU] Simplify `su_info` caching system, should use less resources and computing power
- [MagiskSU] Reduce the amount of broadcasting to Magisk Manager
- [ImgTool] Separate all ext4 image related operations to a new applet called "imgtool"
- [ImgTool] Use precise free space calculation methods
- [ImgTool] Use our own set of loop devices hidden along side with sbin tmpfs overlay. This not only eliminates another possible detection method, but also fixes apps that mount OBB files as loop devices (huge thanks to dev of Pzizz for reporting this issue)

### v16.4

- [Daemon] Directly check logcat command instead of detecting logd, should fix logging and MagiskHide on several Samsung devices
- [Daemon] Fix startup Magisk Manager APK installation on Android P
- [MagiskPolicy] Switch from AOSP u:r:su:s0 to u:r:magisk:s0 to prevent conflicts
- [MagiskPolicy] Remove unnecessary sepolicy rules to reduce security penalty
- [Daemon] Massive re-design /sbin tmpfs overlay and daemon start up
- [MagiskInit] Remove `magiskinit_daemon`, the actual magisk daemon (magiskd) shall handle everything itself
- [Daemon] Remove post-fs stage as it is very limited and also will not work on A/B devices; replaced with simple mount in post-fs-data, which will run ASAP even before the daemon is started
- [General] Remove all 64-bit binaries as there is no point in using them; all binaries are now 32-bit only.
  Some weirdly implemented root apps might break (e.g. Tasker, already reported to the developer), but it is not my fault :)
- [resetprop] Add Protobuf encode/decode to support manipulating persist properties on Android P
- [MagiskHide] Include app sub-services as hiding targets. This might significantly increase the amount of apps that could be properly hidden

### v16.3

- [General] Remove symlinks used for backwards compatibility
- [MagiskBoot] Fix a small size calculation bug

### v16.2

- [General] Force use system binaries in handling ext4 images (fix module installation on Android P)
- [MagiskHide] Change property state to disable if logd is disabled

### v16.1

- [MagiskBoot] Fix MTK boot image packaging
- [MagiskBoot] Add more Nook/Acclaim headers support
- [MagiskBoot] Support unpacking DTB with empty kernel image
- [MagiskBoot] Update high compression mode detection logic
- [Daemon] Support new mke2fs tool on Android P
- [resetprop] Support Android P new property context files
- [MagiskPolicy] Add new rules for Android P

### v16.0

- [MagiskInit] Support non `skip_initramfs` devices with slot suffix (Huawei Treble)
- [MagiskPolicy] Add rules for Magisk Manager
- [Compiler] Workaround an NDK compiler bug that causes bootloops

### v15.4

- [MagiskBoot] Support Samsung PXA, DHTB header images
- [MagiskBoot] Support ASUS blob images
- [MagiskBoot] Support Nook Green Loader images
- [MagiskBoot] Support pure ramdisk images
- [MagiskInit] Prevent OnePlus angela `sepolicy_debug` from loading
- [MagiskInit] Obfuscate Magisk socket entry to prevent detection and security
- [Daemon] Fix subfolders in /sbin shadowed by overlay
- [Daemon] Obfuscate binary names to prevent naive detections
- [Daemon] Check logd before force trying to start logcat in a loop

### v15.3

- [Daemon] Fix the bug that only one script would be executed in post-fs-data.d/service.d
- [Daemon] Add `MS_SILENT` flag when mounting, should fix some devices that cannot mount magisk.img
- [MagiskBoot] Fix potential segmentation fault when patching ramdisk, should fix some installation failures

### v15.2

- [MagiskBoot] Fix dtb verity patches, should fix dm-verity bootloops on newer devices placing fstabs in dtb
- [MagiskPolicy] Add new rules for proper Samsung support, should fix MagiskHide
- [MagiskInit] Support non `skip_initramfs` devices using split sepolicies (e.g. Zenfone 4 Oreo)
- [Daemon] Use specific logcat buffers, some devices does not support all log buffers
- [scripts] Update scripts to double check whether boot slot is available, some devices set a boot slot without A/B partitions

### v15.1

- [MagiskBoot] Fix faulty code in ramdisk patches which causes bootloops in some config and fstab format combos

### v15.0

- [Daemon] Fix the bug that Magisk cannot properly detect /data encryption state
- [Daemon] Add merging `/cache/magisk.img` and `/data/adb/magisk_merge.img` support
- [Daemon] Update to upstream libsepol to support cutting edge split policy custom ROM cil compilations

### v14.6 (1468)

- [General] Move all files into a safe location: /data/adb
- [Daemon] New invincible implementation: use `magiskinit_daemon` to monitor sockets
- [Daemon] Rewrite logcat monitor to be more efficient
- [Daemon] Fix a bug where logcat monitor may spawn infinite logcat processes
- [MagiskSU] Update su to work the same as proper Linux implementation:
  Initialize window size; all environment variables will be migrated (except HOME, SHELL, USER, LOGNAME, these will be set accordingly),
  "--preserve-environment" option will preserve all variables, including those four exceptions.
  Check the Linux su manpage for more info
- [MagiskBoot] Massive refactor, rewrite all cpio operations and CLI
- [MagiskInit][magiskboot] Support ramdisk high compression mode

### v14.5 (1456)

- [Magiskinit] Fix bootloop issues on several devices
- [misc] Build binaries with NDK r10e, should get rid of the nasty linker warning when executing magisk

### v14.5 (1455)

- [Daemon] Moved internal path to /sbin/.core, new image mountpoint is /sbin/.core/img
- [MagiskSU] Support switching package name, used when Magisk Manager is hidden
- [MagiskHide] Add temporary /magisk removal
- [MagiskHide] All changes above contributes to hiding from nasty apps like FGO and several banking apps
- [Magiskinit] Use magiskinit for all devices (dynamic initramfs)
- [Magiskinit] Fix Xiaomi A1 support
- [Magiskinit] Add Pixel 2 (XL) support
- [Magiskboot] Add support to remove avb-verity in dtbo.img
- [Magiskboot] Fix typo in handling MTK boot image headers
- [script] Along with updates in Magisk Manager, add support to sign boot images (AVB 1.0)
- [script] Add dtbo.img backup and restore support
- [misc] Many small adjustments to properly support old platforms like Android 5.0

### v14.3 (1437)

- [MagiskBoot] Fix Pixel C installtion
- [MagiskBoot] Handle special `lz4_legacy` format properly, should fix all LG devices
- [Daemon] New universal logcat monitor is added, support plug-and-play to worker threads
- [Daemon] Invincible mode: daemon will be restarted by init, everything should seamlessly through daemon restarts
- [Daemon] Add new restorecon action, will go through and fix all Magisk files with selinux unlabled to `system_file` context
- [Daemon] Add brute-force image resizing mode, should prevent the notorious Samsung crappy resize2fs from affecting the result
- [resetprop] Add new "-p" flag, used to toggle whether alter/access the actual persist storage for persist props

### v14.2

- [MagicMount] Clone attributes to tmpfs mountpoint, should fix massive module breakage

### v14.1

- [MagiskInit] Introduce a new init binary to support `skip_initramfs` devices (Pixel family)
- [script] Fix typo in update-binary for x86 devices
- [script] Fix stock boot image backup not moved to proper location
- [script] Add functions to support A/B slot and `skip_initramfs` devices
- [script] Detect Meizu boot blocks
- [MagiskBoot] Add decompress zImage support
- [MagiskBoot] Support extracting dtb appended to zImage block
- [MagiskBoot] Support patching fstab within dtb
- [Daemon/MagiskSU] Proper file based encryption support
- [Daemon] Create core folders if not exist
- [resetprop] Fix a bug which delete props won't remove persist props not in memory
- [MagicMount] Remove usage of dummy folder, directly mount tmpfs and constuct file structure skeleton in place

### v14.0

- [script] Simplify installation scripts
- [script] Fix a bug causing backing up and restoring stock boot images failure
- [script] Installation and uninstallation will migrate old or broken stock boot image backups to proper format
- [script] Fix an issue with selabel setting in `util_functions.sh` on Lollipop
- [rc script] Enable logd in post-fs to start logging as early as possible
- [MagiskHide] magisk.img mounted is no longer a requirement
  Devices with issues mounting magisk.img can now run in proper core-only mode
- [MagiskBoot] Add native function to extract stock SHA1 from ramdisk
- [b64xz] New tool to extract compressed and encoded binary dumps in shell script
- [busybox] Add busybox to Magisk source, and embed multi-arch busybox binary into update-binary shell script
- [busybox] Busybox is added into PATH for all boot scripts (post-fs-data.d, service.d, and all module scripts)
- [MagiskSU] Fully fix multiuser issues
- [Magic Mount] Fix a typo in cloning attributes
- [Daemon] Fix the daemon crashing when boot scripts opens a subshell
- [Daemon] Adjustments to prevent stock Samsung kernel restrictions on exec system calls for binaries started from /data
- [Daemon] Workaround on Samsung device with weird fork behaviors

### v13.3

- [MagiskHide] Update to bypass Google CTS (2017.7.17)
- [resetprop] Properly support removing persist props
- [uninstaller] Remove Magisk Manager and persist props

### v13.2

- [magiskpolicy] Fix magiskpolicy segfault on old Android versions, should fix tons of older devices that couldn't use v13.1
- [MagiskHide] Set proper selinux context while re-linking /sbin to hide Magisk, should potentially fix many issues
- [MagiskBoot] Change lzma compression encoder flag from `LZMA_CHECK_CRC64` to `LZMA_CHECK_CRC32`, kernel only supports latter
- [General] Core-only mode now properly mounts systemless hosts and magiskhide

### v13.1

- [General] Merge MagiskSU, magiskhide, resetprop, magiskpolicy into one binary
- [General] Add Android O support (tested on DP3)
- [General] Dynamic link libselinux.so, libsqlite.so from system to greatly reduce binary size
- [General] Remove bundled busybox because it casues a lot of issues
- [General] Unlock all block devices for read-write support instead of emmc only (just figured not all devices uses emmc lol)
- [Scripts] Run all ext4 image operations through magisk binary in flash scripts
- [Scripts] Updated scripts to use magisk native commands to increase compatibility
- [Scripts] Add addon.d survival support
- [Scripts] Introduce `util_functions.sh`, used as a global shell script function source for all kinds of installation
- [MagiskBoot] Moved boot patch logic into magiskboot binary
- [MagiskSU] Does not fork new process for each request, add new threads instead
- [MagiskSU] Added multiuser support
- [MagiskSU] Introduce new timeout queue mechanism, prevent performance hit with poorly written su apps
- [MagiskSU] Multiple settings moved from prop detection to database
- [MagiskSU] Add namespace mode option support
- [MagiskSU] Add master-mount option
- [resetprop] Updated to latest AOSP upstream, support props from 5.0 to Android O
- [resetprop] Renamed all functions to prevent calling functions from external libc
- [magiskpolicy] Updated libsepol from official SELinux repo
- [magiskpolicy] Added xperm patching support (in order to make Android O work properly)
- [magiskpolicy] Updated rules for Android O, and Liveboot support
- [MagiskHide] Remove pseudo permissive mode, directly hide permissive status instead
- [MagiskHide] Remove unreliable list file monitor, change to daemon request mode
- [MagiskHide] MagiskHide is now enabled by default
- [MagiskHide] Update unmount policies, passes CTS in SafetyNet!
- [MagiskHide] Add more props for hiding
- [MagiskHide] Remove background magiskhide daemon, spawn short life process for unmounting purpose
- [Magic Mount] Ditched shell script based mounting, use proper C program to parse and mount files. Speed is SIGNIFICANTLY improved

### v12.0

- [General] Move most binaries into magisk.img (Samsung cannot run su daemon in /data)
- [General] Move sepolicy live patch to `late_start` service
  This shall fix the long boot times, especially on Samsung devices
- [General] Add Samsung RKP hexpatch back, should now work on Samsung stock kernels
- [General] Fix installation with SuperSU
- [MagiskHide] Support other logcat `am_proc_start` patterns
- [MagiskHide] Change /sys/fs/selinux/enforce(policy) permissions if required
  Samsung devices cannot switch selinux states, if running on permissive custom kernel, the users will stuck at permissive
  If this scenario is detected, change permissions to hide the permissive state, leads to SafetyNet passes
- [MagiskHide] Add built in prop rules to fake KNOX status
  Samsung apps requiring KNOX status to be 0x0 should now work (Samsung Pay not tested)
- [MagiskHide] Remove all ro.build props, since they cause more issues than they benefit...
- [MagiskBoot] Add lz4 legacy format support (most linux kernel using lz4 for compression is using this)
- [MagiskBoot] Fix MTK kernels with MTK headers

### v11.5/11.6

- [Magic Mount] Fix mounting issues with devices that have separate /vendor partitions
- [MagiskBoot] Whole new boot image patching tool, please check release note for more info
- [magiskpolicy] Rename sepolicy-inject to magiskpolicy
- [magiskpolicy] Update a rule to allow chcon everything properly
- [MagiskHide] Prevent multirom crashes
- [MagiskHide] Add patches for ro.debuggable, ro.secure, ro.build.type, ro.build.tags, ro.build.selinux
- [MagiskHide] Change /sys/fs/selinux/enforce, /sys/fs/selinux/policy permissions for Samsung compatibility
- [MagiskSU] Fix read-only partition mounting issues
- [MagiskSU] Disable -cn option, the option will do nothing, preserved for compatibility

### v11.1

- [sepolicy-inject] Add missing messages
- [magiskhide] Start MagiskHide with scripts

### v11.0

- [Magic Mount] Support replacing symlinks.
  Symlinks cannot be a target of a bind mounted, so they are treated the same as new files
- [Magic Mount] Fix the issue when file/folder name contains spaces
- [BusyBox] Updated to v1.26.2. Should fix the black screen issues of FlashFire
- [resetprop] Support reading prop files that contains spaces in prop values
- [MagiskSU] Adapt communication to Magisk Manager; stripped out unused data transfer
- [MagiskSU] Implement SuperUser access option (Disable, APP only, ADB Only, APP & ADB)
  phh Superuser app has this option but the feature isn't implemented within the su binary
- [MagiskSU] Fixed all issues with su -c "commands" (run commands with root)
  This feature is supposed to only allow one single option, but apparently adb shell su -c "command" doesn't work this way, and plenty of root apps don't follow the rule. The su binary will now consider everything after -c as a part of the command.
- [MagiskSU] Removed legacy context hack for TiBack, what it currently does is slowing down the invocation
- [MagiskSU] Preserve the current working directory after invoking su
  Previously phh superuser will change the path to /data/data after obtaining root shell. It will now stay in the same directory where you called su
- [MagiskSU] Daemon now also runs in u:r:su:s0 context
- [MagiskSU] Removed an unnecessary fork, reduce running processes and speed up the invocation
- [MagiskSU] Add -cn option to the binary
  Not sure if this is still relevant, and also not sure if implemented correctly, but hey it's here
- [sepolicy-inject] Complete re-write the command-line options, now nearly matches supolicy syntax
- [sepolicy-inject] Support all matching mode for nearly every action (makes pseudo enforced possible)
- [sepolicy-inject] Fixed an ancient bug that allocated memory isn't reset
- [uninstaller] Now works as a independent script that can be executed at boot
  Fully support recovery with no /data access, Magisk uninstallation with Magisk Manager
- [Addition] Busybox, MagiskHide, hosts settings can now be applied instantly; no reboots required
- [Addition] Add post-fs-data.d and service.d
- [Addition] Add option to disable Magisk (MagiskSU will still be started)

### v10.2

- [Magic Mount] Remove apps/priv-app from whitelist, should fix all crashes
- [phh] Fix binary out-of-date issue
- [scripts] Fix root disappear issue when upgrading within Magisk Manager

### v10

- [Magic Mount] Use a new way to mount system (vendor) mirrors
- [Magic Mount] Use universal way to deal with /vendor, handle both separate partition or not
- [Magic Mount] Adding **anything to any place** is now officially supported (including /system root and /vendor root)
- [Magic Mount] Use symlinks for mirroring back if possible, reduce bind mounts for adding files
- [Magisk Hide] Check init namespace, zygote namespace to prevent Magic Mount breakage (a.k.a root loss)
- [Magisk Hide] Send SIGSTOP to pause target process ASAP to prevent crashing if unmounting too late
- [Magisk Hide] Hiding should work under any conditions, including adding libs and /system root etc.
- [phh] Root the device if no proper root detected
- [phh] Move `/sbin` to `/sbin_orig` and link back, fix Samsung no-suid issue
- [scripts] Improve SuperSU integration, now uses sukernel to patch ramdisk, support SuperSU built in ramdisk restore
- [template] Add PROPFILE option to load system.prop

### v9

- **[API Change] Remove the interface for post-fs modules**
- [resetprop] New tool "resetprop" is added to Magisk to replace most post-fs modules' functionality
- [resetprop] Magisk will now patch "ro.boot.verifiedbootstate", "ro.boot.flash.locked", "ro.boot.veritymode" to bypass Safety Net
- [Magic Mount] Move dummy skeleton / mirror / mountinfo filesystem tree to tmpfs
- [Magic Mount] Rewritten dummy cloning mechanism from scratch, will result in minimal bind mounts, minimal file traversal, eliminate all possible issues that might happen in extreme cases
- [Magic Mount] Adding new items to /systen/bin, /system/vendor, /system/lib(64) is properly supported (devices with seperate vendor partition is not supported yet)
- [Magisk Hide] Rewritten from scratch, now run in daemon mode, proper list monitoring, proper mount detection, and maybe more.....
- [Boot Image] Add support for Motorola boot image dtb, it shall now unpack correctly
- [Uninstaller] Add removal of SuperSU custom patch script

### v8

- Add Magisk Hide to bypass SafetyNet
- Improve SuperSU integration: no longer changes the SuperSU PATH
- Support rc script entry points not located in init.rc

### v7

- Fully open source
- Remove supolicy dependency, use my own sepolicy-injection
- Run everything in its own selinux domain, should fix all selinux issues
- Add Note 7 stock kernel hex patches
- Add support to install Magisk in Magisk Manager
- Add support for image merging for module flashing in Magisk Manager
- Add root helpers for SuperSU auto module-ize and auto upgrading legacy phh superuser
- New paths to toggle busybox, and support all root solutions
- Remove root management API; both SuperSU and phh has their own superior solutions

### v6

- Fixed the algorithm for adding new files and dummy system
- Updated the module template with a default permission, since people tend to forget them :)

### v5

- Hotfix for older Android versions (detect policy before patching)
- Update uninstaller to NOT uninstall Magisk Manager, since it cause problems

### v4

- Important: Uninstall v1 - v3 Magisk before upgrading with the uninstaller in the OP!!
- Massive Rewrite Magisk Interface API! All previous mods are NOT compatible! Please download the latest version of the mods you use (root/xposed)
- Mods are now installed independently in their own subfolder. This paves the way for future Magisk Manager versions to manage mods, **just like how Xposed Modules are handled**
- Support small boot partition devices (Huawei devices)
- Use minimal sepolicy patch in boot image for smaller ramdisk size. Live patch policies after bootup
- Include updated open source sepolicy injection tool (source code available), support nearly all SuperSU supolicy tool's functionality

### v3

- Fix bootimg-extract for Exynos Samsung devices (thanks to @phhusson), should fix all Samsung device issues
- Add supolicy back to patch sepolicy (stock Samsung do not accept permissive domain)
- Update sepolicy-injection to patch su domain for Samsung devices to use phh's root
- Update root disable method, using more aggressive approach
- Use lazy unmount to unmount root from system, should fix some issues with custom roms
- Use the highest possible compression rate for ramdisk, hope to fix some devices with no boot partition space
- Detect boot partition space insufficient, will abort installer instead of breaking your device

### v2

- Fix verity patch. It should now work on all devices (might fix some of the unable-to-boot issues)
- All scripts will now run in selinux permissive mode for maximum compatibility (this will **NOT** turn your device to permissive)
- Add Nougat Developer Preview 5 support
- Add systemless host support for AdBlock Apps (enabled by default)
- Add support for new root disable method
- Remove sepolicy patches that uses SuperSU's supolicy tool; it is now using a minimal set of modifications
- Removed Magisk Manager in Magisk patch, it is now included in Magisk phh's superuser only

### v1

- Initial release
