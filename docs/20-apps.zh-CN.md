# 20 - 应用

[English](20-apps.md)

## 摘要

应用组共有 431 行：product_a 27 行，system_a 123 行，system_ext_a 278 行，vendor_a 3 行。这些行包括 142 个 Android APK 压缩包（142 个不同的包名）、137 个 ELF odex 文件和 137 个 vdex 容器（对应 136 个包，Horizon 另有第二个 arm 变体）、2 个 ART 应用镜像、2 个 ART 配置文件，以及 11 个指向 /system_ext/lib64 的符号链接。在 APK 包中，103 个为 com.oculus/com.meta/com.facebook 包（主要位于 system_ext_a 和 product_a），37 个为 AOSP 的 com.android 包（主要位于 system_a），另有 2 个属于其他名称。大多数 APK 的清单报告的平台构建版本为 32；在声明了 targetSdkVersion 的 138 个 APK 中，有 84 个的值为 32。对于 15 个深入分析 APK，对于 15 个深入分析 APK，签名使用两个证书族：CN=Android（在 plat_mac_permissions.xml 中映射到 seinfo platform）和 CN=Android Debug（在 system_ext_mac_permissions.xml 中映射到 seinfo fbandroid_debug）。引用这些包名的配置位于本组之外，包括 privapp-permissions、default-permissions 和 sysconfig XML 文件、nonDiscretionaryAppPackageName 列表、system_ext_seapp_contexts（仅包含一个包）以及一个 vendor JSON 键。本组不含固件，也没有任何 init .rc 文件提及 15 个深入分析包中的任何一个。全部 431 行的角色信息位于 roles.tsv 中：306 个 verified，125 个 inferred。

角色表为本组 431 个文件各提供一行（306 verified，0 observed，125 inferred，0 unverified）。文件位于 `data/subsystems/apps/roles.tsv`。

## 已验证的结论

- **c01** (verified): 应用组共有 431 个输入行：product_a 27，system_a 123，system_ext_a 278，vendor_a 3。
- **c02** (verified): 按文件魔数划分的行类型：142 个 zip（APK）、137 个 ELF（odex）、137 个 vdex、11 个 symlink、2 个 ART image、2 个 profile。没有出现其他魔数。
- **c03** (verified): 从挂载点重新计算的全部 420 个普通文件的 SHA-256，与输入中每个文件的 sha256 列一致。
- **c04** (verified): roles.tsv 有一行表头和 431 行数据，每个输入行对应一行，其 (partition, path) 键与输入相同，且没有重复。置信度计数：306 verified，125 inferred。
- **c05** (verified): 全部 11 个符号链接行都指向 /system_ext/lib64 中的绝对路径目标（libjdns_sd.so、libqrcodereader_jni.so、libwis_input_device.so、libvoidactivity_jni.so、libpreloader.so、libpowerstatscollector.so、libfetchstats.so、libquickplaydataloader-jni.so、libcrypoimpl.so、libtourguideapi.so、libcmsservice-headset.so），且每个目标都存在于 system_ext_a 的 lib64 目录中。
- **c06** (verified): 137 个 odex ELF 文件的机器类型：135 个为 AArch64（e_machine 0xb7）；2 个为 ARM32（e_machine 0x28）：app/Horizon/oat/arm/Horizon.odex 和 priv-app/BodyApiService/oat/arm/BodyApiService.odex。
- **c07** (verified): 137 个 odex 文件各自在文件偏移 4096 附近包含 OAT 头字符串 'oat\n199\0'（搜索窗口为 4000-4112 字节）。
- **c09** (verified): 两个 ART 应用镜像为 SystemUX.art 和 SettingsPanelApp.art（art 头版本 099）。两个 ART 配置文件为 SystemUX.apk.prof 和 SettingsPanelApp.apk.prof（pro 头版本 015）。
- **c10** (verified): 有六个 APK 没有同名的 odex/vdex：product_a app/ModuleMetadata/ModuleMetadata.apk、product_a overlay/PermissionControllerRRO.apk、product_a overlay/PlatformRRO.apk、system_ext_a app/DeviceIdentityTest/DeviceIdentityTest.apk、system_ext_a priv-app/DeviceAuthServer/DeviceAuthServer.apk、system_ext_a priv-app/QuickPlayService/QuickPlayService.apk。137 个 odex 文件对应 136 个包（Horizon 有两个）。
- **c11** (verified): 142 个 APK 清单得出 142 个不同的包名。sharedUserId：无 93 个，android.uid.system 37 个，android.media 4 个，android.uid.shared 3 个，android.uid.bluetooth 1 个，android.uid.calendar 1 个，android.uid.networkstack 1 个，android.uid.shell 1 个，com.oculus.uid 1 个。包名前缀：com.oculus/com.meta/com.facebook 103 个，com.android 37 个，其他 2 个（android.ext.shared、com.qualcomm.timeservice）。
- **c12** (verified): 两个 RRO 叠加层都是静态叠加层：PlatformRRO.apk 的目标包为 'android'；PermissionControllerRRO.apk 的目标包为 com.android.permissioncontroller，targetName 为 PermissionControllerRoles。两者都没有组件。
- **c13** (verified): 15 个深入分析 APK 使用两个签名证书族：SHA-256 c8a2e9bccf597c2f...51192ab8（主体 CN=Android, O=Android, emailAddress android@android.com）用于 Settings、Bluetooth、Telecom、PackageInstaller、SettingsProvider、SystemPermissions、MetaSystemUI；SHA-256 fac61745dc090378...91033b9c（主体 CN=Android Debug）用于 SystemWebView、SystemUX、Horizon、Store、Integrity、OSUpdater、Carmel、VrShell。第一个证书族与 plat_mac_permissions.xml 中 seinfo 为 platform 的签名者匹配；第二个与 system_ext_mac_permissions.xml 中 seinfo 为 fbandroid_debug 的签名者匹配。
- **c14** (verified): SELinux 的 mac_permissions 文件中，system_ext_mac_permissions.xml 有 11 个签名者条目（seinfo 值为 echovr、fbandroid_debug、oculus_apps、oculus_apps_v2、oculus_core、oculus_inhouse、oculus_loneechoavalanche、oculus_privapps、oculus_prod、oculus_workrooms、oculus_workrooms_dev），plat_mac_permissions.xml 有 3 个签名者条目（seinfo 值为 platform、media、network_stack）。142 个包名中没有一个作为 package 元素出现在这两个文件中。
- **c15** (verified): 在 15 个深入分析包中，只有 com.oculus.updater 出现在 SELinux seapp 文件中：system_ext_seapp_contexts 第 35 和 36 行，二者的 domain 均为 osupdater_app。其余 14 个深入分析包名未出现在已挂载 etc 目录树的任何 selinux 目录中。
- **c16** (verified): 已挂载分区中的任何 init .rc 文件都不包含深入分析 APK 的 priv-app/app 路径模式，也不包含 15 个深入分析包名。init 目录（etc/init、system/etc/init、etc/init/hw）中没有匹配的行。
- **c18** (verified): 142 个 APK 的目标 SDK 分布：23（6）、24（2）、25（2）、26（2）、28（14）、29（17）、30（3）、31（3）、32（84）、33（2）、34（1）、35（2）；四个 APK 未声明目标 SDK。minSdkVersion 的取值范围为 23 至 32；versionName 12 出现在 79 个 APK 上。
- **d01** (verified): Settings.apk（system_ext_a/priv-app/Settings）：包名 com.android.settings，versionName 12，versionCode 32，sharedUserId android.uid.system，minSdkVersion 32，targetSdkVersion 32；63099467 字节，sha256 05024d4363869eef685bdbb9d4add1d1091e4277d033982143fba5625fac6887；2434 个 zip 条目；classes.dex 和 classes2.dex，dex 版本均为 039（8102 和 1423 个类定义）；无原生库；343 个组件（282 个 activity、15 个 activity-alias、20 个 receiver、15 个 service、11 个 provider），其中 213 个 activity、15 个 alias、16 个 receiver、10 个 service 和 7 个 provider 已导出；105 个 uses-permission；uses-library 为 org.apache.http.legacy、androidx.window.extensions、androidx.window.sidecar；uses-feature 为 oculus.software.handtracking；application 类为 com.android.settings.SettingsApplication。
- **d01b** (verified): Settings odex（oat/arm64/Settings.odex，AArch64 ELF64 DYN，95152 字节）：OAT 键值存储中读取到 compilation-reason=prebuilt、compiler-filter=verify、concurrent-copying=true、debuggable=false、native-debuggable=false、requires-image=false，其 classpath 以 /system/framework/org.apache.http.legacy.jar、/system_ext/framework/androidx.window.extensions.jar 和 /system_ext/framework/androidx.window.sidecar.jar 开头，与清单中的 uses-library 列表一致。.dex 节为大小 0x31d5c 的 NOBITS。
- **d02** (verified): Bluetooth.apk（system_a/app/Bluetooth）：包名 com.android.bluetooth，versionName 12，sharedUserId android.uid.bluetooth，minSdk 和 targetSdk 为 32；8401460 字节，sha256 eaf2ee6d206d26a1ef3651e3a71b151d1fd88637af534d468c6a316136391d60；95 个 zip 条目；classes.dex 版本 039（5404 个类定义）；无原生库；40 个组件（24 个 service、10 个 activity、4 个 provider、2 个 receiver），其中 22 个 service、2 个 activity、2 个 provider 和 2 个 receiver 已导出；61 个 uses-permission；uses-library 为 javax.obex；application 类为 com.android.bluetooth.btservice.AdapterApp；provider 为 com.android.bluetooth.opp.BluetoothOppProvider，authority 为 com.android.bluetooth.opp（已导出）；使用 CN=Android 证书族签名。
- **d03** (verified): Telecom.apk（system_a/priv-app/Telecom）：包名 com.android.server.telecom，versionName 12，sharedUserId android.uid.system，process 为 'system'，minSdk 和 targetSdk 为 32；10581692 字节，sha256 cbf0ff75fe09a38d7a6cd0780333d688a355a70c219020845b7cde3a7cf9dd24；930 个 zip 条目；classes.dex 版本 039（3092 个类定义）；无原生库；15 个组件（9 个 activity、2 个 activity-alias、2 个 receiver、2 个 service），其中 4 个 activity、2 个 alias、1 个 receiver 和 2 个 service 已导出；28 个 uses-permission，包括 android.permission.CALL_PRIVILEGED 和 BIND_INCALL_SERVICE；无 uses-library；声明了 5 个自定义权限，protectionLevel 为 0x12。
- **d05** (verified): SettingsProvider.apk（system_a/priv-app/SettingsProvider）：包名 com.android.providers.settings，versionName 12，sharedUserId android.uid.system，process 为 'system'；374089 字节，sha256 e0db7e587c4f4f975cec162f0f42d10cd6a742ef8b7462b14065751734667b91；16 个 zip 条目；一个 classes.dex，版本 039，81 个类定义；无原生库；2 个组件（一个 provider、一个 service），均已导出；provider 的 authority 为 'settings'；service WriteFallbackSettingsFilesJobService 要求 android.permission.BIND_JOB_SERVICE；0 个 uses-permission。
- **d06** (verified): SystemWebView.apk（product_a/app/SystemWebView）：包名 com.android.webview，versionName 126.0.6478.122，versionCode 647812201，minSdkVersion 26，targetSdkVersion 34，compileSdkVersion 34；240555911 字节，sha256 20b7ba7f934ea32d09d7aee25cbaceaa3ca467116e8ba4df6bfc4d8a7b79345b；605 个 zip 条目；一个 classes.dex，版本 038（3480 个类定义）；arm64-v8a 和 armeabi-v7a 的原生库：libwebviewchromium.so 和 libcrashpad_handler_trampoline.so；61 个组件（52 个 service、3 个 activity、2 个 activity-alias、4 个 provider），其中 50 个 service、2 个 activity 和 4 个 provider 已导出；6 个 uses-permission；uses-library 为 androidx.window.extensions、android.ext.adservices；uses-feature 为 android.hardware.touchscreen 和 android.software.leanback；META-INF 包含 oculus_prod.OSIG；使用 CN=Android Debug 证书族签名。
- **d07** (verified): SystemUX.apk（system_ext_a/priv-app/SystemUX）：包名 com.oculus.systemux，versionName 71.0.0.605.522，versionCode 665334819，无 sharedUserId，minSdk 和 targetSdk 为 32；26833840 字节，sha256 e2bac78ac3fb06aed7d2d7251dcfe910d6bcaed76b2371a88b9674226611779c；1338 个 zip 条目；classes.dex 和 classes2.dex，版本均为 035（2324 和 11736 个类定义）；超过 10 个 arm64-v8a 原生库（例如 libairlinkjni.so、libappmenupanel.so、libc++_shared.so）；70 个组件（33 个 service、13 个 activity、21 个 receiver、2 个 provider、1 个 activity-alias），其中 27 个 service、13 个 activity、12 个 receiver 和 1 个 alias 已导出；108 个 uses-permission；声明了 6 个自定义权限；application 类为 com.oculus.systemux.SystemUXApplication；使用 CN=Android Debug 证书族签名；随附 SystemUX.art（版本 099）和 SystemUX.apk.prof（版本 015）。
- **d08** (verified): Horizon.apk（system_ext_a/app/Horizon）：包名 com.oculus.horizon，versionName 71.0.0.894.253，versionCode 665345962，sharedUserId com.oculus.uid，minSdk 和 targetSdk 为 23，compileSdk 为 34；83246125 字节，sha256 c3259210a521e710fb0e1bd7b48a2fb60d44cdb31e8b5c5dd9b32dd5e3bb92f0；596 个 zip 条目；classes.dex（版本 035，3523 个类定义）和 classes2.dex（版本 035，9514 个类定义）；arm64-v8a 原生库，包括 libNativeApiManager-V1.so；108 个组件（53 个 service、37 个 receiver、15 个 provider、3 个 activity），其中 17 个 service、16 个 receiver、12 个 provider 和 1 个 activity 已导出；61 个 uses-permission；声明了 20 个自定义权限；uses-feature 为 android.hardware.camera；同时提供 arm（ARM32）和 arm64（AArch64）的 odex 文件；使用 CN=Android Debug 证书族签名。
- **d09** (verified): Store.apk（system_ext_a/app/Store）：包名 com.oculus.store，versionName 228.0.0.19.109，versionCode 374700256，minSdk 和 targetSdk 为 28，compileSdk 为 35；22705166 字节，sha256 ef9c9219c12d81c41b7f82d8adcba64701252e0b07dcaa9bc1c09bf79b7e8575；586 个 zip 条目；classes.dex（版本 035，2634 个类定义）和 classes2.dex（版本 035，5506 个类定义）；arm64-v8a 原生库（例如 libasyncexecutor.so、libbsdiff.so）；34 个组件（14 个 service、3 个 activity、15 个 receiver、2 个 provider），其中 3 个 service、2 个 activity 和 5 个 receiver 已导出；21 个 uses-permission；声明了 2 个自定义权限；使用 CN=Android Debug 证书族签名，签名文件为 META-INF/ANDROIDD.RSA。
- **d11** (verified): Integrity.apk（system_ext_a/priv-app/Integrity）：包名 com.oculus.integrity，versionName 71.0.0.618.530，versionCode 665335042，minSdk 和 targetSdk 为 28，compileSdk 为 35；16765255 字节，sha256 241a66412764818df76c5232a86b521a7940847e8c9a1266bfe006eaecb22ec0；1787 个 zip 条目；classes.dex（版本 035，2331 个类定义）和 classes2.dex（版本 035，5172 个类定义）；arm64-v8a 原生库（例如 libandroid_aware_dlopen.so、libbreakpad.so、libcoremerged.so）；30 个组件（15 个 service、12 个 receiver、2 个 provider、1 个 activity），其中 3 个 service 和 3 个 receiver 已导出；33 个 uses-permission；uses-library 为 org.apache.http.legacy；声明了 2 个自定义权限；使用 CN=Android Debug 证书族签名。
- **d12** (verified): OSUpdater.apk（system_ext_a/priv-app/OSUpdater）：包名 com.oculus.updater，versionName 71.0.0.890.252，versionCode 665332293，minSdk 和 targetSdk 为 26，compileSdk 为 34；6155671 字节，sha256 e96bca4ce8120d3e289e334940e7ae115a6d263dcb267d599c6c5a25eec8d958；134 个 zip 条目；一个 classes.dex（版本 035，6551 个类定义）；无原生库；24 个组件（14 个 receiver、9 个 service、1 个 provider），其中 5 个 receiver 和 3 个 service 已导出；30 个 uses-permission；uses-feature 为 android.hardware.vr.headtracking；声明了 3 个自定义权限；application 类为 com.oculus.updater.OculusUpdaterApplication；使用 CN=Android Debug 证书族签名（ANDROIDD.RSA）。
- **d13** (verified): MetaSystemUI.apk（system_ext_a/priv-app/MetaSystemUI）：包名 com.meta.systemui，versionName 12，无 sharedUserId，minSdk 和 targetSdk 为 32；process 为 com.meta.systemui，persistent 为 true；2655491 字节，sha256 5637bd3c7b6e4a122299a181dd2bd5c313d78e930ec346b3733ead54682e648e；322 个 zip 条目；一个 classes.dex（版本 039，1584 个类定义）；无原生库；2 个 service，均未导出；8 个 uses-permission（包括 INTERNAL_SYSTEM_WINDOW 和 MANAGE_ACTIVITY_TASKS）；声明了 0 个自定义权限；使用 CN=Android platform 证书族签名。
- **d14** (verified): Carmel.apk（system_ext_a/app/Carmel）：包名 com.oculus.browser，versionName 35.1.0.78.167.646680531，versionCode 565301467，minSdkVersion 26，targetSdkVersion 35，compileSdk 35；194804395 字节，sha256 1e44fe1ae8ed0e56749327cca8961bf463d1545c4aba11d953c4356120a090db；3445 个 zip 条目；一个 classes.dex（版本 038，13356 个类定义）；arm64-v8a 原生库 libchrome.so、libchromium_android_linker.so、libopenxr_loader.so 和 libpanelappsdk.so；138 个组件（71 个 service、32 个 activity、24 个 receiver、6 个 provider、5 个 activity-alias），其中 10 个 activity、10 个 receiver、5 个 alias、3 个 service 和 1 个 provider 已导出；73 个 uses-permission；声明了 7 个自定义权限；uses-library 为 androidx.window.extensions、android.ext.adservices；application 类为 com.oculus.browser.HydraApplication；使用 CN=Android Debug 证书族签名。
- **r01** (verified): Settings 在三个配置文件中以名称被引用：system_ext_a/etc/permissions/com.android.settings.xml 第 18 行（privapp-permissions 块，含 47 项权限授予）、system_ext_a/etc/nonDiscretionaryAppPackageName 第 183 行，以及 system_a/system/etc/permissions/privapp-permissions-platform.xml 第 554 行（块中含 2 项授予）。没有 init 或 SELinux 文件引用它。
- **r03** (verified): Telecom（com.android.server.telecom）被引用于 system_a/system/etc/permissions/privapp-permissions-platform.xml 第 274 行（块中含 11 项权限授予），在已挂载的 etc 目录树中别无其他引用。
- **r04** (verified): PackageInstaller（com.android.packageinstaller）被引用于 system_a/system/etc/permissions/privapp-permissions-platform.xml 第 133 行（块中含 7 项权限授予），在已挂载的 etc 目录树中别无其他引用。
- **r05** (verified): com.android.providers.settings 被引用于 system_ext_a/etc/nonDiscretionaryAppPackageName 第 184 行，以及 system_a/system/etc/sysconfig/preinstalled-packages-platform.xml 第 105 行；后者中的 install-in-user-type 条目列出了 SYSTEM 用户类型。
- **r06** (verified): com.android.webview 未在已挂载 etc 目录树下的任何文件中被引用。
- **r07** (verified): SystemUX（com.oculus.systemux）被引用于 system_ext_a/etc/permissions/com.oculus.systemux.xml 第 4 行（27 项 privapp 授予）、system_ext_a/etc/nonDiscretionaryAppPackageName 第 37 行、system_a/system/etc/sysconfig/oculus-hiddenapi-package-whitelist.xml 第 21 行，以及 system_a/system/etc/default-permissions/default-permissions.xml 第 168 行（授予 ACCESS_BACKGROUND_LOCATION 且 fixed=false 的例外块）。
- **r08** (verified): Horizon（com.oculus.horizon）被引用于 system_ext_a/etc/permissions/com.oculus.horizon.xml 第 3 行（privapp 块，3 项授予）、system_ext_a/etc/nonDiscretionaryAppPackageName 第 118 行、system_a/system/etc/sysconfig/oculus-hiddenapi-package-whitelist.xml 第 12 行、system_a/system/etc/sysconfig/oculus-doze-allowlist.xml 第 9 行（allow-in-power-save），以及 system_a/system/etc/default-permissions/default-permissions.xml 第 74 行。product_a/etc/permissions/com.meta.quest.xml 第 10 行中该字符串仅出现在权限名称内（com.oculus.horizon.notifications.receivers.RECEIVE_SILENT_NOTIFS）。
- **r09** (verified): Store（com.oculus.store）被引用于 system_ext_a/etc/nonDiscretionaryAppPackageName 第 41 行，以及 system_ext_a/etc/default-permissions/com.oculus.store.xml 第 4 行；后者是一个例外块，带有 package 属性和一项 POST_NOTIFICATIONS 权限（fixed=true）。
- **r10** (verified): SystemPermissions（com.oculus.systempermissions）仅被引用于 system_ext_a/etc/permissions/com.oculus.systempermissions.xml 第 4 行，即一个含 5 项授予的 privapp-permissions 块。
- **r11** (verified): Integrity（com.oculus.integrity）被引用于 system_ext_a/etc/permissions/com.oculus.integrity.xml 第 4 行（privapp-permissions 块，3 项授予），以及 system_a/system/etc/default-permissions/default-permissions.xml 第 80 行。它不在 nonDiscretionaryAppPackageName 中。
- **r12** (verified): OSUpdater（com.oculus.updater）被引用于：system_ext_a/etc/selinux/system_ext_seapp_contexts 第 35 和 36 行（domain 为 osupdater_app，seinfo 为 fbandroid_debug 和 oculus_prod）；system_ext_a/etc/permissions/com.oculus.updater.xml 第 3 行（privapp 块，10 项授予）；system_ext_a/etc/nonDiscretionaryAppPackageName 第 26 行；system_a/system/etc/sysconfig/oculus-whitelisted-staged-installer.xml 第 6 行（whitelisted-staged-installer）；oculus-hiddenapi-package-whitelist.xml 第 22 行；oculus-doze-allowlist.xml 第 11 行。
- **r13** (verified): com.meta.systemui 未在已挂载 etc 目录树下的任何文件中被引用。
- **r14** (verified): Carmel（com.oculus.browser）被引用于 system_ext_a/etc/nonDiscretionaryAppPackageName 第 137 行，以及 system_a/system/etc/default-permissions/default-permissions.xml 第 57 行（针对 com.oculus.browser 的例外块，BLUETOOTH_CONNECT fixed=true）。它没有 privapp 块。

## 措辞已修正的结论

这些结论实质上成立。以下措辞取代原文。

- **c08** (verified): Settings.vdex 头（魔数 'vdex'，版本 '027' 加 NUL）在偏移 8、12、...、36 处的 32 位值依次为 4、0、60、8、1、0、0、2。该文件大小为 204124 字节；Settings odex 为 95152 字节。该文件不包含 'dex' 加换行的序列。这些值的字段名称未由文件本身确定；第一个值（4）与两个 classes*.dex 条目之间的含义关系尚未解决。
- **c17** (verified): 已挂载的 file_contexts 文件中没有针对 /system_ext/app 或 /system_ext/priv-app 的显式标签。system_ext 路径落入通用规则 '/(system_ext|system/system_ext)(/.*)?  u:object_r:system_file:s0'（plat_file_contexts 第 55 行）。显式的应用目录标签仅存在于 vendor 和 odm：plat_file_contexts 第 13 行（vendor app）和第 29 行（odm app），以及第 14 行（vendor priv-app）和第 30 行（odm priv-app）。plat_file_contexts 没有针对 /system/app 或 /system/priv-app 的显式标签。
- **d04** (verified): PackageInstaller.apk（system_a/priv-app/PackageInstaller）：包名 com.android.packageinstaller，versionName 12，无 sharedUserId，minSdk 和 targetSdk 为 32；3171858 字节，sha256 b7d89294664b1d6ecda8c10bd4313de3b202e13a1c3b3ad51b0b4159e4d4b3db；416 个 zip 条目；classes.dex 版本 039（1374 个类定义）；无原生库；17 个组件（receiver 5、activity 10、service 1、provider 1），其中已导出的 activity 2 个、service 1 个、provider 1 个；15 个 uses-permission（INSTALL_PACKAGES、DELETE_PACKAGES、...）；application 类为 com.android.packageinstaller.PackageInstallerApplication；provider 的 authority 为 com.google.android.packageinstaller.wear.provider（已导出）；声明了 0 个自定义权限。
- **d10** (verified): SystemPermissions.apk（system_ext_a/priv-app/SystemPermissions）：包名 com.oculus.systempermissions，versionName 12，无 sharedUserId，minSdk 和 targetSdk 为 32；1313575 字节，sha256 a561fc1a84d6477d873119b095c241422114c0beb5232fad4190cc2538f8a04d；71 个 zip 条目；classes.dex 版本 039（629 个类定义）；无原生库；3 个 activity（MainActivity、ConsentZeroActivity、WebviewActivity），其中 2 个已导出；16 个 uses-permission（包括 ACCESS_FINE_LOCATION、WRITE_SECURE_SETTINGS）；uses-feature 为 android.hardware.location 和 oculus.software.handtracking；3 项权限声明：com.oculus.permission.USE_ACCOUNTS_META、com.oculus.permission.USE_ACCOUNTS_OCULUS，以及 android.permission.WRITE_SECURE_SETTINGS（此处重新声明了一个平台权限名）；使用 CN=Android 证书族签名。
- **d15** (verified): VrShell.apk（system_ext_a/priv-app/VrShell）：包名 com.oculus.vrshell，versionName 71.0.0.612.529，versionCode 665334768，minSdk 和 targetSdk 为 29，compileSdk 为 34；88072820 字节，sha256 ac4541ca21c32d2fef2886d6f765ce92488fbfbcc0c04b9710bcbd0c92315ca7；1093 个 zip 条目；classes.dex 和 classes2.dex，版本均为 035（2359 和 7393 个类定义）；arm64-v8a 原生库（例如 libNativeApiManager.so、libVertsClient.so、libVideoJni.so）；122 个组件（52 个 service、41 个 activity、21 个 receiver、7 个 provider、1 个 activity-alias）。android:exported="true" 的组件：30 个 service、10 个 receiver、4 个 activity、3 个 provider、1 个 alias。另有 12 个组件（4 个 activity、5 个 service、3 个 receiver）带有 intent filter 且没有 android:exported 属性；在 targetSdkVersion 29 时，这些组件默认导出。115 个 uses-permission；11 个 uses-feature 条目；声明了 16 个自定义权限；application 类为 com.oculus.vrshell.ShellApplication；使用 CN=Android Debug 证书族签名。
- **r02** (verified): com.android.bluetooth 未出现在已挂载 etc 目录树下的任何文件中（0 处匹配），因此 Bluetooth APK 不存在 privapp-permissions、default-permissions、sysconfig 或 SELinux 引用。其他文件中提到了 bluetooth（例如 product_a/etc/permissions/android.hardware.bluetooth.xml），但没有一个包含该包名。
- **r15** (verified): VrShell（com.oculus.vrshell）被引用于：system_ext_a/etc/permissions/com.oculus.vrshell.xml 第 4 行（19 项 privapp 授予）；system_ext_a/etc/permissions/com.oculus.vrshell.desktop.xml 第 4 行（针对 com.oculus.vrshell.desktop）；system_ext_a/etc/nonDiscretionaryAppPackageName 第 12 至 16 行；system_ext_a/etc/default-permissions/com.oculus.vrshell.xml 第 4 行；system_a/system/etc/default-permissions/default-permissions.xml 第 185 行（com.oculus.vrshell）和第 194 行（com.oculus.vrshell.home）；system_a/system/etc/sysconfig/oculus-hiddenapi-package-whitelist.xml 第 24 行（com.oculus.vrshell）和第 25 行（com.oculus.vrshell.home）；以及 vendor_a/etc/eps_properties.json 第 47 行，其中键路径 mixer_thread.package_blocklist 列出了 com.oculus.vrshell 和 com.oculus.firsttimenux。

<!-- generated: apps start -->
## 包索引（生成）

本索引涵盖镜像中的 146 个 `.apk` 文件：本分组的 142 个 APK 压缩包，以及其他分组中的另外 4 个（`framework-res.apk`（framework-and-system-services）、`Provision.apk`（sensors-mcu-meta-hal）、`QuestManagedProvisioning.apk`（sensors-mcu-meta-hal）、`com.oculus.os.platform-res.apk`（sensors-mcu-meta-hal））。
每个文件都由 `tools/apk/apkfacts.py` 使用 androguard 4.1.4 读取。事实位于 `data/apks/<name>.json`，汇总表为 `data/apks/summary.tsv`。本表由 `tools/apk/gen_all_apks_doc.py` 生成。

- 各分区的文件数：system_a 42、system_ext_a 92、product_a 11、vendor_a 1。
- 签名证书族（签名证书前 16 位十六进制数字）：`c8a2e9bccf597c2f` 72、`fac61745dc090378` 53、`a40da80a59d170ca` 11、`28bbfe4a7b97e746` 4、`465983f7791f2abe` 4、`b6f96027df75ec89` 1、`dc2f834e3c194237` 1。
- 具有共享 UID 的包：51 个。按 UID 计：`android.uid.system` 39、`android.media` 4、`android.uid.shared` 3、`android.uid.bluetooth` 1、`android.uid.calendar` 1、`android.uid.networkstack` 1、`android.uid.shell` 1、`com.oculus.uid` 1。
- 附带原生库的包：42 个。
- 不同的包名：146 个，与文件一一对应。
- 未读取的非 APK 容器文件：清单中的 12 个 `.apex` 文件。
- 标注 27 的行为第 27 页功能小节所述的 22 个应用。

| 分组 | 分区 | 文件 | 包名 | 版本 | minSDK | targetSDK | 共享 UID | 签名（前 16 位） | Activities | Services | Receivers | Providers | 权限 | 第 27 页 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| apps | system_a | `system/app/Bluetooth/Bluetooth.apk` | `com.android.bluetooth` | 12 | 32 | 32 | `android.uid.bluetooth` | `c8a2e9bccf597c2f` | 10 | 24 | 2 | 4 | 61 | 27 |
| apps | system_a | `system/app/BookmarkProvider/BookmarkProvider.apk` | `com.android.bookmarkprovider` | 12 | 32 | 32 |  | `a40da80a59d170ca` | 0 | 0 | 0 | 1 | 0 |  |
| apps | system_a | `system/app/CertInstaller/CertInstaller.apk` | `com.android.certinstaller` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 4 | 0 | 0 | 0 | 5 |  |
| apps | system_a | `system/app/CompanionDeviceManager/CompanionDeviceManager.apk` | `com.android.companiondevicemanager` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 1 | 1 | 0 | 1 | 13 |  |
| apps | system_a | `system/app/EnvironmentNuxD/EnvironmentNuxD.apk` | `com.meta.environment.prod.nuxd` | NuxD-477444881 | 29 | 29 |  | `fac61745dc090378` | 0 | 0 | 0 | 0 | 0 |  |
| apps | system_a | `system/app/ExtShared/ExtShared.apk` | `android.ext.shared` | 1 | 32 | 32 |  | `c8a2e9bccf597c2f` | 0 | 0 | 0 | 0 | 0 |  |
| apps | system_a | `system/app/FirstTimeNuxEureka/FirstTimeNuxEureka.apk` | `com.oculus.firsttimenux` | 71.2.0.117.949 | 28 | 28 |  | `fac61745dc090378` | 3 | 11 | 14 | 1 | 38 | 27 |
| apps | system_a | `system/app/GuardianResourcesEureka/GuardianResourcesEureka.apk` | `com.oculus.guardianresources` | 71.0.0.0.0 |  |  |  | `fac61745dc090378` | 0 | 0 | 0 | 0 | 0 |  |
| apps | system_a | `system/app/HTMLViewer/HTMLViewer.apk` | `com.android.htmlviewer` | 12 | 32 | 32 |  | `a40da80a59d170ca` | 1 | 0 | 0 | 0 | 0 |  |
| apps | system_a | `system/app/KeyChain/KeyChain.apk` | `com.android.keychain` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 1 | 1 | 0 | 0 | 2 |  |
| apps | system_a | `system/app/PacProcessor/PacProcessor.apk` | `com.android.pacprocessor` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 0 | 2 |  |
| apps | system_a | `system/app/PlatformCaptivePortalLogin/PlatformCaptivePortalLogin.apk` | `com.android.captiveportallogin` | s_aml_319999900 | 29 | 30 |  | `c8a2e9bccf597c2f` | 1 | 1 | 0 | 0 | 8 |  |
| apps | system_a | `system/app/xrstreamingclient/xrstreamingclient.apk` | `com.oculus.xrstreamingclient` | 71.0.0.601.524 | 24 | 29 |  | `c8a2e9bccf597c2f` | 2 | 2 | 4 | 0 | 27 | 27 |
| framework-and-system-services | system_a | `system/framework/framework-res.apk` | `android` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 25 | 26 | 15 | 1 | 25 |  |
| apps | system_a | `system/priv-app/BackupRestoreConfirmation/BackupRestoreConfirmation.apk` | `com.android.backupconfirm` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 1 | 0 | 0 | 0 | 2 |  |
| apps | system_a | `system/priv-app/BlockedNumberProvider/BlockedNumberProvider.apk` | `com.android.providers.blockednumber` | 12 | 32 | 32 | `android.uid.shared` | `28bbfe4a7b97e746` | 0 | 0 | 0 | 1 | 1 |  |
| apps | system_a | `system/priv-app/BluetoothConnectivityService/BluetoothConnectivityService.apk` | `com.oculus.vrbluetooth` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 2 | 0 | 11 |  |
| apps | system_a | `system/priv-app/CalendarProvider/CalendarProvider.apk` | `com.android.providers.calendar` | 12 | 32 | 32 | `android.uid.calendar` | `a40da80a59d170ca` | 2 | 1 | 4 | 1 | 24 |  |
| apps | system_a | `system/priv-app/ContactsProvider/ContactsProvider.apk` | `com.android.providers.contacts` | 12 | 32 | 32 | `android.uid.shared` | `28bbfe4a7b97e746` | 1 | 0 | 3 | 6 | 18 |  |
| apps | system_a | `system/priv-app/DocumentsUI/DocumentsUI.apk` | `com.android.documentsui` | 12 | 29 | 30 |  | `c8a2e9bccf597c2f` | 6 | 1 | 3 | 3 | 13 |  |
| apps | system_a | `system/priv-app/DownloadProvider/DownloadProvider.apk` | `com.android.providers.downloads` | 12 | 32 | 32 | `android.media` | `465983f7791f2abe` | 0 | 3 | 1 | 2 | 18 |  |
| apps | system_a | `system/priv-app/ExternalStorageProvider/ExternalStorageProvider.apk` | `com.android.externalstorage` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 0 | 0 | 1 | 2 | 5 |  |
| apps | system_a | `system/priv-app/FusedLocation/FusedLocation.apk` | `com.android.location.fused` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 0 | 7 |  |
| apps | system_a | `system/priv-app/InProcessNetworkStack/InProcessNetworkStack.apk` | `com.android.networkstack.inprocess` | 12 | 29 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 2 | 0 | 0 | 0 |  |
| apps | system_a | `system/priv-app/InputDevices/InputDevices.apk` | `com.android.inputdevices` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 0 | 1 | 0 | 0 |  |
| apps | system_a | `system/priv-app/LocalTransport/LocalTransport.apk` | `com.android.localtransport` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 0 | 0 |  |
| apps | system_a | `system/priv-app/MediaProviderLegacy/MediaProviderLegacy.apk` | `com.android.providers.media` | 12 | 32 | 32 | `android.media` | `465983f7791f2abe` | 0 | 0 | 0 | 1 | 0 |  |
| apps | system_a | `system/priv-app/MrServiceEureka/MrServiceEureka.apk` | `com.oculus.mrservice` | 71.0.0.605.498 | 24 |  |  | `fac61745dc090378` | 1 | 1 | 1 | 1 | 23 |  |
| apps | system_a | `system/priv-app/MtpService/MtpService.apk` | `com.android.mtp` | 12 | 32 | 32 | `android.media` | `465983f7791f2abe` | 1 | 2 | 2 | 1 | 8 |  |
| apps | system_a | `system/priv-app/NuxOta/NuxOta.apk` | `com.oculus.nux.ota` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 2 | 1 | 0 | 8 |  |
| apps | system_a | `system/priv-app/PackageInstaller/PackageInstaller.apk` | `com.android.packageinstaller` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 10 | 1 | 5 | 1 | 15 |  |
| apps | system_a | `system/priv-app/PlatformNetworkPermissionConfig/PlatformNetworkPermissionConfig.apk` | `com.android.networkstack.permissionconfig` | 2019-09 | 29 | 30 | `android.uid.networkstack` | `c8a2e9bccf597c2f` | 0 | 0 | 0 | 0 | 0 |  |
| apps | system_a | `system/priv-app/ProxyHandler/ProxyHandler.apk` | `com.android.proxyhandler` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 0 | 1 |  |
| apps | system_a | `system/priv-app/SettingsProvider/SettingsProvider.apk` | `com.android.providers.settings` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 1 | 0 |  |
| apps | system_a | `system/priv-app/SharedStorageBackup/SharedStorageBackup.apk` | `com.android.sharedstoragebackup` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 0 | 3 |  |
| apps | system_a | `system/priv-app/Shell/Shell.apk` | `com.android.shell` | 12 | 32 | 32 | `android.uid.shell` | `c8a2e9bccf597c2f` | 2 | 1 | 3 | 3 | 365 |  |
| apps | system_a | `system/priv-app/SoundPicker/SoundPicker.apk` | `com.android.soundpicker` | 12 | 32 | 32 | `android.media` | `465983f7791f2abe` | 1 | 1 | 1 | 0 | 5 |  |
| apps | system_a | `system/priv-app/StatementService/StatementService.apk` | `com.android.statementservice` | 1.0 | 32 | 29 |  | `c8a2e9bccf597c2f` | 0 | 4 | 11 | 1 | 10 |  |
| apps | system_a | `system/priv-app/Telecom/Telecom.apk` | `com.android.server.telecom` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 9 | 2 | 2 | 0 | 28 | 27 |
| apps | system_a | `system/priv-app/UserDictionaryProvider/UserDictionaryProvider.apk` | `com.android.providers.userdictionary` | 12 | 32 | 32 | `android.uid.shared` | `28bbfe4a7b97e746` | 0 | 0 | 0 | 1 | 1 |  |
| apps | system_a | `system/priv-app/VpnDialogs/VpnDialogs.apk` | `com.android.vpndialogs` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 4 | 0 | 0 | 0 | 4 |  |
| apps | system_a | `system/priv-app/VrDriver/VrDriver.apk` | `com.oculus.systemdriver` | 71.0.0.601.498 | 23 | 24 |  | `fac61745dc090378` | 1 | 4 | 4 | 5 | 55 | 27 |
| apps | system_ext_a | `app/AccountsCenterPWA/AccountsCenterPWA.apk` | `com.meta.AccountsCenter.pwa` | 1.0.0.368134505 | 28 | 29 |  | `b6f96027df75ec89` | 0 | 0 | 0 | 0 | 1 |  |
| apps | system_ext_a | `app/AvatarEditor/AvatarEditor.apk` | `com.oculus.avatareditor` | 98.4.0.16.218 | 29 | 29 |  | `fac61745dc090378` | 1 | 11 | 14 | 1 | 19 |  |
| apps | system_ext_a | `app/Carmel/Carmel.apk` | `com.oculus.browser` | 35.1.0.78.167.646680531 | 26 | 35 |  | `fac61745dc090378` | 32 | 71 | 24 | 6 | 62 | 27 |
| apps | system_ext_a | `app/ChargeControl/ChargeControl.apk` | `com.oculus.os.chargecontrol` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 1 | 1 | 0 | 0 | 0 |  |
| apps | system_ext_a | `app/DeviceIdentityTest/DeviceIdentityTest.apk` | `com.oculus.deviceidentitytest` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 1 | 1 | 0 | 0 | 0 |  |
| apps | system_ext_a | `app/ExploreApp/ExploreApp.apk` | `com.oculus.explore` | 213.0.0.108.81 | 28 | 28 |  | `fac61745dc090378` | 2 | 13 | 12 | 2 | 32 |  |
| apps | system_ext_a | `app/ExternalStorage/ExternalStorage.apk` | `com.oculus.externalstorage` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 5 | 0 | 1 | 0 | 3 |  |
| apps | system_ext_a | `app/ExtraPermissions/ExtraPermissions.apk` | `com.oculus.extrapermissions` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 2 | 0 | 0 | 0 | 5 |  |
| apps | system_ext_a | `app/FitnessTracker/FitnessTracker.apk` | `com.oculus.fitnesstracker` | 104.0.0.20.435 | 28 | 28 |  | `fac61745dc090378` | 2 | 11 | 12 | 3 | 24 |  |
| apps | system_ext_a | `app/GamingActivity/GamingActivity.apk` | `com.oculus.gamingactivity` | 119.0.0.3.162 | 28 | 28 |  | `fac61745dc090378` | 1 | 9 | 12 | 1 | 15 |  |
| apps | system_ext_a | `app/Guidebook/Guidebook.apk` | `com.oculus.guidebook` | 11.0.0.0.0 | 28 | 28 |  | `fac61745dc090378` | 0 | 0 | 0 | 0 | 8 |  |
| apps | system_ext_a | `app/Horizon/Horizon.apk` | `com.oculus.horizon` | 71.0.0.894.253 | 23 | 23 | `com.oculus.uid` | `fac61745dc090378` | 3 | 53 | 37 | 15 | 57 | 27 |
| apps | system_ext_a | `app/JavaService/JavaService.apk` | `com.oculus.javaservice` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 0 | 0 |  |
| apps | system_ext_a | `app/LineFrequencyService/LineFrequencyService.apk` | `com.oculus.linefrequencyservice` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 0 | 5 |  |
| apps | system_ext_a | `app/LogViewer/LogViewer.apk` | `com.oculus.os.logviewer` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 1 | 0 | 0 | 0 | 3 |  |
| apps | system_ext_a | `app/Metacam/Metacam.apk` | `com.oculus.metacam` | 118.0.0.251.161 | 28 | 28 |  | `fac61745dc090378` | 3 | 12 | 19 | 2 | 49 | 27 |
| apps | system_ext_a | `app/MrdsService/MrdsService.apk` | `com.oculus.mrds` | 89.0.0.0.55 | 28 | 24 |  | `dc2f834e3c194237` | 0 | 2 | 2 | 0 | 25 |  |
| apps | system_ext_a | `app/OVRMetricsTool/OVRMetricsTool.apk` | `com.oculus.ovrmonitormetricsservice` | 1.0-stub |  |  |  | `fac61745dc090378` | 0 | 0 | 0 | 0 | 2 |  |
| apps | system_ext_a | `app/OculusAppAutomationTREX/OculusAppAutomationTREX.apk` | `com.oculus.appautomation` | 9.0.0.637.596 | 23 | 23 |  | `fac61745dc090378` | 0 | 0 | 0 | 0 | 1 |  |
| apps | system_ext_a | `app/OculusQuickPromotionsService/OculusQuickPromotionsService.apk` | `com.oculus.quickpromotionservice` | 71.0.0.620.696 | 32 | 32 |  | `fac61745dc090378` | 0 | 1 | 3 | 1 | 8 |  |
| apps | system_ext_a | `app/OculusXrCvpPlugin/OculusXrCvpPlugin.apk` | `com.oculus.cvp` | 71.0.0.541.529 | 23 | 23 |  | `fac61745dc090378` | 0 | 1 | 0 | 0 | 4 |  |
| apps | system_ext_a | `app/PresenceTester/PresenceTester.apk` | `com.oculus.presence.test` |  | 25 | 25 |  | `fac61745dc090378` | 0 | 0 | 0 | 0 | 5 |  |
| apps | system_ext_a | `app/QplService/QplService.apk` | `com.oculus.qplservice` | 71.0.0.273.527 | 28 | 29 |  | `fac61745dc090378` | 0 | 1 | 0 | 1 | 2 |  |
| apps | system_ext_a | `app/SocialPlatform/SocialPlatform.apk` | `com.oculus.socialplatform` | 201.0.0.5.218 | 28 | 28 |  | `fac61745dc090378` | 6 | 24 | 30 | 10 | 43 |  |
| apps | system_ext_a | `app/Store/Store.apk` | `com.oculus.store` | 228.0.0.19.109 | 28 | 28 |  | `fac61745dc090378` | 3 | 14 | 15 | 2 | 21 |  |
| apps | system_ext_a | `app/SystemActivities/SystemActivities.apk` | `com.oculus.systemactivities` | 71.0.0.601.498 | 28 | 28 |  | `fac61745dc090378` | 1 | 0 | 0 | 0 | 0 |  |
| apps | system_ext_a | `app/SystemResource/SystemResource.apk` | `com.oculus.systemresource` | 72.0.0.602.491 | 28 | 28 |  | `fac61745dc090378` | 2 | 1 | 4 | 2 | 13 |  |
| apps | system_ext_a | `app/TV/TV.apk` | `com.oculus.tv` | 176.0.0.0.187 | 28 | 28 |  | `fac61745dc090378` | 0 | 0 | 0 | 0 | 4 |  |
| apps | system_ext_a | `app/VrCastService/VrCastService.apk` | `com.oculus.vrcast` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 1 | 2 | 0 | 0 | 5 |  |
| apps | system_ext_a | `app/VrFrameworkPackageStubs/VrFrameworkPackageStubs.apk` | `com.meta.frameworkpackagestubs` | 1.0.1 | 32 | 32 |  | `c8a2e9bccf597c2f` | 4 | 0 | 0 | 0 | 0 |  |
| apps | system_ext_a | `priv-app/AppSafety/AppSafety.apk` | `com.oculus.appsafety` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 0 | 10 | 4 | 2 | 17 |  |
| apps | system_ext_a | `priv-app/AssetDeliveryService/AssetDeliveryService.apk` | `com.oculus.assetdelivery` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 1 | 0 | 1 |  |
| apps | system_ext_a | `priv-app/Assistant/Assistant.apk` | `com.oculus.assistant` | 172.0.0.0.61 | 29 | 29 |  | `fac61745dc090378` | 7 | 24 | 16 | 5 | 50 | 27 |
| apps | system_ext_a | `priv-app/AudioLatencyTester/AudioLatencyTester.apk` | `com.oculus.audio.latencytester` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 1 | 0 | 0 | 0 | 1 |  |
| apps | system_ext_a | `priv-app/BackupTransportService/BackupTransportService.apk` | `com.oculus.backuptransportservice` | 12 | 32 | 32 |  | `a40da80a59d170ca` | 0 | 4 | 0 | 1 | 14 |  |
| apps | system_ext_a | `priv-app/BodyApiService/BodyApiService.apk` | `com.oculus.bodyapiservice` | 71.0.0.541.758 | 23 | 29 |  | `fac61745dc090378` | 1 | 2 | 0 | 1 | 10 | 27 |
| apps | system_ext_a | `priv-app/BugReportService/BugReportService.apk` | `com.oculus.bugreportservice` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 4 | 1 | 2 | 7 |  |
| apps | system_ext_a | `priv-app/CMSHeadset/CMSHeadset.apk` | `com.oculus.os.cm` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 0 | 0 | 1 | 11 |  |
| apps | system_ext_a | `priv-app/CaptionService/CaptionService.apk` | `com.oculus.captionservice` | 12 | 32 | 29 |  | `a40da80a59d170ca` | 0 | 8 | 0 | 1 | 10 |  |
| apps | system_ext_a | `priv-app/ClearActivity/ClearActivity.apk` | `com.oculus.os.clearactivity` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 1 | 0 | 0 | 0 | 1 |  |
| apps | system_ext_a | `priv-app/CompanionServer/CompanionServer.apk` | `com.oculus.companion.server` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 3 | 3 | 1 | 35 |  |
| apps | system_ext_a | `priv-app/Configuration/Configuration.apk` | `com.oculus.configuration` | 71.0.0.632.694 | 31 | 23 |  | `fac61745dc090378` | 0 | 6 | 11 | 1 | 14 |  |
| apps | system_ext_a | `priv-app/DeviceAuthServer/DeviceAuthServer.apk` | `com.oculus.deviceauthserver` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 1 | 3 |  |
| apps | system_ext_a | `priv-app/GatekeeperService/GatekeeperService.apk` | `com.oculus.gatekeeperservice` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 1 | 0 | 4 |  |
| apps | system_ext_a | `priv-app/Guardian/Guardian.apk` | `com.oculus.guardian` | 71.0.0.613.530 | 24 | 23 |  | `fac61745dc090378` | 0 | 3 | 5 | 2 | 47 | 27 |
| apps | system_ext_a | `priv-app/IdentityManagement/IdentityManagement.apk` | `com.oculus.identitymanagement.service` | 71.0.0.545.496 | 29 | 29 |  | `fac61745dc090378` | 1 | 6 | 16 | 3 | 23 |  |
| apps | system_ext_a | `priv-app/Integrity/Integrity.apk` | `com.oculus.integrity` | 71.0.0.618.530 | 28 | 28 |  | `fac61745dc090378` | 1 | 15 | 12 | 2 | 33 | 27 |
| apps | system_ext_a | `priv-app/LabService/LabService.apk` | `com.oculus.labservice` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 2 | 0 | 2 |  |
| apps | system_ext_a | `priv-app/LibraryPanelApp/LibraryPanelApp.apk` | `com.oculus.panelapp.library` | 71.0.0.614.685 | 32 | 32 |  | `fac61745dc090378` | 1 | 6 | 11 | 1 | 29 |  |
| apps | system_ext_a | `priv-app/LocationService/LocationService.apk` | `com.facebook.wearable.system.location.service` | 71.0.0.891.252 | 29 | 29 |  | `fac61745dc090378` | 0 | 12 | 12 | 1 | 30 |  |
| apps | system_ext_a | `priv-app/LocationServiceProxy/LocationServiceProxy.apk` | `com.facebook.wearable.system.location.proxy` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 0 | 8 |  |
| apps | system_ext_a | `priv-app/MagicIslandCastingService/MagicIslandCastingService.apk` | `com.oculus.magicislandcastingservice` | 2.0.0.0.567 | 29 |  |  | `fac61745dc090378` | 0 | 1 | 0 | 0 | 6 |  |
| apps | system_ext_a | `priv-app/MaintenanceBoot/MaintenanceBoot.apk` | `com.oculus.maintenanceboot` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 0 | 1 | 1 | 1 | 9 |  |
| apps | system_ext_a | `priv-app/MetaSystemUI/MetaSystemUI.apk` | `com.meta.systemui` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 0 | 2 | 0 | 0 | 8 |  |
| apps | system_ext_a | `priv-app/MicService/MicService.apk` | `com.oculus.micservice` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 1 | 1 | 5 |  |
| apps | system_ext_a | `priv-app/NotificationProxy/NotificationProxy.apk` | `com.oculus.notification_proxy` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 0 | 0 | 0 | 8 |  |
| apps | system_ext_a | `priv-app/OCMS/OCMS.apk` | `com.oculus.ocms` | 71.0.0.884.253 | 32 | 32 |  | `fac61745dc090378` | 0 | 29 | 37 | 6 | 63 | 27 |
| apps | system_ext_a | `priv-app/OSUpdater/OSUpdater.apk` | `com.oculus.updater` | 71.0.0.890.252 | 26 | 26 |  | `fac61745dc090378` | 0 | 9 | 14 | 1 | 30 |  |
| apps | system_ext_a | `priv-app/OculusFederatedComputing/OculusFederatedComputing.apk` | `com.meta.federatedcomputing.oculus` | 75.0.0.0.195 | 23 | 23 |  | `fac61745dc090378` | 1 | 14 | 13 | 1 | 18 |  |
| apps | system_ext_a | `priv-app/PCLinkService/PCLinkService.apk` | `com.meta.pclinkservice.server` | 0.0.0.0.193 | 32 | 33 |  | `fac61745dc090378` | 0 | 1 | 0 | 0 | 5 |  |
| apps | system_ext_a | `priv-app/PauldronVr/PauldronVr.apk` | `com.meta.automation.pauldron.vr` | 1.0.0.0.3 | 29 | 29 |  | `fac61745dc090378` | 0 | 0 | 0 | 0 | 20 |  |
| apps | system_ext_a | `priv-app/PreShutdownTaskService/PreShutdownTaskService.apk` | `com.oculus.preshutdowntaskservice` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 2 | 1 | 1 | 6 |  |
| apps | system_ext_a | `priv-app/Preloader/Preloader.apk` | `com.oculus.preloader` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 1 | 0 | 4 |  |
| sensors-mcu-meta-hal | system_ext_a | `priv-app/Provision/Provision.apk` | `com.android.provision` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 1 | 0 | 0 | 0 | 4 |  |
| apps | system_ext_a | `priv-app/QrCodeReader/QrCodeReader.apk` | `com.oculus.os.qrcodereader` | 12 | 32 | 32 |  | `a40da80a59d170ca` | 3 | 0 | 0 | 0 | 1 |  |
| sensors-mcu-meta-hal | system_ext_a | `priv-app/QuestManagedProvisioning/QuestManagedProvisioning.apk` | `com.android.managedprovisioning` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 12 | 5 | 5 | 0 | 39 |  |
| apps | system_ext_a | `priv-app/QuickPlayService/QuickPlayService.apk` | `com.oculus.quickplayservice` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 2 | 0 | 0 | 2 |  |
| apps | system_ext_a | `priv-app/RLTrustService/RLTrustService.apk` | `com.meta.rl.trust.service` | 2.0.0.643.723 | 32 | 32 |  | `fac61745dc090378` | 5 | 6 | 12 | 1 | 26 |  |
| apps | system_ext_a | `priv-app/ReprojectedAndroidPanelApp/ReprojectedAndroidPanelApp.apk` | `com.oculus.vrshell.desktop` | 71.0.0.602.523 | 29 | 29 |  | `fac61745dc090378` | 0 | 1 | 3 | 1 | 17 |  |
| apps | system_ext_a | `priv-app/Settings/Settings.apk` | `com.android.settings` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 282 | 15 | 20 | 11 | 105 | 27 |
| apps | system_ext_a | `priv-app/SettingsPanelApp/SettingsPanelApp.apk` | `com.oculus.panelapp.settings` | 9.0.0.655.532 | 32 | 32 |  | `fac61745dc090378` | 0 | 8 | 13 | 2 | 88 |  |
| apps | system_ext_a | `priv-app/ShellEnv/ShellEnv.apk` | `com.oculus.shellenv` | 71.0.0.605.503 | 29 | 29 |  | `fac61745dc090378` | 1 | 1 | 3 | 3 | 35 |  |
| apps | system_ext_a | `priv-app/SkyguardService/SkyguardService.apk` | `com.meta.internal.skyguard` |  | 29 | 29 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 1 | 0 | 1 | 27 |
| apps | system_ext_a | `priv-app/SpatialPersistenceService/SpatialPersistenceService.apk` | `com.facebook.spatial_persistence_service` | 71.0.0.605.527 | 28 | 29 |  | `fac61745dc090378` | 0 | 3 | 2 | 0 | 20 | 27 |
| apps | system_ext_a | `priv-app/StatsCollector/StatsCollector.apk` | `com.oculus.statscollector` | 1.0 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 5 | 1 | 1 | 6 |  |
| apps | system_ext_a | `priv-app/StorageManager/StorageManager.apk` | `com.android.storagemanager` | 12 | 32 | 32 |  | `a40da80a59d170ca` | 2 | 1 | 2 | 0 | 12 |  |
| apps | system_ext_a | `priv-app/SystemPermissions/SystemPermissions.apk` | `com.oculus.systempermissions` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 3 | 0 | 0 | 0 | 16 |  |
| apps | system_ext_a | `priv-app/SystemUX/SystemUX.apk` | `com.oculus.systemux` | 71.0.0.605.522 | 32 | 32 |  | `fac61745dc090378` | 13 | 33 | 21 | 2 | 108 | 27 |
| apps | system_ext_a | `priv-app/SystemUtilities/SystemUtilities.apk` | `com.oculus.systemutilities` | 71.0.0.608.527 | 28 | 28 |  | `fac61745dc090378` | 2 | 16 | 16 | 2 | 42 |  |
| apps | system_ext_a | `priv-app/UserServer2/UserServer2.apk` | `com.oculus.userserver2` | 70.0.0.891.958 | 25 | 25 |  | `fac61745dc090378` | 0 | 7 | 6 | 4 | 19 |  |
| apps | system_ext_a | `priv-app/VoidActivity/VoidActivity.apk` | `com.oculus.os.voidactivity` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 1 | 0 | 0 | 0 | 2 |  |
| apps | system_ext_a | `priv-app/VrAlertService/VrAlertService.apk` | `com.oculus.vralertservice` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 9 | 1 | 1 | 1 | 16 |  |
| apps | system_ext_a | `priv-app/VrBluetoothControlService/VrBluetoothControlService.apk` | `com.oculus.vrbtcontrol` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 1 | 0 | 10 |  |
| apps | system_ext_a | `priv-app/VrBtTest/VrBtTest.apk` | `com.oculus.vrbttest` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 0 | 7 |  |
| apps | system_ext_a | `priv-app/VrLockscreen/VrLockscreen.apk` | `com.oculus.os.vrlockscreen` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 2 | 1 | 1 | 1 | 12 |  |
| apps | system_ext_a | `priv-app/VrShell/VrShell.apk` | `com.oculus.vrshell` | 71.0.0.612.529 | 29 | 29 |  | `fac61745dc090378` | 41 | 52 | 21 | 7 | 115 | 27 |
| apps | system_ext_a | `priv-app/VrUsbApp/VrUsbApp.apk` | `com.oculus.os.vrusb` | 12 | 32 | 32 |  | `28bbfe4a7b97e746` | 9 | 1 | 1 | 1 | 22 |  |
| apps | system_ext_a | `priv-app/VrosConfigWriterDeprecated/VrosConfigWriterDeprecated.apk` | `com.oculus.vrosconfigwriterdeprecated` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 0 | 0 | 2 |  |
| apps | system_ext_a | `priv-app/WISInputDeviceService/WISInputDeviceService.apk` | `com.meta.wearable.emginput` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 1 | 1 | 0 | 11 |  |
| apps | system_ext_a | `priv-app/WearableInputService/WearableInputService.apk` | `com.oculus.wearableinputservice` | 80.0.0.0.0 | 26 | 26 |  | `fac61745dc090378` | 1 | 5 | 1 | 0 | 10 | 27 |
| apps | system_ext_a | `priv-app/WebSafety/WebSafety.apk` | `com.oculus.websafety` | 12 | 32 | 32 |  | `c8a2e9bccf597c2f` | 0 | 4 | 0 | 0 | 5 |  |
| sensors-mcu-meta-hal | system_ext_a | `priv-app/com.oculus.os.platform-res/com.oculus.os.platform-res.apk` | `oculus.platform` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 0 | 0 | 0 | 1 | 0 |  |
| apps | product_a | `app/ModuleMetadata/ModuleMetadata.apk` | `com.android.modulemetadata` | 12 | 28 | 28 |  | `a40da80a59d170ca` | 0 | 0 | 0 | 0 | 0 |  |
| apps | product_a | `app/Q4BMdm/Q4BMdm.apk` | `com.oculus.q4b.mdm` | 103.0.0.0.166 | 31 | 31 |  | `fac61745dc090378` | 3 | 21 | 24 | 1 | 30 | 27 |
| apps | product_a | `app/SystemWebView/SystemWebView.apk` | `com.android.webview` | 126.0.6478.122 | 26 | 34 |  | `fac61745dc090378` | 3 | 52 | 0 | 4 | 6 | 27 |
| apps | product_a | `overlay/PermissionControllerRRO.apk` | `com.oculus.permissioncontroller.rro` | 12 | 32 | 32 |  | `a40da80a59d170ca` | 0 | 0 | 0 | 0 | 0 |  |
| apps | product_a | `overlay/PlatformRRO.apk` | `com.meta.android.rro` | 12 | 32 | 32 |  | `a40da80a59d170ca` | 0 | 0 | 0 | 0 | 0 |  |
| apps | product_a | `priv-app/BugReportUploaderService/BugReportUploaderService.apk` | `com.oculus.bugreportuploader` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 3 | 1 | 1 | 1 | 4 |  |
| apps | product_a | `priv-app/KioskHome/KioskHome.apk` | `com.oculus.panelapp.kiosk` | 4.0.0.0.327 | 32 | 35 |  | `fac61745dc090378` | 1 | 2 | 2 | 0 | 36 |  |
| apps | product_a | `priv-app/OemConfig/OemConfig.apk` | `com.oculus.oemconfig` | 0.0.0.0.1 | 31 | 31 |  | `fac61745dc090378` | 0 | 1 | 0 | 0 | 1 |  |
| apps | product_a | `priv-app/OneTimeInitializer/OneTimeInitializer.apk` | `com.android.onetimeinitializer` | 12 | 32 | 32 |  | `a40da80a59d170ca` | 0 | 0 | 1 | 0 | 3 |  |
| apps | product_a | `priv-app/Q4BService/Q4BService.apk` | `com.oculus.q4bservice` | 71.0.0.896.252 | 31 | 31 |  | `fac61745dc090378` | 4 | 17 | 22 | 5 | 40 | 27 |
| apps | product_a | `priv-app/SpgoUploader/SpgoUploader.apk` | `com.oculus.spgouploader` | 12 | 32 | 32 | `android.uid.system` | `c8a2e9bccf597c2f` | 3 | 1 | 1 | 0 | 2 |  |
| apps | vendor_a | `app/TimeService/TimeService.apk` | `com.qualcomm.timeservice` | 12 | 31 | 33 |  | `c8a2e9bccf597c2f` | 0 | 0 | 1 | 0 | 1 |  |

<!-- generated: apps end -->

## 缺口

- 标记为 inferred 的 125 个 APK 行，其功能描述仅来自包名和组件类名。未反编译或解析任何 dex 代码、resources.arsc 或 assets，SPGO、OCMS、CM、MRDS、QPL、CVP、Q4B 和 NUX 等缩写也未根据内容展开。
- 下方生成的索引给出了每个 `.apk` 文件的身份信息、版本、SDK 级别、共享 UID、签名摘要、组件数量和权限数量。已导出组件、Intent 过滤器和声明的权限在每个文件的 `data/apks/<name>.json` 中；角色说明和详细结论（d01 至 d15）仅涵盖 15 个应用，其中 9 个属于第 27 页的 22 个应用。
- Settings.apk 的 vdex 第一个计数字段读数为 4，而该 APK 有两个 classes*.dex 条目。其含义未得到解决，仅报告了原始头部字段。叠加层 resourcesMap 的值（type0x1:7f020000 和 7f020001）未被解码。
- init 引用仅按包名和 priv-app/app 路径模式进行了搜索。127 个非深入分析包名已在 etc 目录树（init 和 SELinux 文件所在位置）中检查，但未在 .rc 文件中逐一搜索。
- 未找到 /system_ext/app 或 /system_ext/priv-app 的 SELinux 标签行；因此 system_ext 应用仅由通用的 system_file 规则覆盖，设备上实际使用的标签未被观察到。
- 每个 `.apk` 文件的签名证书摘要都已提取，并列在下方的索引中。未逐个文件与 mac_permissions 文件中的签名者条目（结论 c14）比对。'Android Debug' 主体被报告为已观察到，未作评估。
- 有六个 APK 没有 odex 或 vdex（两个叠加层和另外四个）；它们是否被有意不做预编译，尚未确定。
- 本组不包含固件、引导程序或硬件配置文件。11 个符号链接目标位于 system_ext_a 的 lib64 分区，该分区不在本组范围内；其内容未被检查。
- 未观察安装状态、运行时行为、包管理器数据库内容和设备端配置；仅分析了只读分区的内容。根据任务规则，安全评估已被排除。
