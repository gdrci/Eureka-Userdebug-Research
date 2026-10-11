# 19 - 框架与系统服务

[English](19-framework-and-system-services.md)

## 摘要

该组共有 707 行：229 个符号链接和 478 个普通文件，分布在 system_a（681）、system_ext_a（15）、product_a（9）和 odm_a（2）中。它包含框架层：字节码与 ART 产物（services.jar、framework.jar、framework-res.apk、services.odex/.vdex/.art 一组文件、arm 和 arm64 的 boot-framework 镜像）；system/bin 中的原生守护进程和命令客户端（servicemanager、hwservicemanager、用于 zygote 启动的 app_process32/64、toybox 和 toolbox 的 applet 链接、cmd、service，以及约 180 个其他文件）；22 个 APEX 模块包；约 33 个用于 exec app_process 或 cmd 的 shell 包装脚本；以及 XML 配置（platform.xml、privapp-permissions、default-permissions、framework-sysconfig、hidden-API 白名单、功能与库声明）。启动接线可在 init rc 文件（zygote 变体、servicemanager、hwservicemanager）以及二进制文件在 plat_file_contexts 中的条目中看到。挂载的 system_a 树中，init 文件和 SELinux 文件均未提及任何 XML 配置文件。所有角色与事实均取自挂载副本；未执行任何内容。

角色表为该组中的 707 个文件各包含一行（324 verified、281 observed、101 inferred、1 unverified）。该表位于 `data/subsystems/framework-and-system-services/roles.tsv`。

## 已验证的论断

- **C01** (verified): roles.tsv 每个输入行对应一个数据行（共 707 行），表头为 partition、path、role、confidence；没有缺失或重复的 (partition, path) 组合。
- **C02** (verified): 该组的分区划分：system_a 681 行，system_ext_a 15 行，product_a 9 行，odm_a 2 行。
- **C03** (verified): 类型：478 个普通文件和 229 个符号链接。普通文件的 MIME 类型：183 个 application/x-pie-executable、85 个 application/octet-stream、74 个 application/x-sharedlib、54 个 application/vnd.android.package-archive、36 个 text/xml、36 个 text/plain、6 个 application/java-archive、4 个 application/x-executable（合计 478）。符号链接行的 MIME 值为 link，大小为 0。
- **C04** (verified): 从挂载副本重新计算的 sha256 与 classified.tsv 的 sha256 列，对全部 478 个普通文件均一致。
- **C08** (verified): com.android.runtime.apex 是一个包含 10 个成员的 zip 容器：apex_build_info.pb、apex_manifest.pb（清单名称 com.android.runtime）、apex_payload.img（8,880,128 字节）、apex_pubkey、assets/NOTICE.html.gz、resources.arsc、AndroidManifest.xml 以及 META-INF 签名文件。负载在字节偏移 1080 处带有 ext4 签名 0x53ef。/system/bin/linker64 是指向 /apex/com.android.runtime/bin/linker64 的符号链接。
- **C09** (verified): services.jar（归档大小 18,820,024 字节）包含 28 个成员：classes.dex（未压缩 11,441,472 字节）、classes2.dex（7,359,616 字节）、META-INF/MANIFEST.MF、20 个 *_compat_config.xml 文件和 5 个 .uau 文件。成员总大小为 18,842,858 字节。清单声明 Created-By: soong_zip。未确定 .uau 成员的用途。
- **C10** (verified): services.jar 内的 classes.dex 是 dex 039 文件，其字符串表包含 system_server 入口类 Lcom/android/server/SystemServer;。
- **C12** (verified): framework-res.apk（文件大小 28,322,116 字节）是一个 zip，包含 3,259 个成员，其中 3,249 个位于 res/ 下。它包含 AndroidManifest.xml（未压缩 275,872 字节）和 resources.arsc（15,986,932 字节）。1,675 个成员为 stored（存储），1,584 个为 deflated（deflate 压缩）。它没有 classes.dex。
- **C13** (verified): services.odex 是 ELF64、ET_DYN、AArch64 共享对象。其节包括位于 0x1000 的 .rodata、位于 0x67e000（大小 0x1b636e8）的 .text、位于 0x21e2000 的 .data.bimg.rel.ro、位于 0x227f0a0 的 .dynsym，以及位于 0x2280000（大小 0x70）的 .dynamic。文件大小为 36,885,288 字节。配套文件为 services.vdex（248,244 字节）和 services.art（2,019,328 字节）。
- **C14** (observed): services.odex 镜像记录的类路径以 ART 核心 jar 开头，随后是 /system/framework/framework.jar、framework-graphics.jar、ext.jar、telephony-common.jar、voip-common.jar、ims-common.jar、/apex/com.android.i18n/javalib/core-icu4j.jar，以及 system_ext 的 jar：/system_ext/framework/com.oculus.os.platform.jar、vros-framework.jar 和 ocui.jar。Dex 位置记录为 /system/framework/services.jar!classes2.dex。
- **C15** (verified): services.vdex 的魔数为 vdex，后接 027 和 NUL；services.art 的魔数为 art、换行、099、NUL。两者都位于 oat/arm64 中。
- **C16** (verified): arm64 的 boot-framework 镜像集为：boot-framework.oat（ELF64 AArch64 ET_DYN，1,050,096 字节，DT_NEEDED 条目数为零）、boot-framework.art（art 099 镜像，7,344,128 字节）和 boot-framework.vdex（vdex 027，324,332 字节）。arm 变体的 boot-framework.art 为 5,550,080 字节，boot-framework.oat 为 870,420 字节。
- **C17** (observed): boot-framework.oat 的字符串中列出了 bootclasspath 及其校验和表，并列出 ART 核心 jar，随后是 /system/framework/framework.jar 及其 classes2 至 classes4 的 dex 条目。
- **C19** (verified): servicemanager 是 PIE AArch64 可执行文件，解释器为 /system/bin/linker64，包含 11 个 DT_NEEDED 条目：libbase.so、libbinder.so、libvintf.so、libcutils.so、liblog.so、libutils.so、libselinux.so、libc++.so、libc.so、libm.so、libdl.so。大小为 71,728 字节。
- **C21** (verified): init 根据其服务定义启动 servicemanager：class core animation，user system，group system readproc，critical，onrestart restart apexd，audioserver，gatekeeperd，onrestart class_restart main，hal 和 early_hal，task_profiles ServiceCapacityLow ServicePerformance，shutdown critical。init/hw/init.rc 在第 504 行通过 start servicemanager 启动它，早于 hwservicemanager 和 vndservicemanager。
- **C22** (verified): SELinux 将该二进制标记为 servicemanager_exec，将 hwservicemanager 二进制标记为 hwservicemanager_exec。hwservicemanager 的 init 服务定义为 service hwservicemanager /system/bin/hwservicemanager。
- **C23** (verified): app_process64 是 PIE AArch64 可执行文件，解释器为 /system/bin/linker64，包含 13 个 DT_NEEDED 条目（libandroid_runtime.so、libbinder.so、libcutils.so、libhidlbase.so、liblog.so、libnativeloader.so、libsigchain.so、libutils.so、libwilhelm.so、libc++.so、libc.so、libm.so、libdl.so）。其字符串中包含入口类 com.android.internal.os.ZygoteInit 以及参数 --zygote 和 --start-system-server。大小为 20,400 字节。app_process 是指向它的符号链接。
- **C25** (verified): toybox 是 PIE AArch64 可执行文件，包含 8 个 DT_NEEDED 条目（libcrypto.so、libz.so、liblog.so、libprocessgroup.so、libselinux.so、libc.so、libm.so、libdl.so）。其字符串包含 multicall 标识文本 Toybox %s multicall binary。该组中有 173 个符号链接指向它。SELinux 将 /system/bin/toybox 标记为 toolbox_exec（plat_file_contexts 第 463 行）。挂载的 init 目录中没有任何文件提及 toybox 或 toolbox。
- **C27** (verified): platform.xml 是核心权限文件：根 permissions 元素包含 20 个 permission、45 个 assign-permission、13 个 split-permission、6 个 allow-in-power-save、3 个 allow-in-power-save-except-idle、1 个 allow-in-data-usage-save 以及 5 个 library 子元素。library 元素分别指定 android.test.base、android.test.mock、android.test.runner（依赖 android.test.base:android.test.mock）、android.hidl.base-V1.0-java 和 android.hidl.manager-V1.0-java，每个都带有位于 /system/framework 下的 file= 属性。
- **C28** (verified): privapp-permissions-platform.xml 包含 35 个 package 元素（属性 package）、377 个 permission 子元素，以及 213 个不同的权限名。第一个 package 是 android.ext.services，带有 PROVIDE_RESOLVER_RANKER_SERVICE 和 MONITOR_DEFAULT_SMS_PACKAGE。
- **C29** (verified): default-permissions.xml 包含 32 个 exception 元素（属性 package），以及 87 个 permission 授权子元素，其中 25 个带有 fixed=true。第一个 exception 是 com.facebook.wearable.system.location.proxy，带有 ACCESS_BACKGROUND_LOCATION 和 ACCESS_FINE_LOCATION，二者均为 fixed=true。
- **C30** (verified): framework-sysconfig.xml（config 根元素）包含 23 个 allow-implicit-broadcast action 元素、2 个 backup-transport-whitelisted-service 元素（service=com.android.localtransport/.LocalTransportService 和 com.android.encryptedlocaltransport/.EncryptedLocalTransportService）、1 个 bugreport-whitelisted（package com.android.shell）和 1 个 app-link（package com.android.carrierdefaultapp）。
- **C31** (verified): hiddenapi-package-whitelist.xml 包含 36 个 hidden-api-whitelisted-app 元素，每个都带有 package 属性；前几个条目为 android.ext.services、com.android.apps.tag、com.android.basicsmsreceiver 和 com.android.bookmarkprovider。
- **C33** (verified): 挂载的 system_a 树中，没有任何 init 文件或 SELinux 文件按名称引用 platform.xml、privapp-permissions-platform.xml、framework-sysconfig.xml、default-permissions.xml 或 hiddenapi-package-whitelist.xml，且 plat_file_contexts 中没有针对 /system/etc/permissions、/system/etc/sysconfig 或 /system/etc/default-permissions 的条目。
- **C35** (verified): system/bin 中的包装脚本遵循两种模式。模式 A 将 CLASSPATH 设为一个 framework jar，并在某个 com.android.commands 类上运行 app_process（hid：LD_PRELOAD libhidcommand_jni.so，CLASSPATH /system/framework/hid.jar，exec app_process /system/bin com.android.commands.hid.Hid；monkey：exec app_process $base/bin com.android.commands.monkey.Monkey）。模式 B 以服务名运行 cmd（pm：cmd package；settings：cmd settings；wm：cmd window；am：cmd activity，instrument 除外）。svc 将 svc wifi 映射为 cmd wifi set-wifi-enabled。
- **C36** (verified): 两个由 init 运行的脚本：logcatd 是 init 服务脚本，使用 -L 将上一个 logcat 缓冲区读取到持久存储，然后对当前缓冲区运行 logcat；logpersist.start 除非 getprop ro.debuggable 返回 1，否则拒绝运行。
- **C37** (verified): preloaded-classes 是一个纯文本列表，包含 2,482 个 Java 类名（例如 android.accounts.Account、android.accounts.Account$1、android.accounts.AccountManager）。

## 措辞已更正的论断

下方结论以数据所支持的措辞重新表述。

- **C05** (verified): 229 个符号链接解析为：173 个 toybox、6 个 toolbox、5 个 netutils-wrapper-1.0、5 个 iptables、3 个 mke2fs、3 个 fsck.f2fs、2 个 ziptool、2 个 logpersist.start、2 个 bzip2、2 个 /apex/com.android.runtime/bin/linker64、2 个 /apex/com.android.runtime/bin/linker，18 个相对的 ../boot-*.vdex 链接（9 个 boot 名称，各位于 system/framework/arm 和 system/framework/arm64 下），以及其余单个链接：system/bin/linkerconfig 指向 /apex/com.android.runtime/bin/linkerconfig，system/bin/dex2oat 指向 /apex/com.android.art/bin/dex2oat64，system/bin/dalvikvm 指向 /apex/com.android.art/bin/dalvikvm，system/bin/app_process 指向 app_process64，以及引导别名 system/bin/bootstrap/linker_asan -> linker 和 linker_asan64 -> linker64。
- **C06** (verified): 挂载树中存在的 8 个符号链接不在本组中：system/bin/disable-verity、enable-verity、hwclock、tee、touch、ueventd，以及 system/framework/arm 和 arm64 下的两个 boot-com.oculus.os.platform.vdex 链接。它们在 classified.tsv 中属于另外四个子系统：security-keystore-crypto（disable-verity、enable-verity、tee）、display-and-composer（hwclock）、sensors-mcu-meta-hal（touch 以及两个 boot-com.oculus.os.platform.vdex 链接）和 init-and-properties（ueventd）。
- **C07** (verified): 该组中有 22 个 APEX 类型的包：system_a 中 20 个（system/apex），product_a 中 1 个（apex/com.meta.quest.apex），odm_a 中 1 个（apex/com.meta.xr.apex）。两个包的清单名称与文件名不同：com.android.vndk.current.apex（清单名称为 com.android.vndk.v32）和 com.android.tethering.inprocess.capex（清单名称为 com.android.tethering）。每个 .apex 都有一个 ext4 类型的 apex_payload.img。每个 .capex 没有负载，并包含一个 original_apex 成员。com.android.art.capex 的 original_apex 大小为 60,234,888 字节。
- **C11** (verified): framework.jar（归档大小 34,835,598 字节）包含 45 个成员：classes.dex 至 classes4.dex（dex 039）、res/android.mime.types、res/debian.mime.types、res/vendor.mime.types、META-INF/MANIFEST.MF，以及 37 个 android/...*_compat_config.xml 文件。其 dex 字符串表包含 Landroid/app/ActivityThread;。
- **C18** (verified): services.jar 和 org.apache.http.legacy.jar 的配置文件是 ART profile，魔数为 pro、NUL、015、NUL（版本 015）。services.jar.bprof（3,244 字节）以字节 0x1e 开头，后接 /system/framework/services.jar 文本，因此不带 pro 魔数；其 boot-profile 的标识是根据 .bprof 后缀推断的。
- **C20** (observed): servicemanager 使用 libbinder 中定义的 android::os::IServiceManager binder 接口：其动态符号表从 LIBBINDER 导入 IServiceManager 描述符、IServiceManager 析构函数以及 BnServiceManager::onTransact。唯一定义的接口符号是弱符号 BnInterface<IServiceManager>::onAsBinder。
- **C24** (verified): init.zygote64_stub32.rc 第 18 行定义了 zygote_secondary，运行 /system/bin/stub_zygote，并以 /system/bin/app_process32 作为第一个参数。该 stub 二进制导入了 execve，因此可推断其会 exec app_process32。
- **C26** (observed): cmd 会输出消息 'No service specified; use -l to list all running services' 和 'cmd: Can't find service: <name>'。service 接受 list 和 check SERVICE：其用法文本为 Usage: service [-h|-?], service list, service check SERVICE。两个二进制在 plat_file_contexts 中都没有明确的条目。
- **C34** (verified): plat_seapp_contexts 将 system_server 进程（isSystemServer=true）分配到域 system_server_startup。services.jar 与该域的关联是根据 seapp 规则名称推断的；没有任何文件将 services.jar 映射到 system_server_startup。etc/classpaths/systemserverclasspath.pb 将 /system/framework/services.jar 列为 system server 类路径条目，但未指明域。
- **C38** (inferred): 所列的 15 个 ELF 可执行文件（VRStool、vrs-recorder、vrs-recorder2、vrsbuilder、vrfocusserver、xrspd、resource_control、resource_load、stream-publisher、runtimeipcbroker、wattsdown、flags_health_check、clean_scratch_files、profcollectd、profcollectctl）均为 PIE AArch64 二进制。它们的角色是根据名称和链接的库推断的。profcollectd、profcollectctl 和 flags_health_check 是平台组件（plat_file_contexts 第 553、577、607 行），因此它们不是 Meta 或 Oculus 专有的。

## 已排除的论断

下方声明已排除；每条都给出了原因。

| ID | 论断 | 原因 |
|---|---|---|
| C32 | preinstalled-packages-platform.xml 有 2 个 install-in-user-type 元素。其 package 属性为占位符 com.android.example。开头的注释定义了基础用户类型，包括 SYSTEM（用户 0）和 FULL（任何非 profile 的人类用户）。 | 2 个 install-in-user-type 元素的计数正确，基础用户类型 SYSTEM（用户 0）和 FULL（任何非 profile 的人类用户）位于开头的注释块中。package 属性的论断是错误的：两个元素分别带有 package=com.android.providers.settings（install-in SYSTEM、FULL 和 PROFILE）和 package=com.android.wallpaperbackup（install-in FULL）。字符串 com.android.example 仅出现在注释示例中（第 49 至 90 行）。 |
| C39 | 该组的 ELF 可执行文件为 PIE AArch64 二进制（32 位变体为 ARM），且该组中的每个 ELF 都使用 /system/bin/linker64 或 /system/bin/linker 作为其解释器。静态链接的例外为 gdbserver、gdbserver64 和 snapuserd，它们没有解释器行。 | 重新生成的 elf_facts.tsv 内容相同（261 个 ELF 行）。第 5 列计数：178 个 /system/bin/linker64、78 个空解释器（'-'）、3 个 /system/bin/linker、2 个 /system/bin/bootstrap/linker64。因此 183 行具有解释器行（而非 166 行），78 行没有（而非 95 行）。带有 /system/bin/bootstrap/linker64 的两行分别为 apexd 和 secilc，因此并非每个 ELF 都使用 linker64 或 linker。在 78 个没有解释器的条目中，72 个为 .odex、.oat、.art 或 .vdex 镜像，6 个为 gdbserver、gdbserver64、snapuserd、bootstrap/linker、bootstrap/linker64 和 bootstrap/linkerconfig；论断仅列出了三个例外。 |

## 缺口

- 未确定 services.jar 内五个 .uau 成员的用途；其格式未被解码。
- framework-res.apk 及 APK 类型包中的二进制 XML AndroidManifest.xml 未被解码；APK 类型 jar 的包名未被提取。
- 未对 services.jar 和 framework.jar 的 dex 代码进行反汇编；仅读取了 dex 版本和部分字符串表条目，因此未枚举由 SystemServer 启动的服务集合。
- 未解析 services.odex 和 boot-framework.oat 内的 ART OAT 键值头、方法表和校验和；未解码 boot-framework.art 镜像头字段。
- 未挂载或列出 APEX 负载文件系统（ext4）；仅读取了其清单名称和魔数。
- Meta 或 Oculus 专有 ELF 可执行文件，以及 hw/android.hidl.allocator@1.0-service 和 android.system.suspend@1.0-service 的角色均基于名称判断。由于二进制为 ARM 镜像，未执行任何内容。
- 未枚举 toybox applet 列表；173 个 toybox 符号链接的名称仅通过其 readlink 目标给出。
- 除用法和错误字符串外，未枚举 cmd 和 service 导出的 binder 接口；未为 servicemanager 提取 AIDL 描述符列表。
- init 与 SELinux 搜索仅覆盖 system_a/system/etc/init 和 system_a/system/etc/selinux 目录。未搜索 vendor_a、odm_a、product_a 和 system_ext_a 中的 init 与 SELinux 文件，因此这些分区中的引用未知。
- 未验证平台代码在运行时是否加载 platform.xml、privapp-permissions-platform.xml、framework-sysconfig.xml、default-permissions.xml 或 hiddenapi-package-whitelist.xml；挂载的 init 或 SELinux 树中没有任何文件提及它们。
- C06 中列出的、位于本组之外的八个符号链接未在此分类。
- preinstalled-packages-com.android.providers.media.module.xml 没有 XML 声明，仅作为文本读取；未进行结构化解析。
- 未提取 services.odex 和 boot-framework.oat 的 DT_NEEDED 列表，因为这些文件是包裹 ART 代码的 ELF 容器，运行时依赖记录在 OAT 头中。
