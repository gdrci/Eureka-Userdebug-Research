# 15 - 传感器、MCU 与 Meta HAL

[English](15-sensors-mcu-meta-hal.md)

## 摘要

sensors-mcu-meta-hal 组共有 263 行：259 个普通文件和 4 个符号链接，分布于 odm_a（43）、product_a（19）、system_a（67）、system_ext_a（123）和 vendor_a（11）。其核心是 vendor 传感器 HAL：一个位于 odm 分区的 HIDL 服务（vendor.oculus.hardware.sensors@1.0-service），基于 vendor.oculus.hardware.sensors@1.0.so 构建。它读取位于 /vendor/etc/sensorservice.cfg 的 TOML 配置，并在启动时由 persist.vendor.oculus.sensor.service.impl 选择，odm_a/etc/build.prop 将其设为 sensorservice。旁边还有若干替代实现：一个模拟 HAL（已禁用，无启动触发器）、sensoremu 与 sensor-emulator，以及 flicker 和 IAD HAL 服务。框架侧是 system 分区的 sensorservice 守护进程，包含 libsensorservice.so 以及 android.hardware.sensors 1.0/2.0/2.1 客户端库，sensorproxy 则运行于 system_ext。同一分类中的相邻 HAL 包括 sensorlock AIDL HAL、catty、dock、telemetry 和 devicecert，以及 feature 与 privileged-permission XML。该组还包含仅 IMU 的 TorchScript 跟踪器归档、PyTorch Lite 微手势归档，以及 oculus-device 的 odm keylayout 和 keychar。证据中的路径均相对于只读挂载根目录 <image>/mnt。

角色表为本组的 263 个文件各提供一行（175 verified、13 observed、75 inferred、0 unverified）。该表位于 `data/subsystems/sensors-mcu-meta-hal/roles.tsv`。

## 已验证声明

- **C01** (verified)：所有 263 条输入行在 roles.tsv 中各出现且仅出现一次（表头加 263 条数据行），分区与路径键完全一致。置信度计数：verified 175，observed 13，inferred 75。
- **C02** (verified)：对全部 259 条普通文件行重新计算的 sha256 摘要，与清单中的摘要一致。没有两条普通文件行共享同一摘要。
- **C03** (verified)：组成：odm_a 43 行、product_a 19 行、system_a 67 行、system_ext_a 123 行、vendor_a 11 行。类型：259 个文件，4 个符号链接。符号链接为两个指向 ../boot-com.oculus.os.platform.vdex 的 ART vdex 链接（system_a arm64 与 arm）、system_a/system/bin/touch 指向 toybox，以及 vendor_a/bin/touch 指向 toybox_vendor。
- **C04** (verified)：多组同大小的库在不同分区中内容不同：odm_a 与 system_ext_a 中的 lib64/vendor.oculus.hardware.sensors@1.0.so 副本均为 1534496 字节，但摘要不同；telemetry@1.0.so 在 odm_a、product_a 和 system_ext_a 中均为 87296 字节，但有三个不同的摘要。
- **C05** (verified)：传感器 HAL 服务是一个已剥离符号的 aarch64 PIE，解释器为 /system/bin/linker64。其 NEEDED 列表包括 libsensorutils.so、libvrsensors-hidlwrapper.so、android.hardware.sensors@1.0.so 与 @2.0.so、vendor.oculus.hardware.sensors@1.0.so、vendor.oculus.hardware.sensors_java@1.0.so、vendor.oculus.hardware.dock@1.0.so、libsyncboss.so、libcamerahal.so、libsensortelemetry.so、libdevicesensors.so 和 libtrackingvendorutils.so。所检查的全部 NEEDED 库均存在于已挂载分区中。
- **C06** (verified)：传感器 HAL 二进制读取 /vendor/etc/sensorservice.cfg，并解析 TOML 键 default_rate 与 fallback。日志消息涵盖 IMU 与磁力计速率解析，以及回退到 TOML 默认速率的情况。
- **C07** (verified)：sensorservice.cfg 是 TOML 文件，其顶层为 nice_constellation = "ruby"，[imu.default_rate] fallback = 800，[mag.default_rate] fallback = 25。注释称设备特定键必须与 libdeviceid 中的 kDeviceVersionName 匹配。相同的 nice_constellation 行也出现在 mrsystemservice.cfg 和 trackingservice.cfg 中。
- **C09** (verified)：模拟传感器 HAL 服务 vendor.oculus.sensors-mock-hal-1-0 声明为已禁用，其 rc 文件中没有启动触发器。其 onrestart 行会重启 calibration_svr、mrsystemservice、sensorproxy 和 trackingservice。
- **C10** (verified)：odm 传感器 HAL 的 VINTF manifest 声明了 10 个接口（ICameraProvider、IControllerProvider、IExternalTimingProvider、IFlickerSensor、IIad、IImu、IMag、ISensorsMiscDebug、IStructuredLightProjector、IZapper）。HIDL 库中包含 40 个不同的 vendor.oculus.hardware.sensors@1.0::<name> 字符串。SELinux hwservice 上下文列出 19 个与传感器相关的条目，包括 IAccel、ICamera、ICameraStream、IGyro、IEventHaptics 和 IPowerstate。这些列表并不一致，运行时注册情况无法确定。
- **C11** (verified)：vendor 传感器 HIDL 库导出了约 25 个具名接口的 HIDL 接口符号，包括 ICameraStream、IControllerStateProvider、IEventHaptics、IFaceEyeTrackingLed、IMotionSensorProvider 和 ISensorClient，它们超出了 manifest 所列范围。其 SONAME 为 vendor.oculus.hardware.sensors@1.0.so。
- **C12** (verified)：传感器模拟器由两个 init 服务提供。服务 vendor.sensoremu（/odm/bin/sensoremu）以 root 身份运行，为 oneshot 且已禁用。服务 vendor.sensor-emulator（/odm/bin/sensor-emulator）结构相同。两者的启动触发器均由传感器实现属性控制。SELinux 将这两个二进制文件映射到 hal_oculus_sensors_default_exec，并将其数据目录 /data/vendor/misc/sensoremu 与 /data/vendor/misc/sensor-emulator 分别映射到 sensoremu_odm_file 与 sensor-emulator_odm_file。
- **C13** (verified)：sensorlock HAL 是一个 AIDL 服务，注册为 vendor.oculus.hardware.sensorlock.ISensorLock/default。它与一个 AIDL NDK stub 库（ISensorLock、ISensorLockListener）、一个链接 libsmcihandler.so 的 libsensorlock.so 后端，以及一个 /dev/sensorlock 节点配对。该节点标记为 sensorlock_device，其 ueventd 模式为 0444 system system。
- **C14** (verified)：IAD 与 flicker 传感器 HAL 服务是独立的二进制文件。IAD 为服务 vendor.oculus.iad-hal-1-0，位于 early_hal 类中，提供 IIad，并使用 task_profiles trackingPolicy。Flicker 为服务 vendor.oculus.flicker-hal-1-0，位于 hal 类中，提供 IFlickerSensor。flicker 后端 libflickersensor.so 链接 libstals.so 与 libtelemetry.so。SELinux 为 flicker 与 IAD 二进制文件添加标签，并将 flicker 设备节点 /dev/vd6281 和 /dev/vd6281_spi 映射出来。
- **C15** (verified)：catty、dock 与 telemetry HAL 服务是独立的二进制文件。Catty（ICatty、ICattyPipeClient）以服务 vendor.catty-1-0 运行。Dock（IDock、IDockstateCallback）以服务 vendor.dock-1-0 运行。Telemetry（IDeviceTelemetry）从 system_ext 运行，名为 oculus-hardware-telemetry-1-0。兼容性矩阵将 catty 与 dock 列为可选的 HIDL HAL，hwservice 上下文则将它们映射到专用的 hwservice 类型。
- **C16** (verified)：sensorproxy 是一个 system_ext 服务（/system_ext/bin/sensorproxy，类 late_start，用户 system，已禁用）。仅当 persist.ovr.enable.sensorproxy=true 时才在启动时拉起，当其为 false 时停止。其二进制链接 libsensorclientutils、libsensordatatransforms、libcameramuxmodemonitor 和 libfaceeyetrackingledprovider。其文件上下文为 sensorproxy_exec。
- **C17** (verified)：系统 sensorservice 守护进程创建 SensorService，启动 binder 线程池，并注册到默认 service manager。它链接 libsensorservice.so、libsensorprivacy.so 和 libbinder.so。其服务名 sensorservice 映射到 sensorservice_service，android.hardware.sensors::ISensors 映射到 hal_sensors_hwservice。
- **C18** (verified)：libsensorservice.so（lib64）导出 SensorService 构造函数，以及 addProximityActiveListener 和 removeProximityActiveListener 方法。该库读取 debug.sensors 属性，其日志字符串包括 Sensors HAL died, attempting to reconnect。
- **C20** (verified)：sensor_tool 是用于传感器与相机流调试的命令行工具。它解析 TOML 与 JSON 配置（策略值 Realtime、Normal 和 Batch），将帧转储到由 --out 指定的目录，并链接 libEGL、libGLESv3、libgui 和 libvrsensors-hidlwrapper。sensor_bench 是基于 FMQ 的基准测试命令行工具，用于读写 CPU 频率。
- **C21** (observed)：sensor_validation 由系统 svt.sh 脚本调用，使用形如 SensorValidation_<device>_Stu 的设备配置名，例如 SensorValidation_Eureka_Stu。独立变体链接 vendor.oculus.hardware.sensors@1.0.so 和 libsensorclientutils.so。
- **C22** (verified)：组内 ELF 引用的传感器属性与配置键为：persist.vendor.ovr.imu.direct_channel（odm_a 与 system_a 中的 libvrsensors-hidlwrapper.so）、persist.vendor.oculus.sensor.service.enable_high_frequency_telemetry 与 persist.vendor.oculus.sensor.service.disable_telemetry_submission（odm_a 与 system_a 中的 libsensortelemetry.so），以及 debug.sensors（system_a 中的 libsensorservice.so）。
- **C24** (verified)：handtracking 微手势文件是带有 archive/ 前缀的 PyTorch Lite zip 归档，位于 nimble_gesture/microgesture/model 路径下。MGV23、MGV23MG、MGV24MG 和 MGV25MG 是四个变体。跟踪引擎库包含字符串 MGV23、MGV23MG 和 MGV24MG。在所扫描的二进制文件中未发现 MGV25MG 字符串。同一目录中存在空的 .placeholder 文件。
- **C25** (verified)：oculus-device 键布局将扫描码 172 映射到 HOME，159 映射到 FORWARD。键字符映射设置 FORWARD，并以 HOME 作为基础回退。odm IDC 文件 Vendor_2833_Product_0160.idc（不在本组内）将 keyboard.layout 与 keyboard.characterMap 绑定到 oculus-device。
- **C27** (verified)：com.oculus.os.platform 的启动镜像产物为 ART 与 OAT 文件。arm64 OAT 为 ELF。arm64 ART 文件以魔数 art 加 099 开头。oculus-system-services vdex 以 vdex027 开头。arm64 odex 为 ELF。
- **C28** (verified)：传感器 VINTF 数据将 sensorlock AIDL HAL 与传感器 HIDL HAL 作为可选条目列入设备兼容性矩阵。该矩阵列出了传感器 HIDL 接口 ICameraProvider 等，但并未列出库中的每个描述符。

## 修正措辞后的声明

这些声明在实质上成立。下列措辞替换原文。

- **C08** (verified)：debug init 文件定义了 vendor.oculus.sensors-hal-1-0（类 hal，用户 system，组 system camera，capabilities SYS_NICE，task_profiles sensorServicePolicy，接口 vendor.oculus.hardware.sensors@1.0::IExternalTimingProvider default，已禁用）。其唯一的启动触发器在 persist.vendor.oculus.sensor.service.impl=sensorservice 时启动它，该值由 odm_a/etc/build.prop 第 47 行设置。sensoremu 与 sensor-emulator 服务使用同一属性，其值分别为 sensoremu 和 sensor-emulator。该服务还会在运行时由 shell 脚本 virtualcontroller_ctl（system_ext_a/bin）和 syncboss_consumers_ctl（odm_a/bin）按名称启动和停止。
- **C19** (verified)：android.hardware.sensors@2.1.so（lib64）是 HIDL 2.1 客户端库，SONAME 为 android.hardware.sensors@2.1.so。它链接 android.hardware.sensors@1.0.so 和 android.hardware.sensors@2.0.so，以及 libhidlbase.so、liblog.so、libutils.so、libcutils.so、libc++.so、libc.so、libm.so 和 libdl.so。它在 2.1 命名空间中定义了 ISensors 与 ISensorsCallback。其 AOSP 来源为推断，组数据并未显示。
- **C23** (verified)：IMU 跟踪器归档为 TorchScript zip 容器，每个包含 29 个文件条目，且无目录条目。每个都有 *_final.pt 名称、data.pkl、bytecode.pkl、内容为 3 的 version 文件、extra/producer_info.json、code/ 和 constants/ 条目。左、右归档分别为 278366 和 278395 字节。libtrackingengines.so 包含路径字符串 /system/etc/input_tracking/runtime/eureka/ruby，指向该目录。
- **C26** (verified)：94 个 XML 行为 permission、default-permissions 与 sysconfig 文件。product_a 集合中有 17 个文件声明了 14 个 feature 元素和 3 个 privapp-permissions 块。角色文本由构建脚本根据各文件的标签为 94 个 XML 行中的 93 个推导得出；system_a/system/etc/sysconfig/oculus-backup-transport.xml 的角色为手写。system_ext_a 的 permission XML 为众多包声明了特权权限（64 个文件中的 45 个）。
- **C30** (inferred)：system_a/system/framework/arm 下的 32 位 boot-com.oculus.os.platform .art 与 .oat、system_a framework 的 boot-com.oculus.os.platform.vdex、oculus-system-services.art（arm64），以及 Provision 与 QuestManagedProvisioning 的 odex 和 vdex 文件（arm64），仅依据名称和路径分类（置信度为 inferred）。其魔数与内部结构未经检查。arm64 boot-com 的 .art 与 .oat，以及 oculus-system-services 的 .odex 与 .vdex 已经检查，并标记为 verified。
- **C31** (observed)：odm 的 sensors_java HAL（lib64，接口 IPowerstate）在 odm VINTF manifest 中仅声明了 IPowerstate；IPowerstateCallback 出现在库中，但不是 manifest 接口。IPowerstate 描述符映射到 hal_oculus_sensors_hwservice。system_ext_a 中存在同名库，包括 32 位与 64 位版本；64 位的 system_ext_a 副本与 odm_a 副本大小相同，但摘要不同，因此这些副本并非逐字节相同。

## 已排除的声明

这些声明在复现时未成立。

| ID | Claim | Reason |
|---|---|---|
| C29 | 组内权限与 feature 标签存在于 13 个 system_a 与 system_ext_a 包中。示例：com.oculus.mrservice 特权权限，com.ocul | 四个示例均成立：com.oculus.mrservice 具有 privapp-permissions 块，并有默认 USE_SCENE 授权（fixed=true）；com.oculus.xrstreamingclient 具有默认 RECORD_AUDIO（fixed=false）以及 POST_NO |

## 空白与局限

- 未进行反汇编或反编译。HIDL 和 AIDL 方法签名、调用图与运行时行为均未提取；接口事实仅限于描述符字符串和导出符号。
- 所有 ELF 二进制文件均已剥离或仅有少量符号，因此除证据中列出的导入与字符串外，它们的角色均基于名称推断（75 行为 inferred）。
- 某台设备实际运行哪个 HAL 实例，由运行时属性决定。仅检查了 odm_a/etc/build.prop 中的默认值（sensorservice）。其他属性来源，例如 persist 数据、/data 覆盖或 vendor_a/build.prop，均未针对该键进行搜索。
- 接口列表彼此不一致：odm manifest 声明 10 个接口，库中有 40 个 vendor.oculus.hardware.sensors@1.0:: 字符串，SELinux hwservice 上下文列出 19 个传感器条目。运行时注册了哪些接口未能确定。
- SELinux 策略仅通过对文本 .cil 与 *_contexts 文件的 grep 读取。未解码 odm_a/etc/selinux 下的 precompiled_sepolicy 二进制文件及任何策略二进制文件，因此未枚举完整的 allow 集合。
- system_a/lib 与 vendor_a/lib 下的 32 位变体，以及 32 位 odm 与 product 副本，仅检查了名称和大小，未作进一步检查。
- 未识别 system_ext_a/etc/bootanim 下的 meta.bin（1081352 字节），其格式仍未知。
- TorchScript 与 PyTorch Lite 归档未解码。未提取张量形状、输入与输出名称以及版本语义。生产者元数据包含构建主机和用户标识符，这些均有意未予复现。
- sensorservice.cfg 所用的精确 TOML 语法与设备版本名称（libdeviceid 中的 kDeviceVersionName）未被提取，因此键匹配行为仅根据字符串与注释进行描述。
- sensor_validation（8.2 MB）与 libsensorvalidation.so 仅根据名称、NEEDED 列表及 svt.sh 调用方式进行分类，其内部测试用例未予检查。
- 四个符号链接指向 toybox 以及本组之外的 boot vdex 文件；这些目标未作分析。
- 作为传感器 HAL 与模拟器依赖项列出、但不在本组行中的库（libsyncboss.so、libcamerahal.so、libclocksync.so、libwavreader.so、libgammacorrection.so、libcalibrationstore.so）已确认存在，但未检查。
