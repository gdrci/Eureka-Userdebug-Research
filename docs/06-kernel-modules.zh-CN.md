# 06 - 内核模块（vendor 与 ODM dlkm）

[English](06-kernel-modules.md)

## 摘要

内核模块组由两个只读 ext4 镜像中的 306 个文件组成：vendor_dlkm_a（271 个文件，262 个模块）和 odm_dlkm_a（35 个文件，27 个模块）。289 个 .ko 文件均为 64 位小端 AArch64 ELF 可重定位目标文件，全部针对内核 5.10.205-01655-g5c5a360e9c21 构建（vermagic 相同）。每个镜像都包含 modules.alias、modules.dep、modules.load、modules.softdep、build.prop、NOTICE.xml.gz（许可证列表为空）以及两个空的 fs_config 列表；vendor_dlkm_a 另外还包含 modules.blocklist。fstab.eureka 将这些镜像挂载到 /vendor_dlkm 和 /odm_dlkm，而 /vendor/lib/modules 和 /odm/lib/modules 是指向它们的符号链接。模块加载由 /vendor/bin/init.insmod.sh 完成，它读取 init.insmod.charger.cfg（charger 类，37 条具名 modprobe 条目）或 init.insmod.cfg（启动阶段，针对两个 modules.load 文件执行 modprobe -b *）。init.anorak.rc 还直接用 modprobe 加载 wlan 和 sched-walt。从 modinfo 和别名中可见的模块类别包括：vendor_dlkm_a 中的高通 SoC 平台驱动（时钟、互连、SMMU、GLINK/QRTR 传输、KGSL 图形、MSM DRM、cfg80211/mac80211 WLAN 协议栈、WCD/LPASS 音频编解码器、SPMI PMIC、UFS），以及 odm_dlkm_a 中 Oculus/Meta 外设的设备驱动（USB VDM、syncboss、传感器、稳压器、充电底座）。

角色表为该组中 306 个文件各提供一行（278 个 verified，0 个 observed，27 个 inferred，1 个 unverified）。该表位于 `data/subsystems/kernel-modules/roles.tsv`。

## 已验证结论

- **K01** (verified)：该组共 306 行：289 个 .ko 模块和 17 个其他文件。271 行位于 vendor_dlkm_a，35 行位于 odm_dlkm_a（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K01）。
- **K02** (verified)：全部 289 个 .ko 文件均为 ELF64 小端 AArch64 可重定位目标文件（e_machine 183，e_type REL），未剥离符号，带有 intree=Y，且 vermagic 字符串相同（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K02）。
- **K03** (verified)：289 个模块中有 261 个带有 modinfo 描述字符串；28 个没有（在 roles.tsv 中以 inferred 或 unverified 角色列出）（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K03）。
- **K04** (verified)：加载列表和依赖列表与两棵目录树中的 .ko 文件完整且一致：每个 .ko 都在 modules.load 中，modules.dep 每个模块一行（vendor 262 行，odm 27 行）（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K04）。
- **K05** (verified)：fstab.eureka 将 vendor_dlkm 挂载到 /vendor_dlkm，将 odm_dlkm 挂载到 /odm_dlkm，两者均为只读 ext4，并带有 AVB、logical 和 first_stage_mount 标志（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K05）。
- **K06** (verified)：/vendor/lib/modules 和 /odm/lib/modules 是指向 /vendor_dlkm/lib/modules 和 /odm_dlkm/lib/modules 的符号链接（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K06）。
- **K07** (verified)：init.insmod.sh 读取由 action|argument 行组成的配置文件。对于 modprobe，当参数为 '-b *' 时，它会根据两个 modules.load 文件构建参数列表，并以两个模块目录作为 -d 路径运行 modprobe。最后它会设置 vendor.all.modules.ready 和 vendor.all.devices.ready（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K07）。
- **K10** (verified)：init.anorak.rc 定义了两个 oneshot 服务，分别使用两个配置文件运行 init.insmod.sh。insmod_charger_sh 运行在 charger 类中，由 'on charger' 启动。insmod_sh 处于禁用（disabled）状态，由 'on init && property:vendor.skip.init=0' 启动（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K10）。
- **K11** (verified)：init.anorak.rc 直接为两个模块调用 modprobe，二者都以 u:r:vendor_modprobe:s0 身份并带 exec_background 运行：wlan（country_code=0）在 'on boot' 中加载，sched-walt 在 'on boot && property:vendor.scheduler.schedutil=false' 中加载（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K11）。
- **K12** (verified)：vendor_dlkm_a 的 modules.blocklist 有 69 行黑名单记录，涉及 68 个不重复的模块。其中 4 个被列入黑名单的名称与模块目录树中的 .ko 文件匹配：llcc_perfmon、sched-walt、usbip-vudc 和 wlan（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K12）。
- **K13** (inferred)：wlan 和 sched-walt 出现在 vendor 的 modules.load 中（第 189 和 216 行），因此 init.insmod.cfg 中的 modprobe -b * 路径会考虑它们。init.anorak.rc 则不带 -b 按名称加载这两个模块。其行为遵循通常的 modprobe -b 约定；modprobe 二进制文件不在证据范围内（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K13）。
- **K14** (verified)：SELinux 定义了 vendor_modprobe 域。init 可以切换到该域，并且该域可以读取 proc_modules。vendor_file_contexts 将 /vendor/bin/init.insmod.sh 标记为 init-insmod-sh_exec（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K14）。
- **K15** (verified)：平台文件上下文将 /vendor_dlkm 和 /odm_dlkm（以及它们带 vendor/ 前缀的形式）标记为 vendor_file。没有任何 vendor 文件上下文条目指定模块目录或 dlkm 挂载点（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K15）。
- **K17** (verified)：wlan.ko（vendor）：WLAN 主机设备驱动，双许可 BSD/GPL，staging 状态，直接依赖 cnss2、cfg80211、cnss_prealloc、cnss_nl 和 cnss_utils。它有六个模块参数，其中包括 country_code（charp），init 会将其设为 0。它未导出任何符号，导入 503 个全局未定义符号。其在 modules.dep 中的传递依赖闭包有 22 项（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K17）。
- **K18** (verified)：cfg80211.ko（vendor）：无线配置支持模块，GPL，无模块依赖。它有三个参数，声明了固件文件 regulatory.db 和 regulatory.db.p7s，为 NETLINK 协议族 nl80211 提供别名，并导出 129 个符号。wlan.ko 和 mac80211.ko 依赖于它（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K18）。
- **K19** (verified)：mac80211.ko（vendor）：IEEE 802.11 子系统，GPL，仅依赖 cfg80211。它有六个参数（ieee80211_default_rc_algo 为 charp，max_nullfunc_tries、max_probe_tries、beacon_loss_count、probe_wait_ms 为 int，minstrel_vht_only 为 bool），导出 119 个符号，导入 312 个符号（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K19）。
- **K24** (verified)：sched-walt.ko（vendor）：模块名 sched_walt，GPL v2，依赖 qcom-cpufreq-hw，导出 7 个 core-control 和 WALT 符号，导入 261 个符号。它没有描述字符串，位于黑名单中（第 46 行），并在 vendor.scheduler.schedutil 为 false 时由 init.anorak.rc 在启动时加载（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K24）。
- **K25** (verified)：qcom_ramdump.ko（vendor）：'Qualcomm Technologies, Inc. Ramdump driver'，GPL v2，无依赖，有一个 int 参数 enable_dump_collection，导出 6 个符号，导入 34 个符号。init.anorak.rc 根据 persist.vendor.ssr.enable_ramdump 写入该参数（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K25）。
- **K26** (verified)：usbvdm.ko（odm）：'Meta USB VDM abstraction library'，GPL v2，staging 状态，无依赖。它导出 14 个符号（usbvdm_register、usbvdm_subscribe、usbvdm_engine_vdm、usbvdm_subscriber_vdm 等），是五个 odm 模块的依赖项：pdfu_initiator、usbvdm-glink、external_battery、usbvdm-cypd 和 charging_dock。init.insmod.charger.cfg 与 usbvdm-cypd.ko 和 usbvdm-glink.ko 一起加载它（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K26）。
- **K27** (verified)：modules.dep 是 depmod 的输出：每个模块一行，格式为 '<绝对路径>: <以空格分隔的传递依赖路径>'，无依赖的模块对应空列表。vendor 路径使用 /vendor/lib/modules，odm 路径使用 /odm/lib/modules，即符号链接名称（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K27）。
- **K28** (verified)：modules.alias 是 kmod 别名输出，包含注释头和 'alias <key> <module>' 形式的行。键为 OF compatible 字符串（alias of:N*T*C<compatible>，并带有一行 C* 通配符）、platform:、i2c:、MHI 或 NETLINK 形式。vendor 文件共 841 行（含注释头）；odm 文件有 27 行别名（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K28）。
- **K30** (verified)：modules.softdep 保存 softdep 规则，格式为 'softdep <module> pre: <modules> [post: <modules>]'。vendor 文件有 16 条规则；odm 文件只有注释头，没有规则。示例：softdep msm_kgsl pre: arm_smmu nvmem_qfprom（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K30）。
- **K31** (verified)：两个 etc/build.prop 文件标识的是同一个构建：设备 eureka，Oculus 品牌，型号 'Quest 3'，Android 版本 12（SDK 32），构建 ID SQ3A.220605.009.A1，userdebug/test-keys，日期为 Tue Nov 19 17:09:18 PST 2024。两者都设置了 persist.sys.usb.config=none（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K31）。
- **K32** (verified)：两个镜像中的 fs_config_dirs 和 fs_config_files 文件均为零字节。两个 NOTICE.xml.gz 文件均为 75 字节，解压后为一个 licenses 元素为空的 XML 文档（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K32）。
- **K33** (verified)：roles.tsv 包含 306 行数据（另加一行表头），每个输入行对应一行，且 partition/path 键唯一。置信度标签：278 个 verified，27 个 inferred，1 个 unverified（blu_fw_driver.ko）（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K33）。
- **K34** (inferred)：289 个模块分为两组。vendor_dlkm_a 包含高通 SoC 平台驱动（时钟、互连、SMMU、GLINK/QRTR 传输、KGSL、MSM DRM、cfg80211/mac80211 WLAN、WCD/LPASS 音频、SPMI PMIC、UFS）。odm_dlkm_a 包含 Meta/Oculus 外设的设备驱动（USB VDM、syncboss、传感器、稳压器）。这种分组依据作者字符串、描述和 DT compatible，而非设备树（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K34）。

## 措辞已修正的结论

下方结论以数据所支持的措辞重新表述。

- **K08** (verified)：两个 insmod 配置共使用 38 条 modprobe 指令、4 条 enable 指令和 4 条 setprop 指令。init.insmod.charger.cfg 中的 37 个具名模块均可在两棵模块目录树中解析到对应的 .ko。只有一个条目 pmic-glink.ko 与其文件名（pmic_glink.ko）不同；其余 36 个完全匹配，因此该条目依赖名称规范化（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K08）。
- **K09** (verified)：init.insmod.cfg（启动配置）包含一条加载全部模块的 modprobe 指令 'modprobe|-b *'。其后是 setprop vendor.all.modules.ready，然后是针对 ADSP 启动、CDSP 启动和 CVP 启动节点（/sys/devices/virtual/cvp/cvp/boot）的 enable 指令，最后是 setprop vendor.all.devices.ready（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K09）。
- **K16** (verified)：init.anorak.rc 写入属于模块的 sysfs 路径：/sys/module/drm/parameters/vblankoffdelay（第 124 行）没有被两棵 dlkm 目录树中的任何模块声明，因此它不是 msm_drm.ko；将其归属于 drm 核心属于推断（模块目录树中没有 drm.ko）。它还写入 /sys/module/qcom_ramdump/parameters/enable_dump_collection（第 403 和 409 行），与 qcom_ramdump.ko 的 enable_dump_collection 参数相符（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K16）。
- **K20** (verified)：msm_kgsl.ko（vendor）：'3D Graphics driver'，GPL v2。直接依赖为九个模块（qcom_iommu_util、qcom-scm、cmd-db、llcc-qcom、clk-qcom、log_gpu_error、mdt_loader、secure_buffer、mem_buf_dev）。它有软依赖 pre arm_smmu nvmem_qfprom，匹配 DT compatible qcom,kgsl-3d0，导出 6 个符号，导入 506 个符号。其 modules.dep 传递闭包有 14 项。powerhint.cfg（不在 init 和 SELinux 范围内）是由电源 HAL 应用的 TOML 配置；它列出了 kgsl-3d0 的 devfreq 节点及其取值（例如第 177 行的 devfreq/max_freq = 690000000），但实际写入操作由使用者完成，而非由该文件完成（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K20）。
- **K21** (verified)：msm_drm.ko（vendor）：两个描述字符串（'MSM DRM Driver' 和 'MSM SMMU driver'），GPL 与 GPL v2 许可标签，作者为 Rob Clark。它有 13 个直接依赖，软依赖 pre msm-mmrm，七个参数（包括 dsi_display0、dsi_display1、modeset、vram），16 行 modules.alias 将 mdp4、mdss、sde-kms、smmu_sde_unsec、smmu_sde_sec、smmu_sde_nrt_unsec、smmu_sde_nrt_sec 和 msm-hdcp DT compatible 映射到 msm_drm（每个都有精确形式和 C* 通配符形式），导出 41 个符号，导入 701 个符号。其 modules.dep 传递闭包有 31 项。它由 init.insmod.charger.cfg 加载（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K21）。
- **K22** (verified)：qcom_glink.ko（vendor）：'Qualcomm GLINK driver'，GPL v2，无依赖，导出 8 个符号（qcom_glink_native_probe、qcom_glink_native_start、glink_native_pm_ops 等），导入 91 个符号。它出现在另外 53 个 vendor 模块的 modules.dep 行中，例如 audio_prm_dlkm.ko（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K22）。
- **K23** (verified)：qrtr.ko（vendor）：带有 nameserver 描述的高通 IPC 路由器驱动，GPL v2 与双许可 BSD/GPL 标签，无依赖。它是网络协议族 42（net-pf-42），导出 8 个符号（qrtr_endpoint_register、qrtr_ns_init、qrtr_get_service_id 等）。qmi_helpers.ko 有软依赖 pre qrtr。init.anorak.rc 定义了服务 vendor.qrtr-ns（第 324 行，class core，user 和 group 为 vendor_qrtr，NET_BIND_SERVICE），没有显式的 start 行，因此它作为 class core 服务运行；其运行用户为 vendor_qrtr（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K23）。
- **K29** (verified)：modules.load 是纯文本的 .ko 文件名列表，每行一个，不含路径和选项。vendor 有 262 项，odm 有 27 项，两个列表都是完整的模块集合（K04）。在为 -b * 路径构建的参数中，vendor 列表排在 odm 列表之前；顺序是否会改变加载顺序属于推断，未经验证。vendor 的前几项是 lzo-rle.ko、lzo.ko、qti-regmap-debugfs.ko、regmap-spmi.ko 和 zram.ko。odm 的第一项是 as5510.ko（证据见 `data/subsystems/kernel-modules/claims.json`，编号 K29）。

<!-- generated: modules start -->
## 模块索引与符号依赖（生成）

来源：`data/kernel/modules.tsv`（289 个 `.ko` 文件的 `.modinfo` 字段）和 `data/kernel/symbols.tsv`（每个模块的全局与弱符号）。每个文件的 SHA-256 都与清单相符。

- 模块：289 个（vendor_dlkm_a 262、odm_dlkm_a 27）。vermagic 全部相同：`5.10.205-01655-g5c5a360e9c21 SMP preempt mod_unload modversions aarch64`。
- 许可证：GPL v2 260、GPL 26、Dual BSD/GPL 3。
- `depends=` 值为空的模块：127 个。`alias=` 行共 867 行；`parm=` 行共 30 行，分布在 14 个模块中。含 `firmware=` 行的模块：1 个（`cfg80211`，`regulatory.db.p7s;regulatory.db`）。
- 未定义的全局符号引用：15,896 条，涉及 3,489 个不同符号。
  - 恰好由镜像中另一个模块定义：1,572 条引用。
  - 由多个其他模块定义（未归属）：0 条引用。
  - 镜像中没有任何模块定义：14,324 条引用。这些是来自内核或镜像中不存在的模块的导入；内核不在镜像中，因此未与内核核对。
- 从镜像中另一模块导入至少一个符号的模块：162 / 289 个。

向镜像中其他模块提供最多引用的模块：

| 提供模块 | 导入模块数 | 引用数 |
|---|---|---|
| `cfg80211` | 2 | 129 |
| `qcom-scm` | 25 | 110 |
| `clk-qcom` | 13 | 95 |
| `swr_dlkm` | 8 | 70 |
| `wcd_core_dlkm` | 9 | 67 |
| `lpass_cdc_dlkm` | 6 | 64 |
| `mem_buf_dev` | 10 | 62 |
| `qmi_helpers` | 9 | 61 |
| `wlan_firmware_service` | 1 | 61 |
| `mhi` | 5 | 52 |
| `cnss2` | 1 | 52 |
| `rproc_qcom_common` | 14 | 44 |

镜像中没有任何模块定义、出现最频繁的引用：

| 符号 | 引用数 |
|---|---|
| `__stack_chk_fail` | 235 |
| `__stack_chk_guard` | 235 |
| `printk` | 202 |
| `_dev_err` | 190 |
| `__platform_driver_register` | 177 |
| `devm_kmalloc` | 174 |
| `platform_driver_unregister` | 165 |
| `mutex_unlock` | 161 |
| `mutex_lock` | 158 |
| `kfree` | 148 |
| `__mutex_init` | 128 |
| `kmem_cache_alloc_trace` | 120 |

完整符号列表见 `data/kernel/symbols.tsv`（73,279 行）。

<!-- generated: modules end -->

## 缺口

- 未确定模块的运行时行为（probe 函数、创建的设备节点、启动后实际生效的参数值）；所有结论均来自静态分析。
- 未检查 modprobe 二进制文件（vendor_a/bin/modprobe），因此 K13 中关于黑名单和 -b 行为的结论属于推断，未经复现。
- 与 of: 别名键匹配的设备树节点位于该组之外的设备树 blob 中，未予解析。
- 模块所指定的固件文件（cfg80211 的 regulatory.db 和 regulatory.db.p7s，以及通过 wlan_firmware_service 获取的 WLAN 固件）未能在已挂载的分区中找到。
- 27 个模块的角色仅依据名称、别名或导出符号名称得出；blu_fw_driver.ko 未经验证；bcl_pmic5、bcl_soc、icc-test、qcom_iommu_debug、qti_qmi_sensor_v2、rimps_log 和 ddr_cdev 没有基于内容的确认。
- 镜像中没有任何模块定义的未定义符号未归属到具体提供者（见生成的小节）。内核不在镜像中，因此这些符号未与内核核对。
- 深入分析覆盖了 10 个模块、4 个配置或列表文件（modules.dep、modules.alias、modules.blocklist、modules.load），以及 init 和 SELinux 相关引用；其余 279 个模块仅根据 modinfo、别名或导出字符串进行分类。
- 仅为带有 parm 条目的模块提取了参数描述；模块中不包含参数默认值。
- 内核镜像和 CONFIG_ 选项不在该组中，因此未检查 5.10.205-01655-g5c5a360e9c21 的构建配置。
- init.insmod.sh（vendor_a/bin）和 init.anorak.rc 属于其他子系统，仅作为模块加载相关的引用。
- 已定位但未完整分析其他配置文件中对模块提供节点的引用（ueventd.rc 中的 kgsl 和 wcd 节点，powerhint.cfg 中的 sched_walt 和 kgsl 路径）。
