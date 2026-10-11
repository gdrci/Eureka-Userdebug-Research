# 25 - 杂项根目录文件与小文件

[English](25-unclassified.md)

## 摘要

未分类组包含来自两个分区的 13 行记录：12 个符号链接（system_a 中 11 个，vendor_a 中 1 个）和一个普通文件 system/xbin/su。这些链接是指向以下绝对路径的别名：/data/cache、/system/bin、/system/etc、/storage/self/primary、/sys/kernel/debug、/odm/odm_dlkm/etc、/vendor/vendor_dlkm/etc、/data 下的 shell bugreports 目录，以及 /system_ext、/vendor、/product 和 /odm 分区根目录。plat_file_contexts 将这些路径标记为 rootfs、cache_file、vendor_file 或 system_file。su 文件是一个经过剥离的 AArch64 PIE ELF，解释器为 /system/bin/linker64。它依赖 libc.so、libc++.so、libm.so 和 libdl.so，其中 libc、libm 和 libdl 的名称通过符号链接解析到 com.android.runtime APEX 中。其用法文本为 "su [WHO [COMMAND...]]"，默认 shell 为 /system/bin/sh，模式位为 4750，所有者为 0:2000，其 file_contexts 标签为 su_exec。策略中声明了 su 和 su_exec，并包含一条从 shell 到 su 的进程转换。镜像中的系统 build.prop 报告 Android 版本为 12，构建类型为 userdebug。

角色表：`data/subsystems/unclassified/roles.tsv`（13 个文件：13 个 verified，0 个 observed，0 个 inferred，0 个 unverified）。

## 已验证结论

- **U01** (verified)：本组包含 13 行：12 个符号链接和 1 个普通文件。按分区划分：system_a 有 11 个链接和 1 个文件，vendor_a 有 1 个链接。
- **U02** (verified)：文件行 system_a/system/xbin/su 的大小为 11088 字节，sha256 为 9ec7f36532bfbc20bd9702897a10cba7f8d78b54b24a64d836e6d9d2db3fafb6，与 classified.tsv 中记录的值相同。
- **U03** (verified)：su 是一个经过剥离的 64 位小端 AArch64 ELF，类型为 DYN（PIE），含 11 个程序头、25 个节头，入口点为 0x1000。其解释器为 /system/bin/linker64，构建 ID 为 0a22c7e5ae892af0d48d6a83b14ea2fd。
- **U05** (verified)：动态段列出四个 NEEDED 库（libc++.so、libc.so、libm.so、libdl.so），标志为 BIND_NOW 和 NOW PIE，且没有 RPATH 或 RUNPATH 条目。
- **U06** (verified)：su 导入了 22 个未定义的动态符号，均带有来自 libc.so 的 LIBC（版本 2）版本号：__libc_init、__errno、__stack_chk_fail、error、execvp、free、fwrite、getpwnam、getpwuid、getuid、memset、setenv、setgid、setgroups、setuid、stderr（OBJECT）、strcmp、strcpy、strdup、strsep、strtoul、unsetenv。
- **U07** (verified)：ELF 注释段包含一个 Android NT_VERSION 注释，其描述数据为 20 00 00 00，以及一个值为 0a22c7e5ae892af0d48d6a83b14ea2fd 的 GNU build-id 注释。
- **U08** (verified)：在解压后的目录树中，su 的模式位为 4750（-rwsr-x---），所有者为 0，组为 2000；其父目录 xbin 的模式为 750，所有者为 0:2000。
- **U09** (verified)：二进制文件内嵌用法文本 'usage: su [WHO [COMMAND...]]'、一段说明 WHO 默认为 'root'、COMMAND 默认为 sh，以及一段注释：WHO 是按此顺序排列的、以逗号分隔的用户、组和附加组列表。
- **U11** (verified)：`.gnu_debugdata` 段是一个 xz 流，解压后为一个 64 位 AArch64 ELF 目标文件，含五个 FUNC 符号：_start（20 字节）、_start_main（64）、main（672）、pwtoid（220）和 extract_uidgids（592）。解修饰后，两个辅助函数分别为 pwtoid(char const*, unsigned int*, unsigned int*) 和 extract_uidgids(char const*, unsigned int*, unsigned int*, unsigned int*, int*)。
- **U12** (verified)：plat_file_contexts 将 /system/xbin/su 映射到标签 u:object_r:su_exec:s0。该文件中没有其他行提及 xbin。
- **U13** (verified)：plat_sepolicy.cil 声明了类型 su 和类型 su_exec，二者都带有 object_r 角色。su 被声明为 typepermissive。
- **U14** (verified)：su_exec 是 plat_sepolicy.cil 中属性集 file_type（第 738 行）、exec_type（第 740 行）和 system_file_type（第 751 行）的成员。
- **U15** (verified)：plat_sepolicy.cil 包含提及 su 或 su_exec 的策略语句：针对 su_exec 的 execute 和 execute_no_trans 的 neverallow 规则（作用于 base_typeattr_283）、允许 shell 和 su 对 su_exec 执行文件 execute 和 read 的 allow 规则，以及从 shell 执行 su_exec 到进程 su 的 typetransition。
- **U17** (verified)：su 的动态链接器和 bionic 依赖通过符号链接解析。linker64 指向 /apex/com.android.runtime/bin/linker64，libc.so、libm.so 和 libdl.so 指向 /apex/com.android.runtime/lib64/bionic/。libc++.so 作为一个 686024 字节的文件存在于 system/lib64 中，/system/bin/sh 作为一个 316024 字节的文件存在。
- **U19** (verified)：挂载目录树中没有任何 .rc 文件包含字符串 xbin，因此没有 init 脚本引用 /system/xbin 目录或 su。
- **U21** (verified)：另外四个链接指向分区根目录：system/system_ext 指向 /system_ext，system/vendor 指向 /vendor，system/product 指向 /product（均位于 system_a），vendor_a 中的 odm 指向 /odm。
- **U22** (verified)：本组中每个链接的模式均为 644（lrw-r--r--），所有者为 root:root，其 st_size 等于目标字符串的长度（例如 /data/cache 为 11，/storage/self/primary 为 21，bugreports 路径为 50）。
- **U27** (inferred)：这 12 个链接很可能是分区镜像的静态内容，在镜像构建时创建，而非由 init 在启动时创建。该推断基于没有任何 init 语句创建它们，以及所有条目的时间戳都相同，均为 2009-01-01。
- **U28** (verified)：镜像中的系统 build.prop 报告 ro.build.version.release=12 和 ro.build.type=userdebug。
- **U29** (inferred)：su 二进制文件是一个切换用户命令。根据其字符串和导入项，它接受可选的 WHO 和 COMMAND，设置组 ID、用户 ID 和附加组列表，然后执行 COMMAND 或 /system/bin/sh。辅助函数的名称表明，一个辅助函数将 WHO 字符串解析为 ID，另一个提取 UID、GID 和组列表。

## 修正措辞后的结论

- **U04** (verified)：su 有 11 个程序头：PHDR、INTERP、三个 LOAD 段（R、R+E、RW；对齐为 0x1000）、TLS、DYNAMIC、GNU_RELRO、GNU_EH_FRAME、GNU_STACK（RW）和 NOTE。一个未分配的 .gnu_debugdata 段位于文件偏移 0x23de 处，大小为 0x130。
- **U10** (verified)：二进制文件内嵌了 --help、setuid failed、setgid failed、invalid uid/gid '%s'、setgroups failed、not allowed、too many group ids 和 failed to exec %s 等字符串，以及环境变量名 PATH、USER、LOGNAME、IFS。文件中还内嵌了默认 shell 路径 /system/bin/sh 和默认 PATH 字面量（/product/bin:/apex/com.android.runtime/bin:...:/vendor/xbin）。该二进制文件将 PATH、USER 和 LOGNAME 传给 setenv（PATH 的值为默认字面量），并在不同的调用位置用 unsetenv 清除 IFS、USER 和 LOGNAME。它没有导入 getenv，因此这些名称是被设置或清除，而非被读取。
- **U16** (verified)：su_exec 也出现在 vendor 和 system_ext 策略文件以及版本化映射文件中。存放于 vendor_a 上的平台公共版本化策略文件（plat_pub_versioned.cil，第 4253-4255 行）将其重新声明为 su_exec_32_0，并且编译后的 odm precompiled_sepolicy 包含该字符串。
- **U18** (verified)：APEX 归档 com.android.runtime.apex 是一个 8955305 字节的 zip（JAR）文件，其负载为 apex_payload.img（8880128 字节）。它还包含 apex_manifest.pb、apex_pubkey、apex_build_info.pb、resources.arsc、AndroidManifest.xml、assets/NOTICE.html.gz 和 META-INF（CERT.SF、CERT.RSA、MANIFEST.MF），共 10 个条目。本组中未解包该负载。
- **U20** (verified)：system_a 中有八个链接指向绝对路径目标：根目录下六个（cache -> /data/cache、bin -> /system/bin、sdcard -> /storage/self/primary、d -> /sys/kernel/debug、etc -> /system/etc、bugreports -> /data/user_de/0/com.android.shell/files/bugreports），以及 vendor_dlkm/ 和 odm_dlkm/ 目录内向下一级的两个（vendor_dlkm/etc -> /vendor/vendor_dlkm/etc、odm_dlkm/etc -> /odm/odm_dlkm/etc）。system_a 根目录还包含 init -> /system/bin/init，它不在本组中。
- **U23** (verified)：链接目标 /vendor/vendor_dlkm/etc 和 /odm/odm_dlkm/etc 在 vendor_a 和 odm_a 下不存在。etc 目录仅作为真实目录存在于 vendor_dlkm_a 和 odm_dlkm_a 中。目录 vendor_dlkm 和 odm_dlkm 也存在于 system_a 的根目录下（各自仅包含 etc 链接），以及 scratch 分区的 overlay 目录中（不含 etc）。
- **U24** (verified)：plat_file_contexts 为这些链接路径加了标签。分区根目录链接由以下行的模式规则匹配：第 1 行（vendor）、第 23 行（odm）、第 43 行（product）、第 55 行（system_ext）、第 70 行（vendor_dlkm）和第 71 行（odm_dlkm）。第 86 行的模式规则 /cache(/.*)? 和第 125 行的 /data/cache(/.*)? 被标记为 cache_file。对 /d、/bin、/etc、/sdcard 和 /bugreports 的普通路径规则位于第 351、354、355、362 和 384 行，被标记为 rootfs。
- **U25** (verified)：init 和属性脚本引用了这些链接路径或其目标。init.rc 第 580-587 行中的 chown、chmod、restorecon_recursive 和 mkdir 使用了 /cache。init.rc 第 1203 行使用了 /bin/rm。init.environ.rc 第 11 行将 /sdcard 导出为 EXTERNAL_STORAGE。storaged 服务（storaged.rc 第 5 行）以读取模式声明了 /d/mmc0。vendor 的 init.anorak.rc 第 227 行在 late_start 类中将 /system/vendor/bin/cnss-daemon 定义为服务。vendor 的 init.anorak.rc 第 210 行的 bugreport 服务使用了 bugreports 目标路径。
- **U26** (verified)：init.rc 中的符号链接语句未创建本组中的任何链接名称。它们创建的是 /dev/net/tun、/dev/stdin、/dev/stdout、/dev/stderr、/dev/binder、/dev/hwbinder、/dev/vndbinder、/mnt/sdcard、/mnt/runtime/default/self/primary、/dev/fd、/dev/sys/fs/by-name/userdata 和 /dev/sys/block/by-name/userdata。
- **U30** (inferred)：12 个链接中的每一个都是其目标路径的简单别名。其作用仅限于 U20 和 U21 中验证过的目标路径，以及目标在其他分区中的出现。init 脚本通过其链接路径引用 cache、sdcard、bin、d 和 system/vendor（见 U25），因此这些用途有已观察到的路径引用。bugreports、dlkm、etc、product、system_ext 和 odm 链接的用途在所检查的文件中没有确认的使用方。

## 缺口

- com.android.runtime.apex 内的负载 apex_payload.img 未解包。linker64、libc.so、libm.so 和 libdl.so 的内容及其符号版本未被检查，因为它们属于另一个子系统。
- su 的机器码未被反汇编：可用的 objdump 对该 AArch64 文件报告 "can't disassemble for architecture UNKNOWN!"。su 的控制流、确切的系统调用和运行时行为均未确定，且 su 未被执行。
- 内嵌的调试数据仅包含五个 FUNC 符号，没有 DWARF 或类型信息。局部函数、数据对象以及字符串到代码的交叉引用均未解析。
- 未读取符号链接 inode 和 su inode 上的扩展属性及 SELinux 标签。此处给出的标签仅来自 file_contexts 模式，每个路径的实际生效标签未经确认。
- 运行时目标 /data/cache、/data/user_de/0/com.android.shell/files/bugreports、/storage/self/primary 和 /sys/kernel/debug 不在已挂载的集合之内，因此其存在性和内容未经验证。
- vendor_dlkm/etc 和 odm_dlkm/etc 目标在 vendor_a 或 odm_a 目录树下不存在。它们通过独立的 dlkm 分区进行解析，这一结论是根据分区名称推断的，未对照 fstab 进行验证。
- system_a、vendor_a、odm_a、product_a 和 system_ext_a 的设备挂载点是根据链接目标推断的。未读取 fstab 或 init 的挂载条目。
- 编译后的策略 odm_a/etc/selinux/precompiled_sepolicy 仅检查了字符串 su_exec 是否存在。它未被解码，su 和 su_exec 的有效访问规则也未被计算。
- 仅列出了提及 su 或 su_exec 的策略语句。未枚举 su 域和 su_exec 类型的完整规则集。
- 未找到创建这 12 个链接的构建步骤或清单，因此链接的来源仅为推断。
- 模式和所有权值来自解压后的目录树（对挂载副本执行 stat），而非原始镜像的 inode 元数据。
- 为符号链接给出的角色描述的是其目标。它们的用途（别名、遗留路径、兼容性）是根据名称推断的，未得到每个链接使用方的确认。
