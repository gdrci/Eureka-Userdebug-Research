# 11 - 相机

[English](11-camera.md)

## 摘要

相机组包含分布在五个分区中的 83 个文件（vendor_a 57 个、system_a 12 个、system_ext_a 6 个、odm_a 4 个、product_a 4 个）。其核心是一个 QTI Camera HAL3 模块（vendor_a/lib64/hw/camera.qcom.so，hw_module id 为 "camera"，模块 API 3.0），它从 vendor_a/lib64/camera/components 加载 CHI 管线以及 3A/统计组件库；XR 相机 HAL 实现（vendor_a/lib64/libxrcamerahal.so）则通过一个 odm 垫片库（odm_a/lib64/libcamerahal.so，导出 CameraHalC_Init/Start/Stop/Deinit）访问。一组相机 mux-mode 服务（system_ext 中的 cameramuxmodeservice 守护进程、一个客户端监控库、一个 JSON 读取库和一个命令行客户端）由 /product/etc/cameramuxmode/config.json（v1）和 configv2.json（版本 3）配置，其中 configv2.json 在 vendor_a 中有副本，并由 odm 传感器服务读取。CAMERA_ICP 固件是一个 Xtensa ELF32 MDT 镜像，具有 21 个程序头，其中 13 个段文件携带数据。vendor_a/etc/camera 包含 16 个针对 IMX471、OG01A1B 和 OV7251 传感器的调优与传感器模块数据块（Chromatix 头文件、AEC/AWB 表、CVA 和深度 AE JSON），system_a 则包含 32 位和 64 位构建版本的框架及 HIDL 客户端库（camera.common@1.0、camera.device@3.2、camera2 NDK、camera client、camera metadata）。已挂载分区中的 init 引用仅提到 cameramuxmodeservice；SELinux 标签涵盖 mux-mode 服务、其配置目录与属性，以及固件目录。

角色表为本组 83 个文件各包含一行（38 个 verified、43 个 observed、2 个 inferred、0 个 unverified）。该表位于 `data/subsystems/camera/roles.tsv`。

## 已验证的声明

- **C01** (verified)：相机组包含 83 行：vendor_a 57、system_a 12、system_ext_a 6、odm_a 4、product_a 4。已挂载文件位于 B/mnt 下，其中 B 为 <image>。后续声明中的命令均在 B/mnt 下运行，除非路径为绝对路径。
- **C02** (verified)：83 行中的每一行都存在于 B/mnt 下，且其字节大小和 sha256 与记录值相同。
- **C03** (verified)：有三组行的内容完全相同：product_a 与 vendor_a 的 etc/cameramuxmode/configv2.json；vendor_a etc/camera 中的 cvaectuning_ht.json 与 cvaectuning_slam.json；以及八个零字节固件段。
- **C05** (verified)：HAL 模块符号 HMI 解码为 hw_module_t，其 tag 为 0x48574d54，module_api_version 为 0x0300，hal_api_version 为 0x0100，id 为 “camera”，name 为 QTI Camera HAL，author 为 Qualcomm Technologies, Inc.。其 methods 指针为 0xa36fa0。
- **C06** (verified)：camera.qcom.so 有 39 个 NEEDED 条目。其中包括 CamX 核心库（libcamxcommonutils、libcamxhalbuffermanager、libcamxsettingsmanager、libcamxcsl、libcamxfdengine、libcamxhwnodecontext、libcamximageformatutils、libcamxstatscore）、gralloc 与显示 HIDL 库、QMI 与传感器标定库，以及 libcamera_metadata。
- **C08** (observed)：camera.qcom.so 包含 HAL3 入口点的跟踪名称（ConfigureStreams、ProcessCaptureRequest、ProcessCaptureResult、SetTorchMode、GetVendorTagOps、GetCameraInfo 等），以及大量 com.qti.chi.* 厂商标签名称（例如 multicamerainfo、cameraconfiguration、offlinepipelineinfo、statsaec）。
- **C11** (observed)：libxrcamerahal.so 包含一条 Qualcomm HAL 路径、一个黑白 ISP 控制库的依赖名称，以及用于 multistream、foveation、Teton GME/MCTF 和 AEC 统计开关的属性键。它还引用了厂商标签 org.codeaurora.qcamera3.sessionParameters。
- **C15** (verified)：cameratool 是一个 6,746,120 字节的 AArch64 PIE 可执行文件，解释器为 /system/bin/linker64。其 NEEDED 条目包括 libcamerahal.so、libcameracontrol.so、libsyncboss.so、libdevicesensors.so、libtelemetry.so 以及相关的遥测库。
- **C16** (observed)：cameratool 的说明涵盖帧流、元数据转储，以及按相机集合选择的标志（例如 --teton、--esker、--pikes、--cascade、--grays、--canyon、--maryland、--jackson、--glacier）。其 -O 选项相对于 /vendor/etc/camera 解析路径，并且它有 --syncboss-only 和 --disable-syncboss 两种模式。
- **C18** (verified)：cameramuxmodeservice 是一个 134,448 字节的 AArch64 PIE 可执行文件，解释器为 /system/bin/linker64。其 NEEDED 条目共 17 个库，包括 libcameramuxmodereader.so、libbinder.so、libjsoncpp.so、libtrackingvendorutils.so 和 vendor.oculus.hardware.sensors@1.0.so。
- **C19** (verified)：cameramuxmodeservice 的 init 服务在 late_start 阶段从 /system_ext/bin 运行，用户为 system，用户组为 system 和 camera，并持有 SYS_NICE 能力。它还会在 factorytest 触发器上启动。
- **C20** (observed)：cameramuxmodeservice 包含其配置路径 /product/etc/cameramuxmode/config.json、binder 服务名 cameramuxmodeservice、一个 CMMv2 功能开关字符串（enabled 为 true 或 false），以及用于模式切换和客户端提升的日志文本。它还包含符号名 getCameraMuxModeConfigJsonFileContent 和 FeatureChecks::isCMMv2Enabled。
- **C21** (verified)：SELinux 的文件、服务和属性上下文为 cameramux 组件设置了标签：/product/etc/cameramuxmode 为 system_cameramuxmode_file，服务二进制为 cameramuxmodeservice_exec，binder 名称 cameramuxmodeservice 为 cameramuxmode_service，属性前缀 persist.cmm. 为 cameramuxmode_prop。
- **C22** (verified)：system_ext 的 SELinux 策略声明了 cameramuxmodeservice 域，从 init 进行域转换，并授予其对 cameramuxmode_service 的 service-manager add/find 权限、与 servicemanager、system_server 和 priv_app 之间的 binder 调用权限，以及对 cameramuxmode_prop 和 system_cameramuxmode_file 的读取权限。
- **C24** (verified)：mux-mode 监控库是 mrsystemservice、trackingservice、sensorproxy、trackingfidelityservice、cameramuxmodeservice_tool、stream-publisher 和 libcmsservice-headset.so 的 NEEDED 依赖项，也是系统库 libvrsrecorder.so 和 libvrsrecorder_utils_camera.so 的 NEEDED 依赖项。
- **C25** (observed)：监控库包含其 binder 客户端的 IPC 与错误文本：连接 CameraMuxModeService、设置 mux 模式、名为 on_camera_mux_mode_changed 的回调，以及事务名 set_active_camera_mux_mode。它还包含 consumer-frame-type 报告格式。
- **C26** (verified)：libcameramuxmodereader.so 是一个 23,888 字节的 AArch64 共享对象，其唯一的相机导出符号为 OVR::OS::getCameraMuxModeConfigJsonFileContent(const char*)。其 NEEDED 条目包括 libjsoncpp.so 和 liblog.so 等。
- **C27** (verified)：libcameramuxmodereader.so 被 cameramuxmodeservice、libmxrcamera_image_provider.so、libtokaido_device_features.so、libvrsrecorder.so、libvrsrecorder_utils_camera.so 和 libvrsrecorder_utils_common.so 列为 NEEDED 依赖。
- **C28** (verified)：cameramuxmodeservice_tool 是一个 15,856 字节的 AArch64 PIE 可执行文件，其 NEEDED 条目包括 libcameramuxmodemonitor.so、libgflags.so、libhidlbase.so 和 libjsoncpp.so 等。
- **C29** (observed)：mux-mode 命令行客户端接受 -info 和 -set_active <name> 参数，输出当前 mux 模式，并报告 setActiveCameraMuxMode 的 binder 失败。
- **C31** (verified)：product_a/etc/cameramuxmode/configv2.json（版本 3）的 cameraSystems 包含 worldTracking（niceEnabled 为 true，configurations 为 empty 和 default，16 个模式）和 color（niceEnabled 为 false，configurations 为 empty、default 和 display，5 个模式）。每个 configuration 都有 frameRateVariant、frameRateMultiplier 和 slots。color/display 使用 frameRateVariant display，倍率为 72；其他 configuration 使用 utility，倍率为 1.0。
- **C32** (verified)：configv2.json 中的每个模式都包含 name、configuration、purposes（name、slots 和可选的 cameraIds）、controlPurposes（name 和 slots）、capabilities、frameTags 和 fidelities（name 和 levelRange）。在 worldTracking 中，模式 lowLightHandsDepth 使用 default configuration，在 slot 3 上具有 depth_projector capability，frameTags 为 [1, 0, 1, 0]，并包含 hands（levelRange [1, 4]）和 iot（levelRange [2, 5]）等 fidelities。
- **C35** (observed)：device@3.2 库中列出了 HIDL 接口 ICameraDevice、ICameraDeviceCallback 和 ICameraDeviceSession，并包含客户端、服务端和 passthrough 方法标签，例如 open、getCameraCharacteristics、getResourceCost 和 configureStreams。
- **C36** (verified)：系统兼容性矩阵 5 声明了版本 2.4-6 的 android.hardware.camera.provider，接口为 ICameraProvider；hwservice 上下文将 android.hardware.camera.provider::ICameraProvider 标记为 hal_camera_hwservice。
- **C37** (observed)：64 位系统框架库导出了预期的 API：libcamera2ndk.so 导出 ACameraManager_openCamera、ACaptureRequest_* 和 ACameraCaptureSession_* 函数；libcamera_client.so 导出 BpCameraService::connect 和 android::Camera 符号；libcamera_metadata.so 导出 allocate_camera_metadata、append_camera_metadata 以及 find_camera_metadata_entry 一类的函数。
- **C38** (verified)：Chromatix 调优参数文件以魔数 QTI Chromatix Header 开头，并携带解析器版本字符串 Parameter Parser V3.4.0 (2106301010)。偏移 0x1c 处的 32 位小端大小字段等于文件大小减 4。对于 com.qti.tuned.eureka_imx471_left.bin 和 com.qti.tuned.default.bin，最后四个字节是其之前字节的 CRC32。
- **C39** (verified)：六个 com.qti.sensormodule.camNN bin 文件共享 Chromatix 头部。它们在 0x1c 处的 u32 等于文件大小，末尾字为 0xffffffff（没有 CRC 匹配）。每个文件都嵌入了其模块名称和字符串 cameraModuleData。
- **C42** (observed)：AEC 与 AWB 核心调优表是带有文本标签的二进制表。AECoreTuning_imx471.bin（13157 字节）与 AECoreTuning_imx471_v2.bin（15775 字节）在内容和标签上不同（v2 中的触发器为 luxIndexTrigger 和 gyroSqrTrigger）。AWBCoreTuning_imx471.bin 与 AWBCoreTuning_imx471_CIQ_Vivid.bin 大小相同（27947），但字节内容不同。基于魔数的检测将 AWB 文件识别为 Targa 图像，这是一个错误匹配。
- **C43** (verified)：cvaectuning_slam.json、cvaectuning_ht.json 和 depthaectuning.json 是 JSON 对象，具有相同的四个顶层键：basic、metering、convergence 和 factorization。前两个文件的字节完全相同。
- **C44** (verified)：CAMERA_ICP.mdt 是面向 Tensilica Xtensa 的 ELF32 可执行文件（e_flags 为 0x300），入口为 0x0，包含 21 个程序头。它是一个拆分固件镜像中的头部部分。
- **C45** (verified)：程序头索引 N 对应段文件 CAMERA_ICP.bNN，并且在全部 21 种情况下，文件大小都与程序头的 FileSiz 相等。其中 13 个段文件携带数据（b00 至 b09、b17、b18、b20）。8 个为零字节（b10 至 b16 和 b19）。段 8 大小为 857980 字节，位于 vaddr 0x1000；段 17 大小为 13124 字节，位于 0xe0400000；段 18 大小为 38884 字节，位于 0xe0420000。段 20（PT_NULL 类型）包含 7000 字节。
- **C46** (observed)：段 8 包含命名 CPAS、BPS 和 IPE 命令描述符块的文本，以及一条 error-fatal 消息。段 9 包含 BPS 和 IPE 的入队函数以及内存映射文本。段 20 包含类似 X.509 的文本（Eureka_FW_Root、Meta Platforms, Inc.、Menlo Park）以及有效期日期，与证书块相符。
- **C47** (verified)：vendor 的 SELinux 文件上下文将 /vendor/firmware 下的所有内容标记为 firmware_file。不存在针对特定路径的相机固件规则。
- **C48** (observed)：vendor_dlkm 相机内核模块中携带固件名字符串 CAMERA_ICP.elf。已挂载的固件集提供的是 CAMERA_ICP.mdt，因此模块中的名称与所提供的文件名不匹配。
- **C50** (observed)：vendor.qti.hardware.camera.postproc@1.0 是一个 HIDL 包，包含 IPostProcService、IPostProcServiceCallBacks 和 IPostProcSession。service-impl 库导出 RegisterIPostProcService，并包含文本字符串 CameraPostProc_Create 和 CameraPostProc_Destroy，其 NEEDED 条目为 26 个库。该包在已挂载分区中没有 init、SELinux 或 vintf 条目；com.qti.chi.override.so 引用了它。
- **C51** (observed)：相机 warp-grid 提供方接口是一个 AIDL 包，包含 ICameraWarpGridProvider、ICameraWarpGridProviderClient、WarpGridComputeRequest、WarpGridBufferFormat 和 SystemTimestamp 类型。odm_a 与 system_ext_a 中该接口 NDK 存根的副本大小相同（63192 字节），但构建 ID 和 sha256 值不同。mrsystemservice.rc 声明了该接口实例。
- **C52** (observed)：已挂载的 init 与 ueventd 文件通过设备节点组以及 cgroup/cpuset 设置引用了相机组：video、media、v4l-subdev 和 synx 设备节点被设置为 camera 组，init 创建了 camera-daemon 的 cpu、stune 和 cpuset 组，以及 /data/misc/cameraserver。
- **C53** (observed)：属性上下文将相机属性族分配给各标签：persist.vendor.camera. 和 vendor.camera. 映射到 vendor_camera_prop，vendor.debug.camera. 映射到 vendor_camera_prop，persist.cmm. 映射到 cameramuxmode_prop；系统策略中，camera.disable_zsl_mode 及类似键映射到 camera_config_prop。
- **C55** (observed)：在已挂载的 init 服务与组定义中，名称包含 camera 的唯一服务是 cameramuxmodeservice。其他 init 文件仅将 camera 用作组名，或出现在 cameraserver 重启相关内容中。

## 修正措辞后的声明

下方结论以数据所支持的措辞重新表述。

- **C07** (observed)：camera.qcom.so 中保存了其文件与组件位置的字面路径（/vendor/lib64/camera/components/、/vendor/etc/camera、/data/vendor/camera、/data/vendor/camera/metadata）、格式模式 %s/%s%s_%s_%d.bin、%s/GlobalUsecase%s_%d.bin 和 %s/PerUsecase%s_%d.bin，以及属性键，例如 persist.vendor.camera.isp.enablePiAEC、persist.vendor.camera.eeprom.reload 和 persist.vendor.camera.enablemetahashmap。
- **C12** (verified)：odm_a/lib64/libcamerahal.so 是一个已剥离符号的 AArch64 共享对象（SONAME 为 libcamerahal.so，36,896 字节）。其 C 链接导出符号为 CameraHalC_Init、CameraHalC_Start、CameraHalC_Stop 和 CameraHalC_Deinit；它还导出 CameraHal 命名空间下的 14 个 C++ 修饰符号。其 NEEDED 条目包括 libbase、libimagebuffer、liblog、libmros_vendor_sysprops、libsensortelemetry、libui 和 libutils（另有 libc++、libc、libm 和 libdl）。
- **C13** (observed)：odm 相机 HAL 垫片库通过 dlopen 加载其实现，并按名称引用该库中的 CreateCameraHal 入口点；它并不导出未修饰的 CreateCameraHal 符号。其候选名称包括 libfastpathcamerahal.so 以及一个以 xrcamera 命名的库（libxrcamerahal.so）。其属性为 persist.vendor.camera.hal 和 vendor.camera.hal.debug_file。
- **C17** (observed)：syncboss_camera_tests 是一个已剥离符号的 AArch64 共享对象类型可执行文件，解释器为 /system/bin/linker64。它是 SyncBoss 相机路径的测试程序（根据其名称和日志文本推断），包含 SyncBoss 日志文本，并且其中包含字符串 libcamerahal.so，但该库不在其 NEEDED 条目中（NEEDED 条目为 liblog.so、libm.so、libdl.so、libc.so）。
- **C23** (verified)：libcameramuxmodemonitor.so（lib64）是一个 115,544 字节的 AArch64 共享对象，具有 168 个已定义和 194 个未定义的具名动态符号（若计入索引 0 的空条目，则未定义条目为 195 个）。它导出 CameraMuxModeMonitor 以及 ICameraMuxModeClient 和 ICameraMuxModeService binder 接口符号。其 NEEDED 条目共 13 个库，包括 libvrsensors-hidlwrapper.so 和 libvrsensors-utils.so。
- **C30** (verified)：product_a/etc/cameramuxmode/config.json 是一个 JSON 对象，包含六个顶层模式表（default、HandTracking、HandTrackingUseIOT、RubyTracking、default_popstudy_hack、HandTracking_popstudy_hack）。每个表都包含 consumerToFrameTypesMap、frameConfiguration 和 frameTypeToExposureAlgorithmMap。例如，default 将 GENERIC_SLOT_0 映射到 InsideOutTracking，将 GENERIC_SLOT_1 映射到 FlexibleFrame，将 GENERIC_SLOT_3 映射到 ProjectorAssistedDepth。该文件没有 version 字段。
- **C33** (observed)：configv2.json 由 odm 和系统组件从 /vendor/etc/cameramuxmode/configv2.json 读取：odm_a 中的 odm 传感器服务（及其 -mock 变体）、sensoremu、libsensorscommon.so 和 libsensoremulator.so，以及 system_a 中的系统库 libstreampublisher.so 和 vrs-recorder、vrs-recorder2 二进制文件。cameramuxmodeservice 不包含 configv2.json 引用，并读取 /product/etc/cameramuxmode/config.json。没有任何 SELinux 文件上下文条目覆盖 /vendor/etc/cameramuxmode。
- **C40** (verified)：六个 com.qti.sensormodule.camNN bin 文件的命名格式为 com.qti.sensormodule.camNN_<token>_eureka_<sensor>_<side>_csiN.bin（标记为 verified 的 token）：cam00 none og01a1b left csi4；cam01 none og01a1b right csi1；cam02 liteon ov7251 left csi5；cam03 liteon ov7251 right csi2；cam04 none imx471 left csi0；cam05 none imx471 right csi3。第二个 token 的含义（模块厂商，其中 none 表示没有模块厂商）为推断结果（inferred）。
- **C41** (observed)：camera.qcom.so 中提到一个示例传感器模块文件 com.qti.sensormodule.truly_ov8856.bin 及其双通道变体，它们体现了较短的 com.qti.sensormodule.<sensor>.bin 命名方式。本组中的六个 bin 文件采用较长的 com.qti.sensormodule.camNN_<vendor>_eureka_<sensor>_<side>_csiN.bin 格式（见 C40）。在 camera.qcom.so 中，既没有与本组文件名字面匹配的内容，也没有 eureka、cam04、imx471、og01a1b 或 ov7251 这些 token 的字面匹配。

## 已排除的声明

下方声明已排除；每条都给出了原因。

| ID | 声明 | 原因 |
|---|---|---|
| C04 | camera.qcom.so 是一个 10,921,800 字节、已剥离符号的 ELF64 AArch64 共享对象，类型为 DYN，具有 1054 个已定义和 493 个未定义的动态符号。 | 大小 10921800、ELF64、AArch64、类型 DYN 和已剥离符号均可复现。符号计数不能复现。动态符号表共有 1545 个条目（包括索引 0 处的空条目）：1052 个已定义的具名符号和 492 个未定义的具名符号，另加空条目（即 493 个 UND 行）。所声称的过滤条件还计入了两行输出标题行（1052 + 2 = 1054）。 |
| C09 | 其他 vendor 库和 HAL 提供方配置通过路径或名称引用 camera.qcom.so：com.qti.chi.override.so、libispcontroller.so、libmonocameracontrol.so 和 libxrcamerahal.so。已挂载的 init 目录中没有任何 init 服务启动相机提供方，且提供方可执行文件仅通过 vendor_file_contexts 中的一条模式规则进行标注。 | 四个引用库均可复现：com.qti.chi.override.so、libispcontroller.so、libmonocameracontrol.so 和 libxrcamerahal.so。已挂载目录树中的配置文本文件（xml、json、rc、cfg、conf、prop）均未引用 camera.qcom.so，vendor 或 odm 的 vintf manifest 中均不存在相机条目，因此'HAL 提供方配置'这一部分缺乏依据。init 的结果（没有 init rc 文件启动相机提供方）以及文件上下文标注（vendor_file_contexts 第 19 至 23 行）均可复现。 |
| C10 | libxrcamerahal.so 是一个 461,024 字节、已剥离符号的 AArch64 共享对象，SONAME 为 libxrcamerahal.so，具有 473 个已定义和 157 个未定义的动态符号。其唯一的相机专用导出符号为 CreateCameraHal。其 NEEDED 条目包括 libcamerahal.so、libcamerafencewrapper.so、libcamxhalbuffermanager.so、libdevicesensors.so 和 libcamera_metadata.so 等其他库。 | 大小 461024、SONAME、AArch64 共享对象以及 NEEDED 条目（libcamerahal.so、libcamerafencewrapper.so、libcamxhalbuffermanager.so、libdevicesensors.so、libcamera_metadata.so 等其他库）可复现。计数不能复现：具名符号为 471 个已定义和 156 个未定义（共 628 个条目，含空条目）；声明为 473 和 157。关于'唯一的相机专用导出'的表述有误：该库导出 150 个以 _ZN9CameraHal 或 _ZNK9CameraHal 开头、经名称修饰（mangled）的 GLOBAL 符号（例如 _ZN9CameraHal11XrCameraHal16InitializeModuleEv）。CreateCameraHal 是唯一未经名称修饰的相机专用导出符号；声明所用的筛选在得出结论前已去掉经名称修饰的名称。 |
| C14 | libfastpathcamerahal.so 不在已挂载分区中。垫片库的其他依赖存在：libsyncboss.so 和 libdevicesensors.so 位于 odm_a 中，libcamxcsl.so、libcamxhalbuffermanager.so、libcamxsettingsmanager.so 和 libcamerafencewrapper.so 位于 vendor_a 中。libcameracontrol.so 的 NEEDED 条目包含 libcamerahal.so 和 libsyncboss.so。 | libfastpathcamerahal.so 不在任何已挂载分区中。其他所列库的存在性可复现，libcameracontrol.so 的 NEEDED 条目包含 libcamerahal.so 和 libsyncboss.so。归属有误：垫片库的 NEEDED 列表（libbase、libimagebuffer、liblog、libmros_vendor_sysprops、libsensortelemetry、libui、libutils、libc++、libc、libm、libdl）不包含 libsyncboss.so、libdevicesensors.so 或任何 libcamx 库。这些库被其他组件以 NEEDED 方式引用。 |
| C34 | android.hardware.camera.device@3.2.so（lib64）是一个 195,888 字节的 AArch64 共享对象，具有 265 个已定义和 168 个未定义的动态符号。其 NEEDED 条目包括 android.hardware.camera.common@1.0.so、android.hardware.graphics.common@1.0.so 和 libhidlbase.so 等。它定义了 93 个名称包含 ICameraDevice 的符号。 | 195888 字节、AArch64 共享对象、NEEDED 列表以及 93 个以 ICameraDevice 命名的符号可复现。计数不能复现：具名动态符号为 263 个已定义和 167 个未定义（共 431 个条目，包括空条目）。声称的 265 和 168 包含输出中的两个表头行以及空条目。 |
| C49 | CHI 管线节点、3A 库和统计算法组件导出其工厂符号：libcorepi_3a_v1.so 和 libcorepi_3a_v2.so 导出 createCorePiAWBInterface（v2 还导出 createCorePiAEInterface）；com.qti.stats.awb.so 导出 CreateAWBAlgorithm；com.qti.stats.aec.so 导出 CreateAECAlgorithm；com.qti.stats.aecxcore.so 导出 CreateAecXCoreAlgo；com.meta.node.gipp.so 具有 GIPPNodeCreate；com.meta.node.fastAI.so 具有 FastAINodeProcRequest；com.meta.node.foveation.fusion.so 具有 FoveationBlend；com.qti.eisv3.so 具有 eis_utility 函数。 | 已确认的导出：com.qti.stats.awb.so 导出 CreateAWBAlgorithm；com.qti.stats.aec.so 导出 CreateAECAlgorithm；com.qti.stats.aecxcore.so 导出 CreateAecXCoreAlgo；com.qti.eisv3.so 导出 eis_utility_* 函数。不匹配之处：（a）libcorepi_3a_v1.so 也导出 createCorePiAEInterface，因此 'v2 also' 这一限定语有误；（b）com.meta.node.gipp.so（GIPPNodeCreate）和 com.meta.node.fastAI.so（FastAINodeProcRequest）中这些名称仅以字符串形式出现，没有动态或静态符号条目；（c）com.meta.node.foveation.fusion.so 中的 FoveationBlend 仅出现在日志文本中。 |
| C54 | camera_click.ogg 和 camera_focus.ogg 是短促的 Ogg Vorbis 立体声音频，采样率为 44100 Hz，分别用于相机快门咔嗒声和对焦事件。其角色来自文件名，并且在已挂载分区中未发现对它们的 init、SELinux 或应用程序引用。 | 格式（Ogg Vorbis、立体声、44100 Hz）和文件大小（7010 和 9376 字节）可复现；时长未测量，'short' 是根据大小和比特率推断的。关于无引用的陈述有误：camera_click.ogg 和 camera_focus.ogg 这两个字面字符串出现在 system_a/system/framework/framework.jar 中，位于一个未压缩的成员内，与 camera_autorotate 等其他相机名称并列。未发现 init 或 SELinux 引用。 |

## 缺口

- 相机提供方服务二进制文件（加载 camera.qcom.so 的 android.hardware.camera.provider@2.x-service）不在已搜索的已挂载分区中，因此其 NEEDED 列表、注册方式及其 init rc 均无法确定。
- 已挂载的 init 目录中，没有任何 init rc 启动 QTI 相机 HAL、cameratool、libcamerahal、postproc HIDL 服务或 CAMERA_ICP 的固件加载器；这些启动均未确定。
- 在已挂载的 vintf 目录中，未找到 android.hardware.camera.provider 或 vendor.qti.hardware.camera.postproc 的 vintf manifest 条目；仅找到了系统兼容性矩阵。
- CAMERA_ICP 加载器尚未确定。camera.ko 中的名称为 CAMERA_ICP.elf，而该固件集中包含的是 CAMERA_ICP.mdt 和各段文件。加载顺序、程序头标志的段类型语义以及任何验证流程均未确定，也未作评估。
- ICP 固件段载荷的含义（代码与数据布局、命令格式以及段 20 中的签署文本）仅限于文本标签和头部信息；未进行反汇编。
- Chromatix 调优数据块和传感器模块数据块仅在头部、大小和校验和层面进行了解码。参数段及其数值未被解析。
- AEC 与 AWB 核心调优表（二进制，无头部魔数）仅解码到大小、开头字节和文本标签的程度。
- CHI 管线图定义（描述 com.meta.node.* 和 com.qti.* 节点的 XML 或 JSON）不在本组中；节点拓扑未确定。
- 由于 libfastpathcamerahal.so 不在已挂载分区中，odm libcamerahal 垫片库在 XR HAL 与 fastpath HAL 之间的选择机制未确定。
- 未找到 configv2.json 的字段级解析器。libcameramuxmodereader.so 只导出一个文件内容获取函数，cameramuxmodeservice 仅读取 v1 路径字面量。究竟是哪个二进制文件解析 v3 键（version、cameraSystems、modes），尚未确定。
- system_a 中的 32 位库（lib/ 目录）仅依据 SONAME、NEEDED 和导出名称进行了抽样，其角色也仅基于这些名称。
- camera_click.ogg 和 camera_focus.ogg 的角色仅根据文件名推断。
- cameratool 与 syncboss_camera_tests 均未执行；其行为依据选项和日志文本进行描述。
- 依赖闭包仅针对本次审阅中列出的库进行了检查。来自其他分区的库（如 libsyncboss.so、libdevicesensors.so 和 libcamxcsl.so）已确认存在，但未对每一行计算完整的 NEEDED 解析。
- vendor_dlkm_a 和 odm_dlkm_a 仅搜索了 camera.ko 字符串；其其他内容不在本组范围内。
