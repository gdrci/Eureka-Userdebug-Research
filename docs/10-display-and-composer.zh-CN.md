# 10 - 显示与合成器

[English](10-display-and-composer.md)

## 摘要

该组共有 132 行（odm_a 6、system_a 65、system_ext_a 7、vendor_a 54；其中 130 个普通文件、2 个符号链接），构成一台搭载 Oculus VR（VROS）组件、基于高通 SDM/SDE 的设备的 Android 显示、图形合成与 DRM 栈。VINTF 中注册了两个 IComposer 2.4 HAL 实现：高通的 vendor.qti.hardware.display.composer-service（IQtiComposer 3.2、DisplayConfig AIDL），以及 Oculus 的 StrataComposer 服务（实例名 "display"，开机时禁用，仅当 persist.device_config 属性为 true 时启用）。surfaceflinger 是这两者的 HIDL 客户端，gralloc、mapper、allocator 和 DisplayConfig 客户端库位于其下层。显示调优参数来自 /system/etc/display.conf（JSON）、/vendor/etc/display/DPU930.xml（高通 SDE930 参数表，由 libdisplayqos.so 读取）以及厂商属性 vendor.display.*。约 70 行属于 DRM 与 media-DRM 相关内容（ClearKey 和 Widevine HAL 服务、DRM 框架、MediaDrm 指标、forward-lock 引擎、memtrack）。此处的 libdrm.so 是用户空间内核 DRM 库，而非 Android 内容保护 DRM。

角色表为本组的 132 个文件各提供一行（114 verified、13 observed、5 inferred、0 unverified）。该表位于 `data/subsystems/display-and-composer/roles.tsv`。

## 已验证的论断

- **C02**（verified）：roles.tsv 包含一行表头和 132 行数据，每个输入行对应一行数据，键与输入完全一致。置信度分布：114 verified、13 observed、5 inferred、0 unverified。
- **C03**（verified）：高通 composer 服务是面向 AArch64 的 ELF64 PIE，动态链接至 /system/bin/linker64，已剥离符号，含 41 个 NEEDED 条目。
- **C04**（verified）：高通 composer 服务链接了 composer HIDL 2.1 至 2.4 以及厂商 QTI 3.0 至 3.2 接口，另外还链接了 libsdmcore.so、libqservice.so、libdisplayconfig.qti.so、libdrm.so 和 libdisplaydebug.so。其中最后三个位于本组中。
- **C05**（verified）：高通 composer 服务注册了 IQtiComposer 3.2（实例 default）、IComposer 2.4（实例 default），以及 DisplayConfig AIDL（vendor.qti.hardware.display.config 版本 10，fqname IDisplayConfig/default）。清单中还列出了 vendor.display.config IDisplayConfig 2.0。
- **C06**（verified）：高通 composer 服务的 init 定义（服务名 vendor.qti.hardware.display.composer）位于一个独立的 rc 文件中，同时也出现在 init.anorak.rc 中。后者的副本标记为 override 和 disabled，在重启时重启 strata 类，并在 vendor.syncbosshal.fw_update_complete=true 时启动。
- **C07**（verified）：SELinux 将高通 composer 服务标记为 hal_graphics_composer_default_exec，并在 hwservice 标签 hal_graphics_composer_hwservice 上注册 IQtiComposer。
- **C10**（verified）：Oculus StrataComposer 服务是面向 AArch64 的 ELF64 PIE，含 30 个 NEEDED 条目。其中包括 composer HIDL 2.1 至 2.4、mapper 2.0、3.0 和 4.0、allocator 4.0、power 1.3、libEGL.so、libGLESv2.so 以及 vendor.qti.hardware.display.mapper@4.0.so。
- **C11**（verified）：Oculus composer 在 VINTF 中以实例名 display 声明 IComposer 2.4，并标记 override=true。其二进制文件包含字符串 'Starting VROS HWComposer HAL' 和 'StrataComposerService_2_4'。
- **C12**（verified）：Oculus composer 服务在 init 中被禁用，仅当 persist.device_config.oculus_shared_os_services.mros_enable_strata_composer 为 true 时才会启用。该属性默认为 false。启用后，init 会将 debug.sf.hwc_service_name 和 ro.ovr.hwc_service_name 设为 display。
- **C14**（verified）：SELinux 将 Oculus composer 服务标记为 strata_exec，并通过类型转换（type transition），使 init 执行它时赋予其 strata 域。
- **C15**（verified）：surfaceflinger 是面向 AArch64 的 ELF64 PIE，含 43 个 NEEDED 条目，其中包括 composer HIDL 2.1 至 2.4、allocator 2.0 和 3.0、android.frameworks.displayservice@1.0.so 以及 libdisplayservicehidl.so。
- **C17**（observed）：surfaceflinger 读取 debug.sf.* 属性（例如 debug.sf.hwc_service_name、debug.sf.enable_layer_caching、debug.sf.disable_writeback_display），以及来自 libSurfaceFlingerProp.so 的 sysprop has_writeback_dpu_cac。
- **C20**（verified）：display.conf 是一个包含 10 个键的 JSON 对象：DIRECTION 为 3，SHUTTER_TYPE 为 0，两个值为 COMMENT 的键，SWAP_TIMING_FRONT_BUFFER 和 SWAP_TIMING_SWAPPED_BUFFER（6 元素数组），NIGHT_DISPLAY_AVAILABLE 为 true，以及夜间显示色温的最小值 2220、默认值 2856、最大值 6504。
- **C23**（verified）：composer 2.4 库的系统副本导出了 V2_4 IComposerClient 和 IComposerCallback 的 proxy 与 cast 函数。HIDL 接口名为 android.hardware.graphics.composer::IComposer，标签为 hal_graphics_composer_hwservice。厂商副本是另一个文件，定义了 345 个动态符号。
- **C24**（verified）：64 位 mapper 4.0 QTI 实现导出了 HIDL_FETCH_IMapper 和 HIDL_FETCH_IQtiMapper，以及 QtiMapperExtensions 方法。其 NEEDED 列表包含 libgralloccore.so、libgrallocutils.so、libqdMetaData.so、厂商 QTI mapper 4.0 以及 mapperextensions 1.0 至 1.2。其标签为 same_process_hal_file。
- **C25**（verified）：64 位 gralloc 模块导出了 HMI module-info 符号，以及 gralloc_lock、gralloc_unlock、gralloc_register_buffer、gralloc_unregister_buffer、mapBuffer、terminateBuffer、mapFrameBufferLocked 和 fb_device_open。它有 6 个 NEEDED 条目，标签为 same_process_hal_file。
- **C26**（verified）：64 位 DisplayConfig 客户端库导出了 DisplayConfig::ClientImpl 方法，如 SetActiveConfig、SetPowerMode、IsHDRSupported 和 SetCWBOutputBuffer。它定义了 150 个动态符号，NEEDED 列表包含 vendor.display.config@2.0.so。其标签为 same_process_hal_file。
- **C28**（observed）：display_tool 访问以下内核和 sysfs 节点：/sys/class/backlight/、/sys/class/drm/card0-sde-crtc-0/、/sys/class/drm/card0/card0-sde-crtc-0/backlight_temperature_override，以及 /sys/kernel/debug/dri/0/DSI-1/dcs_cmd_tx 和 dcs_cmd_tx_unicast。
- **C29**（verified）：display_property_handler 是一次性辅助程序。当 vendor.skip.init=0 且设置了 display.enable_ddic_cac 时，init 会以 display.enable_ddic_cac 属性作为参数启动它。它打开 DRM render 节点，并通过 libdrm 发出 DRM_IOCTL_MSM_DISPLAY_CAC。其 SELinux 标签为 disp_prop_handler_exec。
- **C30**（verified）：ClearKey DRM HAL 服务是含 17 个 NEEDED 条目的 ELF64 PIE。它为接口 1.0 至 1.4 注册 clearkey DRM 和 crypto 工厂。该服务为惰性服务，在 init 启动之前保持禁用，以用户 media 身份运行，标签为 vendor_hal_drm_clearkey_exec。其服务名为 vendor.drm-clearkey-hal-1-4。
- **C31**（verified）：drmserver 是面向 ARM 的 ELF32 PIE。它是 Android DRM 框架守护进程，仅当 drm.service.enabled 设为 true 或 1 时才会启动。以用户 drm 身份运行。其 NEEDED 列表包含 libdrmframeworkcommon.so、libbinder.so、libselinux.so 和 libstagefright_foundation.so，标签为 drmserver_exec。其 service manager 名称为 drm.drmManager。
- **C32**（verified）：libdisplayservicehidl.so 实现了 android.frameworks.displayservice@1.0 HIDL 服务：包括 DisplayService::getEventReceiver，以及带有 requestNextVsync、setVsyncRate 和 init 的 DisplayEventReceiver。其标签为 fwk_display_hwservice，并列于 framework manifest 中。
- **C33**（verified）：odm 分区中的 anorak Oculus composer 1.1 实现导出了 HIDL_FETCH_IComposer。它 NEEDED 了 libdisplayconfig.qti.so 和 libdrm.so，其 VINTF 片段声明了 IComposer 1.1，transport 为 passthrough，实例名为 default。
- **C34**（observed）：VINTF 中声明了两个 IComposer 2.4 HAL 服务，实例名不同：高通服务使用实例 default，Oculus 服务使用实例 display。仅当 Oculus 服务启用时，由 sysprop 选择的服务名才会设为 display。
- **C35**（observed）：在已挂载的分区中，libdrm.so 文件是用户空间 libdrm 库：它们导出 drmGetDevice2、drmAddBufs 及相关函数。它们有别于 Android DRM 框架库（libdrmframework、libmediadrm）以及 HAL 库。
- **C36**（verified）：mediadrm 签名者是一个 jar（仅包含 classes.dex 和 MANIFEST.MF），通过权限 XML 注册为共享库。其 arm 和 arm64 的 vdex 文件字节完全相同。
- **C37**（verified）：两个 hwclock 行是符号链接：system_a/system/bin/hwclock 指向 toybox，vendor_a/bin/hwclock 指向 toybox_vendor。
- **C38**（verified）：Widevine 和 ClearKey HAL 服务二进制文件均为惰性服务，启动前保持禁用，并使用相同的接口编号。Widevine 服务为接口 1.0 至 1.4 注册 widevine DRM 和 crypto 工厂。

## 措辞已修正的论断

下方结论以数据所支持的措辞重新表述。

- **C01**（verified）：该组有 132 个输入行：130 个普通文件和 2 个符号链接，分布于 odm_a（6）、system_a（65）、system_ext_a（7）和 vendor_a（54）。四个分区均以只读方式挂载。每个普通文件都存在于挂载点下，且与其记录的大小和 sha256 一致。2 个符号链接行（system_a/system/bin/hwclock -> toybox、vendor_a/bin/hwclock -> toybox_vendor）未记录 sha256（字段为 '-'），仅作为链接进行检查。
- **C08**（observed）：高通 composer 服务二进制文件包含 41 个以 vendor.display.、vendor.gralloc. 或 vendor.debug. 开头的字符串；其中 39 个是具体的属性键（另外两个是单纯的前缀 vendor.display. 以及模板 vendor.display.cac_ipd_adjust_%zu）。示例：vendor.display.enable_verbose_log、vendor.display.cac_scale_red、vendor.display.idle_time、vendor.display.disable_skip_validate、vendor.gralloc.disable_ubwc。
- **C09**（observed）：高通 composer 服务二进制文件包含路径字符串 /vendor/etc/display/hwc_display_resolution_extn.xml、/data/vendor/display 和 /data/vendor/display/frame_dump_external，以及转储文件名格式 %s/input_layer%d_%dx%d_%s_frame%d.raw 和 %s/frame_dump_disp_id_%02u_%s。已挂载的 vendor_a/etc/display/（仅包含 DPU930.xml）中不存在该分辨率 XML。这些路径的用途（扩展分辨率列表、帧转储）是根据其名称推断的。
- **C13**（verified）：当 strata composer 属性为 true 且 ro.vros.composer.cac_mode 为 writeback 时，Oculus composer 服务的 rc 会将 ro.surface_flinger.has_writeback_dpu_cac 设为 0（第 29 行）；只要 ro.vros.composer.cac_mode 为 writeback，就会将其设为 1（第 34 行），且不受 strata 属性条件限制。strata 启用时两个动作同时满足，init 按文件顺序执行，因此最终值为 1（根据 init 按顺序执行推断）；rc 注释中的 'unless set by Strata above' 并未被强制执行。该服务二进制文件读取 ro.vros.composer.cac_mode 和 ro.vros.composer.orientation。
- **C16**（verified）：surfaceflinger 在 system_a/system/etc/init/surfaceflinger.rc 中定义，为 init 类 core 和 animation 中的 /system/bin/surfaceflinger（class core animation），具有 SYS_NICE 以及三个 PDX 套接字（pdx/system/vr/display/client、manager、vsync）。其 SELinux 标签为 surfaceflinger_exec。厂商的 init.anorak.rc 同样定义了该服务，标记为 override 和 disabled，并在 vendor.syncbosshal.fw_update_complete=true 动作中启动它；文本未能确定 init 实际采用哪个定义。
- **C18**（verified）：DPU930.xml 是一个 XML 文档，大小为 2108 字节，sha256 为 e46c07687f208cc0b715d4ae1ff8c79e6873d15e9ed4b715e35683384e73c4c6。其根元素为 <XML>，唯一的子元素为 SDE930 元素。SDE930 包含 74 个有效的 Param 条目（Param0 至 Param73），以及第 83 至 89 行处一个被注释掉的 Overrides 示例。数值参数的含义未确定。
- **C19**（observed）：libdisplayqos.so（64 位和 32 位副本）的字符串表中包含 /vendor/etc/display/ 以及 DPU 型号文件名 DPU660.xml、DPU670.xml、DPU6__.xml、DPU720.xml、DPU7__.xml、DPU820.xml、DPU830.xml、DPU8__.xml 和 DPU930.xml。它是唯一包含 DPU930.xml 名称的已挂载二进制文件，因此很可能是 DPU 参数文件的读取者（推断；未追踪读取这些文件的代码）。
- **C21**（verified）：在已挂载文件中，唯一引用 /system/etc/display.conf 的是 system_ext_a 中的 vrdevicemanagerserver，它没有本组的行（不属于本组）。已挂载分区中的 init rc 或 SELinux 文件均未引用 display.conf。若直接搜索 display\\.conf，也会匹配 display.config 服务上下文名称，因此检查必须使用单词边界。
- **C22**（verified）：厂商 QTI composer 3.2 库（vendor_a/lib64）导出了 IQtiComposer 和 IQtiComposerClient 类（castFrom 及相关函数），定义了 269 个动态符号，并 NEEDED 了 graphics common 1.0 至 1.2 库、composer 2.1 至 2.4，以及厂商 QTI composer 3.0 和 3.1 库。
- **C27**（observed）：display_tool 是面向 AArch64 的 ELF64 PIE，已剥离符号，并使用 linker64。它会为众多子命令打印用法，包括 brightness、configure_cabc、configure_cabc_gray_interp、configure_cac、lock_backlight、overide_display_temperature、read_dcs、write_dcs、write_dcs_unicast、toggle_cabc、toggle_cabc_gray_interp、toggle_cac、toggle_display_power 和 refresh_rate。它会打印 'Root permissions required. Run adb root and try again.'。它 NEEDED 了 vendor.oculus.hardware.graphics.composer@1.1.so。已挂载分区中没有其他文件包含字符串 display_tool。

## 缺口

- 二进制文件已剥离符号。行为信息取自动态导出符号、字符串、NEEDED 列表以及 init/VINTF/SELinux 文本。未进行反汇编或反编译，因此内部调用图和参数语义未知。
- 高通 composer 服务与 Oculus StrataComposer 之间的运行时选择未确定。它取决于设备上 persist.device_config.oculus_shared_os_services.mros_enable_strata_composer 的值（该值不在镜像中）以及 init 的启动顺序。
- 服务 vendor.qti.hardware.display.composer 存在两个定义（独立 rc 文件，以及带有 override 和 disabled 的 init.anorak.rc）。init 实际生效的定义无法从文本中确定。
- DPU930.xml 中 74 个数值 Param 条目的含义，以及 display.conf 键（除名称和取值之外）的含义均未确定。未检查 libdisplayqos.so 的解析逻辑。
- display.conf 在本组内没有使用者。已确认的唯一使用者是 system_ext_a 中的 vrdevicemanagerserver，不在本组之内。
- 在文本策略和 file_contexts 文件中未找到 system_ext_a/bin/display_tool 或 display_validator 的 SELinux 标签。未解析编译后的 precompiled_sepolicy 二进制文件。
- 未对若干 32 位库进行导出符号枚举：libhdr_tm.so、libsdedrm.so、libwvdrmengine.so、libdrmfs.so、libdrmtime.so 和 libmemtrackproxy.so。它们的用途取自 NEEDED 列表和名称；libdrmfs、libdrmtime 以及两个 libmemtrackproxy 行均标记为 inferred。
- 本组之外的依赖项未作分类，仅列出名称：libsdmcore.so、libqservice.so、libqdutils.so、libqdMetaData.so、libgralloccore.so、libgrallocutils.so、libgralloctypes.so、vendor.qti.hardware.display.mapper@4.0.so、vendor.display.config@2.0.so、android.hardware.graphics.common@1.x、android.hardware.power@1.3.so、libhardware.so、libprotobuf-cpp-lite-3.9.1.so 和 libQSEEComAPI.so。
- libdisp-aba 中自适应背光缩写 CABL、FOSS 和 SVI 的全称未经确认。仅观察到符号前缀。
- 在运行中的设备上加载的是 32 位还是 64 位 anorak composer 实现，尚未确定。
- 未分析 Widevine 引擎（libwvdrmengine.so）的内部结构以及 DRM 栈的内容保护行为。安全属性不在范围之内。
- 此处描述的内容是各分区的挂载副本，而非原始 super 镜像。
- 五个 inferred 行（libmemtrackproxy.so 的两个变体，以及 libdrmfs.so 和 libdrmtime.so 的 32 位与 64 位变体）的用途仅依据 NEEDED 列表和文件名。
- 运行中设备上 display.vendor.skip.init 和 display.enable_ddic_cac 的值未确定，因此 display_property_handler 是否运行未知。
