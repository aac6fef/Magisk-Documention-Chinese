# 内部细节

## 文件结构

###  "Magisk tmpfs 文件夹" 中的路径

Magisk 会挂载  `tmpfs` 目录来存储一些临时数据. 对于一些有 `/sbin` 文件夹的设备, it will be chosen as it will also act as an overlay to inject binaries into `PATH`. From Android 11 onwards, the `/sbin` folder might not exist, so Magisk will randomly create a folder under `/dev` and use it as the base folder.

```
可以使用使用`magisk --path`命令来获得 Magisk 目前正在使用的基础文件夹。
例如 magisk 、magiskinit 这样的二进制文件，以及所有程序的链接都会被直接存储在这个路径中。
这意味着当基础文件夹是 /sbin 时，这些二进制文件将直接存储在在PATH中。
MAGISKBASE=$(magisk --path)

# Magisk 内部组件
MAGISKTMP=$MAGISKBASE/.magisk

# Magisk's BusyBox directory. Within this folder stores
# the busybox binary and symlinks to all of its applets.
# Any usage of this directory is deprecated, please
# directly call /data/adb/magisk/busybox and use
# BusyBox's ASH Standalone mode.
# The creation of this path will be removed in the future.
$MAGISKTMP/busybox

# /data/adb/modules will be bind mounted here.
# The original folder is not used due to nosuid mount flag.
$MAGISKTMP/modules

# The current Magisk installation config
$MAGISKTMP/config

# Partition mirrors
# Each directory in this path will be mounted with the
# partition of its directory name.
# e.g. system, system_ext, vendor, data ...
$MAGISKTMP/mirror

# Block devices Magisk creates internally to mount mirrors.
$MAGISKTMP/block

# Root directory patch files
# On system-as-root devices, / is not writable.
# All pre-init patched files are stored here and bind mounted.
$MAGISKTMP/rootdir
```

### Paths in `/data`

Some binaries and files should be stored on non-volatile storages in `/data`. In order to prevent detection, everything has to be stored somewhere safe and undetectable in `/data`. The folder `/data/adb` was chosen because of the following advantages:

- It is an existing folder on modern Android, so it cannot be used as an indication of the existence of Magisk.
- The permission of the folder is by default `700`, owner as `root`, so non-root processes are unable to enter, read, write the folder in any possible way.
- The folder is labeled with secontext `u:object_r:adb_data_file:s0`, and very few processes have the permission to do any interaction with that secontext.
- The folder is located in _Device encrypted storage_, so it is accessible as soon as data is properly mounted in FBE (File-Based Encryption) devices.

```
SECURE_DIR=/data/adb

# Folder storing general post-fs-data scripts
$SECURE_DIR/post-fs-data.d

# Folder storing general late_start service scripts
$SECURE_DIR/service.d

# Magisk modules
$SECURE_DIR/modules

# Magisk modules that are pending for upgrade
# Module files are not safe to be modified when mounted
# Modules installed through the Magisk app will be stored here
# and will be merged into $SECURE_DIR/modules in the next reboot
$SECURE_DIR/modules_update

# Database storing settings and root permissions
MAGISKDB=$SECURE_DIR/magisk.db

# All magisk related binaries, including busybox,
# scripts, and magisk binaries. Used in supporting
# module installation, addon.d, the Magisk app etc.
DATABIN=$SECURE_DIR/magisk

```

## Magisk 启动进程

### Pre-Init

`magiskinit` will replace `init` as the first program to run.

- Early mount required partitions. On legacy system-as-root devices, we switch root to system; on 2SI devices, we patch fstab and execute the original `init` to mount partitions for us.
- Load sepolicy either from `/sepolicy`, precompiled sepolicy in vendor, or compile split sepolicy
- Patch sepolicy rules and dump to `/sepolicy` or `/sbin/.se` or `/dev/.se`
- Patch `init` or `libselinux.so` to force the system to load the patched policies
- Inject magisk services into `init.rc`
- Execute the original `init` to continue the boot process

### post-fs-data

This triggers on `post-fs-data` when `/data` is decrypted and mounted. The daemon `magiskd` will be launched, post-fs-data scripts are executed, and module files are magic mounted.

### late_start

Later in the booting process, the class `late_start` will be triggered, and Magisk "service" mode will be started. In this mode, service scripts are executed.

## Resetprop

Usually, system properties are designed to only be updated by `init` and read-only to non-root processes. With root you can change properties by sending requests to `property_service` (hosted by `init`) using commands such as `setprop`, but changing read-only props (props that start with `ro.` like `ro.build.product`) and deleting properties are still prohibited.

`resetprop` is implemented by distilling out the source code related to system properties from AOSP and patched to allow direct modification to property area, or `prop_area`, bypassing the need to go through `property_service`. Since we are bypassing `property_service`, there are a few caveats:

- `on property:foo=bar` actions registered in `*.rc` scripts will not be triggered if property changes does not go through `property_service`. The default set property behavior of `resetprop` matches `setprop`, which **WILL** trigger events (implemented by first deleting the property then set it via `property_service`). There is a flag `-n` to disable it if you need this special behavior.
- persist properties (props that starts with `persist.`, like `persist.sys.usb.config`) are stored in both `prop_area` and `/data/property`. By default, deleting props will **NOT** remove it from persistent storage, meaning the property will be restored after the next reboot; reading props will **NOT** read from persistent storage, as this is the behavior of `getprop`. With the flag `-p`, deleting props will remove the prop in **BOTH** `prop_area` and `/data/property`, and reading props will be read from **BOTH** `prop_area` and persistent storage.

## SELinux规则

Magisk将对当前的的`sepolicy`进行修补，以确保root和Magisk的操作能够以安全的方式运行。新的域`magisk`实际上是宽容模式，这就是`magiskd`和所有root shell将运行的地方。`magisk_file`是一个新的文件类型，被设置为允许被每个域访问（不受限制的文件上下文）。

在Android 8.0之前，所有被允许的su客户域都被允许直接连接到`magiskd`，并与守护进程建立连接，以获得远程root shell。Magisk还必须放宽一些`ioctl`操作，以便root shell能够正常运行。

在安卓8.0之后，为了减少安卓沙盒中规则的宽容模式，部署了一个新的SELinux模型。`magisk`二进制文件被标记为`magisk_exec`文件类型，以允许的su客户域运行的进程执行`magisk`二进制文件（这包括`su`命令）将通过使用`type_transition`规则转到`magisk_client`。规则严格限制只有`magisk`域进程可以将文件归属到`magisk_exec`。不允许直接连接到`magiskd`的套接字；访问守护进程的唯一方法是通过`magisk_client`进程。这些变化使我们能够保持沙盒的完整性，并使Magisk的特定规则与其他策略分开。

整套规则可以在`magiskpolicy/rules.cpp`中找到。
