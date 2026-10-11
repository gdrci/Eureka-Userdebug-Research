# 16 - 电源、热管理与充电

[English](16-power-thermal-charging.md)

## 摘要

电源-热管理-充电组包含 21 个文件：product_a 中 3 个，system_a 中 7 个，vendor_a 中 11 个。其中包括：Android 充电模式二进制文件（AArch64 PIE）及其动画描述和两个 PNG 图标；HIDL 热管理 HAL 1.0 和 2.0 接口库，有 32 位和 64 位 system 副本以及 64 位 vendor 副本（system 与 vendor 的 64 位副本仅 build-id 不同）；热管理辅助库 libthermalhalutils.so 及其 JSON 传感器和散热设备配置，由 odm 的 perf_service 二进制链接并读取；供电源 HAL 使用的两个 TOML 电源提示配置；由 init.insmod.sh 运行的充电器类 init 模块列表；由音频 HAL 通过 dlopen 加载的电池监听库；被健康 HAL 实现链接的外部电池客户端库；以及被 libsensorservice 链接的 IBatteryStats AIDL 库。热管理 HAL 的服务进程（odm perf_service）和外部电池 HAL 的服务进程（odm vendor.oculus.hardware.externalbattery@1.0-service）位于 odm_a 中，不属于本组。深入分析覆盖：13 项。

角色表：`data/subsystems/power-thermal-charging/roles.tsv`（21 个文件：19 个 verified，2 个 observed，0 个 inferred，0 个 unverified）。

## 已验证的论断

- **C01** (verified)：本组有 21 个输入行：product_a 中 3 个，system_a 中 7 个，vendor_a 中 11 个。之后每条命令使用的基础目录为 E=<image>。
- **C02** (verified)：全部 21 个文件均存在于 $E/mnt/<partition>/<path> 下，其 sha256 值与 classified.tsv 的 sha256 列相等。
- **C03** (verified)：system/bin/charger 是已剥离符号的 ELF64 AArch64 PIE，解释器为 /system/bin/linker64，大小 261472 字节，其 DT_NEEDED 条目包括 android.hardware.health@2.0.so、android.hardware.health@2.1.so、libbase.so、libcutils.so、libhidlbase.so、libiad-utils.so、liblog.so、libutils.so、libpng.so、libc++.so、libc.so、libm.so 和 libdl.so。
- **C04** (observed)：运行充电器的 init 服务定义为：在 charger 类中有 service charger /system/bin/charger，seclabel 为 u:r:charger:s0，用户 system，用户组 system graphics input，capabilities 为 SYS_BOOT，可访问 /dev/kmsg 和两个 pstore 控制台文件，且初始为禁用状态。
- **C05** (observed)：SELinux 将 /system/bin/charger 映射为类型 charger_exec；启动模式属性 sys.boot_from_charger_mode 和 ro.enable_boot_charger_mode 分别映射为 charger_status_prop 和 charger_config_prop；类型 charger 和 charger_exec 在 plat_pub_versioned.cil 中声明。
- **C06** (observed)：当 sys.boot_from_charger_mode 被设为 1 时，系统 init 脚本会停止 charger 类并触发 late-init，因此充电器进程会交接给完整启动流程。
- **C07** (observed)：充电器二进制读取以下系统属性：ro.charger.no_ui、ro.charger.draw_split_screen、ro.charger.draw_split_offset、ro.charger.enable_suspend、ro.charger.disable_init_blank、ro.enable_boot_charger_mode、sys.boot_from_charger_mode、ro.minui.overscan_percent、ro.minui.pixel_format、ro.minui.default_rotation、ro.boot.fake_battery、ro.boot.bootloader_screen_rotation 和 ro.product.charger.unplugged_shutdown_time。
- **C08** (observed)：充电器通过 /sys/class/power_supply 下按电源设备的格式路径读取电池状态（capacity、status、present、current_now、voltage_now、temp、type、health、online、cycle_count 等），并访问 /sys/power/wakeup_count、/sys/power/state、/dev/graphics/fb0、/dev/dri、/dev/input 和 /proc/last_kmsg。
- **C09** (observed)：充电器在以下路径中查找其动画描述和图像：/product/etc/res/values/charger/animation.txt、/res/values/charger/animation.txt、/product/etc/res/images/、/system/etc/res/images/ 和 /res/images/。它以相对名称引用 charger/battery_scale.png 和 charger/battery_fail.png。
- **C10** (verified)：充电器的 DT_NEEDED 中没有 libexternalbattery.so，但其字符串中包含 IExternalBattery 接口名 vendor.oculus.hardware.externalbattery@1.0::IExternalBattery，以及 IExternalBattery 方法的 HIDL 客户端、服务端和 passthrough 插桩名称。
- **C12** (verified)：健康 HAL 实现 android.hardware.health@2.0-impl-2.1-anorak.so 的 DT_NEEDED 包含 libexternalbattery.so，并导入四个 ExternalBattery 方法：构造函数、getCombinedBatteryLevel、isChargerPlugged 和 isConnected。
- **C13** (verified)：libexternalbattery.so（64 位 vendor 副本，SONAME 为 libexternalbattery.so，定义了 102 个动态符号）导出包装类 hardware::oculus::health::ExternalBattery（getBatteryLevel、getChargingStatus、getCombinedBatteryLevel(int)、isChargerPlugged、isConnected、getServiceLocked、handleServiceDeath、DeathRecipient），以及 IExternalBattery 1.0 的代理类和存根类（BpHwExternalBattery 包含 _hidl_getBatteryLevel、_hidl_getStatus、_hidl_isChargerPlugged、_hidl_isConnected；BnHwExternalBattery；BsExternalBattery）。它依赖 libbase.so、libcutils.so、libhidlbase.so、liblog.so 和 libutils.so。
- **C14** (verified)：android.hardware.thermal@2.0.so（64 位 system 副本，SONAME 为 android.hardware.thermal@2.0.so，定义了 174 个动态符号）声明了 HIDL 接口 android.hardware.thermal@2.0::IThermal 和 android.hardware.thermal@2.0::IThermalChangedCallback。它导出 2.0 版 IThermal 的方法 getCurrentTemperatures、getTemperatureThresholds、getCurrentCoolingDevices、registerThermalChangedCallback 和 unregisterThermalChangedCallback，继承自 1.0 版的方法 getTemperatures、getCpuUsages 和 getCoolingDevices，以及回调方法 notifyThrottling。它依赖 android.hardware.thermal@1.0.so。
- **C15** (verified)：android.hardware.thermal@1.0.so（64 位 system 副本，SONAME 为 android.hardware.thermal@1.0.so，定义了 92 个动态符号）声明了 android.hardware.thermal@1.0::IThermal，其方法包括 getTemperatures、getCpuUsages、getCoolingDevices、debug 和 getDebugInfo。它依赖 libhidlbase.so、liblog.so、libutils.so、libcutils.so、libc++.so、libc.so、libm.so 和 libdl.so。
- **C16** (verified)：android.hardware.thermal@1.0.so 和 android.hardware.thermal@2.0.so 的 64 位 system 与 vendor 副本大小相同，且恰好有 16 个字节不同，位于文件偏移 0x298 至 0x2a7。这些字节是第二个 NOTE 段（位于 0x288）中的 build-id。两个副本的 BuildID 不同。
- **C18** (observed)：odm 的 perfservice init 服务运行 /odm/bin/hw/perf_service，并注册 android.hardware.thermal@1.0::IThermal 和 android.hardware.thermal@2.0::IThermal 的 default 实例。其 VINTF manifest 条目列出 android.hardware.thermal 的 1.0 和 2.0 版本，实例为 default。
- **C19** (verified)：perf_service 链接 libthermalhalutils.so，并导入 V2_0::IThermal::registerAsService、解析函数 ParseSensorInfo 和 ParseCoolingDevice，以及 ThermalFiles::readThermalFile 和 addThermalFile。因此它是注册热管理 HAL 并使用该辅助库的进程。
- **C20** (verified)：libthermalhalutils.so（64 位 vendor 副本，SONAME 为 libthermalhalutils.so，定义了 29 个动态符号）导出 ParseSensorInfo、ParseCoolingDevice、ThermalFiles 的方法 getThermalFilePath、readThermalFile 和 addThermalFile，以及 V2_0::CoolingType 和 V2_0::TemperatureType 的 hidl_enum_values。它使用 jsoncpp 解析 JSON（Json::CharReaderBuilder、Json::Value::asFloat、asString、asBool、isString），并依赖 android.hardware.thermal@1.0.so 和 @2.0.so。
- **C22** (observed)：libthermalhalutils.so 中的解析器可识别以下 JSON 键：Sensors、CoolingDevices、Name、Type、Monitor、HotThreshold、ColdThreshold、HotHysteresis、ColdHysteresis、Multiplier 和 VrThreshold。其默认值记录在消息中：ColdThreshold 默认为 NAN，HotHysteresis 和 ColdHysteresis 默认为 0.0，Monitor 默认为 false。
- **C23** (verified)：配置文件在所有传感器中均省略了 ColdThreshold 和 ColdHysteresis。Monitor 键在 53 个传感器中的 37 个中存在，在 16 个中缺失。Multiplier 在 45 个传感器中为 0.001，在 8 个中为 1。传感器的键集合为 Name、Type、HotThreshold、HotHysteresis、VrThreshold、Monitor 和 Multiplier。
- **C26** (verified)：powerhint_holdout.cfg 是有效的 TOML，包含 27 个表。每个表名在 powerhint.cfg 中也存在。其 default 表有 102 个条目，其中 sched_migration_cost_ns 为 500000，而 powerhint.cfg 中为 0。其文件头说明内容是从 1/1/2024 的 powerhint.cfg 复制而来，并去掉了实验内容。
- **C27** (observed)：powerhint.cfg 的文件头定义了格式。该文件为 TOML。路径与值的配对为 TOML 基本字符串。duration 和 priority 为整数，作用于其所在的整个表。default 表为必需，clocks_cpu 和 clocks_gpu 表也为必需。实验内容放在 Experiments Section 下，回退内容放在 Backout Section 下。
- **C28** (observed)：perf_service 读取 /vendor/etc/powerhint.cfg、/vendor/etc/powerhint_holdout.cfg、/vendor/etc/irqhint.cfg 和 /vendor/etc/thermal_info_config.json。set_hint 读取 /vendor/etc/powerhint.cfg。当缺少 default 表时，两者都会打印消息 powerhint.cfg configuration is missing default table!。
- **C29** (verified)：init.insmod.charger.cfg 是以 action|argument 形式列出的 37 个 modprobe 动作、2 个 setprop 动作和 1 个 enable 动作的列表。它由 charger 类中的 oneshot、disabled 服务 insmod_charger_sh 运行，该服务使用此文件调用 /vendor/bin/init.insmod.sh。脚本以 IFS='|' 读取每一行，并按 insmod、setprop、enable、modprobe 和 wait 分派。最后设置 vendor.all.modules.ready 和 vendor.all.devices.ready。
- **C32** (verified)：battery_scale.png 是 8 位调色板 PNG，宽 120、高 1440。其 tEXt 块中的 Frames 值为 12，因此该图为 12 个 120x120 帧上下堆叠。它与 animation.txt 中的 12 个 frame 指令相符。
- **C33** (verified)：battery_fail.png 是 96x96 的 4 位调色板 PNG，包含 IHDR、PLTE、IDAT 和 IEND 块。视觉上显示一个位于圆形内的电池轮廓。充电器日志消息表明：若存在自定义的 battery_fail 图像则加载该图像，否则使用内置图像。
- **C34** (verified)：libbatterystats_aidl.so（32 位和 64 位 system 副本）实现了 AIDL 接口 com.android.internal.app.IBatteryStats。描述符在 64 位副本中以 UTF-16LE 存储于偏移 0x61fc 处，在 32 位副本中位于 0x3924 处。导出符号包括 IBatteryStats::asInterface、getDefaultImpl、setDefaultImpl、descriptor 和 BnBatteryStats::onTransact。它依赖 libbinder.so。
- **C35** (verified)：在已挂载分区中，libbatterystats_aidl.so 唯一的 DT_NEEDED 使用者是 libsensorservice.so，32 位和 64 位 system 副本中均如此。
- **C36** (verified)：libbatterylistener.so（64 位 vendor 副本）导出七个 C 函数：battery_properties_listener_init、battery_properties_listener_deinit、battery_properties_is_charging、battery_properties_remaining_capacity、battery_properties_current_temperature、pause_battery_listener 和 resume_battery_listener。它实现了 IHealthInfoCallback 2.0 和 2.1，导入 IHealth 2.1 的 getService，并依赖 health@1.0、@2.0、@2.1、power@1.2 和 libaudioutils.so。
- **C37** (verified)：音频 HAL audio.primary.anorak.so 在运行时使用 dlopen 和 dlsym 加载 libbatterylistener.so，所用绝对路径为 /vendor/lib64/libbatterylistener.so。它的 DT_NEEDED 中没有该库的条目。其 AudioExtn 包装函数声明了 battery_properties_listener_init(void (*)(bool, int, int))、battery_properties_is_charging()、battery_properties_remaining_capacity()、battery_properties_current_temperature()、battery_properties_listener_deinit()、pause_battery_listener() 和 resume_battery_listener()。
- **C38** (inferred)：每个 HotThreshold 和 HotHysteresis 数组中的七个槽位对应热管理 HAL 的七个节流严重等级。索引到等级的映射在这些二进制文件中未得到确认。
- **C39** (inferred)：clocks_cpu 的值单位为 kHz，clocks_gpu 的值单位为 Hz。这些单位是根据数值量级和表名推断的，未经代码确认。

## 措辞已修正的论断

- **C11** (inferred)：充电器的 DT_NEEDED 中没有 libexternalbattery.so 条目，也没有 .symtab 段。它已被剥离，但其 .gnu_debugdata 段中保存了一个压缩的符号表，其中定义了 IExternalBattery 代理类和存根类，以及 hardware::oculus::health::ExternalBattery 包装类。因此该二进制文件包含这些代码自身编译的副本，而不是在运行时加载 libexternalbattery.so。
- **C17** (verified)：热管理 HAL 库被 15 个文件链接。在 odm_a 中：perf_service（1.0 和 2.0）和 wattsup（2.0）。在 system_a 中：lib 和 lib64 中的 libandroid_servers.so（1.0）；libvrsrecorder.so、libvrsrecorder_plugin_temperaturereader.so、libvrsrecorder_utils_camera.so 和 libvrsrecorder_utils_sensorapishim.so（2.0）；以及 android.hardware.thermal@2.0.so 的 lib 和 lib64 副本（各自链接 @1.0）。在 system_ext_a 中：perfstream、vrdevicemanagerserver 和 libperfstreamprovider.so（2.0）。在 vendor_a 中：libthermalhalutils.so（1.0 和 2.0）的 vendor 副本，以及 android.hardware.thermal@2.0.so（@1.0）的 vendor 副本。
- **C25** (verified)：非表键包括 629 个路径条目（以 / 开头的键）、14 个时钟级别条目（clocks_cpu 和 clocks_gpu 中的 VR_MODE_*）、106 个 priority 条目和 9 个 duration 条目。
- **C30** (observed)：animation.txt 是一个指令文件，包含一条 animation 指令（3 个循环，2 次首帧重复，文件为 charger/battery_scale）、一条 percent_display 指令和 12 条 frame 指令。每条 frame 指令的显示时间（disp_time 列）为 750，文件未说明单位。六个电量区间 0-9、10-24、25-49、50-74、75-99 和 100-100 各由两帧使用。充电器使用模式 c %d %d %d %d %d %n%*s%n 扫描 percent_display 行，使用 frame: %d %d %d 扫描 frame 行。
- **C31** (inferred)：percent_display 行的扫描方式为：字面量 c，后跟五个整数（1150、216、216、216、255）和一个字体名。文件头在字体名之前列出了六个数值字段（x、y、r、g、b、a），因此这五个值与文件头字段的对应关系既未被文件确定，也未被充电器确定。
- **C40** (observed)：SELinux 的 file-context 和 hwservice-context 文件将热管理 HAL 服务端二进制文件和外部电池服务映射到各自的标签，并将热管理和外部电池 HIDL 服务映射到 hwservice 标签。热管理树和外部电池供电设备的 sysfs 标签定义在 vendor SELinux 策略文件中。
- **C41** (observed)：vendor 属性上下文将 persist.vendor.enable.thermal.genl 映射到 vendor_thermal_prop。其使用者不在这 21 个组文件之中；该属性名出现在 odm_a/bin/hw/perf_service 中，而 vendor_a/build.prop 第 64 行将其设为 true。未确定其中哪一个读取了该属性。

## 已排除的论断

| ID | 论断 | 原因 |
|---|---|---|
| C21 | thermal_info_config.json 是一个 JSON 对象，有两个顶层键：Sensors（53 个对象，从第 2 行开始）和 CoolingDevices（6 个对象，从第 1178 行开始）。每个传感器都有 7 元素的 HotThreshold 和 HotHysteresis 数组。传感器类型为 BATTERY 4、SKIN 14、CPU 14、GPU 8、NPU 3、BCL_VOLTAGE 6、BCL_CURRENT 2 和 USB_PORT 2。散热设备为 battery（BATTERY）、cdsp、panel0-backlight（均为 COMPONENT）、pwm-tach-fan0（FAN），以及 thermal-cpufreq-0 和 thermal-cpufreq-1（均为 CPU）。 | 已确认：顶层键为 Sensors 和 CoolingDevices，53 个传感器和 6 个散热设备，行号 2 和 1178，八个类型计数，所有 53 个传感器中 HotThreshold 均为 7 元素，以及散热设备的名称和类型。已反驳：HotHysteresis 仅出现在 53 个传感器中的 38 个（原始键出现 38 次），因此 15 个传感器缺少该键。每个传感器都有 7 元素 HotHysteresis 数组这一论断不成立。 |
| C24 | 字面字符串 BATTERY、SKIN、CPU、GPU、NPU、USB_PORT 和 FAN 不出现在 libthermalhalutils.so 或 perf_service 中。字符串 BCL_VOLTAGE、BCL_CURRENT、BCL_PERCENTAGE、POWER_AMPLIFIER 和 COMPONENT 出现在 libthermalhalutils.so 中。SKIN 和 USB_PORT 出现在 system_a 中的 framework.jar 和 services.jar 中。 | BATTERY、SKIN、NPU、USB_PORT 和 FAN 在 libthermalhalutils.so 和 perf_service 中均不存在。CPU 和 GPU 不存在于 libthermalhalutils.so 中，但作为独立字符串字面量出现在 perf_service 中，紧邻 CPU frequencies、GPU frequencies 和 Failed to get CPU usages 等日志文本。其余事实成立：BCL_VOLTAGE、BCL_CURRENT、BCL_PERCENTAGE、POWER_AMPLIFIER 和 COMPONENT 各在 libthermalhalutils.so 中出现一次，SKIN 和 USB_PORT 在 framework.jar 和 services.jar 中为独立标记。 |

## 缺口

- 未进行反汇编。充电器主循环、热管理 HAL 算法、电池监听线程，以及 IBatteryStats 的事务码和方法列表均未确定。
- 配置中的 Type 值 BATTERY、SKIN、CPU、GPU、NPU、USB_PORT 和 FAN 未作为字面字符串出现在 libthermalhalutils.so 或 perf_service 中。SKIN 和 USB_PORT 仅出现在 framework.jar 和 services.jar 中。解析器如何将这些配置值映射为枚举值尚未确定。
- 每个传感器的七个阈值槽位的含义仅为推断（C38）。powerhint.cfg 中的时钟单位仅为推断（C39）。
- 充电器的命令行选项集合（其 getopt 字符串）未确定。
- 充电器是否静态链接了 IExternalBattery 代理仅为推断（C11）。该二进制文件已剥离且没有符号表。
- libbatterylistener.so 的 DT_NEEDED 中有 android.hardware.power@1.2.so 条目，但该库中没有未定义符号引用 android::hardware::power::V1_2::IPower。其对电源 HAL 的使用情况未确定。
- percent_display 行开头的 c 标记是字面扫描标记。1150 及 RGBA 数值与位置和颜色的对应关系仅根据文件头注释推断（C31）。
- battery_scale.png 通过其文件头、tEXt 块和视觉渲染进行了检查。未解码逐帧像素内容。
- 动画资源（product、system 和 res 路径）的运行时搜索顺序仅通过字符串得知。
- libthermalhalutils.so 内嵌了构建时的源文件路径 vendor/meta/services/perf/lib/libthermalutils/src/config_parser.cpp 和 thermal_files.cpp。这些只是观察到的字符串，未与任何源码树对照。
- odm_a/etc/init/vendor.oculus.hardware.externalbattery@1.0-service.rc 被列为包含外部电池服务，但未读取。服务端二进制文件 odm_a/bin/hw/vendor.oculus.hardware.externalbattery@1.0-service 未进行分析。
- DT_NEEDED 使用者列表仅涵盖已挂载分区（odm_a、odm_dlkm_a、product_a、system_a、system_ext_a、vendor_a、vendor_dlkm_a）。基于 dlopen 的使用者（如音频 HAL）仅通过字符串搜索发现。
- 未在各二进制文件中搜索 persist.vendor.enable.thermal.genl 的使用者（vendor_property_contexts 第 30 行）。
- roles.tsv 文件包含一个表头行，后接 21 个数据行。其中 battery_fail.png 和充电器二进制文件的角色描述为 observed 而非 verified，因为其功能是从字符串和 init 文本中得出的。
