# 24 - 原生工具与 HAL 二进制文件

[English](24-native-tools-and-hal-binaries.md)

## 摘要

该组包含 346 行：157 个普通文件（132 个 PIE 可执行文件、17 个共享对象格式的可执行文件，以及 8 个脚本）和 189 个符号链接，分布在 vendor_a（241）、system_ext_a（55）、odm_a（38）、system_a（10）和 product_a（2）中。核心部分包括一个 toybox 多调用二进制文件（172 个 applet 符号链接）和一个 toolbox 多调用二进制文件（6 个符号链接）、一个 mksh shell、一个 vendor binder 服务管理器，以及 14 个 HAL 服务可执行文件（vendor_a/bin/hw 中 10 个，odm_a/bin/hw 中 4 个），这些服务注册 HIDL 或 AIDL 接口，并由 init 服务启动。odm 分区包含 SyncBoss 传感器控制器工具集：fw_init（一个 oneshot 固件更新程序，读取 /odm/etc/fw_init.cfg），以及 17 个共享对象格式的工具，每个都导出 main。system_ext_a 包含 50 个平台服务和命令行可执行文件以及 4 个 shell 包装器；product_a 包含 2 个崩溃上传器可执行文件。所有 157 个文件的 sha256 和字节大小均与清单一致，init 或 SELinux 文本中有 62 个通过完整路径引用了它们。证据中的命令以挂载根目录作为工作目录运行：<image>/mnt。

该角色表为本组中 346 个文件各有一行（190 verified，139 observed，16 inferred，1 unverified）。它位于 `data/subsystems/native-tools-and-hal-binaries/roles.tsv`。

## 已验证的断言

- **G01** (verified)：该组提取内容包含 346 行，分区为：odm_a 38、product_a 2、system_a 10、system_ext_a 55 和 vendor_a 241。
- **G02** (verified)：157 行为普通文件（132 个 application/x-pie-executable、17 个 application/x-sharedlib、7 个 text/plain、1 个 text/x-shellscript），189 行为符号链接。
- **G03** (verified)：磁盘上全部 157 个普通文件的 sha256 和字节大小均与清单一致。
- **G04** (verified)：全部 189 个符号链接行在已挂载分区上都作为符号链接存在。
- **G05** (verified)：符号链接目标为：172 个指向 toybox_vendor，6 个指向 toolbox，1 个指向 meta_screencap，另有 10 个从 system_a/odm/* 指向 /vendor/odm/<name> 的绝对链接（app、bin、etc、firmware、framework、lib、lib64、overlay、priv-app、usr）。
- **G06** (verified)：toybox 的 172 个 applet 链接名中，有 171 个以精确字符串形式出现在 toybox_vendor 字符串表中；名称 '[' 没有。
- **G07** (observed)：toybox_vendor 将其版本标识为 0.8.4-android，其顶层用法行为 'usage: toybox [--long | --help | --version | [COMMAND] [ARGUMENTS...]]'。
- **G08** (verified)：toybox_vendor 是 AArch64 位置无关可执行文件，已剥离符号，解释器为 /system/bin/linker64，NEEDED 项为 libcrypto.so、libz.so、liblog.so、libprocessgroup.so、libselinux.so、libc.so、libm.so 和 libdl.so；它没有 SONAME。
- **G09** (verified)：toolbox 提供六个 applet 链接（getprop、setprop、start、stop、getevent、modprobe），且每个 applet 名称在 toolbox 字符串表中只出现一次。
- **G11** (observed)：vendor shell 是 MirBSD KSH R59（mksh）构建版本，其默认 ENV 文件为 /vendor/etc/mkshrc，默认 PATH 列出 /odm/bin 和 /vendor/bin。
- **G12** (verified)：/vendor/etc/mkshrc 作为 389 字节的文件存在于 vendor_a/etc 中。
- **G13** (observed)：vendor binder 服务管理器由 init 服务启动，该服务位于 class core 中，绑定到 /dev/vndbinder，以 system 用户运行，属组为 system readproc；当其重启时，会一并重启 class main、hal 和 early_hal。
- **G14** (observed)：vndservicemanager 在 vendor 文件上下文中被标记为 vndservicemanager_exec，vendor vndservice 上下文文件将该管理器条目映射到 service_manager_vndservice。
- **G15** (observed)：vndservice 是 vendor 服务管理器的命令行客户端，子命令包括 list、check SERVICE，以及 call SERVICE CODE 后接类型化参数。
- **G16** (verified)：hal_proxy_daemon 的 NEEDED 项为 libnl.so、libcutils.so、liblog.so、libwifi-hal-qcom.so、libhardware_legacy.so、libc++.so、libc.so、libm.so 和 libdl.so。
- **G18** (verified)：在已挂载的 init 或 SELinux 文本中，hal_proxy_daemon 没有完整路径或基本名称引用。
- **G19** (observed)：light HAL 由 class hal 中的 init 服务 light-hal-2-0 启动，以 system 用户运行，注册 android.hardware.light@2.0::ILight 和 vendor.oculus.hardware.light@2.0::ILight；早期启动块为 /sys/class/leds/{blue,green,red,white:indicator} 节点设置属主和模式。
- **G20** (observed)：light HAL 在 vendor 文件上下文中被标记为 hal_light_default_exec，vendor hwservice 上下文将 vendor.oculus.hardware.light::ILight 映射到 hal_light_hwservice。
- **G22** (observed)：KeyMint 服务由 'on init'（start vendor.keymint-qti）启动，以服务 vendor.keymint-qti 运行，位于 class early_hal 中，属组为 system drmrpc，并在 vendor 文件上下文第 489 行被标记为 vendor_hal_keymint_qti_exec。
- **G24** (verified)：KeyMint 二进制文件的 NEEDED 项包括 libqtikeymint.so、libcrypto.so、android.hardware.keymaster@4.0.so、libhidlbase.so 以及三个 android.hardware.security.*-V1-ndk_platform.so 库，此外还有常见的 C 和 C++ 运行时库。
- **G27** (observed)：fw_init 是一个 oneshot、开机时禁用的 init 服务，属于 class main，以 system 用户运行，属组为 system、graphics 和 input，并写入 /dev/kmsg。
- **G28** (observed)：vendor init 脚本将 vendor.syncbosshal.wait_for_service 设为 true，然后启动 fw_init，并附注释说明 fw_init 会在 sensorservice 启动前抑制 SyncBoss 重置。
- **G29** (verified)：fw_init 二进制文件与清单中的 sha256 62380bdbfd4280475b8ad8deddeed1589804e6d8b89e960ae6fafe21b1b59be7 一致。
- **G30** (verified)：fw_init 是 PIE，解释器为 /system/bin/linker64，NEEDED 项为 libsyncboss.so、libovrfwupdate.so、libiad-utils.so、libpng.so、libdeviceid.so、vendor.oculus.hardware.telemetry@1.0.so 以及常见运行时库；定义了 157 个动态符号。
- **G31** (observed)：fw_init 含有以下字面字符串：SyncBoss 设备节点、锁文件模式、fw_init.cfg 路径、启动时长属性，以及其 SWD 和 SPL 更新及 nRF 版本消息。
- **G32** (observed)：/odm/etc/fw_init.cfg 有三个节：[syncboss]（路径 /odm/firmware/syncboss.bin，id 1，busy_path 位于 /sys/devices/virtual/misc/syncboss0 之下）、[secondary]（路径 /odm/firmware/santana_manager.bin，id 0）和 [blu]（路径 /vendor/firmware/blu.bin，version_path /vendor/firmware/blu.toml，part bb51-8kb，uart /dev/ttyHS13，bootloader 和 reset sysfs 路径，bootloader_update_delay_s 5，app_update_delay_ms 500）。
- **G33** (verified)：fw_init.cfg 所指定的固件镜像中，有两个存在于 odm_a/firmware 下：syncboss.bin（366076 字节）和 santana_manager.bin（60240 字节），均为 ARM Cortex-M 镜像；/vendor/firmware/blu.bin 和 blu.toml 在 vendor_a/firmware 中不存在。
- **G34** (observed)：fw_init 在 vendor 文件上下文中通过 odm 路径（第 97 行）被标记为 syncboss_exec；SyncBoss 设备节点 /dev/syncboss.* 被标记为 syncboss_device（第 221 行）。
- **G35** (observed)：对 vendor.fw_init.duration_ms 的属性触发器会将 FwInitDuration 值写入 bootstat。
- **G36** (verified)：odm_a/bin 下全部 17 个共享对象格式的可执行文件都定义了 main 符号并使用解释器 /system/bin/linker64，因此尽管其 ELF 类型为共享对象，它们仍会作为程序启动。
- **G37** (verified)：syncboss_getstatus 的 NEEDED 项仅有 liblog.so、libm.so、libdl.so 和 libc.so（libsyncboss 代码不是 NEEDED 依赖），并将 main 定义为位于 0x5ed4、长 304 字节的全局函数。
- **G38** (observed)：SyncBoss 工具引用一组共享的属性键：persist.vendor.syncbosshal.disable_fw_version_check、persist.vendor.syncbosshal.force_single_pulsar_tx_slot、persist.vendor.syncbosshal.sensor_controller.imu、vendor.syncbosshal.pulsar_afh 和 vendor.syncbosshal.pulsar_blocklist。
- **G39** (observed)：vendor property contexts 将 syncbosshal 键分配给 vendor_syncboss_prop，将 fw_init 键分配给 vendor_fw_init_prop。
- **G40** (observed)：SyncBoss 工具的选项如下：print_fw_version（--check-fw-version、--check-fw-flavor、--short、--build-flavor、--help）；powerstate（--get、--watch）；prox（--get、--get-raw、--get-raw-interactive、--watch、--watch-raw、--run-calibration 等）。
- **G41** (observed)：syncboss_consumers_ctl 是一个 shell 脚本，按顺序启动、停止或重启固定的七个服务列表，支持 --exclude，启动期间在 /data/local/tmp/fw-init-syncboss.lck 上获取 flock，并在停止时将 vendor.oculus.sensors-mock-hal-1-0 视为额外服务。
- **G42** (verified)：挂载分区中的任何 init rc 或 SELinux 文本都未引用 syncboss_consumers_ctl；其唯一的文本引用位于其自身文件中。
- **G43** (observed)：board_init 是 class core 中的 oneshot init 服务，以 system 用户运行，调用方式为 board_init --setprops，其选项文本说明它的作用是设置属性，而不是读取并转储属性。
- **G44** (verified)：board_init 的 NEEDED 项包括 libbase.so、libcutils.so、libdeviceid.so、libgflags.so、liblog.so、libosutils.so、libc++.so、libc.so、libm.so 和 libdl.so，其属性名为 ro.product.device、ro.vendor.hw.deviceid 和 ro.vendor.product.model_extended。
- **G45** (observed)：perf_service 由 init 服务 perfservice 启动，参数为 --schedutil，并将 IPower 1.0 至 1.3、IThermal 1.0 和 2.0、IClocks 1.0 作为默认实例进行注册；在 vendor 文件上下文第 102 行被标记为 hal_perfservice_exec，vendor.scheduler. 属性归入 vendor_scheduler_prop。
- **G46** (verified)：perf_service 的 NEEDED 项包括 android.hardware.power@1.0 至 1.3、android.hardware.thermal@1.0 和 2.0、vendor.oculus.hardware.clocks@1.0.so、libthermalhalutils.so 和 libmros_vendor_sysprops.so 等。
- **G47** (observed)：USB HAL 1.2 服务（odm）是 init 服务 vendor.usb-hal-1-2，位于 class hal 中，以 root 运行，注册 IUsb 1.0 至 1.2 和 IUsbGadget 1.0 至 1.2，具有 20 秒的延迟关机，并在 vendor 文件上下文第 101 行被标记为 hal_usb_oculus_exec；其 configfs 字符串指向 /config/usb_gadget/g1。
- **G48** (observed)：vrdevicemanagerserver 是 init 服务 vrdevice，位于 class late_start 中，以 system 用户运行并具有 SYS_NICE，在 system_ext 文件上下文第 82 行被标记为 vrdevicemanagerserver_exec，并读取 /system/etc/device_props.json 和 /system/etc/display.conf。
- **G49** (verified)：vrdevicemanagerserver 的 NEEDED 项包括 android.hardware.power@1.0 和 1.1、android.hardware.thermal@2.0、vendor.oculus.hardware.clocks@1.0、vendor.oculus.hardware.graphics.composer@1.1、libjsoncpp.so 和 libprocessgroup.so 等。
- **G50** (verified)：本组中的 32 位可执行文件使用解释器 /system/bin/linker，64 位可执行文件使用 /system/bin/linker64；boringssl_self_test32 属于 32 位文件。
- **G52** (verified)：在 157 个普通文件中，94 个在已挂载的 init 或 SELinux 文本中没有完整路径或基本名称引用，62 个有完整路径引用，1 个仅有基本名称引用。
- **G53** (verified)：odm_a/etc/selinux 只包含二进制预编译策略和两个校验和文件，因此 odm 二进制文件的标签取自带有 (odm|vendor/odm) 前缀的 vendor 文件上下文条目。
- **G54** (verified)：vendor 或 system_ext 文件上下文中都没有 /vendor/bin/toolbox、/vendor/bin/sh 或 /vendor/bin/toybox_vendor 的显式文件上下文条目。
- **G55** (verified)：roles.tsv 包含一行表头和 346 行数据，且分区与路径组合唯一；置信度合计为 190 个 verified、139 个 observed、16 个 inferred 和 1 个 unverified。

## 更正措辞的断言

这些断言的实质内容成立。以下措辞替换原文。

- **G10** (observed)：toolbox 含有 getprop 和 setprop 的用法字符串，从 libc 导入 __system_property_foreach 和 __system_property_read_callback，其字符串表包含路径 /dev/__properties__/property_info。通过该文件读取属于推断，依据是路径字符串；没有任何导入或 open 调用能证明这一点。
- **G17** (observed)：hal_proxy_daemon 导入 wifi_* 函数（例如 wifi_initialize、wifi_get_ifaces 和 wifi_start_gscan，它们是由其 NEEDED 库 libwifi-hal-qcom.so 解析的未定义动态符号），其字符串表中保存编号为 1 至 13 的 WifiLogger 命令描述。
- **G21** (observed)：light HAL 二进制文件将 android.hardware.light.V2_0.ILight、vendor.oculus.hardware.light.V2_0.ILight 和 android.hidl.base.V1_0.IBase 的方法符号作为未定义的动态符号导入引用（共 31 个符号，二进制文件中均未定义）。
- **G23** (observed)：KeyMint 二进制文件包含 android::hardware::security::keymint 命名空间下的符号，涉及 IKeyMintDevice、IKeyMintOperation 和 IRemotelyProvisionedComponent。它们的 ::hash 成员已定义（弱符号），而 IKeyMintDevice 和 IRemotelyProvisionedComponent 的 ::descriptor 成员为未定义导入。
- **G25** (verified)：在 *_contexts 文件中，keymint 仅出现在 vendor_file_contexts（第 70 和 489 行）中，vendor service 和 hardware-service 上下文文件中没有 keymint 条目。在整个 etc/selinux 目录中，keymint 还作为类型名出现在 vendor_sepolicy.cil（51 处匹配）和 plat_pub_versioned.cil（43 处匹配）中。
- **G26** (observed)：health HAL 2.1 服务 health-hal-2-1（classes hal 和 charger；用户和用户组为 system；capabilities WAKE_ALARM 和 BLOCK_SUSPEND）在 vendor_file_contexts 第 41 行被标记为 hal_health_default_exec。其二进制文件将 android.hardware.health.V2_0.IHealth 和 android.hardware.health.V2_0.IHealthInfoCallback 作为未定义导入引用。

## 被排除的断言

这些断言在复现时未能成立。

| ID | 断言 | 原因 |
|---|---|---|
| G51 | 本组 149 个 ELF 镜像中没有未剥离的；每个都报告为已剥离。 | 有两行报告为 'not stripped'：odm_a/bin/peripheralconfig-tool（含 debug_info 节）和 system_ext_a/bin/bpftrace（含 .symtab 节）。所声称的零计数来自对带连字符的 'no |

## 缺口

- vendor_a/bin 中的四个 toybox applet 链接（groups、hwclock、tee、touch）被归类在 config-files、display-and-composer、security-keystore-crypto 和 sensors-mcu-meta-hal 下；它们不在本组的 172 个 toybox 行之中。
- toybox applet '[' 的链接目标已验证，但其名称未以精确字符串形式出现在 toybox_vendor 中，其 applet 行为未得到确认。
- 大多数 SyncBoss 共享对象格式工具（vendor_input_tool、syncboss_rftest、syncboss_streammag、syncboss_streamimu、syncboss_per_tool、syncboss_perf_tool、syncboss_input_tool、syncboss_catty、syncboss_rf_blocklist_tool）的用途仅依据名称和共享字符串推断；syncboss_per_tool 的用途未确定。
- 157 个文件中有 94 个在已挂载的 init 或 SELinux 文本中没有完整路径或基本名称引用，因此它们的调用路径（其他二进制文件、脚本或手动使用）未能确定；其中包括 hal_proxy_daemon 和大多数 SyncBoss 工具。
- odm_a/etc/selinux 仅包含预编译的二进制策略，未经解码；odm 二进制文件的标签来自 vendor_file_contexts 的正则条目，未与编译后的策略核对确认。
- 没有运行任何可执行文件；启动顺序、开机时的属性值以及配置键的效果（例如 persist.vendor.syncbosshal.disable_fw_version_check）均未观察到，仅以键名的形式报告。
- 在已挂载的 init 或 SELinux 文本中未找到 syncboss_consumers_ctl 的使用者；锁文件名暗示它与 fw_init 有关联，但这一点未经验证。
- fw_init.cfg 所指定的固件镜像（syncboss.bin、santana_manager.bin）被归类在 dsp-and-remote-processors 下；其内容未经分析。/vendor/firmware/blu.bin 和 blu.toml 不在挂载中。
- NEEDED 列表中的共享库（libsyncboss.so、libqtikeymint.so、libwifi-hal-qcom.so、libovrfwupdate.so、libmros_vendor_sysprops.so 等）未与其他子系统核对，其自身内容也未经分析。
- 除 roles.tsv 中的角色行之外未作深入分析：set_hint、flashswap.sh、set_usb_cdc_mac.sh、virtualcontroller_ctl、trackingservice_ctl、killtop.sh、btcontrol.sh、pdfu_test、bpftrace（51.9 MB）以及其余 system_ext_a 服务可执行文件。
- system_ext_a/bin/screencap 是指向 meta_screencap 的符号链接，后者不在本组之内，未经分析。
- roles.tsv 中 16 个 inferred 和 1 个 unverified 角色是基于名称的（例如 syncboss_streammag、dramstress、xrframeconverter、syncboss_per_tool），未从内容中确认。
