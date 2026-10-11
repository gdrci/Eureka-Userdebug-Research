# 21 - 原生共享库

[English](21-native-libraries.md)

## 摘要

原生库组共有 1371 行，分布在五个分区中：system_a 787 行，vendor_a 368 行，system_ext_a 169 行，odm_a 27 行，product_a 20 行。其中包含 1361 个普通文件和 10 个符号链接。普通文件包括 1351 个 ELF 共享对象（854 个 ELF64 AArch64，497 个 ELF32 ARM，均为 ET_DYN）和 10 个 LLVM 位码文件（system/lib 和 system/lib64 下的 libclc 内置库）。符号链接包括 8 个 bionic 链接（system/lib 和 system/lib64 中的 libc、libdl、libdl_android、libm），它们指向挂载中不存在的 /apex/com.android.runtime，另有 odm_a 和 vendor_a 中指向 dlkm 分区的 lib/modules 链接。system_a 包含 AOSP 平台库和 NDK 公开库，以及 HIDL/AIDL 接口库和 Rust dylib。vendor_a 包含高通 HIDL/AIDL 客户端、HAL 模块（lib/hw、lib64/hw）、FastRPC、音频（AGM/PAL）以及相机和显示库。system_ext_a、product_a 和 odm_a 包含设备专用库（Oculus/Meta 遥测、追踪、XR 加载器、维护启动、Vulkan 层）。管辖该组的配置包括 system/etc/public.libraries.txt（29 个条目）和 vendor/etc/public.libraries.txt（8 个条目），以及 SELinux 标签，位于 plat_file_contexts 第 121 行和 vendor_file_contexts 第 142-153、196 行。角色表为 roles.tsv，共 1371 行：453 verified，857 inferred，61 unverified。在`<image>/mnt` 是镜像各分区的挂载点。

该组中的 1371 个文件各在角色表中占一行（453 verified，0 observed，857 inferred，61 unverified）。角色表位于 `data/subsystems/native-libraries/roles.tsv`。

## 已验证的声明

- **G1** (verified)：该组共 1371 行：1361 个普通文件和 10 个符号链接。各分区计数为 system_a 787、vendor_a 368、system_ext_a 169、odm_a 27、product_a 20（证据见 `data/subsystems/native-libraries/claims.json`，编号 G1）。
- **G2** (verified)：全部 1361 个普通文件的大小和 sha256 均与清单中记录的一致；挂载的副本与已分类的行逐字节相同（证据见 `data/subsystems/native-libraries/claims.json`，编号 G2）。
- **G3** (verified)：格式划分：1351 个 ELF ET_DYN 共享对象（854 个 ELF64 EM_AARCH64，497 个 ELF32 EM_ARM）和 10 个 LLVM 位码文件，没有其他文件格式（证据见 `data/subsystems/native-libraries/claims.json`，编号 G3）。
- **G4** (verified)：10 个符号链接中，8 个为 bionic 链接（system/lib 和 system/lib64 中的 libc.so、libdl.so、libdl_android.so、libm.so），指向 /apex/com.android.runtime；另外 2 个为内核模块目录链接（odm_a 和 vendor_a 的 lib/modules）（证据见 `data/subsystems/native-libraries/claims.json`，编号 G4）。
- **G6** (verified)：角色表对每个输入行恰好覆盖一次：1371 行，453 verified，857 inferred，61 unverified。标记为 unverified 的角色仅是根据名称的猜测（证据见 `data/subsystems/native-libraries/claims.json`，编号 G6）。
- **G7** (verified)：1371 行中共有 810 个不重复的基本文件名；许多名称在 lib 和 lib64 中重复出现，并分布在两个或更多分区中（证据见 `data/subsystems/native-libraries/claims.json`，编号 G7）。
- **G8** (verified)：系统 SELinux 映射将所有 system/lib 和 system/lib64 路径标记为 system_lib_file，其中引导副本标记为 system_bootstrap_lib_file（证据见 `data/subsystems/native-libraries/claims.json`，编号 G8）。
- **D2.1** (verified)：system/lib64/libbinder.so 是一个 ELF64 AArch64 DYN 对象，SONAME 为 libbinder.so，带有 build-id 注释和 28 个节，已剥离符号。它有 1392 个已定义的动态符号和 245 个未定义条目（包括空条目）（证据见 `data/subsystems/native-libraries/claims.json`，编号 D2.1）。
- **D2.2** (verified)：libbinder.so 依赖 liblog、libcutils、libutils、libc++、libc、libm 和 libdl。其导出项包括 C++ Binder API，例如 _ZN7android12ProcessState15startThreadPoolEv（ProcessState::startThreadPool）。它未列在 public.libraries.txt 中（证据见 `data/subsystems/native-libraries/claims.json`，编号 D2.2）。
- **D3.1** (verified)：system/lib64/libutils.so 是一个 AArch64 DYN 对象，SONAME 为 libutils.so，有 389 个已定义的动态符号。其导出项包括 Looper 和 RefBase，它依赖 libcutils、liblog、libprocessgroup、libvndksupport、libc++、libc、libm 和 libdl（证据见 `data/subsystems/native-libraries/claims.json`，编号 D3.1）。
- **D4.1** (verified)：system/lib64/liblog.so 是 Android 日志库。它导出 __android_log_print、__android_log_write 以及带有 @@LIBLOG 等版本标签的 android_log_* 符号，并且只依赖 libc++、libc、libm 和 libdl。它列在 NDK 公开库文件中（证据见 `data/subsystems/native-libraries/claims.json`，编号 D4.1）。
- **D5.1** (verified)：system/lib64/libcutils.so 是 Android C 工具库。它导出 property_get、ashmem_create_region、android_get_control_socket 和 socket_local_client，有 120 个已定义符号，依赖 liblog、libbase、libc++、libc、libm 和 libdl（证据见 `data/subsystems/native-libraries/claims.json`，编号 D5.1）。
- **D6.2** (verified)：该位码由 libRSCpuRef.so 使用，该库包含字面路径 /system/lib64/libclcore.bc、libclcore_debug.bc 和 libclcore_debug_g.bc（证据见 `data/subsystems/native-libraries/claims.json`，编号 D6.2）。
- **D7.1** (verified)：system/lib64/hw/android.hidl.memory@1.0-impl.so 是一个直通（passthrough）HIDL 实现，SONAME 为 android.hidl.memory@1.0-impl.so。它在 android::hidl::memory::V1_0::implementation 中导出 AshmemMapper 和 AshmemMemory 类，并依赖 android.hidl.memory@1.0.so 和 libhidlbase.so（证据见 `data/subsystems/native-libraries/claims.json`，编号 D7.1）。
- **D7.2** (verified)：系统 VINTF 清单将 android.hidl.memory 1.0 声明为 32 位和 64 位架构上的直通 HAL，因此该实现会被加载到客户端进程中（证据见 `data/subsystems/native-libraries/claims.json`，编号 D7.2）。
- **D8.1** (verified)：vendor/lib64/hw/bootctrl.anorak.so 是一个 AArch64 HAL 模块，SONAME 为 bootctrl.anorak.so。它导出 HMI 模块信息对象（568 字节）、boot_control_init、set_active_boot_slot、slot_suffix_arr、gpt_disk_alloc 和 gpt_utils_is_ufs_device，并有 28 个已定义的动态符号（证据见 `data/subsystems/native-libraries/claims.json`，编号 D8.1）。
- **D8.2** (verified)：bootctrl.anorak.so 依赖 libcutils、liblog、libz、libc++、libc、libm 和 libdl。其字符串表包含引导槽位 API 名称 get_current_slot、get_number_slots、is_slot_bootable、is_slot_marked_successful、set_active_boot_slot 和 set_slot_as_unbootable（证据见 `data/subsystems/native-libraries/claims.json`，编号 D8.2）。
- **D8.3** (inferred)：使用该模块的引导 HAL 服务在 init 中声明为 vendor.boot-hal-1-2，运行 /vendor/bin/hw/android.hardware.boot@1.2-service，在 early_hal 类中提供 IBootControl 1.0、1.1 和 1.2 接口。服务会加载 bootctrl.anorak.so，这一点是根据 HMI 导出项和共享的槽位 API 名称推断的，而不是来自服务二进制文件中的导入（证据见 `data/subsystems/native-libraries/claims.json`，编号 D8.3）。
- **D9.1** (verified)：vendor/lib64/vendor.qti.hardware.display.mapper@4.0.so 是 IQtiMapper v4.0 的直通 HIDL 接口库。它依赖 graphics common 1.0/1.1/1.2、graphics mapper 4.0、mapperextensions 1.0 接口和 libhidlbase，并导出 101 个已定义符号，包括 IQtiMapper::getService 和 BnHwQtiMapper（证据见 `data/subsystems/native-libraries/claims.json`，编号 D9.1）。
- **D9.2** (verified)：vendor VINTF 清单在名为 android.hardware.graphics.mapper-impl-qti-display.xml 的片段中，将 vendor.qti.hardware.display.mapper 4.0 声明为直通，接口为 IQtiMapper（证据见 `data/subsystems/native-libraries/claims.json`，编号 D9.2）。
- **D10.1** (verified)：system_ext/lib64/libopenxr_loader.so 是 OpenXR 加载器：SONAME 为 libopenxr_loader.so，有 55 个已定义符号和 162 个未定义符号，导出 xr* 入口点，例如 xrCreateInstance、xrDestroyInstance 和 xrAcquireSwapchainImage。它依赖 libandroid、libEGL、liblog、libnativewindow 和 libvulkan（证据见 `data/subsystems/native-libraries/claims.json`，编号 D10.1）。
- **D11.1** (verified)：system_ext/lib64/libVkLayer_khronos_validation.so 是一个 Vulkan 层。它是一个没有 SONAME、大小为 21482200 字节的 AArch64 DYN 对象。它导出 7 个 vk* 加载器接口符号，包括 vkNegotiateLoaderLayerInterfaceVersion，并依赖 libdl、liblog、libandroid、libm 和 libc（证据见 `data/subsystems/native-libraries/claims.json`，编号 D11.1）。
- **D12.2** (verified)：libadsprpc.so 列在 vendor/etc/public.libraries.txt 中，并在 vendor_file_contexts 中标记为 same_process_hal_file。其字符串包括 fastrpc_mmap、fastrpc_munmap 和 fastrpc_async_get_status（证据见 `data/subsystems/native-libraries/claims.json`，编号 D12.2）。
- **D13.1** (verified)：odm/lib64/libmaintenanceboot.so 是一个维护启动库，SONAME 为 libmaintenanceboot.so，有 122 个已定义符号。它导出 OVR::OS MiscWriter、ReadMiscVirtualAbMessage、update_bootloader_message_in_struct、clear_bootloader_message 以及 android::fs_mgr fstab 读取器。它依赖 android.hardware.health@2.1.so、libosutils、libutils、libbase、libcutils 和 liblog（证据见 `data/subsystems/native-libraries/claims.json`，编号 D13.1）。
- **D13.2** (verified)：维护启动 HAL 服务在 odm init 中声明为 vendor.oculus.hardware.maintenanceboot.maintenanceboot-service，接口为 aidl vendor.oculus.hardware.maintenanceboot.IMaintenanceBoot/default，类为 hal。没有任何 init 或 SELinux 文件直接提到 libmaintenanceboot.so；服务二进制文件标记为 hal_maintenanceboot_default_exec（证据见 `data/subsystems/native-libraries/claims.json`，编号 D13.2）。
- **D15.1** (verified)：vendor/etc/public.libraries.txt 列出 8 个 vendor 条目：libadsprpc.so、libcdsprpc.so、libeva.so、libhexagon.so、libsdsprpc.so、libsynx.so（这六个都存在于本组中），以及 libdrm.so 和 libOpenCL.so（存在于 display-and-composer 和 gpu 子系统中）（证据见 `data/subsystems/native-libraries/claims.json`，编号 D15.1）。

## 修正措辞的声明

下方声明以数据所支持的措辞重新表述。

- **G5** (verified)：1300 个 ELF 对象带有 DT_SONAME。没有 DT_SONAME 的 51 个对象是 50 个 .dylib.so 对象和 libVkLayer_khronos_validation.so。这 50 个 .dylib.so 对象遵循 Rust crate-dylib 命名方式；其中 40 个在 NEEDED 中列出了 libstd.dylib.so，而没有该条目的 9 个小型 crate（例如 libbitflags.dylib.so）的 Rust 来源是根据其名称推断的（证据见 `data/subsystems/native-libraries/claims.json`，编号 G5）。
- **G9** (verified)：vendor SELinux 映射将本组中的 vendor libadsprpc.so、vendor.qti.hardware.display.mapper@2.0、@3.0 和 @4.0 库，以及 vendor.qti.hardware.display.mapperextensions@1.0、@1.1 和 @1.2 库标记为 same_process_hal_file（vendor_file_contexts 第 161-166 行和第 196 行）。vendor_file_contexts 第 17 行将引导 HAL 服务可执行文件标记为 hal_bootctl_default_exec，第 108 行将 odm 维护启动服务可执行文件标记为 hal_maintenanceboot_default_exec（证据见 `data/subsystems/native-libraries/claims.json`，编号 G9）。
- **D1.1** (verified)：system/lib64/libc.so 是一个 46 字节的符号链接，指向 /apex/com.android.runtime/lib64/bionic/libc.so。该路径未在挂载中实体化，system_a/apex 为空。APEX 包本身位于挂载中的 system/apex/com.android.runtime.apex，其 apex_payload.img 包含 /lib64/bionic/libc.so（1322360 字节）（证据见 `data/subsystems/native-libraries/claims.json`，编号 D1.1）。
- **D6.1** (verified)：system/lib64/libclcore.bc（及 _debug、_debug_g、_g 和 _neon 变体）是以魔数字节 42 43 c0 de 开头的 LLVM IR 位码文件（verified）。lib64 中它们的大小分别为 330432、353076、1077160、1124700 和 333936 字节（verified）。其用途（用于 OpenCL 和 RenderScript 的 libclc 内置库）是根据名称推断的（inferred）（证据见 `data/subsystems/native-libraries/claims.json`，编号 D6.1）。
- **D10.2** (verified)：system/etc/openxr/1 下的系统 OpenXR 运行时清单（active_runtime.aarch64.json、active_runtime.armv7a-vfp.json、active_runtime.x86_64.json）均在 VrDriver.apk 中指定了 libopenxr_forwardloader.so；没有一个指定 libopenxr_loader.so。挂载中的文本文件（init、SELinux、XML、JSON 或其他）均未引用 libopenxr_loader.so。二进制文件则引用了它：system/lib64/libxrapp.so 和 system/lib64/libvrui_jni.so 在 NEEDED 中列出了 libopenxr_loader.so（证据见 `data/subsystems/native-libraries/claims.json`，编号 D10.2）。
- **D11.2** (verified)：挂载中没有任何层清单或配置文本文件提到 libVkLayer_khronos_validation.so。该名称确实作为字符串出现在二进制文件 system/lib64/libvulkan.so 和 system/lib/libvulkan.so 中，这两个文件还包含 debug.vulkan.layers 和 debug.vulkan.layer.<priority> 字符串；挂载中没有文本文件设置这些属性，因此无法从挂载中确定注册路径（证据见 `data/subsystems/native-libraries/claims.json`，编号 D11.2）。
- **D12.1** (verified)：vendor/lib64/libadsprpc.so 是 FastRPC 用户空间库（SONAME 为 libadsprpc.so，有 44 个已定义符号，导出 remote_handle_control、remote_session_control、remote_mem_map、remote_munmap64 和 rpcmem_to_fd）。它与音频 DSP 的关联是根据 adsp 名称前缀推断的（inferred）（证据见 `data/subsystems/native-libraries/claims.json`，编号 D12.1）。
- **D14.1** (verified)：system/etc/public.libraries.txt 在注释头之后列出 29 个条目。条目 libneuralnetworks.so 末尾带有 nopreload 标记。列出的五个名称（libicu.so、libicui18n.so、libicuuc.so、libnativehelper.so 和 libneuralnetworks.so）在这些确切名称下没有文件或已分类的行，并且不在挂载中。子字符串 libneuralnetworks 只匹配 libneuralnetworks_packageinfo.so（native-libraries，system_a）（证据见 `data/subsystems/native-libraries/claims.json`，编号 D14.1）。

## 已排除的声明

下方声明已排除；每条都给出了原因。

| ID | 声明 | 原因 |
|---|---|---|
| D14.2 | 在 29 个公开条目中，有 23 个以文件或链接的形式出现在本组行中。libaaudio.so、libEGL.so、libGLESv2.so 和 libvulkan.so 等条目存在 | 未能复现。D14.1 中五个缺失的名称使 29 个条目中有 24 个存在，而非 23 个；未确定造成差异的文件。 |

<!-- generated: libraries start -->
## 库索引（生成）

原生库分组共有 1,361 个文件。来源：`data/elf/libraries.tsv`（头部与动态符号）。每个文件的 SHA-256 都与清单相符（1,361 个中有 1,361 个）。

- 1,351 个文件为 ELF 共享对象（ET_DYN）：854 个为 ELFCLASS64、机器类型 EM_AARCH64，497 个为 ELFCLASS32、机器类型 EM_ARM。
- 10 个文件为 LLVM 位码，不是 ELF：`system_a/system/lib/libclcore.bc`、`system_a/system/lib/libclcore_debug.bc`、`system_a/system/lib/libclcore_debug_g.bc`、`system_a/system/lib/libclcore_g.bc`、`system_a/system/lib/libclcore_neon.bc`、`system_a/system/lib64/libclcore.bc`、`system_a/system/lib64/libclcore_debug.bc`、`system_a/system/lib64/libclcore_debug_g.bc`、`system_a/system/lib64/libclcore_g.bc`、`system_a/system/lib64/libclcore_neon.bc`。
- 12 个文件保留了 .symtab 节；其余 1,339 个只保留动态符号表（已剥离）。
- 1,343 个 ELF 文件（共 1,351 个）带有 GNU build-id 注释，8 个没有。51 个文件没有 SONAME。
- 12 个文件的第一个 .comment 字符串为 `Android (7485623, based on r416183b1) clang version 12.0.7 (https://android.googlesource.com/toolchain/llvm-project c935d99d7cf2016289302412d708641d52d2f7ee)`。

### 按分区

| 分区 | 文件数 | ELF 文件数 |
|---|---|---|
| system_a | 779 | 769 |
| vendor_a | 367 | 367 |
| system_ext_a | 169 | 169 |
| odm_a | 26 | 26 |
| product_a | 20 | 20 |

### NEEDED 依赖图

1,351 个 ELF 文件共列出 11,630 条 NEEDED 边，涉及 613 个不同的库。其中 10,845 条边指向本组内某个文件的 SONAME（共 453 个不同名称）。其余 785 条边指向 160 个不在本组内的不同库。最常见的 NEEDED 条目：

| NEEDED | 库数 |
|---|---|
| `libc.so` | 1,343 |
| `libdl.so` | 1,284 |
| `libm.so` | 1,267 |
| `libc++.so` | 1,224 |
| `liblog.so` | 921 |
| `libutils.so` | 677 |
| `libcutils.so` | 574 |
| `libhidlbase.so` | 348 |
| `libbase.so` | 288 |
| `libbinder.so` | 246 |
| `libbinder_ndk.so` | 123 |
| `libosutils.so` | 115 |
| `libui.so` | 62 |
| `libz.so` | 58 |
| `libjsoncpp.so` | 52 |

本组中没有 SONAME 的 NEEDED 名称（最常见的）：

| NEEDED | 边数 | 清单中其他分组的同名文件 |
|---|---|---|
| `libcrypto.so` | 45 | security-keystore-crypto |
| `libstd.dylib.so` | 40 | native-libraries |
| `libEGL.so` | 31 | gpu |
| `libGLESv2.so` | 25 | gpu |
| `android.hardware.graphics.mapper@4.0.so` | 25 | display-and-composer |
| `libdisplaydebug.so` | 23 | display-and-composer |
| `libaudioclient.so` | 21 | audio |
| `libnativehelper.so` | 18 | 无 |
| `libstagefright_foundation.so` | 16 | media-codecs |
| `libaudioutils.so` | 15 | audio |
| `libvrsensors-hidlwrapper.so` | 15 | sensors-mcu-meta-hal |
| `libstatssocket.so` | 14 | 无 |

在没有 SONAME 匹配本组的 160 个名称中，有 152 个在清单中有同名文件。匹配仅依据文件名，因此并不确认 SONAME。列中为 `native-libraries` 表示该文件在本组中，但 SONAME 不同。其余名称在镜像中没有同名文件。

### 动态符号

在 ELF 文件中，动态符号表共有 572,803 个已定义符号和 137,597 个未定义符号。这些计数不含索引 0 的空项。未定义符号最多的库：

| 库 | 机器类型 | 未定义 | 已定义 |
|---|---|---|---|
| `system_a/system/lib/libandroid_runtime.so` | EM_ARM | 2400 | 1878 |
| `system_a/system/lib64/libandroid_runtime.so` | EM_AARCH64 | 2398 | 1856 |
| `system_a/system/lib/libandroid_servers.so` | EM_ARM | 815 | 908 |
| `system_a/system/lib64/libandroid_servers.so` | EM_AARCH64 | 815 | 908 |
| `system_a/system/lib/libhwui.so` | EM_ARM | 804 | 121 |
| `system_a/system/lib64/libhwui.so` | EM_AARCH64 | 794 | 121 |
| `system_a/system/lib/libmediaplayerservice.so` | EM_ARM | 784 | 1638 |
| `system_a/system/lib/libmedia_jni.so` | EM_ARM | 679 | 542 |
| `system_a/system/lib64/libmedia_jni.so` | EM_AARCH64 | 677 | 542 |
| `vendor_a/lib/libfastcvopt.so` | EM_ARM | 584 | 5950 |
| `vendor_a/lib64/libfastcvopt.so` | EM_AARCH64 | 576 | 6000 |
| `vendor_a/lib64/libwvhidl.so` | EM_AARCH64 | 535 | 3421 |

库之间的符号级匹配未在此计算。

<!-- generated: libraries end -->

## 缺口

- /apex/com.android.runtime 下的 bionic 目标（libc、libdl、libm、libdl_android）未在挂载中实体化：system_a/apex 为空，因此无法跟随这 8 个 bionic 符号链接读取其内容。APEX 包 system/apex/com.android.runtime.apex 在挂载中，其 payload 包含这些库（见结论 D1.1）。
- libicu.so、libicui18n.so、libicuuc.so、libnativehelper.so 和 libneuralnetworks.so 在 system/etc/public.libraries.txt 中被列出，但未出现在任何子系统清单中；未确定 APEX 模块是否提供它们。
- 857 行的角色是根据文件名推断的，453 个已验证的角色来自导出符号或 NEEDED 条目；未检查每个文件的行为。61 个未验证的行仅是基于名称的猜测（例如 com.qti.stats.common.so、libgipp.so、libkll.so、libfruit.so），并在 roles.tsv 中相应标记。
- 全部 1,351 个 ELF 对象的 NEEDED 边已在上方生成的小节中列出。边先与本组内文件的 SONAME 匹配。组外的 NEEDED 名称仅按文件名与清单中的文件匹配，未读取这些文件的 SONAME；镜像中没有文件的名称可能来自未在挂载中实体化的 APEX 模块。
- 链接器配置（system/linkerconfig、system/etc/linker.config.pb）为二进制文件，未经解码。对 libadsprpc.so、libopenxr_loader.so 和 libmaintenanceboot.so 进行的字面 grep 在 linker.config.pb 中未找到匹配项，但搜索规则和命名空间搜索路径均未确定。
- 未提取 bootctrl.anorak.so 的 HMI 模块 ID 字符串，也未发现 android.hardware.boot@1.0-impl-1.2-qti.so 与 bootctrl.anorak.so 之间的导入链接。使用关系仅为推断。
- 挂载中未找到 libVkLayer_khronos_validation.so 的 Vulkan 层清单，因此其注册和启用路径未知。
- 未确定 OpenXR 加载器的使用方应用。运行时清单指向 VrDriver.apk 中的另一个库。
- 除 Rust NEEDED 检查和位码魔数字节外，50 个 Rust .dylib.so crate 对象和 10 个位码文件仅根据名称和格式进行了分类。
- 15 个深入分析项是按名称和角色从最重要的文件中挑选的。其余二进制文件除角色表外未作深入分析。
- pyelftools（facts.jsonl 中）给出的未定义符号计数比 readelf 少一，原因是 readelf 会计入空条目。声明引用的是 readelf 的计数。
