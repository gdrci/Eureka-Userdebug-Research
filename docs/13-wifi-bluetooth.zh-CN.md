# 13 - Wi-Fi 与蓝牙

[English](13-wifi-bluetooth.md)

## 摘要

wifi-bluetooth 组是一台设备的 Wi-Fi 与蓝牙协议栈，该设备搭载高通 QCA6490 组合芯片（QCA6490 出现在蓝牙补丁构建字符串中，也出现在 SELinux 文件上下文中所命名的 cnss-qca6490 sysfs 节点中）。厂商 HAL 服务暴露 IWifi（HIDL 1.0 至 1.5）、ISupplicant（1.0 至 1.4）、IHostapd（1.0 至 1.3）和 IBluetoothHci（1.0）。这些服务由 wpa_supplicant 与 hostapd 2.10-devel-12 守护进程、高通 WLAN 服务 cnss-daemon 与 cnss_diag，以及以单记录 TLV 容器存储的高通蓝牙 rampatch 与 NVM 数据块支撑。system 分区包含 wificond 守护进程、带有 JNI 库的蓝牙协议栈 libbluetooth.so、Wi-Fi APEX com.android.wifi（framework-wifi.jar、service-wifi.jar、ServiceWifiResources.apk），以及特权权限与功能标志 XML。odm 分区上有 Oculus 厂商 Wi-Fi HAL（vendor.oculus.hardware.wifi@1.0），system_ext 分区上有一个遥测守护进程。配置以 wpa_supplicant 与驱动覆盖文本文件、ICM 通道配置、蓝牙协议栈 trace 配置以及设备 ID 文件的形式提供。

角色表对本组中的 104 个文件各有一行（91 个 verified，13 个 observed，0 个 inferred，0 个 unverified）。该表位于 `data/subsystems/wifi-bluetooth/roles.tsv`。

## 已验证的声明

- **WB-01** (verified)：该组包含 104 行：103 个普通文件和 1 个符号链接。各分区计数为 odm_a 3、product_a 7、system_a 31、system_ext_a 5 以及 vendor_a 58。
- **WB-03** (verified)：组内没有两个文件共享同一个 sha256 摘要。大小相同的配对内容各不相同：system_a 与 vendor_a 中的 android.hardware.wifi@1.0.so（均为 1018168 字节），以及 odm_a 与 system_ext_a 中的 vendor.oculus.hardware.wifi@1.0.so（均为 171728 字节）。
- **WB-04** (verified)：组内的 64 位 ELF 对象为 AArch64。lib/ 中的对象为 32 位 ARM EABI5，lib64/ 与 bin/ 中则是 64 位的那一组。
- **WB-06** (verified)：组内可执行文件的 SELinux file_contexts 标签如下：/vendor/bin/hw/wpa_supplicant 为 hal_wifi_supplicant_default_exec，hostapd 为 hal_wifi_hostapd_default_exec，Wi-Fi HAL 服务为 hal_wifi_default_exec，蓝牙 HAL 服务为 hal_bluetooth_default_exec，cnss-daemon 与 cnss_diag 为 vendor_wcnss_service_exec，/system/bin/wificond 为 wificond_exec，/system_ext/bin/wifitelemetry 为 wifitelemetry_exec。数据路径标签涵盖 /data/vendor/bluetooth、/data/vendor/wifi/hostapd 和 /data/misc/bluetooth。
- **WB-07** (verified)：HIDL hwservice 标签将 IWifi 绑定到 hal_wifi_hwservice，ISupplicant 绑定到 hal_wifi_supplicant_hwservice，IHostapd 绑定到 hal_wifi_hostapd_hwservice，IKeystore 绑定到 system_wifi_keystore_hwservice，IBluetoothHci 绑定到 hal_bluetooth_hwservice。Oculus 接口 IWifi 在 vendor 上下文文件中标记为 hal_oculus_wifi_hwservice。
- **WB-08** (verified)：vendor VINTF 清单声明了以下版本的 Wi-Fi HAL 接口：IWifi 1.5（default 实例）、ISupplicant 1.4（default）和 IHostapd 1.3（default），均通过 hwbinder 提供。
- **WB-09** (verified)：product 分区声明了七个功能标志：android.hardware.bluetooth、android.hardware.bluetooth_le、android.hardware.wifi、android.hardware.wifi.aware、android.hardware.wifi.direct、android.hardware.wifi.rtt 和 oculus.software.wifi.sleep（最后一个用于 Wi-Fi 休眠策略）。
- **WB-10** (verified)：com.android.wifi.capex 是一个 zip 文件，包含 original_apex（本身也是 zip）和 AndroidManifest。其内部负载是一个 ext2 镜像（apex_payload.img），包含九个文件：javalib/framework-wifi.jar、javalib/service-wifi.jar、priv-app/ServiceWifiResources/ServiceWifiResources.apk、app/OsuLogin/OsuLogin.apk、etc/security/cacerts_wfa 下的三个证书、etc/classpaths/bootclasspath.pb，以及 apex_manifest.pb（包名 com.android.wifi）。
- **WB-11** (verified)：bt_firmware/image 中的每个蓝牙固件或 NVM 文件都是一条 TLV 记录：第 0 字节为记录类型（.tlv rampatch 文件为 0x01，.bin NVM 文件为 0x02），第 1 至 3 字节为小端序长度，六个文件的文件大小都等于该长度加 4。
- **WB-12** (verified)：hpbtfw21.tlv 携带构建字符串 BTFW.HSP.2.1.0-00632-PATCHZ-1.63111.4，构建日期为 Mar 14 2024，并包含一行 'Patch Release PF=QCA6490ROM= 0201'。hpbtfw20.tlv 携带 BTFW.HSP.2.0.1-00252-HSP_PATCH-1，以及 'PF=QCA6490ROM= 0200'。
- **WB-13** (observed)：hpbtfw21.tlv 在 4 字节记录头之后的前几个字节，是小端序字 0x00024fbc 和 0x00024f98，二者都位于文件内部。字节 70 b5 82 b0 位于偏移 0x28 处。
- **WB-14** (verified)：hpnv21.bin 是一条类型为 0x02、长度为 6606 的记录，在偏移 0x10 处有 ASCII 标记 EVT2。hpnv21g.bin 长度为 6446，具有相同标记。hpnv20.bin 长度为 6315，且该偏移处没有 EVT2 标记。
- **WB-15** (observed)：蓝牙 HAL 实现（lib64/hw/android.hardware.bluetooth@1.0-impl-qti.so）包含多个芯片系列的固件与 NVM 文件名字面量，包括 apbtfw10.tlv、cmbtfw13.tlv、crbtfw32.tlv、gnbtfw20.tlv、hmtbtfw20.tlv、hpbtfw21.tlv、hpnv21xm.bin 和 msnv12.bin。它还包含模式字符串 (btfw_(([a-z1-9][a-z1-9])|[a-z1-9])) 以及位于 /vendor/bt_firmware/image/ 与 /vendor/firmware/ 下的回退文件名 tpbtfw00.tlv 和 tpnv00.bin。
- **WB-17** (observed)：芯片身份为高通 QCA6490：厂商 SELinux 文件上下文为位于 b0000000.qcom,cnss-qca6490 平台设备下的一个 sysfs 节点加了标签，蓝牙补丁构建中也命名了 PF=QCA6490ROM。
- **WB-18** (verified)：wpa_supplicant 是经过剥离的 64 位 AArch64 PIE，解释器为 /system/bin/linker64。其 DT_NEEDED 条目包括 ISupplicant 库 android.hardware.wifi.supplicant@1.0 至 @1.4、libkeystore-wifi-hidl.so、libkeystore-engine-wifi-hidl.so、libnl.so、libcrypto.so、libssl.so、libhidlbase.so 和 libbase.so。
- **WB-19** (observed)：wpa_supplicant 包含字面版本字符串 'wpa_supplicant v2.10-devel-12'，并以独立字面量的形式包含这些配置键：update_config、eapol_version、ap_scan、fast_reauth、p2p_add_cli_chan、sae_pwe、disable_scan_offload、p2p_disabled、tdls_external_control、wowlan_triggers、bss_max_count、interworking、p2p_no_group_iface、persistent_reconnect 和 p2p_go_he。键 pmf 和 oce 仅以子串形式出现。
- **WB-20** (verified)：wpa_supplicant 服务定义于 vendor_a/etc/init/hw/init.anorak.rc 中，参数为 -O/data/vendor/wifi/wpa/sockets、-puse_p2p_group_interface=1 和 -g@android:wpa_wlan0。它运行在 class main 中，拥有一个 wpa_wlan0 dgram 套接字（模式 660，wifi:wifi），并且同时处于 disabled 与 oneshot 状态。该二进制文件的标签为 hal_wifi_supplicant_default_exec。
- **WB-21** (verified)：hostapd 是经过剥离的 64 位 AArch64 PIE，解释器为 /system/bin/linker64。其 DT_NEEDED 条目包括 IHostapd 库 android.hardware.wifi.hostapd@1.0 至 @1.3、libnl.so、libssl.so、libcrypto.so 和 libhidlbase.so。其横幅为 'hostapd v%s'，版本字符串为 2.10-devel-12。
- **WB-22** (observed)：hostapd 包含默认控制接口路径 /data/vendor/wifi/hostapd/ctrl、用法行 'usage: hostapd [-hdBKtv] [-P <PID file>] [-e <entropy file>]'，以及键 ctrl_interface、ctrl_interface_group 和 dump_file。dump_file 键被报告为已弃用。
- **WB-23** (verified)：hostapd 从 vendor_a/etc/init/hostapd.android.rc 启动。post-fs-data 步骤创建 /data/vendor/wifi/hostapd 及其 sockets 子目录，所有者为 wifi:wifi。该服务以用户 wifi、NET_ADMIN 与 NET_RAW 能力在 class main 中运行 /vendor/bin/hw/hostapd，并且同时处于 disabled 与 oneshot 状态。其 SELinux 标签为 hal_wifi_hostapd_default_exec，/data/vendor/wifi/hostapd 的标签为 hostapd_data_file。
- **WB-24** (verified)：android.hardware.wifi@1.0-service 是依赖 android.hardware.wifi@1.0 至 @1.5、libwifi-hal.so、libwifi-system-iface.so、libxml2.so、libnl.so 和 libhidlbase.so 的 64 位 AArch64 PIE。其 init 服务 vendor.wifi_hal_legacy 为接口 @1.0 至 @1.5 注册 IWifi default 实例。它以用户 wifi、组 wifi gps 在 class hal 中运行，具有 NET_ADMIN、NET_RAW 和 SYS_MODULE 能力。其 SELinux 标签为 hal_wifi_default_exec。
- **WB-25** (observed)：Wi-Fi HAL 服务二进制文件包含以下日志与类字符串：'Wifi HAL started'、'Wifi HAL stopped'、'WifiVendorHal'、'No active wlan interfaces in use! Using default'，以及 'Check failed: size >= 0 && size <= kMaxWifiUsableChannels'。
- **WB-26** (verified)：64 位 android.hardware.wifi@1.5.so 的 SONAME 为 android.hardware.wifi@1.5.so。其动态符号携带版本标记 V1_0（304）、V1_1（9）、V1_2（31）、V1_3（14）、V1_4（26）和 V1_5（766）。它依赖 @1.0 至 @1.4 库。IWifi、IWifiIface、IWifiApIface 和 IWifiNanIface 类出现在修饰名中。
- **WB-27** (verified)：64 位 android.hardware.wifi.supplicant@1.4.so 的 SONAME 为 android.hardware.wifi.supplicant@1.4.so。它携带版本标记 V1_0（376）、V1_1（19）、V1_2（26）、V1_3（25）和 V1_4（921）。其修饰名包括 ISupplicant、ISupplicantIface 和 ISupplicantNetwork。hwservice 上下文将 android.hardware.wifi.supplicant::ISupplicant 绑定到 hal_wifi_supplicant_hwservice（plat_hwservice_contexts 第 76 行）。
- **WB-28** (verified)：libwifi-hal-qcom.so 是 64 位共享对象，SONAME 为 libwifi-hal-qcom.so，共有 571 个已定义的动态符号，其中 73 个以 wifi_ 开头（例如 wifi_get_roaming_capabilities）。它依赖 libnetutils.so、liblog.so、libcld80211.so、libwifi-hal-ctrl.so、libcrypto.so 和 libnl.so。其 C++ 类包括 WifiVendorCommand、LLStatsCommand、GScanCommand 和 NanCommand。
- **WB-29** (observed)：libwifi-hal-qcom.so 包含 QCA_WLAN_VENDOR_ATTR_* 属性名，它会检查这些属性是否存在，例如 QCA_WLAN_VENDOR_ATTR_FEATURE_SET、QCA_WLAN_VENDOR_ATTR_CHANNEL、QCA_WLAN_VENDOR_ATTR_DRV_INFO_BUS_SIZE 和 QCA_WLAN_VENDOR_ATTR_GSCAN_RESULTS_CAPABILITIES_MAX_HOTLIST_BSSIDS。它还包含日志文本 '%s: Error wifi_handle NULL'。
- **WB-30** (verified)：wificond 是位于 /system/bin/wificond 的经过剥离的 64 位 AArch64 PIE。其 init 服务在 class main 中以用户 wifi、组 wifi net_raw net_admin 运行，具有 NET_RAW 和 NET_ADMIN 能力。其 DT_NEEDED 条目包括 libwifi-system-iface.so、android.system.wifi.keystore@1.0.so、android.system.keystore2-V1-ndk_platform.so、libbinder_ndk.so、libminijail.so 和 libcrypto.so。其 SELinux 标签为 wificond_exec。
- **WB-31** (observed)：wificond 记录了 'Driver is in a bad state, restarting wificond'、'New wificond event callback registered' 和 'Failed to find registered wificond event callback'。其 binder 接口引用了 keystore AIDL 类型 IKeystoreService、IKeystoreOperation 和 IKeystoreSecurityLevel。
- **WB-32** (verified)：libbluetooth.so（lib64）是经过剥离的 64 位共享对象，SONAME 为 libbluetooth.so，且没有 .symtab 节。其 DT_NEEDED 条目包括 android.hardware.bluetooth@1.0 和 @1.1、android.hardware.bluetooth.audio@2.0 和 @2.1、android.hardware.bluetooth.a2dp@1.0、android.system.suspend@1.0、libchrome.so、libgrpc++.so、libprotobuf-cpp-lite.so、libflatbuffers-cpp.so、libaaudio.so 和 libfmq.so。
- **WB-34** (verified)：libbluetooth.so 中包含字面路径 /etc/bluetooth/bt_stack.conf，以及字面覆盖路径 /odm/etc/bluetooth/bt_did_override.conf。协议栈键 TraceConf、TRC_BTM、TRC_HCI、TRC_L2CAP、TRC_SMP、TRC_BTAPP、PTS_SecurePairOnly、PTS_SmpOptions 和 LoggingVModule 均为字面量。设备 ID 键 primaryRecord、vendorIdSource、productId、clientExecutableURL、serviceDescription 和 documentationURL 均为字面量。基础路径 /etc/bluetooth/bt_did.conf 不以字面量形式出现，因此其加载途径未能确定。
- **WB-35** (verified)：蓝牙 HAL 实现（lib64/hw/android.hardware.bluetooth@1.0-impl-qti.so）是 64 位共享对象，SONAME 为 android.hardware.bluetooth@1.0-impl-qti.so，定义了 821 个动态符号。它导出 HIDL_FETCH_IBluetoothHci，并实现 android::hardware::bluetooth::V1_0::implementation::BluetoothHci，同时包含 AsyncFdWatcher 与 BluetoothAddress 辅助类。其 DT_NEEDED 条目包括 libdiag.so、libqmi_cci.so、libbtnv.so、libsoc_helper.so、libbtatomreport.so 和 android.hardware.bluetooth@1.0.so。
- **WB-36** (verified)：android.hardware.bluetooth@1.0-service-qti 是 64 位 PIE，依赖 android.hardware.bluetooth@1.0.so、libhidlbase.so、libutils.so 和 libcutils.so。其 init 服务 vendor.bluetooth-1-0-qti 在 class hal 中以用户 bluetooth、组 bluetooth system wakelock oem_2901 net_raw 运行，ioprio 为 rt 4，能力为 BLOCK_SUSPEND、NET_ADMIN 和 SYS_NICE。其 SELinux 标签为 hal_bluetooth_default_exec。
- **WB-39** (verified)：WCNSS_qcom_cfg.ini 是 WLAN 驱动的文本覆盖文件，包含 99 行 key=value 与 14 行注释。其第一条注释为 'This file allows user to override the factory defaults for the WLAN Driver'。键包括 gDot11Mode=0、gChannelBondingMode5GHz=1、gEnableSelfRecovery=1、RoamRssiDiff=5 和 gSoftApMaxPeers=10。gActiveMaxChannelTime 出现在第 22 行（40）和第 101 行（60），gEnableDFSMasterCap 出现在第 12 行和第 119 行。组内没有任何二进制文件包含该文件名。
- **WB-40** (verified)：cnss-daemon 是经过剥离的 64 位 AArch64 PIE，依赖 libqmi_cci.so、libqmi_common_so.so、libcld80211.so、libperipheral_client.so、libmdmdetect.so、libnl.so 和 libcutils.so。其 init 服务在 init.anorak.rc 中定义为 /system/vendor/bin/cnss-daemon -n -l，在 class late_start 中以用户 system、组 system inet net_admin wifi 运行，具有 NET_ADMIN 能力。/vendor/bin/cnss-daemon 的 SELinux 标签为 vendor_wcnss_service_exec。
- **WB-41** (observed)：cnss-daemon 包含平台 QMI 消息 'Failed to init Platform QMI client' 和 'CNSS Platform QMI disconnected: %d'，校准路径模式 /data/vendor/wifi/wlfw_cal_%02d.bin，/data/vendor/wifi/iotap_ps.bin，以及本地套接字 /data/vendor/wifi/sockets/cnss_user_client 和 cnss_user_server。它接受 -n（不以守护进程方式运行）和 -l（输出到 logcat）。
- **WB-43** (verified)：设备 init 片段写入 WLAN 电源与恢复节点：/sys/kernel/cnss/charger_mode（值为 1）、/sys/kernel/cnss/fs_ready（值为 1）、/sys/kernel/cnss/recovery（值为 1），以及在 sys.shutdown.requested 时写入的 /sys/kernel/shutdown_wlan/shutdown。它还创建 Wi-Fi 与蓝牙数据目录以及 wpa 套接字目录。
- **WB-44** (verified)：odm 上的 Oculus Wi-Fi HAL（odm_a/bin/hw/vendor.oculus.hardware.wifi@1.0-service，64 位 AArch64）依赖 android.hardware.wifi.hostapd@1.1、android.hardware.wifi.supplicant@1.0、vendor.oculus.hardware.wifi@1.0.so、libvrsensors-hidlwrapper.so、libwpa_client.so 和 libnl.so。其 init 服务 vendor.oculus.wifi-hal-1-0 在 class hal 中以用户 system 为 vendor.oculus.hardware.wifi@1.0::IWifi 注册 default 实例。其 SELinux 标签为：二进制文件 hal_oculus_wifi_default_exec，接口 hal_oculus_wifi_hwservice。
- **WB-45** (verified)：wifitelemetry 是位于 /system_ext/bin/wifitelemetry 的 64 位 AArch64 守护进程。其 init 服务在 class late_start 中以用户 system 运行，组为 system wifi inet net_admin，restart_period 为 1800。它依赖 libwifitelemetry.so、libnativewakelock.so、libosutils.so 和 libbinder.so，其 SELinux 标签为 wifitelemetry_exec。
- **WB-47** (observed)：wifi_cli（system_ext_a/bin/wifi_cli）是 64 位命令行工具，通过名称 dlopen 一个 OSSDK Wi-Fi 模块。其命令包括 createWifiHandler、destroyWifiHandler、setStreamClassV4、setStreamClassV6、clearStreamClass 和 dumpInfo。其用法行列出了 stream-class 命令的参数。该模块本身不在本组中。
- **WB-48** (observed)：libwpa_drv_oem_hmd.so 是供 wpa_supplicant 使用的驱动封装，会发送 OEM 厂商命令：OEM_CMD_SET_CUSTOM_TX_POWER、OEM_CMD_SET_POWER_SURGE、OEM_CMD_GET_TX_DUTY_CYCLE、OEM_CMD_GET_TX_DUTY_CYCLE_STATS、OEM_CMD_GET_CUSTOM_TX_POWER 和 OEM_CMD_GET_POWER_SURGE，并报告 set_ac_config、set_burst_duration 和 get_power_surge_config 的分配失败。该库在 lib/ 中有 32 位副本，在 lib64/ 中有 64 位副本。
- **WB-51** (verified)：icm.conf 配置了一个无线电（id_0_radio_ifname=phy0、num_radios=1、walflag=cfg、daemon=0、server_mode=0）。其接口 id_0 为频段 0（2.4 GHz）上的 softap0，PHY 规格为 7（11NG），信道 6；其 id_1 位于频段 1（5 GHz），PHY 规格为 6（11NA），HT40PLUS 宽度为 2，信道 44。p2p0 和 ath2 接口名被注释掉了。
- **WB-52** (verified)：wpa_supplicant 配置片段设置了以下键：基础文件 vendor_a/etc/wifi/wpa_supplicant.conf（update_config=1、eapol_version=1、ap_scan=1、fast_reauth=1、pmf=1、p2p_add_cli_chan=1、oce=1、sae_pwe=2）；wpa_supplicant_overlay.conf（disable_scan_offload=1、p2p_disabled=1、tdls_external_control=1、wowlan_triggers=magic_pkt、bss_max_count=400、interworking=1）；p2p_supplicant_overlay.conf（disable_scan_offload=1、p2p_no_group_iface=1、persistent_reconnect=1、bss_max_count=400、p2p_go_he=1）。
- **WB-53** (verified)：蓝牙音频策略文件 etc/bluetooth_audio_policy_configuration_7_0.xml 声明了模块 'bluetooth'，halVersion 为 2.0，混合端口为 'a2dp output'、'hearing aid output' 和 'le audio output'，设备端口包括 A2DP（BT A2DP Out、Headphones、Speaker）、BT Hearing Aid Out，以及 BLE Headset Out 和 BLE Speaker Out。

## 修正措辞的声明

下方结论以数据所支持的措辞重新表述。

- **WB-02** (verified)：全部 104 个输入行都位于其挂载路径上。103 个普通文件的大小与 sha256 与记录一致（ok=103 bad=0）。唯一的符号链接 system_a/system/app/Bluetooth/lib/arm64/libbluetooth_jni.so 存在，其目标 /system/lib64/libbluetooth_jni.so 在 system_a 挂载中存在。
- **WB-05** (verified)：组内的 init 服务定义位于：vendor_a/etc/init/hw/init.anorak.rc（wpa_supplicant、cnss-daemon）、vendor_a/etc/init/hw/init.anorak.debug.rc（vendor.cnss_diag）、vendor_a/etc/init/android.hardware.wifi@1.0-service.rc（vendor.wifi_hal_legacy）、vendor_a/etc/init/android.hardware.bluetooth@1.0-service-qti.rc（vendor.bluetooth-1-0-qti）、vendor_a/etc/init/hostapd.android.rc（hostapd）、system_a/system/etc/init/wificond.rc（wificond）、system_ext_a/etc/init/wifitelemetry.rc（wifitelemetry）以及 odm_a/etc/init/vendor.oculus.hardware.wifi@1.0-service.rc（vendor.oculus.wifi-hal-1-0）。system_a/system/etc/init/wifi.rc 未定义任何服务，它只包含 restorecon 与内核 tracing 步骤。
- **WB-16** (observed)：蓝牙 HAL 实现包含以下 persist 与 ro 属性的字面名称：persist.vendor.bluetooth.alt_path_for_fw、persist.vendor.bluetooth.patch_config、persist.vendor.bluetooth.modem_nv_support、persist.vendor.bluetooth.rx_delay、persist.vendor.qcom.bluetooth.soc、ro.vendor.bt.bdaddr_path 和 ro.vendor.bluetooth.wipower。它还包含格式字符串 'Downloading NVM TAG %d' 和 'NVM_TAG_ERROR (0x%x) detected!!!'。这些字符串并未显示其中哪些属性在运行时被读取。
- **WB-33** (verified)：libbluetooth.so 将 bluetoothInterface 导出为动态符号。字面字符串 btif_a2dp_audio_interface_init、btif_a2dp_audio_interface_deinit、btif_a2dp_audio_interface_start_session、btif_a2dp_audio_interface_end_session、btif_a2dp_audio_interface_restart_session、btif_a2dp_audio_on_started、btif_a2dp_audio_on_stopped 和 btif_a2dp_audio_on_suspended 出现在该二进制文件中，但不在其动态符号表中。它有 695 个已定义的修饰 C++ 名称，包含 bt、Bt、btif 或 bluetooth，并包含 Rust cxxbridge 导出，例如 bluetooth$shim$rust$cxxbridge1$controller_supports_connected_iso_stream_central。
- **WB-37** (verified)：bt_stack.conf 设置 TraceConf=true，并将 19 个 TRC_* 键（从 TRC_BTM 到 TRC_HID_DEV）设为级别 2。其 PTS_* 测试键全部被注释掉（5 行以 '#PTS_' 开头）。除 TraceConf 和 TRC_* 键外，不存在其他激活的键。
- **WB-38** (verified)：两个设备 ID 文件定义了 DID1：bt_did.conf（system）中 primaryRecord=true、productId=0x1200、version=0x1436；bt_did_override.conf（odm）中 primaryRecord=true、vendorId=0x01AB、productId=0x0504、version=0x0100。两个文件中 DID2 与 DID3 的节标题都存在且未注释，但其下的每一行键都被注释掉了。
- **WB-46** (observed)：libwifitelemetry.so 导出 OVR::OS::nl80211::Parsers::station、OVR::OS::nl80211::EventListener::subscribeToScanMulticast、OVR::OS::nl80211::Connector::getP2pIfaceName 和 OVR::OS::WifiHalConnector::getLinkLayerStat。字符串 oculus_mobile_wifi_telemetry 和 'Failed to create telemetryHandler!' 位于 wifitelemetry 守护进程二进制文件（/system_ext/bin/wifitelemetry）中，而不在 libwifitelemetry.so 中。
- **WB-49** (observed)：libwifi-system-iface.so 导出 android::wifi_system::InterfaceTool 的方法，用于接口的 up/down 状态（GetUpState、SetUpState、SetWifiUpState）、MAC 地址设置（SetMacAddress）以及出厂 MAC 查询（GetFactoryMacAddress），还有网桥的创建、删除、添加与移除（createBridge、deleteBridge、addIfaceToBridge、removeIfaceFromBridge）。libbluetooth_jni.so 通过 android::register_com_android_bluetooth_* 函数（例如 sdp、hearing_aid 和 a2dp）注册 com.android.bluetooth 原生方法。它定义了 43 个名称包含 btgatt 的动态符号（即 btgatt 客户端与服务器回调），但并不导入它们。libwifirpc.so 使用 nng PAIR、PUB、REP、REQ 和 SUB 套接字，并引用 JNI 类 WifiRpc。
- **WB-50** (verified)：vendor_a 中的 Wi-Fi 与蓝牙 lib64 辅助库是 ARM aarch64 共享对象，大小为 15 KB 至 514 KB。libwifi-hal.so（514,040 字节）导出 77 个 wifi_* C 名称，例如 wifi_get_valid_channels、wifi_select_tx_power_scenario、wifi_set_link_stats 和 wifi_cleanup。libwpa_client.so（28,248 字节）导出 wpa_ctrl_open、wpa_ctrl_request、wpa_ctrl_recv、wpa_ctrl_get_fd 和 wpa_ctrl_detach。libkeystore-wifi-hidl.so（15,224 字节）导出 keystore_get，libkeystore-engine-wifi-hidl.so（46,984 字节）导出 EVP_PKEY_from_keystore。
- **WB-54** (verified)：system 分区中该组有四个非 ELF 对象：system_a/system/apex/com.android.wifi.capex（Wi-Fi APEX 包）、system_a/system/etc/bluetooth/bt_stack.conf 和 system_a/system/etc/bluetooth/bt_did.conf（蓝牙协议栈与设备 ID 配置），以及 system_a/system/etc/permissions/com.oculus.vrbluetooth.xml，后者将 android.permission.BLUETOOTH_PRIVILEGED 作为 privapp 权限授予包 com.oculus.vrbluetooth。APEX 清单中的包名为 com.android.wifi。

## 已排除的声明

下方声明已排除；每条都给出了原因。

| ID | 声明 | 原因 |
|---|---|---|
| WB-42 | cnss_diag 以服务 vendor.cnss_diag 运行 /system/vendor/bin/cnss_diag -q -f -b 128 -t HELIUM，位于 class main 中，用户为 system，组中包括 diag。It is dis | 该服务已禁用，在 class main 中以用户 system 运行，组中包括 diag，并带有标签 vendor_wcnss_service_exec（vendor 第 411 行）。启动条件有误。The rc file has two  |

## 缺口

- 已剥离符号的可执行文件与库（wpa_supplicant、hostapd、wificond、libbluetooth.so、cnss-daemon、HAL 服务以及蓝牙 JNI 库）仅通过头部、动态表、导出名称和字符串进行了检查。其运行时控制流尚未确定。
- hpnv20g.bin 和 hpnv21g.bin 中后缀 'g' 的含义，以及蓝牙 HAL 中命名的 'xm' 变体的含义，均未确定。本组中仅存在 g 文件。
- 蓝牙固件与 NVM 数据主体除 4 字节记录头和 EVT2 标记外均未解码。HAL 记录的 NVM 标签结构（NVM TAG 消息）未被解析，hpbtfw20.tlv 也未与 hpbtfw21.tlv 进行比较，仅比较了头部与构建字符串。
- rampatch 负载（偏移 0x28 处的字节 70 b5 82 b0）的 Thumb 代码解释未作为代码进行验证。
- 基础设备 ID 文件 /etc/bluetooth/bt_did.conf 不以字面量形式出现在 libbluetooth.so 中，因此加载该文件并合并 odm 覆盖的机制未确定。
- 未确定 WCNSS_qcom_cfg.ini 的使用者。组内没有任何二进制文件包含该文件名，因此推测读取者为 WLAN 内核驱动或本组之外的组件，且未分析 vendor_dlkm 或 odm_dlkm 中的内核模块。
- wifi_cli 通过 dlopen 加载的 OSSDK Wi-Fi 模块不在本组中，因此其名称与行为均未确定。
- libwifi-hal-ctrl.so（32 位与 64 位）的用途仅根据其五个导出项和有限的导入项推断得出。其导出名称未逐一列出。
- SELinux 策略仅在文本上下文文件和 CIL 文件中搜索。编译后的 precompiled_sepolicy 二进制文件未被解析，因此除标签外，未列举这些域的 allow 规则。
- init 中对 /system/vendor/bin/cnss-daemon 的引用，与 /vendor/bin/cnss-daemon 的 SELinux file_contexts 规则未经核对。未验证本设备上 /system/vendor 是否为指向 /vendor 的符号链接。
- system_a/system/lib 与 vendor_a/lib 中的 32 位库仅验证了 ELF 类别、SONAME 和 NEEDED 条目。其动态导出项未逐一列举。
- 在 12 个 Wi-Fi APEX 负载文件中，仅检查了文件列表。framework-wifi.jar、service-wifi.jar、ServiceWifiResources.apk 和 OsuLogin.apk 的内容未经检查。
- 未检查 VINTF 清单中蓝牙 HAL 的 HIDL 版本，仅读取了 Wi-Fi 清单。
- DT_NEEDED 条目中指向本组之外的依赖项（libcld80211.so、libnl.so、libqmi_cci.so、libdiag.so、libbtnv.so、libsoc_helper.so、libvrsensors-hidlwrapper.so 等）未被解析为具体文件，因此未描述其角色。
- roles.tsv 中 13 个 'observed' 行各自被赋予的角色，依据的是字符串或导出名称，未经结构性检查。'g' 变体文件、Oculus 遥测库和 wifi_cli 工具均在此集合中。
