# 内部细节

## 文件结构

###  "Magisk tmpfs 文件夹" 中的路径

Magisk 会挂载  `tmpfs` 目录来存储一些临时数据. 对于一些有 `/sbin` 文件夹的设备, 它将被选中，因为它也将作为一个覆盖层，将二进制文件注入`PATH`。从Android 11开始，`/sbin`文件夹可能不存在，所以Magisk会在`/dev`下随机创建一个文件夹，并将其作为基础文件夹。

```
可以使用使用`magisk --path`命令来获得 Magisk 目前正在使用的基础文件夹。
例如 magisk 、magiskinit 这样的二进制文件，以及所有程序的链接都会被直接存储在这个路径中。
这意味着当基础文件夹是 /sbin 时，这些二进制文件将直接存储在在PATH中。
MAGISKBASE=$(magisk --path)

# Magisk 内部组件
MAGISKTMP=$MAGISKBASE/.magisk

# Magisk的BusyBox目录。在这个文件夹中存放着
# busybox二进制文件和它所有小程序的符号链接。
# 对这个目录的任何使用都是过时的，请
# 直接调用/data/adb/magisk/busybox并使用
# BusyBox的ASH独立模式。
# 这个路径的创建将在未来被删除。
$MAGISKTMP/busybox

# /data/adb/modules将被绑定挂载在这里。
# 由于nosuid挂载标志，原始文件夹不被使用。
$MAGISKTMP/modules

# 当前的Magisk安装配置
$MAGISKTMP/config

# 分区镜像
# 这个路径中的每个目录都会被挂载到
# 分区的目录名。
# 例如：system, system_ext, vendor, data ...
$MAGISKTMP/mirror

# Magisk在内部创建块状设备来挂载镜像。
$MAGISKTMP/block

# 根目录的补丁文件
# 在系统为根的设备上，/是不可写的。
# 所有启动前的补丁文件都存储在这里，并绑定安装。
$MAGISKTMP/rootdir
```

### `/data`中的路径

一些二进制程序和文件应该存储在`/data`的非易失性存储器中。为了防止被发现，所有的东西都必须存储在`/data`中一个安全的、不被发现的地方。选择`/data/adb`文件夹是因为有以下优点。

- 它是现代安卓系统上的一个现有文件夹，所以它不能被用作Magisk存在的标志。
- 该文件夹的权限默认为`700`，所有者为`root`，所以非root进程无法以任何可能的方式进入、读取、写入该文件夹。
- 该文件夹被标记为secontext `u:object_r:adb_data_file:s0`，很少有进程有权限与该secontext做任何交互。
- 该文件夹位于_设备加密存储中，所以只要数据被正确挂载到FBE（基于文件的加密）设备中，就可以访问它。

```
SECURE_DIR=/data/adb

# 存储一般post-fs-data脚本的文件夹
$SECURE_DIR/post-fs-data.d

# 存储一般后期服务脚本的文件夹
$SECURE_DIR/service.d

# Magisk模块
$SECURE_DIR/modules

# 等待升级的Magisk模块
# 模块文件在安装时不能安全地被修改
# 通过Magisk应用程序安装的模块将被保存在这里
# 并在下次重启时合并到$SECURE_DIR/modules中去
$SECURE_DIR/modules_update

# 存储设置和root权限的数据库
MAGISKDB=$SECURE_DIR/magisk.db

# 所有与magisk相关的二进制文件，包括busybox,
# 脚本，和magisk二进制文件。用于支持
# 模块安装、addon.d、Magisk应用程序等。
DATABIN=$SECURE_DIR/magisk

```

## Magisk 启动进程

### 预启动

`magiskinit`将取代`init`成为第一个运行的程序。

- 早期挂载所需的分区。在传统的system-as-root设备上，我们将root切换到system；在2SI设备上，我们修补fstab并执行原来的`init`来为我们装载分区。
- 从`/sepolicy`中加载sepolicy，在供应商中预编译sepolicy，或者编译分割sepolicy
- 给sepolicy规则打补丁并转储到`/sepolicy`或`/sbin/.se`或`/dev/.se`。
- 给`init`或`libselinux.so`打上补丁，强迫系统加载打过补丁的策略。
- 将Magisk服务注入到`init.rc`中。
- 执行原始的`init'以继续启动过程

### post-fs-data

当`/data`被解密和挂载时，在`post-fs-data`上触发。守护进程`magiskd`将被启动，post-fs-data脚本被执行，模块文件被神奇的挂载。

###后期启动

在启动过程的后期，`late_start`类将被触发，Magisk的 "服务 "模式将被启动。在这个模式下，服务脚本被执行。

## Resetprop

通常，系统属性被设计为只能由`init`更新，对非root进程来说是只读的。在root下，你可以使用`setprop`等命令向`property_service`（由`init`托管）发送请求来改变属性，但改变只读属性（以`ro.`开头的属性，如`ro.build.product`）和删除属性仍然被禁止。

`resetprop`是通过从AOSP中提炼出与系统属性相关的源代码来实现的，并打了补丁，允许直接修改属性区域，或`prop_area`，绕过了需要通过`property_service`。由于我们绕过了`property_service`，有一些注意事项。

- 如果属性变化没有经过`property_service'，`on property:foo=bar'在`*.rc'脚本中注册的动作将不会被触发。`resetprop`的默认设置属性行为与`setprop`相匹配，它***会触发事件（通过首先删除属性然后通过`property_service`设置它来实现）。如果你需要这种特殊行为，有一个标志`-n`可以禁用它。
- 持久属性（以`persist.`开头的道具，如`persist.sys.usb.config`）被存储在`prop_area`和`/data/property`中。默认情况下，删除道具将***不从持久性存储中删除，这意味着该属性将在下次重启后恢复；读取道具将***不从持久性存储中读取，因为这是`getprop`的行为。如果使用标志`-p`，删除道具将在*** `prop_area`和`/data/property`中删除道具，而读取道具将从*** `prop_area`和持久化存储读取。

## SELinux规则

Magisk将对当前的的`sepolicy`进行修补，以确保root和Magisk的操作能够以安全的方式运行。新的域`magisk`实际上是宽容模式，这就是`magiskd`和所有root shell将运行的地方。`magisk_file`是一个新的文件类型，被设置为允许被每个域访问（不受限制的文件上下文）。

在Android 8.0之前，所有被允许的su客户域都被允许直接连接到`magiskd`，并与守护进程建立连接，以获得远程root shell。Magisk还必须放宽一些`ioctl`操作，以便root shell能够正常运行。

在安卓8.0之后，为了减少安卓沙盒中规则的宽容模式，部署了一个新的SELinux模型。`magisk`二进制文件被标记为`magisk_exec`文件类型，以允许的su客户域运行的进程执行`magisk`二进制文件（这包括`su`命令）将通过使用`type_transition`规则转到`magisk_client`。规则严格限制只有`magisk`域进程可以将文件归属到`magisk_exec`。不允许直接连接到`magiskd`的套接字；访问守护进程的唯一方法是通过`magisk_client`进程。这些变化使我们能够保持沙盒的完整性，并使Magisk的特定规则与其他策略分开。

整套规则可以在`magiskpolicy/rules.cpp`中找到。
