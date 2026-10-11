# 14 - 音频

[English](14-audio.md)

## 摘要

音频组包含 239 行：238 个普通文件和 1 个符号链接，分布在 product_a（51）、system_a（111）、system_ext_a（12）和 vendor_a（65）中。该组包含：Android 音频框架（audioserver 可执行文件、AudioFlinger 与 AudioPolicy 库、AAudio、AIDL 客户端存根，以及 4.0 至 7.0 版的 HIDL 音频接口库）；厂商 HAL 服务和 HAL 模块（为 anorak 平台链接 vendor.qti PAL 与 AGM 组件的 primary HAL，以及通用的 primary、USB、remote-submix 和 A2DP 模块）；包含 26 个 AELI 效果库的效果栈（每个 lib 目录 13 个，另有效果工厂和 HAL 实现库）；策略、效果、IO 配置文件和 mixer-path 配置文件；system_ext 中的 audiotapper 服务，以及延迟和录音测试工具；49 个 Vorbis-in-Ogg 格式的界面、闹钟、通知和铃声音频片段。init 与 SELinux 引用涵盖 audioserver（系统 init 文件、进程和服务标签）、厂商 HAL 服务（同一服务名的两个 init 定义，以及 HAL 标签）、audiotapper 对（带属性触发器的已禁用服务及 SELinux 类型），以及高通 ADSP RPC 守护进程。roles 文件包含一行表头和 239 行数据。

角色表：`data/subsystems/audio/roles.tsv`（239 个文件：230 个 verified，2 个 observed，7 个 inferred，0 个 unverified）。

## 已验证的论断

- **A01**（已验证）：音频组有 239 行：product_a 51、system_a 111、system_ext_a 12、vendor_a 65；其中 238 个是普通文件，1 个是符号链接。
- **A02**（已验证）：全部 238 个普通文件的大小和 sha256 与记录一致；符号链接 system_ext_a/priv-app/AudioLatencyTester/lib/arm64/libaudiolatencytester_jni.so 解析到 /system_ext/lib64/libaudiolatencytester_jni.so。
- **A03**（已验证）：238 个普通文件包括 93 个 ELF64 共享对象、72 个 ELF32 共享对象、7 个 ELF64 PIE 可执行文件、49 个 Ogg 文件、13 个 XML 文档、3 个 ASCII 文本文件和 1 个 shell 脚本。
- **A06**（已验证）：有 26 个共享对象导出效果库符号 AELI：vendor_a/lib/soundfx 下 13 个，vendor_a/lib64/soundfx 下 13 个（bundlewrapper、volumelistener、visualizer、ldnhncr、qcomvisualizer、qcompostprocbundle、dynproc、downmix、qcomvoiceprocessing、effectproxy、audiopreprocessing、reverbwrapper、hapticgenerator）。
- **A07**（已验证）：组内全部 165 个共享对象的 SONAME 与其文件名相同；7 个 ELF 可执行文件没有 SONAME。
- **D01**（已验证）：system_a/system/bin/audioserver 是 AArch64 PIE（ELF64，解释器 /system/bin/linker64，已剥离符号表）。它有 20 个 DT_NEEDED 条目，包括 libaudioflinger.so、libaudiopolicyservice.so、libaaudioservice.so、libmedia.so 和 libbinder.so。其未定义符号包括 AudioFlinger::instantiate 与 AudioPolicyService::AudioPolicyService。
- **D02**（已验证）：init 服务 audioserver 定义于 system_a/system/etc/init/audioserver.rc：class core、user audioserver、包含 audio、media 和 wakelock 的 group 列表、capability BLOCK_SUSPEND，以及 onrestart 行，用于重启 vendor.audio-hal、vendor.audio-hal-4-0-msd、audio_proxy_service、vendor.audio-hal-2-0 和 audio-hal-2-0。它还会创建 /dev/socket/audioserver。
- **D03**（已验证）：SELinux 文件上下文将 /system/bin/audioserver 标记为 audioserver_exec，服务上下文将 media.audio_flinger、media.audio_policy、media.aaudio、media.log 和 media.sound_trigger_hw 标记为 audioserver_service。
- **D04**（已验证）：system_a/system/lib64/libaudioflinger.so 导出 AudioFlinger::instantiate（即 audioserver 导入的符号）。其 DT_NEEDED 列表包括 libaudiohal.so、libaudioclient.so、libaudioprocessing.so、libaudiofoundation.so 和 libaudiotapper.oculus.so。
- **D05**（已验证）：libaudiopolicymanagerdefault.so 中保存了文件名 audio_policy_configuration.xml、audio_policy_configuration_a2dp_offload_disabled.xml 和 audio_policy_configuration_bluetooth_legacy_hal.xml，以及搜索前缀 /system/etc、/vendor/etc 和 /vendor/etc/audio/sku_。libaudiopolicyservice.so 中保存了路径 /system/etc/audio_effects.conf 和 /vendor/etc/audio_effects.conf。
- **D06**（已验证）：vendor_a/bin/hw/android.hardware.audio.service_64 是 AArch64 PIE，有 10 个 DT_NEEDED 条目（libcutils、libbinder、libhidlbase、liblog、libutils、libhardware、libc++、libc、libm、libdl）。其字符串中列出 HIDL 接口 IDevicesFactory（4.0 至 7.0）、IEffectsFactory（4.0 至 7.0）、ISoundTriggerHw（2.0 至 2.3）、IBluetoothAudioProvidersFactory（2.0、2.1）和 IBluetoothAudioOffload（1.0），另有字符串 audiohalservice 以及属性 persist.vendor.audio.service.hwbinder.size_kbyte。
- **D07**（已验证）：有两个 init 源为同一可执行文件定义了服务 vendor.audio-hal：vendor_a/etc/init/android.hardware.audio.service_64.rc（第 1 行，capabilities BLOCK_SUSPEND，无 override）和 vendor_a/etc/init/hw/init.anorak.rc（第 281 行，第 282 行以 'override' 开头，第 286 行为 'capabilities BLOCK_SUSPEND SYS_NICE'）。两种定义的运行时效果未被复现。
- **D10**（已验证）：system_a/system/lib64/libaudiohal@7.0.so 导出 createIDevicesFactory 和 createIEffectsFactory。它是 7.0 HAL 接口的客户端封装。
- **D11**（已验证）：vendor_a/lib64/hw/audio.primary.anorak.so 导出 HMI 符号，有 23 个 DT_NEEDED 条目，包括 libar-pal.so、libagm.so、libmros_vendor_sysprops.so、vendor.qti.hardware.pal@1.0.so 和 vendor.qti.hardware.AGMIPC@1.0.so；其字符串包括路径 /vendor/etc/default_volume_tables.xml、/vendor/etc/microphone_characteristics.xml、/data/vendor/audio/dsp_config_render 和 /data/vendor/audio/dsp_config_capture，以及属性 persist.vendor.audio.va_concurrency_mute_enabled 和 vendor.audio.feature.* 属性。
- **D12**（已验证）：init.anorak.rc 创建 /data/vendor/audio（模式 0770，属主 audio）以及 acdbdata 子目录（第 92 至 97 行）。vendor_file_contexts 第 243 行将 /data/vendor/audio(/.*)? 标记为 vendor_audio_data_file。
- **D13**（已验证）：vendor_a/lib64/hw/android.hardware.audio@7.0-impl.so 导入 hw_get_module_by_class，包含字符串 primary，并在其 DT_NEEDED 条目中列出 libhardware.so、libmedia_helper.so 和 libmediautils_vendor.so。它实现了 audio::V7_0 类。
- **D14**（已验证）：vendor_a/etc/audio_policy_configuration.xml 为 audioPolicyConfiguration 7.0 版，包含一个模块（name 为 primary，halVersion 为 2.0），defaultOutputDevice 为 bus1_default，globalConfiguration 的属性 speaker_drc_enabled 和 call_screen_mode_supported 设为 true。模块列表中包含 usb_audio_policy_configuration.xml、r_submix_audio_policy_configuration.xml 和 bluetooth_audio_policy_configuration_7_0.xml，末尾包含 audio_policy_volumes.xml 和 default_volume_tables.xml。
- **D15**（已验证）：primary 模块列出 12 个 mixPort、13 个 devicePort 和 13 个 route 元素。mixPort 名称包括 mixport_bus1 至 mixport_bus5、mixport_bus1_mmap、hifi_playback、raw input、primary input、mmap_no_irq_in、assistant input 和 usb_device input。指向 bus1_default 的 route 以 mixport_bus1 和 mixport_bus1_mmap 为源。devicePort 的 tagName 包括 bus1_default 至 bus5_spatial、Wired Headset、Wired Headphones、USB Headset Out、USB Device Out、Built-In Mic、Wired Headset Mic、USB Device In 和 USB Headset In。
- **D16**（已验证）：system_a/system/etc/audio_effects.conf 采用旧版格式。其 libraries 块有 6 个活动条目（bundle、reverb、visualizer、downmix、loudness_enhancer、dynamics_processing），每个条目的路径都在 /vendor/lib/soundfx 下。其 effects 块有 12 个活动条目，每个条目都有 library 和 uuid 键。另有 10 行 uuid 被注释掉。
- **D17**（已验证）：vendor_a/etc/audio_effects.xml（schema 为 http://schemas.android.com/audio/audio_effects_conf/v2_0）有 8 个活动的 library 元素和 14 个活动的 effect 元素。其 pre_processing 库、agc、aec 和 ns 效果，以及 preprocess 和 postprocess 块都位于 XML 注释中，因此该文件没有定义任何活动的前处理或后处理 stream 块。
- **D18**（已验证）：三个库（system_a/system/lib64/libeffectsconfig.so、vendor_a/lib64/libeffectsconfig.so 和 vendor_a/lib/libeffectsconfig.so）包含字符串 audio_effects.xml 和 audio_effects_conf。audio_effects.xml 所列的降噪库 libnseffect.so 不在已挂载的分区中，也不在 classified.tsv 中。
- **D19**（已验证）：vendor_a/etc/mixer_paths_anorak_max98388.xml 只有一个 mixer 根元素（第 33 至 965 行），包含 190 个 path 块和 493 个 ctl 条目。path 名称包括 hifi-filter-coefficients、gsm-mode、amic1、amic2、amic3、speaker、headphones 和 handset。
- **D20**（已验证）：system_a/system/lib64/libaudioroute.so 包含字符串 mixer_paths.xml，并导出 audio_route_init、audio_route_apply_path 和 audio_route_reset。未确定该通用名称与 vendor_a/etc/mixer_paths_anorak_max98388.xml 之间的对应关系。
- **D22**（已验证）：vendor_a/lib64/soundfx/libaudiopreprocessing.so 导出 AELI 效果符号（OBJECT，56 字节，位于 0xe0800）。其解修饰后的导出包括 webrtc::EchoCanceller、webrtc::AudioProcessingImpl、webrtc::rnn_vad 和 webrtc::ApmDataDumper。其 DT_NEEDED 列表为 liblog.so、libutils.so、libc++.so、libc.so、libm.so 和 libdl.so。在 vendor_a/etc/audio_effects.xml 中，它的唯一引用（即该库以及 agc、aec 和 ns 效果）都位于注释中。
- **D23**（已验证）：system_ext_a/bin/audiotapper 是 AArch64 PIE，依赖 libaudiotapperservice.so、libbinder.so、libcutils.so、liblog.so、libutils.so 和 packagemanager_aidl-cpp.so。它会输出日志 'audiotapper service started' 和 'audiotapper service failed to start (%d): %s'。system_ext_a/bin/audiomonitor 是一个 /system/bin/sh 脚本，通过 dumpsys 查询 media.audiotapperhost。
- **D24**（已验证）：system_ext_a/etc/init/audiotapper.rc 定义了服务 audiotapper（/system_ext/bin/audiotapper；class core、user system、group system、disabled）。它在 post-fs-data 阶段创建 /data/audiotapper（0770 system system），并根据属性 persist.ovr.audiotapper.enable_service 启动或停止该服务。system_ext_a/etc/init/audiotapperhost.rc 定义了服务 audiotapperhost（class late_start、disabled），并等待 init.svc.audiotapper 处于运行状态。
- **D25**（已验证）：audiotapper 这一对的 SELinux 标签为：/data/audiotapper 为 audiotapper_file，/system_ext/bin/audiotapper 为 audiotapper_exec，/system_ext/bin/audiotapperhost 为 audiotapperhost_exec。system_ext_sepolicy.cil 声明了以下类型：audiotapper、audiotapper_exec、audiotapperhost、audiotapperhost_exec、audiotapper_prop、audiotapper_service、audiotapperhost_service 和 audiotapper_file。
- **D26**（已验证）：vendor_a/bin/audioadsprpcd 是 AArch64 PIE，解释器为 /system/bin/linker64，有 5 个 DT_NEEDED 条目（liblog、libc++、libc、libm、libdl）。其字符串包括 DSP_LIBRARY_PATH 相关消息，以及源码路径 vendor/qcom/anorak/opensource/audio-hal/primary-hal/adsprpcd/adsprpcd.c。它由 vendor_a/etc/init/vendor.qti.audio-adsprpc-service.rc 以 vendor.audioadsprpcd_audiopd 的名义启动（class main、user media、group audio media），其 SELinux 标签由 vendor_file_contexts 第 429 行定义为 vendor_audioadsprpcd_exec。
- **D27**（已验证）：组内的七个 ELF 可执行文件为 audioserver、audio_capture、audiotapperhost、audio_tool、audiotapper、audioadsprpcd 和 android.hardware.audio.service_64。audio_tool 包含选项名 --config_file、--getparam、--setparam（XRA 参数），以及 AAudio 延迟和压力测试选项，并链接 libaudiolatencytester_engine.so。
- **D29**（已验证）：roles 文件包含一行表头和 239 行数据，每个输入行对应一行数据，没有缺失或重复的行。置信度计数为 230 已验证、2 已观察和 7 推断。

## 措辞已修正的论断

- **A04**（已验证）：product_a/media/audio/ 下音频组的全部 49 个 .ogg 片段（闹钟 9、通知 11、铃声 12、界面 17）均为 Ogg 容器中的 Vorbis 编码，采样率为 44100 或 48000 Hz，声道数为 1 或 2。该目录共有 51 个 .ogg 文件；camera_click.ogg 和 camera_focus.ogg 属于相机组，未计入。
- **A05**（已验证）：有 10 个共享对象导出 HAL 模块符号 HMI：audio.a2dp.default.so（system 下的 lib 与 lib64 中各一个），以及 audio.r_submix.default.so、audio.primary.default.so、audio.usb.default.so 和 audio.primary.anorak.so，每个都在 vendor 的 lib 和 lib64 中各一份。
- **D08**（已验证）：根据 vendor_file_contexts 第 347 行，/vendor/bin/hw/android.hardware.audio.service* 的标签为 hal_audio_default_exec，该条目的模式即为该服务的通配符。第 2 和第 3 行为 @2.0- 与 @7.0-service.example 两种路径形式标注了相同的标签。HIDL 名称 android.hardware.audio::IDevicesFactory 的标签为 hal_audio_hwservice（plat_hwservice_contexts 第 10 行）。
- **D21**（已验证）：vendor_a/etc/audio_io_policy.conf 定义了 11 个输出配置（default、proaudio、proaudio_2、voip_rx、deep_buffer、direct_pcm_16、direct_pcm_24、direct_pcm_32、compress_passthrough、compress_offload_16、compress_offload_24）和 3 个输入配置（record_16bit、record_24bit、record_32bit）。输出配置包含 flags、formats、sampling_rates、bit_width 和 app_type 键；输入配置包含 formats、sampling_rates、bit_width 和 app_type 键，且没有 flags 键。在所搜索的 lib 目录中，没有任何音频库包含字符串 audio_io_policy。
- **D28**（已验证）：音频组引用的其他子系统中的音频相关文件：bluetooth_audio_policy_configuration_7_0.xml（wifi-bluetooth；由 audio_policy_configuration.xml 包含）、default_volume_tables.xml 和 microphone_characteristics.xml（config-files；由 audio.primary.anorak.so 指定）、libar-pal.so 和 libagm.so（native-libraries；位于 audio.primary.anorak.so 的 NEEDED 列表中），以及 libeffectsconfig.so（native-libraries；由音频组的 libaudiopolicyservice.so 指定）。audioserver.rc（init-and-properties）通过服务名而非文件名与音频 HAL 相连。libsndcardparser.so 和音频内核模块位于同一子系统中，但没有音频组文件引用它们。
- **I01**（推断）：android.hardware.audio@7.0-impl.so 导入 hw_get_module_by_class 并包含字符串 primary，因此它很可能通过 hw_get_module_by_class 加载 primary HAL 模块（即导出 HMI 的 audio.primary.* 模块）。运行时的模块选择过程未被追踪。
- **I02**（推断）：在此构建中，audio_effects.xml 里的 pre_processing 库以及 agc、aec 和 ns 条目位于 XML 注释中，因此它们并未启用，且该 XML 没有定义任何启用的前处理或后处理块。框架在运行时读取哪个配置文件未被追踪，因此正在使用的效果库尚无法确定。一个已启用的 v2.0 条目指定了 noise_suppressor（libnseffect.so），而该文件不在镜像中。

## 已排除的论断

| ID | 论断 | 原因 |
|---|---|---|
| D09 | system_a/system/lib64/android.hardware.audio@7.0.so 的 SONAME 为 android.hardware.audio@7.0.so，并依赖 android.hardware.audio.common@7.0.so、android.hardware.audio.effect@7.0.so 和 android.hidl.safe_union@1.0.so。它列出了接口 IDevice、IDevicesFactory、IPrimaryDevice、IStream、IStreamIn、IStreamOut、IStreamOutCallback 和 IStreamOutEventCallback，并定义了 927 个动态符号。 |  |

## 缺口

- 未确定 vendor_a/etc/audio_io_policy.conf 的读取者：system_a/system/lib64、system_a/system/lib、vendor_a/lib64 或 vendor_a/lib 中的任何音频库都不包含字符串 audio_io_policy。
- audio_effects.xml 所列的降噪库 libnseffect.so 不在已挂载的分区中，也不在 classified.tsv 中。
- 未确定 libaudioroute.so 中通用的 mixer_paths.xml 字符串与 vendor_a/etc/mixer_paths_anorak_max98388.xml（或其他 mixer_paths_* 文件）之间的对应关系。
- 两个 vendor.audio-hal 服务定义（vendor_a/etc/init/android.hardware.audio.service_64.rc 以及包含 override 关键字的 vendor_a/etc/init/hw/init.anorak.rc）的运行时效果未被复现，仅记录了文本差异。
- 未找到 /vendor/etc/audio_policy_configuration.xml、/vendor/etc/audio_effects.xml、/system/etc/audio_effects.conf 或 /vendor/etc/audio_io_policy.conf 的明确 SELinux file_contexts 规则；未确定适用于它们的通用标签。
- Ogg 片段仅在容器和编解码器头部层面进行了检查；未确定时长、响度和播放路径。
- 框架和 HAL 库的行为（AudioFlinger 线程、策略引擎决策、AudioTrack 与 AudioRecord 流程）未作分析。libaudiohal@4.0、@5.0 和 @6.0（两个 lib 目录树）、libaudiopolicyenginedefault.so 与 libaudioprocessing.so 的角色仅依据导出名称，且 libaudiohal@4.0、@5.0 和 @6.0 被标记为推断。
- 音频二进制文件所引用、但归类于其他子系统的库未作分析：libar-pal.so、libagm.so、libmros_vendor_sysprops.so、vendor.qti.hardware PAL 与 AGM HIDL 库、libsndcardparser.so、蓝牙音频 HAL、音频内核模块、audioserver.rc，以及 audiotapper 的 rc 文件。
- audioprovider 有 SELinux 标签（system_ext_a/etc/selinux/system_ext_file_contexts 第 42 行中的 /system/bin/audioprovider），但已挂载分区中不存在该可执行文件（ls 对 system_a/system/bin 和 system_ext_a/bin 均报告没有此类文件），也没有对应的 classified 行。
- 除二进制字符串中的选项名外，未确定 audio_tool 的 XRA 参数键（用于 --getparam 和 --setparam）的语义。
- 除导出和字符串证据外，未分析 audio.a2dp.default.so 的内部实现（Rust cxxbridge 与蓝牙 shim 代码）。
- audioadsprpcd 与 DSP 之间的运行时交互（DSP_LIBRARY_PATH 的处理和重启行为）未被复现；仅读取了字符串和 init 服务定义。
- 除 android.hardware.audio.service_64 中存在的字符串外，未确定本设备实际使用的厂商 HAL 版本集合（即 IDevicesFactory 4.0 至 7.0 中哪些实际完成注册）。
- 策略音量和引擎 XML 文件（audio_policy_volumes.xml、audio_policy_engine_stream_volumes.xml、audio_policy_engine_product_strategies.xml）仅统计了元素名，未完整解析。
