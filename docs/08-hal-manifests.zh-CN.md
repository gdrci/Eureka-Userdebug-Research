# 08 - HAL 清单（VINTF）

[English](08-hal-manifests.md)

## 摘要

hal-manifests 组由 62 个 VINTF XML 文件构成，这些文件全部是挂载分区上仅有的 etc/vintf 文件：vendor_a 20 个，odm_a 18 个，system_ext_a 12 个，system_a 11 个，product_a 1 个。没有任何文件是二进制、库或固件镜像。这些文件为框架（system_a manifest.xml、基于级别的兼容性矩阵 3 至 6，以及一个设备矩阵）、system_ext、vendor 和 odm 各层声明 HAL 的包名、版本、传输方式和实例。大多数已声明的 HAL 也在 init 服务条目以及 SELinux 的 hwservice_contexts 或 service_contexts 文件中被提及。基于名称的交叉引用将 414 个清单和矩阵条目与上下文文件进行了匹配；另有 7 个矩阵需求名称在上下文文件中没有对应条目。两个设备矩阵需求（vendor.oculus.hardware.mixedreality 和 vendor.oculus.hardware.peripheralconfig@1.0，均为可选）在这 62 个文件中没有提供方。对于两个 odm HAL（sensors 和 usb），init rc 或 hwservice 上下文声明的接口或版本比清单片段更多。

角色表为本组中的 62 个文件各提供一行（59 verified，0 observed，3 inferred，0 unverified）。它位于 `data/subsystems/hal-manifests/roles.tsv`。

## 已验证结论

- **GRP-01** (verified)：该组有 62 行，这些正是挂载分区上存在的 62 个 etc/vintf 文件。按分区统计：vendor_a 20，odm_a 18，system_ext_a 12，system_a 11，product_a 1。其他分区（odm_dlkm_a、vendor_dlkm_a、scratch）均没有 vintf 文件。
- **GRP-02** (verified)：这 62 个文件的 sha256 值均与 classified.tsv 中记录的值一致，且全部 62 个文件都是格式良好的 XML。
- **GRP-03** (verified)：在这 62 个文件中，56 个的根元素为 manifest，6 个的根元素为 compatibility-matrix。HAL 格式属性：hidl 309，aidl 61，native 1，未设置 14。传输方式：hwbinder 57，passthrough arch=32+64 5。hal 上的 optional 属性：true 268，false 38，未设置 79。hal 上的 override 属性：2。
- **GRP-05** (verified)：框架 manifest system_a/system/etc/vintf/manifest.xml 的版本为 4.0，类型为 framework，包含 9 个 HAL 声明，并支持 system-sdk 版本 28 至 32。它声明了 android.frameworks.displayservice IDisplayService、android.frameworks.schedulerservice ISchedulingPolicyService（max-level 5）、android.frameworks.sensorservice ISensorManager、android.hidl.manager IServiceManager@1.2、android.hidl.memory IMapper（passthrough，arch 32+64，instance ashmem）、android.hidl.token ITokenManager、android.system.net.netd INetd@1.1、android.system.wifi.keystore IKeystore，以及一个 native 类型的 netutils-wrapper@1.0 条目。
- **GRP-07** (verified)：框架兼容性矩阵（级别 3、4、5 和 6）分别有 57、61、66 和 78 个 HAL 需求条目。每个矩阵都包含本级别的内核需求行，例如级别 6 要求 4.19.191（8 个条目）、5.4.86（9 个）和 5.10.43（9 个）。它们的头部注释提到 matrix_kernel_config_*.xml 输入，而这些文件在挂载分区上都不存在。
- **GRP-08** (verified)：设备 manifest vendor_a/etc/vintf/manifest.xml 的版本为 4.0，类型为 device，target-level 为 4。它有 8 个 HAL 声明（其中包括 android.hardware.bluetooth IBluetoothHci、android.hardware.keymaster @4.1 IKeymasterDevice、android.hardware.media.omx IOmx 和 IOmxStore、vendor.qti.hardware.qseecom IQSEECom）。它的 sepolicy 版本为 32.0，内核 target-level 为 6。平台 sepolicy 版本文件的内容为 32.0。
- **GRP-09** (verified)：设备兼容性矩阵 vendor_a/etc/vintf/compatibility_matrix.xml 的版本为 4.0，类型为 device。它列出了 6 个框架 HAL 需求（android.frameworks.sensorservice ISensorManager、android.hidl.allocator IAllocator、android.hidl.manager IServiceManager、android.hidl.memory IMapper、android.hidl.token ITokenManager、android.system.wifi.keystore IKeystore）、vendor-ndk 32 和 system-sdk 32。
- **GRP-10** (verified)：system_ext_a/etc/vintf/manifest.xml 未声明任何 HAL。它的唯一内容是版本为 32 的 vendor-ndk 条目，并带有一条 XML 注释，将 system_ext_manifest.default.xml 列为输入。
- **GRP-12** (verified)：android.hardware.security.keymint-service-qti.xml 在包 android.hardware.security.keymint 下声明了四个 AIDL 条目（sharedsecret/secureclock）：IKeyMintDevice/default、ISharedSecret/default、ISecureClock/default 和 IRemotelyProvisionedComponent/default。对应的 SELinux service_contexts 条目位于 plat_service_contexts 第 16 至 19 行。init 服务 vendor.keymint-qti 运行 /vendor/bin/hw/android.hardware.security.keymint-service-qti（class early_hal，挂载分区上为 36656 字节）。
- **GRP-13** (verified)：android.hardware.light@2.0-service.anorak.xml 声明了 android.hardware.light@2.0 ILight 和 vendor.oculus.hardware.light@2.0 ILight，两者都是通过 hwbinder 传输的 hidl，实例为 default。init 服务 light-hal-2-0 注册了相同的两个接口，并运行 /vendor/bin/hw/android.hardware.light@2.0-service.anorak（101480 字节）。SELinux 将 vendor.oculus.hardware.light::ILight 映射到 hal_light_hwservice。AIDL 名称 android.hardware.light.ILights/default 出现在 plat_service_contexts 中，但没有任何组内 manifest 声明它。
- **GRP-15** (verified)：odm 的 android.hardware.usb@1.2-service.xml 声明了 IUsb 1.2 和 IUsbGadget 1.2。odm 的 init 服务 vendor.usb-hal-1-2 为 /odm/bin/hw/android.hardware.usb@1.2-service.ovr（193360 字节）注册了 IUsb 1.0、1.1 和 1.2，以及 IUsbGadget 1.0、1.1 和 1.2。因此 init 条目列出的接口版本包含比清单更早的版本。SELinux 的 hwservice 上下文包含 android.hardware.usb::IUsb 和 android.hardware.usb.gadget::IUsbGadget。清单文件名省略了二进制文件所带的 .ovr 后缀。
- **GRP-16** (verified)：odm 的 perfservice.xml 声明了 IPower 1.3（android.hardware.power）、IThermal 1.0 和 2.0（android.hardware.thermal），以及 IClocks 1.0（vendor.oculus.hardware.clocks），均为 hwbinder 传输，实例为 default。odm 的 perfservice.rc 启动 /odm/bin/hw/perf_service（501832 字节），并注册 IPower 1.0 至 1.3、IThermal 1.0 和 2.0，以及 IClocks 1.0。其重启由 init.svc.zygote=restarting 条件控制。它的 hwservice 标签位于 plat_hwservice_contexts（power 和 thermal）以及 system_ext_hwservice_contexts（clocks）中。
- **GRP-17** (verified)：system_ext 的 tracking_manifest.xml 在名称 oculus.internal.tracking 下声明了四个可选 AIDL HAL：IFaceEyeTrackingService、IHandTrackingService、IInsideOutBodyTrackingService 和 IMemoryBrokerService，实例均为 default。system_ext 的 service_contexts 文件列出了七个 oculus.internal.tracking 条目（第 60 至 66 行），其中包括本清单未声明的 IEyeTrackingService 和 ITrackingService。第 60 行是 IAssetReceiver/encoder。
- **GRP-20** (verified)：组内只有两个 HAL 条目带有 override=true。两者都位于 system_ext：vendor.oculus.hardware.composer-service.xml 为实例 display 覆盖了 android.hardware.graphics.composer 2.4 IComposer；emgservice_manifest.xml（manifest 版本 2.0）覆盖了 com.meta.wearable.emg IEmgService/default（AIDL，可选）。
- **GRP-21** (verified)：system_a 中有两个框架片段的 SELinux 标签相互匹配。android.frameworks.stats@1.0 IStats 的 max-level 为 5，并带有一个 aidl IStats/default 条目；plat_hwservice_contexts 将 android.frameworks.stats::IStats 映射到 fwk_stats_hwservice，plat_service_contexts 将 android.frameworks.stats.IStats/default 映射到 fwk_stats_service。android.system.keystore2-service.xml 声明了 AIDL IKeystoreService/default；plat_service_contexts 将其映射到 keystore_service。
- **GRP-22** (verified)：clearkey 和 widevine DRM 清单除实例名不同外完全相同。两者都是 945 字节，均声明了 android.hardware.drm，其中 IDrmFactory 和 ICryptoFactory 版本为 1.4，通过 hwbinder 传输。只有 fqname 行不同。
- **GRP-23** (verified)：product_a 的 crashuploaderservice-default.xml 声明了 AIDL meta.internal.diagnostics ICrashUploaderService/default。该名称出现在 system_ext_service_contexts 第 24 行，并且 product_a/etc/init/ovr_crashuploader.rc 和 product_a/bin/ovr_crashuploader 都存在于挂载分区上。未读取该 init rc 文件。
- **GRP-25** (inferred)：vintf 目录遵循同一种布局。vendor_a/etc/vintf 包含 manifest.xml、compatibility_matrix.xml 和一个 manifest/ 目录。odm_a/etc/vintf 和 product_a/etc/vintf 各自只包含一个 manifest/ 目录。system_ext_a/etc/vintf 包含 manifest.xml 和 manifest/。system_a/system/etc/vintf 包含 manifest.xml、compatibility_matrix.*.xml 和 manifest/。因此，框架或设备加载器很可能将每个分区的 manifest.xml 与其 manifest/ 片段组合成一个 manifest。
- **GRP-26** (inferred)：分区到路径的映射与 init 服务路径一致：odm_a 对应 /odm，vendor_a 对应 /vendor，system_ext_a 对应 /system_ext，system_a/system 对应 /system。示例包括 /odm/bin/hw/perf_service、/vendor/bin/hw/android.hardware.light@2.0-service.anorak 和 /system_ext/bin/bpfagent。

## 修正措辞后的结论

下方结论以数据所支持的措辞重新表述。

- **GRP-04** (observed)：三个文件被归类为 text/xml（parfait.xml 和两个 DRM 清单），file(1) 对这三个文件的判断与该分类一致，均报告为 XML 1.0 document。在 59 个 text/plain 文件中，file(1) 对其中 18 个（例如 system_a/system/etc/vintf/manifest.xml）报告为 exported SGML document，对其中 41 个报告为 plain ASCII text。
- **GRP-06** (verified)：由 56 个 manifest 根元素（在 62 个文件中）提供的名称级覆盖率为：30 个设备矩阵名称中有 28 个。没有提供方的两个名称是 vendor.oculus.hardware.mixedreality（optional true）和 vendor.oculus.hardware.peripheralconfig 1.0（optional true）。完整 HAL 名称 vendor.oculus.hardware.peripheralconfig 仅出现在挂载分区上的 system_a/system/etc/vintf/compatibility_matrix.device.xml 中；裸词 peripheralconfig 还出现在其他 10 个文件中，作为库、工具和路径名称出现。
- **GRP-11** (verified)：这四个接口的 SELinux 标签出现在三个上下文文件中：vendor_a/etc/selinux/vendor_hwservice_contexts（IQtiComposer 位于第 9 行，vendor.display.config::IDisplayConfig 位于第 2 行）、vendor_a/etc/selinux/vendor_service_contexts（AIDL vendor.qti.hardware.display.config.IDisplayConfig/default 位于第 4 行），以及 system_a/system/etc/selinux/plat_hwservice_contexts（android.hardware.graphics.composer::IComposer 位于第 36 行）。
- **GRP-14** (verified)：四个 odm init rc 文件从 /odm/bin/hw 启动 vendor.oculus.hardware.sensors@1.0 服务，分别是 debug、mock、iad 和 flicker 对应的 rc 文件。第五个 odm rc 文件 vendor.oculus.hardware.sensorlock-service.rc 也从 /odm/bin/hw 启动一个服务（一个 AIDL sensorlock 服务）。当 persist.vendor.oculus.sensor.service.impl=sensorservice 时，debug 服务随开机启动，而 mock 服务被禁用。18 个 hwservice 条目位于 vendor_a/etc/selinux/vendor_hwservice_contexts 第 31 至 48 行。
- **GRP-18** (verified)：bpfagent_manifest.xml 声明了版本为 1 的 AIDL vendor.oculus.hardware.bpfagent，包含 IConfigurator/default 和 IStrobemeta/default，且未标记为可选。system_ext 的 init 服务 bpfagent 将这两者声明为 aidl 接口，并且启动时处于禁用状态。SELinux 将这些名称映射到 bpfagent_configurator_service 和 bpfagent_strobemeta_service。CIL 策略源文件 vendor_a/etc/selinux/plat_pub_versioned.cil（文本，而非已编译的二进制策略）在第 666 至 671 行定义了 hal_bpfagent_* 类型属性。
- **GRP-19** (verified)：instsampd_manifest.xml 声明了一个可选 AIDL HAL：oculus.internal IInstructionSampler/default。system_ext 的 init 服务 instsampd（/system_ext/bin/instsampd，396880 字节）声明了同一接口，运行于 late_start 类中，并在启动时处于禁用状态。它的启动、停止和启用触发器受 persist.device_config.oculus_shared_os_services.oculus_mobile_instruction_sampler_enabled 和 ro.vendor.build.type 控制；另外两个属性触发器（ovr.instsampd.kptr_restrict_access 和 ovr.instsampd.set_perf_event_limits）各自作用于自己的属性。SELinux 将该名称映射到 instsamp_service。
- **GRP-27** (verified)：兼容性矩阵包含 hidl（261）和 aidl（37）需求。在 298 个矩阵需求条目中，260 个为 optional true，38 个为 optional false。在这 62 个文件的全部 385 个 hal 元素中，268 个为 optional true，38 个为 false，79 个没有该属性（后两组包括清单条目）。框架矩阵 compatibility_matrix.device.xml 的 30 个条目全部为 optional true；设备类型矩阵 vendor_a/etc/vintf/compatibility_matrix.xml 的 6 个条目全部为 optional false。

## 已排除的结论

下方声明已排除；每条都给出了原因。

| ID | 结论 | 原因 |
|---|---|---|
| GRP-24 | 对组内全部 385 个 hal 元素（421 个 interface/fqname 对：100 个来自清单，321 个来自矩阵）与 hwservice_contexts 和 service_contexts 文件进行名称级交叉引用：414 个对与上下文标签匹配，7 个不匹配。这 7 个不匹配的需求全部是兼容性矩阵条目，且均未被组内任一清单声明：android.hardware.wifi.offload IOffload（矩阵 3）、vendor.oculus.hardware.peripheralconfig IPeripheralConfig（device）、android.hardware.automotive.occupant_awareness IOccupantAwareness（矩阵 5 和 6）、android.hardware.automotive.sv ISurroundViewService（矩阵 5 和 6），以及 aidl android.hardware.neuralnetworks IDevice（矩阵 6，regex 实例）。矩阵 6 中的 hidl neuralnetworks IDevice 需求与 plat_hwservice_contexts 第 50 行匹配。 | 在 421 个 interface/fqname 对中，414 个与上下文标签匹配，7 个不匹配。这 7 个不匹配的需求全部是兼容性矩阵条目，且均未被组内任一清单声明：android.hardware.wifi.offload IOffload（矩阵 3）、vendor.oculus.hardware.peripheralconfig IPeripheralConfig（device）、android.hardware.automotive.occupant_awareness IOccupantAwareness（矩阵 5 和 6）、android.hardware.automotive.sv ISurroundViewService（矩阵 5 和 6），以及 aidl android.hardware.neuralnetworks IDevice（矩阵 6，regex 实例）。矩阵 6 中的 hidl neuralnetworks IDevice 需求与 plat_hwservice_contexts 第 50 行匹配。 |

## 缺口

- 组内没有二进制文件、库或固件（这 62 行全部是 VINTF XML）。init 所命名的二进制文件仅检查了是否存在及其大小，未作分析（没有 ELF 依赖或符号审查）。
- 挂载分区上没有 HIDL .hal 和 AIDL 接口源码。方法签名、数据类型以及声明版本号之外的版本兼容性均未确定。
- 兼容性矩阵头部提到的内核配置输入（matrix_kernel_config_*.xml）缺失，因此内核需求行无法与内核配置匹配。
- 基于名称的覆盖率统计仅使用这 62 个组内清单。挂载分区上不存在其他清单，因此 vendor.oculus.hardware.mixedreality 和 vendor.oculus.hardware.peripheralconfig 在挂载分区上没有提供方。设备镜像中挂载分区之外是否存在提供方，尚未确定。
- 交叉引用仅按名称和实例进行。hwservice_contexts 条目不包含版本，因此未针对声明的版本进行版本级匹配。
- SELinux 策略编译文件（plat_pub_versioned.cil、vendor_sepolicy.cil）仅搜索了名称匹配项。未分析各 HAL 的域允许规则。
- 仅对明确指向组内 HAL 的 vendor、system 和 product HAL 服务的 init rc 文件进行了阅读。未与组内 HAL 关联的服务（例如 ovr_crashuploader.rc）未被阅读。
- 已记录 init 接口行与清单之间的版本不匹配：usb（init 列出 1.0 至 1.2，清单仅 1.2）和 perfservice（init 列出 IPower 1.0 至 1.3，清单仅 1.3）。这些不匹配是否影响运行时行为未经测试。
- sensors 上下文文件列出 vendor.oculus.hardware.sensors 包的 18 个接口，odm 清单列出 10 个。运行时注册表内容未被观察，因此不知道传感器服务实际注册了哪些接口。
- 未观察运行时行为（服务启动、向 hwservicemanager 或 servicemanager 注册、启动时的 VINTF 检查）。关于运行时行为的所有陈述均仅基于文件内容。
- 这些 XML 文件的 file(1) 标签不一致（XML document 与 exported SGML document）。文件类型标签未被用作证据，仅记录了这一不一致。
