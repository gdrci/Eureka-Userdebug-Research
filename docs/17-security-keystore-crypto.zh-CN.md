# 17 - 密钥库、keymaster 与加密

[English](17-security-keystore-crypto.md)

## 摘要

该组包含 60 行已分类记录：56 个普通文件和 4 个符号链接，分布在 system_a（40）、system_ext_a（1）和 vendor_a（19）中。它涵盖四个层面。（1）系统 keystore2 技术栈：Rust 实现的 keystore2 守护进程、其 AIDL/NDK 与 C++ 桩代码、Rust 加密与认证（attestation）辅助库、keystore 客户端 CLI，以及 wait_for_keymaster 启动辅助程序。（2）Gatekeeper：gatekeeperd 与 gatekeeperserver 守护进程、gatekeeper 核心库、OVR gatekeeper 服务与客户端库，以及高通 gatekeeper 1.0 HAL 服务及其实现。（3）Keymaster 与 KeyMint：系统分区和 vendor 分区中的 HIDL 3.0、4.0 和 4.1 接口库，软件 keymaster 库，高通 keymaster 4.1 HAL 服务及其后端，以及供 vendor 服务使用的 qseecomd TEE 守护进程。（4）fs-verity 与 AVB verity：fsverity_init（由 init.rc 启动）、set-verity-state 及其 enable-verity 和 disable-verity 链接、libfsverity，以及 fs-verity 发布用 X.509 证书。跨模块共用的库包括 BoringSSL libcrypto（32 位和 64 位）以及 Widevine OEMCrypto 库。全部 56 个普通文件的 sha256 和大小均与其记录一致。二进制文件已去除符号信息，因此其行为依据导出及导入的符号、NEEDED 条目、字符串，以及 init 和 SELinux 文本文件推断得出。

角色表：`data/subsystems/security-keystore-crypto/roles.tsv`（60 个文件：40 个 verified，11 个 observed，9 个 inferred，0 个 unverified）。

## 已验证的论断

- **cov-1** (verified)：本组有 60 条输入行（56 个普通文件，4 个符号链接）。roles.tsv 每条输入行对应一行，分区与路径集合相同，且无重复。
- **cov-2** (verified)：全部 56 个普通输入文件的 sha256 和大小均与其记录一致。
- **cov-3** (verified)：四行为符号链接：system_a/system/bin/disable-verity 和 enable-verity 指向 set-verity-state；system_a/system/bin/tee 指向 toybox；vendor_a/bin/tee 指向 toybox_vendor。
- **ks2-format** (verified)：keystore2 是为 Android 构建的、已去除符号的动态链接 AArch64 PIE（ELF64，解释器为 /system/bin/linker64），大小 1546696 字节，BuildID 18995cead41f204d5b127934d0ec6651。它由 Rust 源码构建，且没有 .comment 节。
- **ks2-deps** (verified)：keystore2 有 14 个 NEEDED 条目。其中 libkm_compat_service.so 和 libsqlite.so 不在本组内。
- **ks2-config** (observed)：keystore2 使用系统属性 keystore.boot_level 和 keystore.crash_count，以及数据目录 /data/misc/keystore。其 init 服务将该目录作为第一个参数传入，init.rc 在启动期间设置 boot-level 属性。
- **ks2-selinux** (verified)：keystore2 的标签为 keystore_exec。其数据目录的标签为 keystore_data_file，其 AIDL 服务名映射到 keystore_service，keystore2 的密钥上下文位于 plat_keystore2_key_contexts 中。
- **ks2-source** (observed)：keystore2 由 system/security/keystore2/src 下的 29 个 Rust 源文件构建，包括 keystore2_main.rs、service.rs、database.rs、super_key.rs、legacy_blob.rs 和 remote_provisioning.rs，并带有 Rust 1.55.0 标准库路径。
- **libcrypto64-format** (verified)：lib64/libcrypto.so 是由 external/boringssl 构建的 BoringSSL。它导出约 2386 个已定义的动态符号，且仅依赖 libc、libm 和 libdl。32 位的 system/lib/libcrypto.so 具有相同的 BoringSSL 标记，并导出 2388 个符号。
- **set-verity-format-cfg** (observed)：set-verity-state 是已去除符号的 AArch64 PIE，用于启用或禁用 verity。它既可直接调用（用法为 [1|0]），也可通过 enable-verity 和 disable-verity 名称调用，并读写 avb.managed_verity_mode 属性。它依赖 libfec、libfs_mgr_binder 和 libcrypto_utils。
- **fsverity-cert** (verified)：fsverity-release.x509.der 是 DER 编码的 X.509 v3 自签名 CA 证书，主体与颁发者均为 CN=Android (O=Google Inc., OU=Android)，RSA 4096 位密钥，公钥指数 65537，有效期为 2020-03-20 至 2050-03-20，SHA-256 指纹为 9F:97:31:5D:A5:EC:FF:D9:5B:78:C1:EE:B2:1D:3B:3C:27:1B:C3:0D:AB:61:C0:FA:74:2C:89:E6:20:BE:AC:7D。
- **libfsverity** (verified)：libfsverity.so 是 fs-verity 用户态库。它导出 libfsverity_enable、libfsverity_enable_with_sig、libfsverity_sign_digest、libfsverity_compute_digest 以及哈希辅助函数，并依赖 libcrypto.so。在已挂载分区中，它唯一的 NEEDED 使用者是 /system/bin/odsign，该程序不在本组内。
- **gatekeeperd** (observed)：gatekeeperd 是已去除符号的 AArch64 PIE 守护进程，通过 binder 暴露 android.service.gatekeeper.IGateKeeperService。它依赖 gatekeeper HIDL 1.0 接口和 libgatekeeper，由 init 在 late_start 阶段启动。
- **libgatekeeper64** (verified)：lib64/libgatekeeper.so 是 gatekeeper 核心库。它导出 GateKeeper::Enroll、GateKeeper::Verify、GateKeeper::MintAuthToken、CreatePasswordHandle、ThrottleRequest 和 IncrementFailureRecord，以及请求与响应的序列化类。32 位的 system/lib/libgatekeeper.so 导出 46 个 gatekeeper:: 符号。
- **keystore-cli** (observed)：keystore_cli_v2 是已去除符号的 AArch64 keystore2 客户端，提供以下命令：confirmation、decrypt、delete、encrypt、generate、get-chars、list、list-brillo-tests、sign-verify 和 brillo-platform-test。它依赖 libchrome、libprotobuf-cpp-lite 和 libkeymint_support，其源码为 system/security/keystore/keystore_cli_v2.cpp。
- **wait-for-keymaster** (observed)：wait_for_keymaster 是启动期辅助程序，由 init 以 root 身份、优先级 -20 运行。它等待 keystore2 的 IKeystoreService/default 实例和 keymaster 维护服务就绪，其源码为 system/vold/wait_for_keymaster.cpp。其标签为 wait_for_keymaster_exec。
- **libkeymaster-portable** (verified)：libkeymaster_portable.so 是可移植的软件 keymaster 实现。它导出 AndroidKeymaster 类、用于 RSA、对称密钥和升级消息的操作类，以及 ae_* 认证加密辅助函数。它依赖 libcrypto、libcppbor_external 和 libcppcose_rkp。
- **vendor-keymaster41-hidl** (verified)：vendor 的 lib64/android.hardware.keymaster@4.1.so 是 HIDL IKeymasterDevice 4.1 接口库。它导出 V4_1::IKeymasterDevice 及其 Bp、Bn 代理与桩类，并在 vendor VINTF 清单中以 @4.1::IKeymasterDevice/default 形式发布。
- **vendor-keymaster41-service** (observed)：bin/hw/android.hardware.keymaster@4.1-service-qti 是高通 keymaster 4.1 HAL 服务。它由 init 以 vendor.keymaster-4-1 的名称在 early_hal 类中启动，链接 libqtikeymaster4 以及 4.0 和 4.1 HIDL 库，其源码为 vendor securemsm keymaster/4.1/service.cpp。
- **qseecomd** (observed)：qseecomd 是 vendor TEE 路径使用的 QSEECom 守护进程。它在 core 类中以 root 身份运行，为 drmrpc 组持有一个 notify-topology 套接字，打开 /dev/smcinvoke，并链接 libQSEEComAPI、libdrmfs、libminkdescriptor 和 libdmabufheap。其标签为 tee_exec，/dev/qseecom 节点的标签为 tee_device。
- **libqtikeymaster4** (observed)：lib64/libqtikeymaster4.so 是高通 keymaster 4.x 后端，vendor keymaster 4.1 服务链接了它。它依赖 vendor keymaster 设备库与工具库、libqcbor，以及 4.0 和 4.1 HIDL 接口。它导出 239 个已定义符号，其中包括一组 CR* 系列的 getter 函数。

## 措辞已修正的论断

- **ks2-interfaces** (observed)：keystore2 的二进制字符串中包含 android.system.keystore2 服务名（IKeystoreService/default、IKeystoreOperation），以及维护、指标、授权、compat、APC、legacy keystore 和远程配置（remote provisioning）接口名称，另有 KeyMint、安全时钟和共享密钥 HAL 接口名称。它导入了 AServiceManager_addService，并在 IKeystoreService/default、APC、授权、指标、远程配置、维护和 legacy keystore 名称旁记录日志 'Successfully registered Keystore 2.0 service.'。无法仅凭字符串确定每个列出的接口是由该二进制提供服务，还是仅被引用。
- **set-verity-refs** (verified)：已挂载分区中有四个二进制文件包含 verity 状态命令名：system_a/system/bin/init、snapshotctl、update_engine 以及 set-verity-state 本身（其中除 set-verity-state 外为三个）。verity 状态相关行（set-verity-state、enable-verity、disable-verity）以及 tee 符号链接行，在已检查分区的任何 init.rc 或 file_contexts 条目中均未被提及。
- **fsverity-init** (observed)：fsverity_init 是已去除符号的 AArch64 PIE，它从 /system/etc/security/fsverity 和 /product/etc/security/fsverity 加载已验证的 fs-verity 密钥（.der 文件），按名称加载额外密钥，并锁定 .fs-verity 密钥环。init.rc 在 'on late-fs' 块（第 647 行）中以 --load-verified-keys 参数运行它，并在 'on post-fs-data' 块（第 981 行）中以 --lock 参数运行它。
- **gatekeeperserver** (verified)：gatekeeperserver 是一个小型 OVR gatekeeper 服务宿主。它导入了 GatekeeperNativeService::createProdInstance 和 ServiceHost，由 init 在 main 类中启动。它链接 libgatekeepernativeservice.so，该库的字符串中包含 /data/oculus/gatekeepers/gks.db。该库导出覆盖与清除方法（overrideGatekeeper、clearOverrideGatekeeper、clearOverriddenGatekeepers、clearGatekeeper）以及一个 dump 入口点。

## 缺口

- 已去除符号的二进制文件（keystore2、gatekeeperd、gatekeeperserver、fsverity_init、set-verity-state、wait_for_keymaster、keystore_cli_v2、qseecomd，以及两个 HAL 服务进程）：函数级行为、内部状态布局和磁盘数据库格式均未确定。描述依赖导出与导入的符号、NEEDED 条目、字符串以及 init/SELinux 文本。
- 未分析的本组外依赖：libkm_compat_service.so 和 libsqlite.so（keystore2 的 NEEDED 条目），libkeyutils.so、libselinux.so、libfs_mgr_binder.so、libfec.so、libQSEEComAPI.so、libqcbor.so、libcppbor_external.so、libcppcose_rkp.so、libkeymint_support.so、libchrome.so、libprotobuf-cpp-lite.so、libgsi.so、libkvstore.so、libosutils.so。
- KeyMint 服务二进制文件（vendor_a/bin/hw/android.hardware.security.keymint-service-qti，及其 init rc 和 vintf 清单，它们位于 vendor_a 中）被本组的 VINTF 和 SELinux 文件引用，但不在本组内，因此未作分析。
- 根据 NEEDED 条目，libkm_compat_service.so 的使用者是 keystore2，但该库的实现不在本组内，未予检查。
- odsign（system_a/system/bin/odsign，NEEDED libfsverity.so）不在本组内。其对 libfsverity 的使用仅依据其 NEEDED 条目进行了核查。
- 未找到 set-verity-state、enable-verity、disable-verity、tee、keystore_cli_v2 或本组共享库的显式 file_contexts 条目。适用于它们的默认标签规则未确定。
- fsverity_init 的运行时密钥加载路径（其在启动时实际加载哪些密钥文件）以及 fs-verity 签名与验证流程均未复现。发布证书的作用是根据文件名和搜索路径字符串推断的。
- 32 位和 64 位 gatekeeper 核心库、32 位 vendor gatekeeper 实现（vendor_a/lib/hw/android.hardware.gatekeeper@1.0-impl-qti.so）、32 位 vendor keymasterdeviceutils 库、libkeymaster4support 与 libkeymaster_messages 的 vendor 副本，以及 vendor 的 3.0 和 4.0 HIDL 库，均是依据名称、NEEDED 列表和大小识别的，未经过符号级检查。libkeymaster4support 和 libkeymaster_messages 的 vendor 副本与其 system 副本大小相同，但 sha256 值不同，原因未确定。
- libqtikeymaster4 中 CR* getter 系列的含义，以及 gatekeeper 实现中 vendor.gatekeeper.disable_spu 键的含义均未确定。liboemcrypto 中的 vendor.wv.oemcrypto.debug.* 键仅作了列举，未作解释。
- vendor 的 boringssl_self_test init rc（vendor_a/etc/init/boringssl_self_test.rc）及其与 libcrypto 的关系未作检查。
- SELinux 源文件仅有编译后的 CIL（没有 .te 文件）。keystore2 和 gatekeeperd 的域转换未被完整追踪；仅确认了 exec 标签和类型声明。
- tee 符号链接背后的 toybox 小程序未作检查。
