# 23 - 配置文件

[English](23-config-files.md)

## 摘要

config-files 组共有 392 行，分布在五个分区中：system_a 270 行、system_ext_a 73 行、vendor_a 28 行、odm_a 11 行、product_a 10 行（389 个普通文件，3 个符号链接）。该组包括 123 个信任锚 PEM 证书（system_a 的 security/cacerts，每个文件以其 OpenSSL 主题哈希命名）、41 个已编译的 terminfo 条目、37 个断字（hyphenation）许可证文本、5 个 gzip 压缩的 NOTICE 索引、38 个 zip 容器模型归档（.ptl、.pt、.jit）、4 个 ExecuTorch 程序文件（.pte、.ff），以及 3 个 octet-stream 类型的 .ptl 模型，分别用于身体、手部、面部、输入、物体、IoT、被动追踪、唤醒词、深度和音频功能；此外还有运行时配置，由 init、进程组库、动态链接器、字体栈、相机和音频 HAL、OpenXR 加载器以及遥测服务读取。二进制项包括两个 eBPF 对象（netd.o、clatd.o）、一个 AArch64 PIE 辅助程序（ip-up-vpn）、一份 Cap'n Proto HMD 配置、一个二进制链接器命名空间 protobuf，以及一个 fs_config 目录表。每一行对应的文件都存在，且 sha256 与记录一致。15 个深入分析条目给出了格式事实、键，以及在挂载镜像中找到的字面量或 SELinux 引用。

角色表为该组中的 392 个文件各有一行（261 项已验证，96 项已观察，35 项为推断，0 项未验证）。它位于 `data/subsystems/config-files/roles.tsv`。

## 已验证的论断

- **C01**（已验证）：该组共 392 行：system_a 270、system_ext_a 73、vendor_a 28、odm_a 11、product_a 10；其中 389 个为普通文件，3 个为符号链接。
- **C02**（已验证）：每一行的文件都存在于 mnt/<partition>/<path>，且全部 392 行的 sha256 都与记录值相等。
- **C03**（已验证）：该组的 MIME 分布：194 个 text/plain、42 个 application/zip、39 个 application/x-terminfo、31 个 application/octet-stream、30 个 application/json、24 个 inode/x-empty、13 个 text/xml、6 个 application/gzip、3 个 text/x-tex、2 个 image/png、2 个 application/x-object、2 个 application/x-dc42-floppy-image、1 个 application/x-pie-executable，以及 3 个链接行。
- **C04**（已验证）：123 个 security/cacerts 行都是单证书 PEM 文件；每个文件名都等于该证书的 subject_hash_old（123 个中的 123 个）。示例：01419da9.0 是 Microsoft ECC Root Certificate Authority 2017；9d6523ce.0 是 Chunghwa Telecom ePKI Root Certification Authority；8d6437c3.0 是 DigiCert Assured ID Root G2。
- **C05**（已验证）：system_a/system/etc/security/cacerts 下的两个磁盘文件（a3896b44.0 和 a7605362.0）被归类到子系统 gpu，而非 config-files，因此不属于本组，也不在 roles.tsv 中。
- **C06**（已验证）：全部 41 个 terminfo 行都能被 infocmp 解码。全部 41 个文件都以旧版 terminfo 魔数 0x011a（字节 1a 01）开头。其中两行（x/xterm 和 x/xterm-xfree86）在分类中被标记为 application/x-dc42-floppy-image，这是对同一 terminfo 格式的误识别。
- **C07**（已验证）：5 个 NOTICE.xml.gz 行都是 gzip 压缩的 XML <licenses> 索引，其中文件名条目数分别为：system_a 3497、vendor_a 559、system_ext_a 28、odm_a 5、product_a 4。
- **C08**（已验证）：24 行为零字节（sha256 e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855）。它们是六个 .placeholder 标记、fs_config_dirs 和 fs_config_files 桩文件、group 和 passwd 桩文件，以及 system 上的零字节 xtables.lock。
- **C09**（已验证）：3 行为符号链接。两个 odm_a 的 openxr active_runtime 清单指向绝对路径 /apex/com.meta.xr/，这些路径在挂载分区中不存在。vendor_a/bin/groups 指向 toybox_vendor。
- **C10**（已验证）：zip 容器模型行（例如 system_ext_a/etc/assistant/en_US/ww_model.pt 和 system_a/system/etc/handtracking/runtime/dpe/DPEV23.ptl）包含诸如 <prefix>/extra/producer_info.json 之类的条目。42 个 zip 类型的行包括 2 个旋转 CSV 归档、1 个 otacerts 归档、1 个 gltf 场景归档和 38 个模型归档。
- **C11**（已验证）：roles.tsv 包含一行表头和 392 行数据，每行四个以制表符分隔的列，且 (partition, path) 键与输入行完全一致，每个键恰好出现一次。
- **D01a**（已验证）：system/etc/cgroups.json（1018 字节，JSON）的顶层键为 Cgroups 和 Cgroups2。Cgroups 列出五个控制器：blkio、cpu、cpuacct、cpuset、memory（memory 的 Optional 为 true）。Cgroups2 的 path 为 /sys/fs/cgroup，其中只有一个 freezer 控制器，path 为 '.'。
- **D01b**（已观察）：init 和进程组库按路径引用该文件：init.rc 第 14 行的注释写道 cgroups 是 "using list from /etc/cgroups.json" 挂载的；libprocessgroup_setup.so 包含字面量 /etc/cgroups.json 和 /vendor/etc/cgroups.json；plat_file_contexts 将 /(vendor|system/vendor)/etc/cgroups.json 映射到 vendor_cgroup_desc_file（第 6 行），将 /system/etc/cgroups.json 映射到 cgroup_desc_file（第 554 行）。
- **D02a**（已验证）：system/etc/task_profiles.json（12543 字节）的顶层键为 Attributes（10 个）、Profiles（40 个）和 AggregateProfiles（13 个）。CPUSET_SP_TOP_APP 是一个 AggregateProfiles 条目，其 Profiles 为 [MaxPerformance, ProcessCapacityMax, MaxIoPriority, TimerSlackNormal]。
- **D02b**（已观察）：使用方按路径引用该文件：libprocessgroup.so 包含 /etc/task_profiles.json、/vendor/etc/task_profiles.json 和 /etc/task_profiles/task_profiles_%u.json；plat_file_contexts 第 603 行将 /system/etc/task_profiles.json 映射到 task_profiles_file；第 347 行将 /system/etc/task_profiles/task_profiles_[0-9]+.json 映射到 task_profiles_api_file。odm 的 init 脚本 odm.sensor-emulator.rc 第 10 行使用配置名 CPUSET_SP_TOP_APP。
- **D02c**（已验证）：vendor_a/etc/task_profiles.json 定义了 11 个属性、52 个 profile 和 22 个聚合 profile。profile trackingPolicy、sensorServicePolicy、mrSystemPolicy 和 SoftRealtimePerformance 只存在于 vendor 文件中，不存在于 system/etc/task_profiles.json 中。
- **D03a**（已验证）：system/etc/fonts.xml（67158 字节）是一个 familyset，版本为 23，包含 157 个 family 元素和 24 个 alias 元素。它引用了 204 个不同的字体文件，且这 204 个文件全部存在于 system_a/system/fonts 中。别名将诸如 sans-serif-thin 之类的名称映射到带有 weight 属性的 sans-serif。
- **D03b**（已观察）：libandroid.so 包含字面路径 /system/etc/fonts.xml 和 /product/etc/fonts_customization.xml。
- **D04a**（已验证）：product_a/etc/fonts_customization.xml（1104 字节）是一个字体修改文档（版本 1），包含三个新命名的 family 条目：ms-neo-gothic（MSNeoGothic.ttf）、optimistic（五种 OptimisticText 字重）和 optimistic display（OptimisticDisplay-VF.ttf，五种字重）。它引用了七个不同的字体文件。
- **D04b**（已验证）：在 fonts_customization.xml 中引用的七个字体文件里，六个 Optimistic 文件存在于 product_a/fonts 中。MSNeoGothic.ttf 不在 product_a/fonts 中，但存在于 system_a/system/fonts 中。
- **D05a**（已验证）：system/etc/fs_config_dirs 是一个 32 字节的二进制 fs_config 表，只有一个条目：长度 32，模式 0771（八进制），uid 1000，gid 1000，capabilities 0x0，前缀 'persist/'。布局为：uint16 长度、uint16 模式、uint16 uid、uint16 gid、uint64 capabilities，然后是以 NUL 结尾并填充到长度的前缀。
- **D05b**（已观察）：libcutils.so 包含 fs_config_dirs 和 fs_config_files 在 /odm、/oem、/product、/system_ext、/system 和 /vendor 下的字面路径；libadbd_fs.so 和 libcutils.so 携带 fs_config 符号名。
- **D06a**（已验证）：system/etc/linker.config.pb（984 字节）是一个二进制 protobuf，其顶层字段为 3（38 个长度界定的字符串）和 4（19 个字符串）。字段 3 的值是库名，例如 heapprofd_client_api.so、libEGL.so、libGLESv1_CM.so 和 libc.so；字段 4 的值包括 libandroidicu.so、libdexfile.so、libicu.so 和 libnativeloader.so。
- **D06b**（已观察）：linkerconfig 二进制通过路径加载该文件（/system/etc/linker.config.pb 和 /etc/linker.config.pb），其内嵌的构建路径指向 linker_config.proto 定义。字段 3 和 4 的含义是根据名称列表推断的，尚未与 .proto 核对确认。
- **D07a**（已验证）：system/etc/xrs-hmdconfig.capnp.bin（760 字节）是一个 Cap'n Proto 消息：4 字节的段数（0，表示一个段）、4 字节的段大小（94 个字），以及段数据；8 + 94 x 8 = 760 字节。段中的字符串包括 'Meta Quest 3' 和 'Oculus VR'。
- **D07b**（已观察）：该文件在 system_ext 的 SELinux 文件上下文中被标记为 xrs_system_file，libossdk.oculus.so 包含其路径字面量 /system/etc/xrs-hmdconfig.capnp.bin。
- **D08a**（已验证）：system/etc/ppp/ip-up-vpn（11048 字节）是一个面向 AArch64 的小端 ELF64 DYN（PIE）可执行文件，入口地址 0x1000，有六个 NEEDED 库（libcutils.so、liblog.so、libc++.so、libc.so、libm.so、libdl.so），构建 ID 为 8bf65ecfdd62db387cd2d4f103a653ea。其未定义的动态符号包括 __android_log_print、fopen、fwrite、ioctl、inet_pton、socket 和 rename。其字符串包括 /data/misc/vpn/state、/data/misc/vpn/.tmp 和 INTERNAL_ADDR4。
- **D08b**（已观察）：racoon 二进制包含字面量 /etc/ppp/ip-up-vpn。挂载分区中没有任何 init .rc 文件或 SELinux 上下文文件提到该路径。
- **D09b**（已观察）：bpfloader 二进制引用了对象目录 /system/etc/bpf/ 和 /apex/com.android.tethering/etc/bpf/。bpfloader_min_ver 段的字节为 00000000，bpfloader_max_ver 段的字节为 00000100；这些字段的整数宽度和字节序尚未确认。
- **D09c**（已验证）：system/etc/bpf/clatd.o（9496 字节）是一个 ELF64 可重定位 Linux BPF 对象，包含 schedcls 段 ingress6/clat_ether、ingress6/clat_rawip、egress4/clat_ether 和 egress4/clat_rawip，映射 clat_ingress6_map 和 clat_egress4_map，许可证为 Apache 2.0。CLAT（464XLAT）的用途是根据名称推断的。
- **D10a**（已验证）：aarch64、armv7a-vfp 和 x86_64 三个平台的 OpenXR active-runtime 清单均为 file_format_version 1.0.0，其 runtime.library_path 指向 /system/priv-app/VrDriver/VrDriver.apk!/lib/<abi>/libopenxr_forwardloader.so。VrDriver.apk 归档包含 arm64-v8a 和 armeabi-v7a 的 forwardloader 库，没有 x86_64 的 openxr 条目。
- **D10b**（已观察）：libopenxr_loader.so 包含路径片段 /etc/openxr/、openxr/1/api_layers/implicit.d/，以及文件名 /active_runtime.aarch64.json 和 /active_runtime.json。odm_a 的 openxr 符号链接指向 /apex/com.meta.xr/，该路径在挂载分区中不存在。
- **D11a**（已验证）：odm_a/etc/fw_init.cfg（513 字节）是一个 INI 文件，包含以下小节：syncboss（path、id、busy_path）、secondary（path、id）和 blu（path、version_path、part、uart、bootloader_path、reset_path、bootloader_update_delay_s、app_update_delay_ms）。odm_a/etc/catty.cfg（149 字节）的小节为 syncboss（prefix、dev、baud）和 santanamanager（prefix、dev、baud）。
- **D11b**（已观察）：fw_init 服务（odm_a/etc/init/fw_init.rc）将 /vendor/odm/bin/fw_init 作为一个禁用的 oneshot 运行；vendor 的 init.anorak.rc 在第 504 行启动它。fw_init 二进制包含字面量 /odm/etc/fw_init.cfg，catty 服务的二进制包含 /odm/etc/catty.cfg。catty 服务从 odm_a/etc/init/vendor.oculus.hardware.catty@1.0-service.rc 第 1 行启动。
- **D11c**（已验证）：fw_init.cfg 所命名的固件文件部分存在：odm_a/firmware/syncboss.bin（366076 字节）和 santana_manager.bin（60240 字节）存在。blu 小节中命名的 vendor_a/firmware/blu.bin 和 blu.toml 在挂载镜像中不存在。
- **D12a**（已验证）：vendor_a/etc/resourcemanager_anorak_max98388.xml（61849 字节）的根元素为 resource_manager_info，版本为 2.0，有 13 个子元素：version、config_params、config_volume、config_lpm、config_gapless、bt_codecs、gain_db_to_level_mapping、config_voice、sleep_monitor_vote_streams、group_device_cfg、in_streams、device_profile 和 sound_trigger_platform_info。group_device_cfg 的子元素包括 upd_rx、handset、speaker、speaker_voice、upd_rx_handset 和 upd_rx_speaker。
- **D12b**（已观察）：音频库 libar-pal.so 包含字面量 /vendor/etc/usecaseKvManager.xml 和 /vendor/etc/card-defs.xml，以及一条提到 Rm.xml 的日志消息。vendor 二进制文件和库中不存在指向 resourcemanager_anorak_max98388.xml 的字面路径，因此尚未确定该文件的加载者。
- **D13a**（已验证）：vendor_a/etc/peripheralconfig/dynamic_logical_device_manifest.json（6030 字节）包含：targetName 为 'halliday'，logicalDeviceCount 为 1，logicalDeviceXMLnames 为 ['eureka.xml']；一个逻辑设备 LogicalRGB（fwkId 6，primaryCameraIndex 1，visible 为 true），其物理相机为 JacksonRight 和 JacksonLeft；以及一个 xmlConfigSensorIdToName 映射，包含六个传感器条目（og01a1b_left、og01a1b_right、ov7251_left、ov7251_right、imx471_left、imx471_right）。
- **D13b**（已观察）：相机适配器插件 peripheralconfig-plugin-camx-adapter.so 包含字面量 /vendor/etc/peripheralconfig/dynamic_logical_device_manifest.json。
- **D14b**（已观察）：libtrackingengines.so 包含字面量 /system/etc/facetracking/runtime。其中不存在以 facetracking/runtime/config.json 结尾的字面量，因此尚未确定 config.json 的加载者；也未找到 init 或 SELinux 对该文件的引用。
- **D15a**（已验证）：system/etc/threadpolicy.conf（293 字节）类似 JSON，但并非严格的 JSON：ELEVATED_SOFT_REALTIME 条目之后的尾随逗号会使严格解析器在第 12 行第 6 列失败。它定义了两个策略：ELEVATED（policy SCHED_FIFO，rtprio 47，taskprofile SoftRealtimePerformance）和 ELEVATED_SOFT_REALTIME（policy SCHED_OTHER，nice -20，taskprofile SoftRealtimePerformance）。
- **D15b**（已观察）：vrdevicemanagerserver 二进制包含字面量 /system/etc/threadpolicy.conf 和符号字符串 setThreadPolicy。taskprofile 名称 SoftRealtimePerformance 定义在 vendor_a/etc/task_profiles.json 中，而不在 system 的 task_profiles.json 中。
- **X02**（已观察）：SELinux 文件上下文映射了若干组内文件：/system/etc/event-log-tags 映射到 system_event_log_tags_file（plat_file_contexts 第 579 行），/system/etc/task_profiles/cgroups_[0-9]+.json 映射到 cgroup_desc_api_file（第 343 行），/system/etc/task_profiles/task_profiles_[0-9]+.json 映射到 task_profiles_api_file（第 347 行）。组内其他许多文件（fonts、facetracking、perfstream、threadpolicy、openxr、resourcemanager、linker.config.pb、fs_config_dirs）在挂载的上下文文件中没有对应的上下文行。

## 已修正措辞的论断

这些论断实质上成立。下面的措辞取代原文。

- **D09a**（已验证）：system/etc/bpf/netd.o（16552 字节）是一个面向 Linux BPF 的 ELF64 可重定位对象。其程序段为 cgroupskb/ingress/stats、cgroupskb/egress/stats、cgroupsock/inet/create，以及四个 skfilter/*/xtbpf 段（ingress、egress、allowlist、denylist）。存在段 maps、progs、license、critical、bpfloader_min_ver、bpfloader_max_ver、size_of_bpf_map_def 和 size_of_bpf_prog_def。license 段包含 11 字节的字符串 'Apache 2.0' 加一个 NUL 终止符。size_of_bpf_map_def 为 0x30 字节。十个映射名：uid_counterset_map、iface_stats_map、app_uid_stats_map、uid_owner_map、configuration_map、uid_permission_map、cookie_tag_map、iface_index_name_map、stats_map_A、stats_map_B。
- **D14a**（已验证）：system/etc/facetracking/runtime/config.json（9393 字节）的顶层键为 lipsyncSmoothing（minCutoff 2.0，beta 20.0，dCutoff 1.0）、lipsyncBlendshapes（14 个条目）、lipsync（只有一个键 modelPath）和 audioToExpression（17 个条目）。其 modelPath 值位于第 368 和 371 行，分别为 models/lipsync/prod/nn_config.json 和 models/a2e/prod/nn_config.json。这两个 nn_config.json 文件都是后端条目的 JSON 列表（Backend TorchScript；QEngine QNNPACK 或 FBGEMM），分别指向 bolt/model.pt、htp/qnn_model.ptl 和 cpu/model.pt（lipsync），以及在 CpuModel 键下指向 bolt/a2e.ptl 和 cpu/a2e.ptl（a2e）。在这些被引用的模型文件中，只有 bolt/model.pt 和 bolt/a2e.ptl 存在于挂载镜像中。
- **X01**（已观察）：init 的 rc 文件通过路径引用了若干组内文件：vendor 的 init.anorak.rc 第 246 行以 /vendor/etc/init.insmod.cfg 运行 insmod_sh；第 291 行以 /vendor/etc/sec_config 运行 vendor.irsc_util；第 296 行以 /vendor/etc/msm_irqbalance.conf 运行 vendor.msm_irqbalance；system 的 logd.rc 写入 /dev/event-log-tags；system_ext 的 trackingservice.rc 第 18 行为服务 trackingservice 指定了 trackingPolicy 任务配置（task profile），system_ext 的 mrsystemservice.rc 第 18 行指定了 mrSystemPolicy 任务配置。
- **X03**（已验证）：文本文件 vendor_a/etc/msm_irqbalance.conf（467 字节）和 vendor_a/etc/sec_config（331 行）由 X01 中列出的 init 服务使用。挂载镜像中有两个组外的额外 cacerts 文件（a3896b44.0 和 a7605362.0，分类为 gpu），如 C05 所记录；磁盘上全部 41 个 terminfo 文件都在组内。

## 缺口

- system_a 上的两个 security/cacerts 文件（a3896b44.0、a7605362.0）被归类到子系统 gpu，而非 config-files；它们不属于本组，也不在 roles.tsv 中。
- 尚未确定 vendor_a/etc/resourcemanager_anorak_max98388.xml 的加载者：vendor 二进制文件和库中不存在指向该文件名的字面路径（grep 计数为 0），而 libar-pal.so 仅提及 usecaseKvManager.xml 和 card-defs.xml。
- 尚未确定 system/etc/facetracking/runtime/config.json 的加载者：libtrackingengines.so 包含目录字面量 /system/etc/facetracking/runtime，但没有针对该路径、以 config.json 结尾的字面量。
- odm_a/etc/fw_init.cfg 所命名的 vendor_a/firmware/blu.bin 和 blu.toml 均不在挂载镜像中。
- 两个 odm_a openxr 符号链接所指向的 /apex/com.meta.xr 目标不在挂载分区中。system 的 x86_64 清单指向 lib/x86_64/libopenxr_forwardloader.so，但 VrDriver.apk 中没有 x86_64 的 openxr 条目，因此未找到该库。
- linker.config.pb 的字段语义（字段 3 中 38 个条目，字段 4 中 19 个条目）未与 Soong 的 linker_config.proto 核对，该文件不在挂载分区中；给出的标签是根据库名推断的。
- 除文件头或大小外均未解码的内容：vendor_a/etc/Diag-CHGPD.cfg、vendor_a/etc/peripheralconfig/pcl_config.bin、vendor_a/etc/acdbdata/anorak_max98388/workspaceFileXml.qwsp、vendor_a/etc/HRTF_KS.hrtf 内部的记录布局、system/etc/mldepth/stereo_model*.ptl（文件头字节 TH1）、system/etc/scene_objects/mask2former_v5.ptl（文件头字节 etz0）、system_ext_a/etc/assistant 下类 Caffe2 的 NetDef .pb 文件，以及 boot-image.prof 和 boot-image.bprof 前几个字节之后的内容。
- 32 个 zip 容器模型归档和 6 个 bodytracking 归档仅按条目名列出。未读取 producer_info.json、model_version_metadata 和 ExecuTorch 程序内容，也未追踪加载它们的库。
- threadpolicy.conf 不是严格的 JSON。未追踪 vrdevicemanagerserver 如何解析它，以及它是否容忍尾随逗号。
- cgroups.json、task_profiles.json、fonts.xml、linker.config.pb、fs_config_dirs 以及 BPF 对象的使用方仅通过字面字符串识别。未追踪运行时行为。
- 有 35 行被标记为推断，因为它们只依据文件名：除已读取的三个之外的 34 个断字许可证文本，以及 system/etc/seccomp_policy/code_coverage.arm.policy。
- bpfloader_min_ver 和 bpfloader_max_ver 段的字节以原始形式读取（00000000 和 00000100）。其整数宽度和字节序未经确认。
- SELinux 覆盖范围：字面量和正则搜索覆盖了挂载分区中的 file_contexts、property、service 和 seapp 上下文文件。未枚举通配规则，因此对未匹配组内文件的上下文覆盖并不完整。
- 未检查 init.insmod.sh（/vendor/bin/init.insmod.sh）和 vendor_a/etc/init.insmod.charger.cfg（不属于本组）。
- 引用搜索覆盖了挂载的 bin、lib、lib64 和 priv-app 目录中的 .rc 文件、*contexts* 文件以及字面字符串。其他形式的引用可能存在，例如编译进未列出二进制文件中的默认值。
