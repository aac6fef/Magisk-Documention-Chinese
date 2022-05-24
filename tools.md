# Magisk工具

Magisk带有大量的安装工具、守护程序和开发人员使用的实用程序。本文档包括4个二进制文件和所有包含的小程序。这些二进制文件和小程序如下所示。

```
magiskboot /* 二进制文件 */
magiskinit /* 二进制文件 */
magiskpolicy -> /* 二进制文件 */
supolicy -> magiskpolicy
magisk /* 二进制 */
resetprop -> magisk
su -> magisk
```


### magiskboot

一个解压/重新打包启动镜像的工具，解析/修补/提取cpio，修补dtb，修补十六进制文件，以及用多种算法压缩/解压文件。

`magiskboot`原生支持（这意味着它不依赖外部工具）常见的压缩格式，包括`gzip`、`lz4`、`lz4_legacy`（[仅用于LG](https://events.static.linuxfound.org/sites/events/files/lcjpcojp13_klee.pdf)）、`lzma`、`xz`和`bzip2`。

`magiskboot`的概念是使启动镜像的修改更简单。对于解包，它解析头文件并提取映像中的所有部分，如果在任何部分检测到压缩，就立即进行解压。对于重新打包，需要原始的 Boot Image，所以可以使用原始的头文件，只需要改变必要的条目，比如章节大小和校验。如果需要，所有的部分都会被压缩成原始格式。该工具还支持许多CPIO和DTB操作。

```
使用方法。./magiskboot <action> [args...]。

支持的操作。
  unpack [-n] [-h] <bootimg>.
    将<bootimg>解压到，如果有的话，kernel, kernel_dtb, ramdisk.cpio,
    second, dtb, extra, 和 recovery_dtbo到当前目录。
    如果提供了'-n'，它将不会尝试解压kernel或
    ramdisk.cpio的原始格式。
    如果提供了'-h'，它将把头信息转储到'header'。
    在重新打包的时候会被解析。
    返回值。
    0:有效 1:错误 2:chromeos

  repack [-n] <origbootimg> [outbootimg)
    重新打包当前目录下的 Boot Image 组件
    到 [outbootimg]，如果没有指定，则为 new-boot.img。
    如果提供了'-n'，它将不会尝试重新压缩ramdisk.cpio。
    否则，它将压缩ramdisk.cpio和内核，其格式与
    与<origbootimg>中的格式相同，如果提供的文件还没有被压缩的话。
    如果环境变量PATCHVBMETAFLAG被设置为 "true"，所有禁用的标志将被设置在
    将被设置在vbmeta头中。


  hexpatch <文件> <hexpattern1> <hexpattern2
    在<file>中搜索<hexpattern1>，然后用<hexpattern2>替换。

  cpio <incpio> [命令...]
    对<incpio>做cpio命令（修改是在原地进行的）
    每个命令都是一个参数，为每个命令加上引号
    支持的命令。
      exists ENTRY
        如果ENTRY存在则返回0，否则返回1
      rm [-r] ENTRY
        删除ENTRY，指定[-r]可以递归删除。
      mkdir MODE ENTRY
        在权限MODE中创建目录ENTRY
      ln TARGET ENTRY
        为TARGET创建一个名为ENTRY的符号链接
      mv SOURCE DEST
       将SOURCE移到DEST
      add MODE ENTRY INFILE
        在权限MODE中添加INFILE作为ENTRY；如果存在，将取代ENTRY。
      extract [ENTRY OUT]
        将ENTRY提取到OUT，或者将所有条目提取到当前目录下。
      test
        测试当前cpio的状态
        返回值为0或以下数值的位数或值。
        0x1:Magisk    0x2:不支持  0x4:索尼
      patch
        应用ramdisk的补丁
        使用环境变量进行配置: KEEPVERITY KEEPFORCEENCRYPT
      backup ORIG
        从 ORIG 创建ramdisk备份
      restore
        从存储在incpio中的ramdisk备份中恢复ramdisk
      sha1
        如果以前在ramdisk中备份过，则输出库存启动SHA1。
  dtb <input> <action> [args...]
    对<input>做dtb相关的操作
    支持的行为：
      print [-f]
        打印dtb的所有内容以进行调试
        指定[-f]，只打印fstab节点
      patch
        搜索fstab并删除verity/avb
        直接对文件进行就地修改
        用环境变量进行配置：KEEPVERITY

  拆分<input>。
    将image.*-dtb分成kernel + kernel_dtb

  sha1 <file>
    打印<文件>的SHA1校验和

  cleanup
    清理当前工作目录

  compress[=format] <infile> [outfile]
    检测格式并压缩 <infile>, [outfile]是可选的
    <infile>/[outfile] can be '-' to be STDIN/STDOUT
    支持的格式: gzip zopfli xz lzma bzip2 lz4 lz4_legacy lz4_lg

  decompress <infile> [outfile]
    检测格式并解压缩 <infile>, [outfile]是可选的
    <infile>/[outfile] can be '-' to be STDIN/STDOUT
    支持的格式: gzip zopfli xz lzma bzip2 lz4 lz4_legacy lz4_lg
```

### magiskinit

这个二进制文件将取代Magisk修补启动镜像的ramdisk中的`init`。它最初是为支持使用system-as-root的设备而创建的，但是这个工具被扩展到支持所有的设备，成为Magisk的一个重要部分。更多的细节可以在**预启动**部分中找到。 [Magisk 启动过程](details.md#magisk-booting-process).

### magiskpolicy

(这个工具的别名为`supolicy`，以便与SuperSU的sepolicy工具兼容)

这个工具可用于高级开发人员修改SELinux策略。在常见的情况下，比如Linux服务器管理员，他们会直接修改SELinux策略源（`*.te`）并重新编译`sepolicy`二进制文件，但在Android上，我们直接对二进制文件（或运行时策略）打补丁。

所有从Magisk守护进程产生的进程，包括root shells和它的所有分叉，都在`u:r:magisk:s0`的上下文中运行。在所有安装了Magisk的系统上使用的规则可以被看作是原始`sepolicy`，有这些补丁。`magiskpolicy --magisk 'allow magisk * * *'`.

```
用法: ./magiskpolicy [--options...] [policy statements...]

选项。
   --help             显示政策声明的帮助信息
   --load FILE        从FILE加载单体的sepolicy
   --load-split       从预编译的sepolicy中加载或编译拆分的cil策略
   --compile-split    编译分离的cil策略
   --save FILE        向FILE转储单一的sepolicy。
   --live             立即将sepolicy加载到内核中
   --magisk           应用内置的Magisk sepolicy规则
   --apply FILE       应用FILE中的规则，读取并解析逐行作为策略声明(允许多个 --apply)

如果既没有指定--load、--load-split，也没有指定--compile-split。
它将从当前的实时策略（/sys/fs/selinux/policy）加载。

一个策略声明应被视为一个参数。
这意味着每个策略声明应该用引号括起来。
一条命令中可以提供多个策略声明。

语句的格式为"<rule_name> [args...]"。
标有（^）的参数可以接受一个或多个条目。多个
条目由一个用大括号（{}）分隔的列表组成。
标有(*)的参数与(^)相同，但额外地
支持匹配通配符（*）。

示例: "allow { s1 s2 } { t1 t2 } class *"
将扩展至:

allow s1 t1 class { all-permissions-of-class }
allow s1 t2 class { all-permissions-of-class }
allow s2 t1 class { all-permissions-of-class }
allow s2 t2 class { all-permissions-of-class }

支持的策略声明:

"allow *source_type *target_type *class *perm_set"
"deny *source_type *target_type *class *perm_set"
"auditallow *source_type *target_type *class *perm_set"
"dontaudit *source_type *target_type *class *perm_set"

"allowxperm *source_type *target_type *class operation xperm_set"
"auditallowxperm *source_type *target_type *class operation xperm_set"
"dontauditxperm *source_type *target_type *class operation xperm_set"
- 唯一支持的操作是 'ioctl'
- xperm_set 的格式是 'low-high'或 'value'或 '*'.
  '*' 将会被当作 '0x0000-0xFFFF'.
  All values should be written in hexadecimal.

"permissive ^type"
"enforce ^type"

"typeattribute ^type ^attribute"

"type type_name ^(attribute)"
- 参数 'attribute' 是可选的, 默认为 'domain'

"attribute attribute_name"

"type_transition source_type target_type class default_type (object_name)"
- 参数'object_name'是可选的

"type_change source_type target_type class default_type"
"type_member source_type target_type class default_type"

"genfscon fs_name partial_path fs_context"
```


### magisk

当Magisk二进制文件以`magisk'的名字被调用时，它作为一个实用工具，有许多辅助功能和几个Magisk服务的入口点。

```
用法：magisk [applet [arguments]...] 。
   或者: magisk [options]...

选项。
   -c 打印当前的二进制版本
   -v 打印正在运行的守护程序版本
   -V 打印正在运行的守护程序版本代码
   --list 列出所有可用的小程序
   --remove-modules 删除所有模块并重新启动
   --install-module ZIP 安装一个模块压缩文件

高级选项（内部API）。
   --daemon手动启动Magisk守护进程
   --stop 删除所有magisk的变化并停止守护进程
   --[init trigger] 启动init trigger的服务
                             支持的init触发器。
                             post-fs-data, service, boot-complete
   --unlock-blocks 将所有块设备的BLKROSET标志设为OFF
   --restorecon 在Magisk文件上恢复selinux环境
   --clone-attr SRC DEST clone permission, owner, and selinux context
   --clone SRC DEST 将SRC克隆到DEST。
   --sqlite SQL 向Magisk数据库执行SQL命令
   --path print Magisk tmpfs mount path
   --denylist ARGS denylist config CLI

可用的小程序。
    su, resetprop

用法: magisk --denylist [action [arguments...] ] ]
动作。
   status 返回执行状态
   enable 启用拒绝名单的执行
   disable 停用 denylist 的执行
   add PKG [PROC] 添加一个新的目标到denylist中。
   rm PKG [PROC] 从denylist中删除目标。
   ls 打印当前的 denylist
   exec CMDs...    执行孤立的挂载命令
                   名称空间中执行命令，并进行所有的卸载
```

### su

`magisk`的一个小程序，MagiskSU的入口点。传统的的`su'命令.

```
用法: su [选项] [-] [用户 [参数...]] ]

选项:
  -c, --command COMMAND 将COMMAND传给被调用的shell。
  -h, --help 显示此帮助信息并退出
  -, -l, --login 将 shell 伪装成一个登录 shell
  -m, -p,
  --reserve-environment 保存整个环境
  -s, --shell SHELL 使用SHELL而不是默认的/system/bin/sh
  -v, --version 显示版本号并退出
  -V 显示版本代码并退出
  -mm, -M,
  --mount-master强制在全局mount命名空间运行
```

注意：尽管上面没有列出`-Z, --context`选项，该选项仍然存在，以便与为SuperSU设计的应用程序的CLI兼容。但是这个选项被默默地忽略了，因为它不再相关了。

### resetprop

`magisk`的一个小程序。一个高级的系统属性操作工具。查看 [Resetprop Details](details.md#resetprop) 以了解更多背景信息。

```
用法：resetprop [flags] [options...] 。

选项。
   -h, --help 显示此信息
   (没有参数) 打印所有属性
   NAME 获取属性
   NAME设置属性条目NAME与VALUE
   --file FILE 从FILE中加载prop
   --delete NAME删除属性

标志。
   -v 向 stderr 打印详细的输出信息
   -n 设置prop，不经过property_service
           (这个标志只影响到setprop)
   -p 从持久性存储中读/写prop
           (这个标志只影响getprop和delprop)
```
