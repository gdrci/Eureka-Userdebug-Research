# 09 - 初始化脚本与构建属性

[English](09-init-and-properties.md)

## 摘要

本组涵盖 183 条清单记录（181 个普通文件、2 个符号链接），分布于五个分区（odm_a 24、product_a 2、system_a 80、system_ext_a 31、vendor_a 46）。它包含 init 可执行文件及其两个符号链接、带有 zygote 与 USB 变体的主系统 init 脚本、定义 191 个服务和 371 个 on-trigger 的 170 个 .rc 文件、面向 anorak SoC 和 eureka 设备的厂商 init 脚本、三个厂商 shell 辅助脚本、两个 fstab 文件、两个 ueventd.rc 规则文件，以及包含 395 个不同属性键的五个 build.prop 文件。文件所声明的启动流程如下：init 二进制文件读取 /system/etc/init/hw/init.rc，该文件导入 /init.environ.rc、USB 脚本、/vendor/etc/init/hw/init.${ro.hardware}.rc（即 init.eureka.rc，它又导入 init.anorak.rc），以及 /system/etc/init/hw/init.${ro.zygote}.rc（即 init.zygote64_stub32.rc，由厂商 build.prop 中的 ro.zygote=zygote64_stub32 选定）。由 ro.hardware 选定的 fstab 为 /vendor/etc/fstab.eureka，由 init.anorak.rc 中的 mount_all 加载。ro.hardware 本身的值不在本组的任何 build.prop 中。

角色表为本组 183 个文件各提供一行（178 verified、5 observed、0 inferred、0 unverified）。它位于 `data/subsystems/init-and-properties/roles.tsv`。

## 已验证的声明

- **C01** (verified): 本组有 183 行：181 个普通文件和 2 个符号链接，分布为 odm_a 24、product_a 2、system_a 80、system_ext_a 31、vendor_a 46。
- **C02** (verified): 全部 181 个普通文件的 sha256 值与清单一致；两个符号链接行没有哈希值（'-'）。
- **C03** (verified): 170 个 .rc 文件包含分布于 145 个文件中的 191 个服务定义以及 371 个 on-trigger；rc 文件分布为 odm_a 23、product_a 1、system_a 76、system_ext_a 30、vendor_a 40。
- **C04** (verified): 本组的文本文件以 LF 结尾，不含 CR 字节；唯一没有结尾换行符的文件是 init 二进制文件。
- **C05** (verified): 六个服务名各自在两个或更多 rc 文件中定义：zygote（4 个文件）、zygote_secondary、surfaceflinger、health-hal-2-1、vendor.audio-hal、vendor.qti.hardware.display.composer（各 2 个文件）。共存在 183 个不同的服务名。
- **C06** (verified): init 二进制文件是 aarch64 ELF64 PIE，动态链接，解释器为 /system/bin/bootstrap/linker64，已剥离符号，有 12 个程序头和 30 个节头，入口点为 0xac000。
- **C07** (verified): init 二进制文件的 GNU build ID 为 ab61af44159984e98f5fa1da776a36db，其 Android ident 注释中的值为 0x20（32），与系统 build.prop 中的 ro.build.version.sdk=32 一致。
- **C08** (verified): init 二进制文件列出 20 个 NEEDED 共享库，包括 libselinux.so、libfs_mgr.so、libprocessgroup.so、libcutils.so、liblog.so、libbase.so、libutils.so、libc++.so、libc.so、libm.so 和 libdl.so。
- **C10** (verified): init 二进制文件内嵌了它搜索的路径：/system/etc/init/hw/init.rc、各分区的 init 目录（/product、/system、/system_ext、/vendor、/odm）、fstab 搜索前缀（/system/etc/fstab.、/vendor/etc/fstab.、/odm/etc/fstab.、/first_stage_ramdisk/fstab.）、ueventd 规则路径（/system/etc/ueventd.rc、/vendor/ueventd.rc、/odm/ueventd.rc、/ueventd.），以及属性存储路径 /dev/__properties__ 和 /data/property/persistent_properties。
- **C11** (verified): init 二进制文件包含字符串 ro.hardware 和 ueventd started!，因此同一个二进制文件同时提供 init 角色和 ueventd 角色。
- **C12** (verified): 内建命令名以独立字符串的形式出现在 init 二进制文件中，包括 setprop、class_start、class_reset、mount_all、restorecon、start、stop、trigger、writepid、load_persist_props、load_system_props、symlink、mkdir、write、chmod、chown、insmod、export 和 swapon_all。verity_load_state 和 rm 不是独立字符串。
- **C14** (verified): system_a/init 是指向 /system/bin/init 的符号链接，system_a/system/bin/ueventd 是指向 init 的符号链接（相对目标）。
- **C15** (verified): system_a/system/etc/init/hw/init.rc 导入六个文件：/init.environ.rc、/system/etc/init/hw/init.usb.rc、/init.${ro.hardware}.rc、/vendor/etc/init/hw/init.${ro.hardware}.rc、/system/etc/init/hw/init.usb.configfs.rc 和 /system/etc/init/hw/init.${ro.zygote}.rc。
- **C16** (verified): init.rc 包含 51 个 on-block、6 个服务定义（boringssl_self_test32、boringssl_self_test64、boringssl_self_test_apex32、boringssl_self_test_apex64、ueventd、console）以及 23 行 setprop。其 early-init 块启动 ueventd，late-init 触发 fs、post-fs、post-fs-data 和 zygote-start 阶段。
- **C17** (verified): init.zygote64_stub32.rc 定义 zygote（app_process64，套接字 zygote 和 usap_pool_primary）以及 zygote_secondary（stub_zygote 封装 app_process32）。Class 为 main，优先级 -20，用户为 root。由厂商 build.prop 设置的 ro.zygote=zygote64_stub32 选定该文件。
- **C18** (verified): init.anorak.rc 导入四个 debug、cgroup、power 和 usb 脚本，定义 23 个服务和 36 个 on-block，并在 early 阶段对 /vendor/etc/fstab.persist 调用 mount_all，在 early 和 late 阶段对 /vendor/etc/fstab.${ro.hardware} 调用 mount_all。
- **C20** (verified): init.eureka.rc 导入 init.anorak.rc，并新增一个 post-fs 步骤：等待 vendor.all.modules.ready=1，然后向 nautilus 使能节点写入 1。它有两个 sys.hmt.mounted 触发器，分别写入 external_battery mount_state；还有一个 factorytest 触发器，写入 charge_capacity_limit 值 75。
- **C22** (verified): 系统 ueventd.rc 导入 /vendor/etc/ueventd.rc 和 /odm/etc/ueventd.rc，声明子系统 graphics、drm、input、sound 和 dma_heap，并有 31 条 /dev 权限规则，格式为 devname mode uid gid。它导入的 /odm/etc/ueventd.rc 文件不在挂载的目录树中。
- **C23** (verified): 厂商 ueventd.rc 有 248 行以 / 开头（设备和 sysfs 规则），另有一个子系统条目 mem_buf_vm。
- **C24** (verified): fstab.eureka 有 14 个条目。七个逻辑分区（system、system_ext、vendor、odm、vendor_dlkm、odm_dlkm、product）使用带 avb 和 logical 标志的 ext4，data 分区设置了 fileencryption、metadata_encryption、quota、reservedsize=128M 和 checkpoint=block。其中存在一个 4 GiB 的 zram swap 条目（zramsize=4294967296）。
- **C25** (verified): fstab.persist 只有一个条目：/persist（ext4），带有 noatime、noexec、nosuid、nodev、nosymfollow、barrier=1、sync 和 fs_mgr 标志 wait。init.anorak.rc 在早期通过 mount_all 挂载它。
- **C26** (verified): 系统 build.prop 有 226 行和 173 个键，分为由 generate-common-build-props 生成的若干部分：buildinfo.prop 块、device/meta 下的 anorak-common 与 eureka system.prop，以及 PRODUCT_SYSTEM_PROPERTIES。它设置 ro.build.version.sdk=32、ro.build.version.release=12 和 ro.build.type=userdebug。
- **C27** (verified): 厂商 build.prop 有 143 行和 96 个键，包含 anorak-common 与 eureka vendor.prop、android_info.prop 以及 PRODUCT_VENDOR_PROPERTIES 等部分。它设置 ro.vendor.build.fingerprint 和 ro.vendor.product.cpu.abilist 列表。
- **C28** (verified): ODM build.prop 有 86 行和 63 个键；product build.prop 有 90 行和 52 个键；system_ext build.prop 有 31 行和 22 个键。每个文件都有 generate-common-build-props 头部及其各自的分区标识键。
- **C29** (verified): 各分区的标识键保持一致：brand 为 oculus，device 为 eureka，model 为 Quest 3，name 为 eureka；各分区在各自的键名下都具有相同的构建指纹 oculus/eureka/eureka:12/SQ3A.220605.009.A1/50352750111100511:userdebug/test-keys。
- **C32** (verified): 属性上下文为若干键提供了精确匹配标签：ro.build.fingerprint（fingerprint_prop）、ro.zygote（dalvik_config_prop）、ro.hardware（bootloader_prop）、ro.control_privapp_permissions（enum disable enforce log）、ro.product.board 和 ro.vendor.build.fingerprint（build_vendor_prop）、ro.odm.build.fingerprint（build_odm_prop）。vendor.all.modules.ready 和 vendor.all.devices.ready 是精确条目（vendor_init_prop）。
- **C33** (verified): 基于前缀的属性标签覆盖了厂商 init 设置的其他属性：vendor. 映射到 vendor_default_prop，sys. 映射到 system_prop，persist.device_config.oculus_shared_os_services. 映射到 device_config_vros_shared_services_prop。
- **C34** (verified): init 二进制文件自身的标签在 SELinux 源码中声明：/system/bin/init 标记为 init_exec，/vendor/bin 中的每个 init 辅助脚本都有专用的 exec 类型，并在 vendor_sepolicy.cil 中有相应的类型声明。
- **C36** (verified): init.insmod.sh 读取一个配置文件，其每行格式为 action|argument，分派 insmod、setprop、enable、modprobe 和 wait。脚本结束时无条件设置 vendor.all.modules.ready=1 和 vendor.all.devices.ready=1。insmod_sh 服务使用 /vendor/etc/init.insmod.cfg 运行它。
- **C38** (verified): init.kswapd.sh 接受一个参数，通过 /proc/<pid>/comm 扫描 PID 1 至 499，查找名为 kswapd0 的内核线程，并将该 PID 写入 /dev/cpuset/<arg>/tasks。init.anorak.rc 以参数 system-background（在 oculus_kswapd_affinity=true 触发器上）和 '.'（在 false 触发器上）调用它。
- **C39** (verified): init.set_secnvm_perms.sh 运行两条命令：对 /persist/secnvm/* 执行 chown system:system 和 chmod 0600。它作为 set_secnvm_perms_sh oneshot 服务运行，并由 init.anorak.rc 启动。
- **C40** (verified): init.environ.rc 有一个 early-init 动作，导出 10 个环境变量：ANDROID_BOOTLOGO、ANDROID_ROOT、ANDROID_ASSETS、ANDROID_DATA、ANDROID_STORAGE、ANDROID_ART_ROOT、ANDROID_I18N_ROOT、ANDROID_TZDATA_ROOT、EXTERNAL_STORAGE 和 ASEC_MOUNTPOINT。
- **C41** (verified): anorak cgroup 脚本在 3 个 on-block 中创建 /dev/cpuctl 组（background、dex2oat、foreground、idle、immersive、other、rt、soft-rt、system 以及其他组），共 23 行 mkdir、17 行 write 和 66 行 chown。
- **C42** (verified): USB 脚本分布在厂商与系统之间：init.anorak.usb.rc 有 7 个 on-block 和 24 行 write，并定义了 set_usb_cdc_mac；系统 init.usb.configfs.rc 有 25 个 on-block 和 34 行 write，用于在 adb、mtp、ptp 及其组合之间切换 sys.usb.config。

## 更正表述的声明

这些声明实质上成立。下面的表述替换原始表述。

- **C13** (observed): init 二进制文件的属性服务字符串包括 init.svc. 前缀、ro.boottime.init、ro.cold_boot_done、persist. 和 ro. 前缀（例如 ro.boot. 和 ro.boottime.），以及 /dev/__properties__/property_info。这些字符串与 init 设置 init.svc.<name> 状态属性的行为一致（inferred；未观察到运行时写入）。
- **C19** (inferred): 选择 init.eureka.rc 和 fstab.eureka 的 ro.hardware 值被推断为 eureka。它不在本组的五个 build.prop 文件中设置，其来源在本组之外。该名称与每个 build.prop 中的 ro.product.<partition>.device=eureka（例如 ro.product.vendor.device=eureka），以及厂商 build.prop 中的 ro.product.board=eureka 一致。
- **C21** (verified): 厂商 build.prop 设置了 ro.zygote=zygote64_stub32、ro.board.platform=anorak、ro.product.board=eureka 和 ro.soc.model=SXR2230P。ro.zygote 通过导入 /system/etc/init/hw/init.${ro.zygote}.rc 选定 init.zygote64_stub32.rc。anorak 脚本由 init.eureka.rc 按固定路径导入。本组中没有 rc 文件引用 ro.board.platform、ro.product.board 或 ro.soc.model，因此这三个键在本组中没有选择作用。
- **C30** (verified): 有八个属性键出现在多个 build.prop 中：六个 dalvik.vm.heap* 键（heapgrowthlimit、heapmaxfree、heapminfree、heapsize、heapstartsize 和 heaptargetutilization，各出现在 2 个文件中）、ro.control_privapp_permissions（system_ext 和 vendor，值为 enforce），以及 persist.sys.usb.config（5 个文件，odm、product、system_ext 和 vendor 中值为 none，system 中值为 adb）。并集中的 395 个不同键并未被本组的文件合并。
- **C31** (verified): 系统 build.prop 有 85 个 ro. 键、15 个 persist. 形式的 key=value 条目（共 17 行带 persist. 前缀，其中两行使用空格作为分隔符，不是 key=value）、4 个 debug. 键，以及 69 个其他前缀的键（例如 dalvik.、aaudio.、audio.、af.）。其他四个 build.prop 的前缀计数为：odm 25/36/0/2、product 39/8/1/4、system_ext 19/2/0/1、vendor 50/11/3/32。
- **C35** (verified): 本组中的 file_contexts 文件不包含 init .rc 文件、fstab 文件、build.prop 文件或 init.environ.rc 的条目。唯一的 ueventd 条目是标记为 rootfs 的 /ueventd\..* 规则（plat_file_contexts 第 95 行）。/system 下的路径由标记为 system_file 的目录规则 /system(/.*)? 覆盖（plat_file_contexts 第 88 行）。厂商侧路径的标签未追溯（推断来自目录规则；未验证）。
- **C43** (verified): init 脚本所命名的若干脚本或配置目标在挂载的目录树中缺失：/vendor/bin/install-recovery.sh（由 vendor_flash_recovery.rc 命名）、/system/bin/move_widevine_data.sh（由 widevine drm rc 命名），以及系统 ueventd.rc 导入的 /odm/etc/ueventd.rc。/system/bin/linkerconfig 是一个符号链接，其目标 /apex/com.android.runtime/bin/linkerconfig 位于挂载的目录树之外；没有 init 脚本命名它。init 引用的 /system/bin/bootstrap/linkerconfig 存在。其他已检查的引用目标均存在，包括三个厂商辅助脚本、两个 insmod cfg 文件、/odm/bin/flashswap.sh 和 /odm/bin/board_init。
- **C44** (verified): 本组的行级角色标签为 178 个 verified 和 5 个 observed（共 183 行）。这五个 observed 行是 system_a 中没有服务定义的 rc 文件：flags_health_check.rc、atrace_userdebug.rc、wifi.rc、oculus.perfetto.rc 和 bootstat.rc。它们的角色文本描述的是由触发器驱动的动作。

## 已排除的声明

这些声明在复现时未成立。

| ID | 声明 | 原因 |
|---|---|---|
| C09 | init 二进制文件导出 124 个已定义的动态符号，导入 534 个未定义的符号。已定义的集合包括 fs_mgr fstab 和启动配置函数；导入的 | |
| C37 | init.insmod.cfg（13 行）包含 1 个 modprobe、2 个 setprop 和 3 个 enable 动作。init.insmod.charger.cfg（48 行）列出了充电模式 .ko 模块的 modprobe 行 | |

## 缺口

- 本组的任何 build.prop 中都没有设置 ro.hardware，挂载的目录树中也不存在它；init.eureka.rc 和 fstab.eureka 选择 eureka 是根据文件名推断的。其来源（内核命令行或引导加载程序）未被检查。
- 当同一属性键在多个 build.prop 文件中被设置时（persist.sys.usb.config、dalvik.vm.heap*、ro.control_privapp_permissions），其优先级未确定；init 加载逻辑不在目录树中，且未对二进制文件进行反汇编。
- 内建命令表以及 ueventd 与 init 的模式选择，仅通过 init 二进制文件中的精确字符串存在性进行了检查；未做反汇编或控制流分析。verity_load_state 和 rm 作为独立字符串不存在。
- 未识别出 init 二进制文件的版本或构建横幅字符串。
- 第一阶段 ramdisk 的 init.rc（first_stage_ramdisk）和启动镜像不在本组中；只检查了系统和厂商的副本。
- rc 文件所命名的服务二进制文件（位于 /vendor/bin、/system/bin、/system_ext/bin、/product/bin、/odm/bin 下）属于其他子系统，未被检查；只记录了脚本所声明的内容。
- 较大的脚本未逐行通读全文：init.rc（55,112 字节）、atrace.rc（24,276）、init.anorak.rc（18,409，部分读取）、init.anorak.cgroup.rc（9,637）、init.anorak.usb.rc（8,663）、init.usb.configfs.rc（7,690）、wifi.rc（5,715）、perfetto.rc（4,858）。其结构取自 service、trigger、import 和计数提取。
- 挂载的目录树中缺失：/vendor/bin/install-recovery.sh、/system/bin/move_widevine_data.sh、/system/bin/linkerconfig（bootstrap/linkerconfig 变体存在）、/odm/etc/ueventd.rc。init.insmod.cfg 和 init.insmod.charger.cfg 中命名的 .ko 模块不在本组中。
- 清单中的 2 个符号链接行没有 sha256；其目标仅通过 readlink 检查。
- 已编译的 SELinux 策略（odm_a 中的 precompiled_sepolicy）未被解码；SELinux 证据来自源码形式的 file_contexts、property_contexts 和 CIL。rc 文件和 build.prop 文件的标签继承自未被检查的目录级规则。
- 哪个重复的服务定义在运行时生效（例如四个 zygote 变体，或两个文件中的 surfaceflinger 和 health-hal-2-1）取决于 ro.zygote 和 ro.hardware 的值以及导入顺序；只检查了 zygote 的选择逻辑。
- 属性语义（每个 build.prop 键在运行时的作用）未被检查；角色仅描述分组和内容。
