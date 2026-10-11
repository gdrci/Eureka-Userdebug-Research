# 18 - 媒体与编解码器

[English](18-media-codecs.md)

## 摘要

媒体编解码器组共有 163 个文件：system_a 中 49 个，system_ext_a 中 1 个，vendor_a 中 113 个。system 一侧包含 Stagefright 和 Codec2 框架库（32 位和 64 位）、media.extractor 与 media 服务可执行文件，以及一个 Mainline APEX（com.android.media.swcodec，版本 319999900），该 APEX 内含 mediaswcodec 服务、一组 Codec2 软件插件，以及其自带的 media_codecs.xml、init.rc 和 seccomp 策略。vendor 一侧包含每种位数 29 个 Codec2 软件插件（每个都导出 CreateCodec2Factory）、14 个旧版 OMX 软件插件（createSoftOMXComponent）、高通 QC2 层（libqcodec2_*，包括一个 V4L2 插件）、供 vendor.qti.media.c2 HAL 使用的 Codec2 HIDL 服务端库、一个 ClearKey MediaCas 插件，以及 XML/JSON 配置（media_codecs_anorak.xml、media_codecs_performance_anorak.xml、media_profiles_V1_0.xml、media_profiles_anorak.xml、media_anorak/video_system_specs.json）。变体后缀 _anorak 来自 vendor build.prop 中的 ro.media.xml_variant.codecs 和 ro.media.xml_variant.codecs_performance。其中一行 system_ext_a/lib64/libcodecwrapper.so 是用于可穿戴输入遥测的 protobuf 封装库，不是媒体编解码器。media_codecs_anorak.xml 所引用的若干 XML 包含文件在已挂载的分区中不存在。

角色表：`data/subsystems/media-codecs/roles.tsv`（163 个文件：103 个 verified，56 个 observed，4 个 inferred，0 个 unverified）。

## 已验证的论断

- **grp-01** (verified)：该组共 163 行：system_a 中 49 行，system_ext_a 中 1 行，vendor_a 中 113 行。每行的 kind 均为 file。
- **grp-02** (verified)：全部 163 个文件均存在于 MNT/<partition>/<path>，且大小和 sha256 与记录一致。
- **grp-03** (verified)：roles.tsv 有一行表头和 163 行数据。其 (partition, path) 键与输入完全一致。置信度计数为 103 verified、56 observed、4 inferred 和 0 unverified。
- **apex-01** (verified)：swcodec Mainline 模块是名为 com.android.media.swcodec、版本为 319999900 的 APEX。其负载是带 extents 的 ext2 镜像。
- **apex-02** (verified)：APEX 负载目录树包含 bin/mediaswcodec、etc/{init.rc, ld.config.txt, media_codecs.xml, seccomp_policy/}，以及 lib64 下的 72 个库，其中包括 Codec2 软件插件、libvpx.so、libopus.so 和 libmedia_codecserviceregistrant.so。
- **apex-03** (verified)：APEX 的 init.rc 声明了一个服务 media.swcodec，它以用户 mediacodec 运行 /apex/com.android.media.swcodec/bin/mediaswcodec，并使用任务配置文件 ProcessCapacityHigh 和 HighPerformance。
- **apex-04** (verified)：mediaswcodec 是一个去除了符号的 64 位 AArch64 PIE，它链接 libmedia_codecserviceregistrant.so，并输出日志 'media swcodec service starting'。它注册名称 media.swcodec，并从 APEX 或 vendor 路径读取其 seccomp 策略。
- **mediaextractor-01** (verified)：/system/bin/mediaextractor 是一个 AArch64 PIE，它注册 media.extractor 服务（MediaExtractorService）。它链接 libmedia.so 和 libmediaextractorservice.so，并调用 limitProcessMemory。
- **mediaextractor-02** (observed)：init、SELinux 文件、service 和 domain 文件均引用了 mediaextractor。其 seccomp 策略分别在 /apex/com.android.media/etc/seccomp_policy/mediaextractor.policy 和 /vendor/etc/seccomp_policy/mediaextractor.policy 中查找。
- **mediaserver-02** (observed)：mediaserver 服务名称在平台 service contexts 中声明。该二进制文件的 file context 为 mediaserver_exec。
- **libstagefright-01** (verified)：libstagefright.so（64 位）是核心 Stagefright 库。其导出符号涵盖 MediaCodec 和 ACodec 状态机、MediaCodecList、MPEG4Writer、MediaCodecSource 和 NuMediaExtractor，并导出 136 个 AMEDIAFORMAT_KEY_* 常量。
- **libstagefright-02** (observed)：libstagefright.so 引用了 Codec2 和 OMX 库，以及媒体配置和性能分析文件路径。它指定了 /apex/com.android.media/lib64/extractors，并加载一个可选的 libmediaformatshaper.so。
- **libcodec2_vndk-01** (verified)：libcodec2_vndk.so（system，64 位）是 Codec2 VNDK 框架。它导出 C2 buffer、block 和 store API、平台 store 插件加载器（C2PlatformStorePluginLoader），以及 DMA-heap 和 ION 使用结构。
- **libcodec2_vndk-02** (observed)：libcodec2_vndk.so 读取 debug.c2.use_dmabufheaps 属性，并选择 DMA-heap 路径。system 副本还引用 media.c2.dmabuf.padding，vendor 副本则使用 /dev/dma_heap/system 和 /dev/ion。其 file 和 property context 由 SELinux 设置。
- **libcodec2-01** (verified)：system_a 中的 32 位和 64 位 libcodec2.so 文件都是链接桩。它们不导出任何动态符号，且仅从 libc 导入 __cxa_finalize。
- **libcodec2_client-01** (verified)：libcodec2_client.so（system，64 位）是 Codec2 HIDL 组件 store 的客户端封装。其 Codec2Client::CreateFromAllServices 和 Codec2Client::Component 方法涵盖队列、启动、停止、flush、drain、reset、release、输出 surface 以及 block pool 设置。
- **libsfplugin_ccodec-01** (observed)：libsfplugin_ccodec.so 是将 stagefright MediaCodec 路径连接到 Codec2 的 CCodec 插件。它的配置文件名取自 media_codecs.xml、media_codecs_c2.xml、media_codecs_performance.xml、media_codecs_performance_c2.xml 和 media_codecs_shaping.xml，并从 swcodec APEX 的 etc 目录和 formatshaper 路径读取。
- **xmlparser-01** (verified)：libstagefright_xmlparser.so（64 位）是 MediaCodecsXmlParser。它基于 expat 构建，解析 media_codecs*.xml 的语法：Include、Settings、Decoders、Encoders、MediaCodec、Alias、Limit、Feature、Quirk、Variant、Tuning 和 Mapping。
- **variant-01** (verified)：vendor build.prop 为两个编解码器 XML 文件和目标都选择了 _anorak 变体。SELinux 声明了对应的变体属性。用于选择 media profiles 变体的 ro.media.xml_variant.profiles 未在任何已挂载的 build.prop 中设置。
- **vendor-xml-01** (verified)：vendor_a/etc/media_codecs_anorak.xml 是一个格式良好的 Codec2 编解码器列表，包含 12 个解码器（avc、hevc、vp9 和 av1，每种都有 plain、low_latency 和 secure 变体）以及 4 个编码器（avc、hevc、hevc.cq 和 heic）。它包含 media_codecs_google_audio.xml，以及两个在已挂载分区中缺失的文件。
- **perf-xml-01** (verified)：media_codecs_performance_anorak.xml 是高通的覆盖层。它包含 15 个 MediaCodec 条目，每个都带有 update="true"，用于为 Codec2 解码器和编码器添加 measured-frame-rate-<WxH> 限制。
- **profiles-02** (verified)：media_profiles_anorak.xml 是一个 MediaSettings 变体，包含 CamcorderProfiles、VideoEditorCap（maxInputFrameWidth）和 ExportVideoProfile。其内部 DTD 未声明 cameraId 属性，并拒绝质量值 1080p，共产生 11 个有效性错误。
- **profiles-dtd-01** (observed)：独立的 media_profiles_V1_0.dtd 定义了 MediaSettings 架构。其元素包括 CamcorderProfiles、EncoderProfile（quality、fileFormat、duration）、Video（codec、bitRate、width、height、frameRate）、Audio（codec、bitRate、sampleRate、channels）、ImageEncoding、ImageDecoding（memCap）、Camera、EncoderOutputFileFormat、VideoEncoderCap、AudioEncoderCap、VideoDecoderCap 和 AudioDecoderCap。
- **video-specs-02** (observed)：libqcodec2_platform.so 构建路径 /vendor/etc/media（加上后缀）以及文件名 video_system_specs.json。这些 JSON 键在 QC2 platform、utils 和 V4L2 库中被引用。
- **qc2-v4l2-01** (verified)：libqcodec2_v4l2codec.so（vendor，64 位）是 QC2 V4L2 编解码器插件。它导出 QC2Codec_GetPlugin 和 QC2V4l2CodecPlugin::getSupportedCodecs 条目，并使用 V4L2 和 media 设备节点。
- **qc2-core-01** (observed)：QC2 的 base、core、base-codec、hooks 和 platform 库构成一个分层堆栈。libqcodec2_core.so 导出 QC2ComponentStoreFactoryGetter。libqcodec2_hooks.so 导出 QC2ComponentObserver 及其默认 observer，并链接 1.0 和 1.1 版本的 android.hardware.power HAL。
- **vendor-hidl-01** (verified)：libcodec2_hidl@1.0.so（vendor，64 位）是 android.hardware.media.c2@1.0 的 Codec2 HIDL 服务端实现。它提供 utils::ComponentStore、Component 和 CachedConfigurable，并链接 vendor Codec2 VNDK 以及 filter 插件库。
- **vendor-hal-01** (verified)：vendor.qti.media.c2@1.0-service HAL 二进制文件由 init 以用户 mediacodec 身份作为 vendor-qti-media-c2-hal-1-0 启动。它注册 Codec2 IComponentStore，并从 QC2ComponentStoreFactoryGetter 获取其 store 工厂。其 seccomp 策略为 codec2.vendor.base-arm64.policy 和 codec2.vendor.ext-arm64.policy。它通过 dlopen 打开的库名称尚未确定。
- **soft-plugins-01** (verified)：每种位数下各有 29 个 Codec2 软件组件插件，分别位于 lib 和 lib64 中，另有 libcodec2_soft_common.so。每个插件导出 CreateCodec2Factory 和 DestroyCodec2Factory，并命名一个 c2.android.* 组件。libcodec2_soft_common.so 导出 SimpleC2Component 和 SimpleC2Interface。
- **soft-aacdec-01** (verified)：libcodec2_soft_aacdec.so（vendor，64 位，964592 字节）是 AAC 解码器 Codec2 软件插件。它导出 CreateCodec2Factory 和 DestroyCodec2Factory，并依赖 Codec2 VNDK、通用 soft 库和 CCodec 工具库。
- **legacy-omx-01** (verified)：vendor_a/lib 中有 14 个旧版 OMX 软件插件（libstagefright_soft_*.so）。每个都导出 createSoftOMXComponent。部分插件报告 OMX.google.* 组件名称。libstagefright_softomx.so 提供 SimpleSoftOMXComponent 基类，libstagefright_softomx_plugin.so 导出 createOMXPlugin。
- **selinux-file-01** (observed)：SELinux 文件上下文为编解码器二进制文件和库分配其标签。有两个条目指定的路径在已挂载分区中不存在：/system/bin/mediaswcodec（该二进制文件随 APEX 提供）和 /vendor/bin/hw/android.hardware.media.c2@1.0-service-v4l2-64。
- **hwservice-01** (observed)：Codec2 HIDL 接口在两个分区中均以 hwservice 名称注册，标签为 hal_codec2_hwservice。平台属性 media.c2.dmabuf.padding 的标签为 codec2_config_prop。

## 措辞已修正的论断

- **sysext-01** (verified)：system_ext_a/lib64/libcodecwrapper.so 是基于 protobuf 的可穿戴输入遥测编解码封装库（包括 com::oculus::wearableinputservice 命名空间中的 IMU、EMG、手势、电池、设备状态和触觉消息），不是媒体编解码器。它内嵌 protobuf 运行时，且没有 libprotobuf 依赖。其引用方包括 WearableInputService.apk（其中打包了 lib/arm64-v8a/libcodecwrapper.so 和 lib/arm64-v8a/libwis_input_device.so），以及两个链接它的 system_ext 库：libwis_input_device.so（同样打包在 APK 中）和 wis_handlers.so（一个独立的 system_ext 库）。
- **apex-05** (verified)：APEX 的 media_codecs.xml 列出 9 个 Codec2 解码器和 6 个编码器，均以 c2.android.* 前缀命名。15 个中有 14 个带有 OMX.google 别名；c2.android.opus.encoder 没有别名。该文件中没有 Include 元素。
- **apex-06** (observed)：mediaswcodec 的 seccomp 策略是每行一条目的允许列表。系统调用条目由名称后跟 ': 1' 或参数表达式组成。该文件还包含一条 @include 指令，用于引入 code_coverage 策略。文件头注释指出，ioctl 调用通过 SELinux 策略进行过滤。
- **mediaserver-01** (verified)：/system/bin/mediaserver 是一个 32 位 ARM EABI5 PIE。它实例化 MediaPlayerService 和 ResourceManagerService，配置并加入 HIDL RPC 线程池（configureRpcThreadpool 和 joinRpcThreadpool），并链接 android.hardware.media.omx@1.0.so。其 init 服务名为 media。
- **vendor-xml-02** (observed)：每个 MediaCodec 元素包含 Limit 和 Feature 子元素；16 个中有 13 个还包含一个 Alias 子元素（三个 c2.qti.av1.decoder 条目均没有）。Limit 名称包括 size、alignment、block-size、block-count、blocks-per-second、bitrate、frame-rate、concurrent-instances、complexity、quality 和 performance-point-<WxH>（性能点是 Limit 名称，而非单独的元素）。Feature 名称包括 adaptive-playback、low-latency、intra-refresh、video-minimum-quality、bitrate-modes 和 secure-playback，其中 secure-playback 在每个带有它的条目中都设为 required=true。解析器字符串包括 rank、supports- 和 mapping-。
- **profiles-01** (verified)：media_profiles_V1_0.xml（vendor）是一个 MediaSettings 文档，包含 cameraId 为 0 和 1 的 CamcorderProfiles、25 个 EncoderProfile 条目、6 个 ImageEncoding 条目，以及编码器和解码器能力项（2 个 VideoEncoderCap、5 个 AudioEncoderCap、1 个 VideoDecoderCap、1 个 AudioDecoderCap）。其内部 DTD 还声明了 VideoEditorCap 和 ExportVideoProfile，但文件中没有它们的实例。其内部 DTD 子集产生 61 个有效性错误；将子集替换为对系统 DTD 副本的引用后，验证结果为 0 个错误。
- **video-specs-01** (verified)：media_anorak/video_system_specs.json 是面向高通编解码器平台的带注释 JSON 文件。它包含 Video.Features（3 个布尔值）、Video.Presets（5 个值）、空的 QC2CodecPlugins 和 QC2PreProcessingFilters 列表，以及一个空的 QC2PostProcessingFilters 对象。QC2PreProcessingFilters 这个名称还在注释块中作为多流示例出现过一次（第 56 行）；有效键位于第 60 行。
- **selinux-type-01** (observed)：SELinux 在平台策略中声明了 mediaswcodec（hal_codec2_server 的 service domain），并在 vendor 策略中声明了 mediacodec，其通过 mediacodec_exec 进行 init 转换。平台的 hal_codec2_client 集合包括 cameraserver、mediaserver 和 mediaswcodec。vendor 策略将 mediacodec 放入 hal_codec2 和 hal_codec2_server 集合，因此它是服务端成员，而非客户端。
- **misc-libs-01** (observed)：libstagefright_flacdec.so 是一个 FLAC 解码器（含 FLAC__ 符号），被 FLAC 解码器软件插件 libcodec2_soft_flacdec.so 和 libstagefright_soft_flacdec.so 使用。libstagefright_enc_common.so 提供 cmnMem* 辅助函数，被 AMR-WB 编码器软件插件 libcodec2_soft_amrwbenc.so 和 libstagefright_soft_amrwbenc.so 使用。libclearkeycasplugin.so 是 ClearKey MediaCas 插件（ClearKeyCasPlugin，包含 openSession 和 processEcm）。DTD 和配置文件加载器在 profiles 相关论断中有所描述。
- **seccomp-01** (observed)：seccomp 策略文件共用一种格式：每行一个条目，即 'name: 1' 或 'name: <argument expression>'。例外是 @include 指令：mediacodec.policy 和 mediaextractor.policy 各有两条，mediaswcodec APEX 策略中有一条。Codec2 vendor 的 base 和 ext 策略（vendor_a/etc/seccomp_policy 中的 codec2.vendor.base-arm64.policy 和 codec2.vendor.ext-arm64.policy）没有指令。mediacodec.policy 仅以 system 副本形式存在（system_a/system/etc/seccomp_policy）；mediaextractor.policy 既有 system 副本，也存在于 com.android.media APEX 负载中。两者均没有 vendor 副本。

## 缺口

- 由 vendor 编解码器列表引用的包含 XML 文件 media_codecs_c2_audio.xml 和 media_codecs_google_c2.xml 不存在于任何已挂载分区中，因此其编解码器条目未知。
- stagefright 和 CCodec 库所引用的文件（media_codecs_c2.xml、media_codecs_performance_c2.xml、media_codecs_shaping.xml、system 的 media_codecs.xml、media_profiles.xml）不存在于已挂载分区中。运行时的搜索顺序以及哪个副本生效均未确定。
- media profiles 变体的选择取决于 ro.media.xml_variant.profiles。该属性未在任何已挂载的 build.prop 中设置，因此运行时加载的是 media_profiles_anorak.xml 还是 media_profiles_V1_0.xml 尚不清楚。
- 库中所指定的运行时文件 /data/misc/media/media_codecs_profiling_results.xml 和 /data/vendor/media 不在输入中，因此未作分析。
- 无法从字符串中确定 vendor.qti.media.c2@1.0-service 通过 dlopen 打开的库名称（即 QC2HALService 的 open 消息）。仅读取了其 NEEDED 列表和导出的 QC2ComponentStoreFactoryGetter 名称。
- init 和 SELinux 所引用的两个 vendor 二进制文件 vendor_a/bin/hw/android.hardware.media.omx@1.0-service 和 vendor_a/bin/hw/vendor.qti.media.c2@1.0-service 不属于本组的行。它们仅检查了 NEEDED 列表和字符串。
- system_a/system/apex/com.android.media.capex（2514355 字节）引用了 mediaextractor.policy，但不属于本组的行。未作分析。
- /vendor/bin/hw/android.hardware.media.c2@1.0-service-v4l2-64 的 vendor_file_contexts 条目在 vendor_a 中没有对应的二进制文件，/system/bin/mediaswcodec 的 plat_file_contexts 条目在 system_a 中没有对应的文件。这些条目在运行时的标签结果未确定。
- libcodecwrapper.so（system_ext_a/lib64）是可穿戴输入的 protobuf 封装库，不是媒体编解码器。其所属分组应予复核；已分析但未重新分配。
- 未列举 32 位 system 库以及大多数 32 位 vendor 库的导出符号族。它们的角色依赖于 NEEDED 列表、名称以及对应的 64 位版本。其置信度相应标记为 observed 或 inferred。
- ext2 APEX 负载中的 ld.config.txt（8226 字节）、code_coverage.arm64.policy 和 crash_dump.arm64.policy 已列出，但未作分析。
- mediaswcodec APEX 中的 libcodec2_hidl@1.0.so（282944 字节）与 vendor 副本（283184 字节）大小不同。该差异未作分析。
- ClearKey 许可 protobuf 消息定义和 ClearKey 会话逻辑未作深入分析，仅涉及导出的类名和方法名。
- media_codecs_anorak.xml 中组件级编解码器能力值（limit 范围）未与性能覆盖层的值或运行时查询结果进行比较。
