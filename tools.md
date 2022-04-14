# Magisk工具

Magisk带有大量的安装工具、守护程序和开发人员使用的实用程序。本文档包括3个二进制文件和所有包含的小程序。这些二进制文件和小程序如下所示。

```
magiskboot /* 二进制文件 */
magiskinit /* 二进制文件 */
magiskpolicy -> magiskinit
supolicy -> magiskinit
magisk /* 二进制 */
magiskhide -> magisk
resetprop -> magisk
su -> magisk
```

注意: 你下载的Magisk压缩包只包含`magiskboot`、`magiskinit`和`magiskinit64`。二进制的`magisk`被压缩并嵌入到`magiskinit(64)`中。把`magiskinit(64)`推到你的设备上，然后运行`./magiskinit(64) -x magisk <path>`，从二进制文件中提取`magisk`。

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
        Create a symlink to TARGET with the name ENTRY
      mv SOURCE DEST
        Move SOURCE to DEST
      add MODE ENTRY INFILE
        Add INFILE as ENTRY in permissions MODE; replaces ENTRY if exists
      extract [ENTRY OUT]
        Extract ENTRY to OUT, or extract all entries to current directory
      test
        Test the current cpio's status
        Return value is 0 or bitwise or-ed of following values:
        0x1:Magisk    0x2:unsupported    0x4:Sony
      patch
        Apply ramdisk patches
        Configure with env variables: KEEPVERITY KEEPFORCEENCRYPT
      backup ORIG
        Create ramdisk backups from ORIG
      restore
        Restore ramdisk from ramdisk backup stored within incpio
      sha1
        Print stock boot SHA1 if previously backed up in ramdisk

  dtb <input> <action> [args...]
    Do dtb related actions to <input>
    Supported actions:
      print [-f]
        Print all contents of dtb for debugging
        Specify [-f] to only print fstab nodes
      patch
        Search for fstab and remove verity/avb
        Modifications are done directly to the file in-place
        Configure with env variables: KEEPVERITY

  split <input>
    Split image.*-dtb into kernel + kernel_dtb

  sha1 <file>
    Print the SHA1 checksum for <file>

  cleanup
    Cleanup the current working directory

  compress[=format] <infile> [outfile]
    Compress <infile> with [format] (default: gzip), optionally to [outfile]
    <infile>/[outfile] can be '-' to be STDIN/STDOUT
    Supported formats: gzip zopfli xz lzma bzip2 lz4 lz4_legacy lz4_lg

  decompress <infile> [outfile]
    Detect format and decompress <infile>, optionally to [outfile]
    <infile>/[outfile] can be '-' to be STDIN/STDOUT
    Supported formats: gzip zopfli xz lzma bzip2 lz4 lz4_legacy lz4_lg
```

### magiskinit

This binary will replace `init` in the ramdisk of a Magisk patched boot image. It is originally created for supporting devices using system-as-root, but the tool is extended to support all devices and became a crucial part of Magisk. More details can be found in the **Pre-Init** section in [Magisk Booting Process](details.md#magisk-booting-process).

### magiskpolicy

(This tool is aliased to `supolicy` for compatibility with SuperSU's sepolicy tool)

An applet of `magiskinit`. This tool could be used for advanced developers to modify SELinux policies. In common scenarios like Linux server admins, they would directly modify the SELinux policy sources (`*.te`) and recompile the `sepolicy` binary, but here on Android we directly patch the binary file (or runtime policies).

All processes spawned from the Magisk daemon, including root shells and all its forks, are running in the context `u:r:magisk:s0`. The rule used on all Magisk installed systems can be viewed as stock `sepolicy` with these patches: `magiskpolicy --magisk 'allow magisk * * *'`.

```
Usage: ./magiskpolicy [--options...] [policy statements...]

Options:
   --help            show help message for policy statements
   --load FILE       load monolithic sepolicy from FILE
   --load-split      load from precompiled sepolicy or compile
                     split cil policies
   --compile-split   compile split cil policies
   --save FILE       dump monolithic sepolicy to FILE
   --live            immediately load sepolicy into the kernel
   --magisk          apply built-in Magisk sepolicy rules
   --apply FILE      apply rules from FILE, read and parsed
                     line by line as policy statements
                     (multiple --apply are allowed)

If neither --load, --load-split, nor --compile-split is specified,
it will load from current live policies (/sys/fs/selinux/policy)

One policy statement should be treated as one parameter;
this means each policy statement should be enclosed in quotes.
Multiple policy statements can be provided in a single command.

Statements has a format of "<rule_name> [args...]".
Arguments labeled with (^) can accept one or more entries. Multiple
entries consist of a space separated list enclosed in braces ({}).
Arguments labeled with (*) are the same as (^), but additionally
support the match-all operator (*).

Example: "allow { s1 s2 } { t1 t2 } class *"
Will be expanded to:

allow s1 t1 class { all-permissions-of-class }
allow s1 t2 class { all-permissions-of-class }
allow s2 t1 class { all-permissions-of-class }
allow s2 t2 class { all-permissions-of-class }

Supported policy statements:

"allow *source_type *target_type *class *perm_set"
"deny *source_type *target_type *class *perm_set"
"auditallow *source_type *target_type *class *perm_set"
"dontaudit *source_type *target_type *class *perm_set"

"allowxperm *source_type *target_type *class operation xperm_set"
"auditallowxperm *source_type *target_type *class operation xperm_set"
"dontauditxperm *source_type *target_type *class operation xperm_set"
- The only supported operation is 'ioctl'
- xperm_set format is either 'low-high', 'value', or '*'.
  '*' will be treated as '0x0000-0xFFFF'.
  All values should be written in hexadecimal.

"permissive ^type"
"enforce ^type"

"typeattribute ^type ^attribute"

"type type_name ^(attribute)"
- Argument 'attribute' is optional, default to 'domain'

"attribute attribute_name"

"type_transition source_type target_type class default_type (object_name)"
- Argument 'object_name' is optional

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