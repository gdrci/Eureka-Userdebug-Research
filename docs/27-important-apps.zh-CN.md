# 27 - 重要应用

[English](27-important-apps.md)

本页介绍镜像中的 22 个应用：设置、蓝牙、浏览器、助手、主页、采集、系统界面、VR 运行时、身体与眼动输入、可穿戴输入、首次设置、系统 WebView、设备管理、应用安装、Guardian 边界服务、滥用举报、电话和 Skyguard 服务。这些应用是从 5,685 条清单（`data/inventory/files.tsv`）中人工挑选的，用于覆盖面向用户和系统的功能。每一节的事实来自清单文件（manifest）和 dex 文件。角色描述是根据类名和组件名的读法给出的；超出清单内容的说明标注为“推断”。

每个应用的完整事实在 `data/apks/<name>.json` 中，汇总表为 `data/apks/summary.tsv`。本页由 `tools/apk/gen_apps_doc.py` 从这些文件生成。

## 方法和限制

- 权限：敏感子集按固定规则确定，而非人工判断。规则为：Android 运行时（危险）权限；特殊访问权限（MANAGE_EXTERNAL_STORAGE、SYSTEM_ALERT_WINDOW、REQUEST_INSTALL_PACKAGES、PACKAGE_USAGE_STATS、WRITE_SETTINGS）；以 `_PRIVILEGED` 结尾的名称；以 `COLOCATION_API_` 开头的名称；以及一份固定的特权名称列表（BLUETOOTH_STACK、CALL_PRIVILEGED、CONTROL_INCALL_EXPERIENCE、COPY_PROTECTED_DATA、MEDIA_CONTENT_CONTROL、MODIFY_PHONE_STATE、WRITE_MEDIA_STORAGE、SPATIAL_MEDIA_PLAYBACK、SET_SPATIAL_MEDIA_LISTENER、FACE_EYE_INTERNAL_API、HYPERSPACE_CALL、ACCESS_CAMERA_DATA、RECORD_AUDIO_BACKGROUND）。
- 接口名：仅保留最后一段以 `I` 开头、第二个字符为大写字母且包含小写字母的名称（AIDL 命名模式）。全大写的 Intent 常量被排除。
- 类属性字符串是 dex 字符串池中的匹配。它们表明代码能够命名哪些属性，并不表明运行时实际读取了哪些属性。
- JSON 中的列表上限为 200 项。只有 Settings 的活动列表被截断（列出 200 个，共 282 个）；其导出数量仅针对列出的 200 个。
- 本页不基于任何真机运行结果。

## 签名

本组的签名证书族，以签名证书 SHA-256 摘要的前 16 位十六进制数字区分：

- 族 `fac61745dc090378`（17 个）：Carmel、Assistant、Horizon、Metacam、SystemUX、VrShell、VrDriver、BodyApiService、SpatialPersistenceService、WearableInputService、FirstTimeNuxEureka、SystemWebView、Q4BMdm、Q4BService、OCMS、Guardian、Integrity。
- 族 `c8a2e9bccf597c2f`（5 个）：Settings、Bluetooth、xrstreamingclient、SkyguardService、Telecom。

Settings（`android.uid.system`）、Bluetooth（`android.uid.bluetooth`）、SkyguardService（`android.uid.system`）、Telecom（`android.uid.system`）声明了系统共享 UID。Android 要求声明系统共享 UID 的应用使用平台证书签名，因此当这些应用都在族 `c8a2e9bccf597c2f` 中时，该族很可能是平台证书（根据该规则推断；未与参考证书比对）。

完整摘要见各节。

## Settings（设置）

包名 `com.android.settings`，版本 12（代码 32），minSdk 32，targetSdk 32，共享 UID `android.uid.system`。来源 `system_ext_a/priv-app/Settings/Settings.apk`。

平台设置应用。MAIN 动作的 51 个活动是按功能命名的设置页面（Wi-Fi、显示、无障碍、隐私等）；这一判断来自名称。列出的 200 个活动中没有任何一个带 LAUNCHER 类别。

- 签名证书 SHA-256：`c8a2e9bccf597c2fb6dc66bee293fc13f2fc47ec77bc6b2b0d52c11f51192ab8`。
- 类：9525 个，位于 2 个 dex 文件中。原生库：0。资源（assets）：0。
- 组件：活动 282（已列出 200，其中导出 144）；服务 15（导出 10）；广播接收器 20（导出 16）；内容提供者 11（导出 7）。
- MAIN 动作的活动（51）：`Settings$NetworkDashboardActivity`、`Settings$NetworkProviderSettingsActivity`、`Settings$ConfigureWifiSettingsActivity`、`Settings$WifiInfoActivity`、`WifiConfigInfo`、`Settings$WifiAPITestActivity`、`WifiStatusTest`、`Settings$ApnSettingsActivity`、`Settings$TetherSettingsActivity`、`Settings$WifiTetherSettingsActivity`、`Settings$WifiP2pSettingsActivity`、`Settings$VpnSettingsActivity`、`Settings$DateTimeSettingsActivity`、`Settings$LocalePickerActivity`、`Settings$LanguageAndInputSettingsActivity`、`Settings$SpellCheckersSettingsActivity`、`InputMethodAndSubtypeEnablerActivity`、`Settings$UserDictionarySettingsActivity`、`Settings$ZenModeSettingsActivity`、`ZenSuggestionActivity`、`WallpaperSuggestionActivity`、`StyleSuggestionActivity`、`Settings$ZenModeScheduleRuleSettingsActivity`、`Settings$DisplaySettingsActivity`、`Settings$SmartAutoRotateSettingsActivity`、`Settings$NightDisplaySettingsActivity`、`Settings$DarkThemeSettingsActivity`、`Settings$NightDisplaySuggestionActivity`、`Settings$MyDeviceInfoActivity`、`Settings$ManageApplicationsActivity`、`Settings$RunningServicesActivity`、`Settings$StorageUseActivity`、`Settings$NotificationStationActivity`、`NotificationHistoryActivity`、`Settings$LocationSettingsActivity`、`Settings$SecurityDashboardActivity`、`Settings$PrivacySettingsActivity`、`Settings$DeviceAdminSettingsActivity`、`Settings$IccLockSettingsActivity`、`Settings$AccessibilitySettingsActivity`、`Settings$AccessibilityDaltonizerSettingsActivity`、`Settings$ReduceBrightColorsSettingsActivity`、`Settings$AccessibilityInversionSettingsActivity`、`SetupRedactionInterstitial`、`ScreenLockSuggestionActivity`、`FingerprintEnrollSuggestionActivity`、`Settings$StorageDashboardActivity`、`Settings$PrintSettingsActivity`、`UsageStatsActivity`、`Settings$PowerUsageSummaryActivity`、`CryptKeeper`。
- LAUNCHER 活动（0）：无。
- 服务（15）：`TetherService`、`BluetoothPairingService`、`SettingsDumpService`、`DevelopmentTiles$ShowLayout`、`DevelopmentTiles$GPUProfiling`、`DevelopmentTiles$ForceRTL`、`DevelopmentTiles$AnimationSpeed`、`DevelopmentTiles$WinscopeTrace`、`DevelopmentTiles$SensorsOff`、`DevelopmentTiles$WirelessDebugging`、`DevelopmentTiles$ShowTaps`、`AnomalyCleanupJobService`、`AnomalyConfigJobService`、`AnomalyDetectionJobService`、`SimNotificationService`。
- 广播接收器（已列出 20）：`SettingsInitialize`、`SearchStateReceiver`、`StorageUnmountReceiver`、`BluetoothPairingRequest`、`BluetoothPermissionRequest`、`TestingSettingsBroadcastReceiver`、`BluetoothDiscoverableTimeoutReceiver`、`ProfileUpdateReceiver`、`SimSelectNotification`、`ConnectToWifiHandler`、`SliceBroadcastReceiver`、`SliceRelayReceiver`、`VolumeSliceRelayReceiver`、`AnomalyDetectionReceiver`、`AnomalyConfigReceiver`、`BluetoothPairingReceiver`、`SimSlotChangeReceiver`、`SuwFinishReceiver`、`SimCompleteBootReceiver`、`ProviderRelayReceiver`。
- 请求的权限：105。敏感子集（21）：`ACCESS_FINE_LOCATION`、`BLUETOOTH_ADVERTISE`、`BLUETOOTH_CONNECT`、`BLUETOOTH_PRIVILEGED`、`BLUETOOTH_SCAN`、`CALL_PHONE`、`CAMERA`、`COPY_PROTECTED_DATA`、`MANAGE_EXTERNAL_STORAGE`、`MEDIA_CONTENT_CONTROL`、`MODIFY_PHONE_STATE`、`PACKAGE_USAGE_STATS`、`READ_CONTACTS`、`READ_EXTERNAL_STORAGE`、`READ_PHONE_STATE`、`SYSTEM_ALERT_WINDOW`、`TETHER_PRIVILEGED`、`WRITE_CONTACTS`、`WRITE_EXTERNAL_STORAGE`、`WRITE_MEDIA_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（1）：`com.oculus.settings.wifi.permission.VIEW_WIFI_EAP_CERT_ALIASES`。
- AIDL 风格的接口名（1）：`android.ui.ISurfaceComposer`。
- 类属性字符串（57）；前十个：`debug.com.android.settings.search.crash_on_error`、`debug.force_rtl`、`debug.hwui.force_dark`、`debug.hwui.overdraw`、`debug.hwui.profile`、`debug.hwui.show_dirty_regions`、`debug.hwui.show_layers_updates`、`debug.hwui.show_non_rect_clip`、`persist.bluetooth.a2dp_offload.disabled`、`persist.bluetooth.avrcpversion`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：129。内容 authority（2）：`content://telephony/carriers/preferapn`、`content://telephony/carriers/restore`。

## Bluetooth（蓝牙）

包名 `com.android.bluetooth`，版本 12（代码 32），minSdk 32，targetSdk 32，共享 UID `android.uid.bluetooth`。来源 `system_a/system/app/Bluetooth/Bluetooth.apk`。

蓝牙协议栈应用。其服务包括适配器（AdapterService）、GATT、A2DP 和 A2DP sink、HFP（HeadsetService）、HID host、AVRCP 控制器和目标端、OPP、PBAP、MAP、MAP client、SAP 以及媒体浏览服务（均为类名）。

- 签名证书 SHA-256：`c8a2e9bccf597c2fb6dc66bee293fc13f2fc47ec77bc6b2b0d52c11f51192ab8`。
- 类：5404 个，位于 1 个 dex 文件中。原生库：0。资源（assets）：0。
- 组件：活动 10（导出 2）；服务 24（导出 22）；广播接收器 2（导出 2）；内容提供者 4（导出 2）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（24）：`AdapterService`、`BluetoothOppService`、`BluetoothPbapService`、`BluetoothMapService`、`MapClientService`、`SapService`、`GattService`、`HeadsetService`、`A2dpService`、`A2dpSinkService`、`BluetoothMediaBrowserService`、`AvrcpTargetService`、`AvrcpControllerService`、`HidHostService`、`HidDeviceService`、`PanService`、`HeadsetClientService`、`HfpClientConnectionService`、`PbapClientService`、`HearingAidService`、`LeAudioService`、`AuthenticationService`、`BluetoothInCallService`、`MultiInstanceInvalidationService`。
- 广播接收器（已列出 2）：`BluetoothOppReceiver`、`BluetoothOppHandoverReceiver`。
- 请求的权限：61。敏感子集（23）：`ACCESS_COARSE_LOCATION`、`BLUETOOTH_ADVERTISE`、`BLUETOOTH_CONNECT`、`BLUETOOTH_PRIVILEGED`、`BLUETOOTH_SCAN`、`BLUETOOTH_STACK`、`CALL_PRIVILEGED`、`CONTROL_INCALL_EXPERIENCE`、`GET_ACCOUNTS`、`MEDIA_CONTENT_CONTROL`、`MODIFY_PHONE_STATE`、`PACKAGE_USAGE_STATS`、`READ_CALL_LOG`、`READ_CONTACTS`、`READ_EXTERNAL_STORAGE`、`READ_SMS`、`RECEIVE_SMS`、`SEND_SMS`、`TETHER_PRIVILEGED`、`WRITE_CALL_LOG`、`WRITE_CONTACTS`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（2）：`android.permission.ACCESS_BLUETOOTH_SHARE`、`com.android.permission.ALLOWLIST_BLUETOOTH_DEVICE`。
- AIDL 风格的接口名（1）：`android.net.networkstack.aidl.quirks.IPv6ProvisioningLossQuirkParcelable`。
- 类属性字符串（16）；前十个：`persist.bluetooth.a2dp_offload.disabled`、`persist.bluetooth.avrcpversion`、`persist.bluetooth.btsnoopdefaultmode`、`persist.bluetooth.btsnooplogmode`、`persist.bluetooth.disableinbandringing`、`persist.bluetooth.enablenewavrcp`、`persist.bluetooth.factoryreset`、`persist.bluetooth.mapversion`、`persist.bluetooth.maxconnectedaudiodevices`、`persist.bluetooth.pts`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：52。内容 authority（5）：`content://com.android.bluetooth.avrcpcontroller.AvrcpCoverArtProvider`、`content://com.android.bluetooth.map.MmsFileProvider`、`content://com.android.bluetooth.opp/btopp`、`content://media/external/images/media/`、`content://mms-sms/canonical-address/`。

## Carmel（浏览器）

包名 `com.oculus.browser`，版本 35.1.0.78.167.646680531（代码 565301467），minSdk 26，targetSdk 35，无共享 UID。来源 `system_ext_a/app/Carmel/Carmel.apk`。

浏览器应用。按类名前缀统计，其 dex 主要由 Chromium 包构成：org.chromium.chrome、org.chromium.components 和 org.chromium.content。其 MAIN 动作活动包括 WebXR 和 WebVR 活动，PanelActivity 是 LAUNCHER 活动。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：13356 个，位于 1 个 dex 文件中。原生库：4。资源（assets）：36。
- 组件：活动 32（导出 10）；服务 71（导出 3）；广播接收器 24（导出 10）；内容提供者 6（导出 1）。
- MAIN 动作的活动（4）：`WebXRCustomTabActivity`、`WebVRActivity`、`WebVRVolumeActivity`、`PanelActivity`。
- LAUNCHER 活动（1）：`PanelActivity`。
- 服务（71）：`BrowserRestartService`、`DecoderService`、`DownloadForegroundService`、`DownloadBroadcastManager`、`NotificationService`、`NotificationJobService`、`BackgroundTaskJobService`、`CustomTabsConnectionService`、`PostMessageService`、`PaymentDetailsUpdateService`、`PanelService`、`NotificationIntentInterceptorService`、`MediaCaptureNotificationService`、`ChromeMediaNotificationControllerServices$PlaybackListenerService1`、`ChromeMediaNotificationControllerServices$PlaybackListenerService2`、`ChromeMediaNotificationControllerServices$PlaybackListenerService3`、`ChromeMediaNotificationControllerServices$PlaybackListenerService4`、`ChromeMediaNotificationControllerServices$PlaybackListenerService5`、`ChromeMediaNotificationControllerServices$PlaybackListenerService6`、`ChromeMediaNotificationControllerServices$PlaybackListenerService7`、`ChromeMediaNotificationControllerServices$PresentationListenerService`、`ChromeMediaNotificationControllerServices$CastListenerService`、`BluetoothNotificationService`、`UsbNotificationService`、`SandboxedProcessService0`、`SandboxedProcessService1`、`SandboxedProcessService2`、`SandboxedProcessService3`、`SandboxedProcessService4`、`SandboxedProcessService5`、`SandboxedProcessService6`、`SandboxedProcessService7`、`SandboxedProcessService8`、`SandboxedProcessService9`、`SandboxedProcessService10`、`SandboxedProcessService11`、`SandboxedProcessService12`、`SandboxedProcessService13`、`SandboxedProcessService14`、`SandboxedProcessService15`、`SandboxedProcessService16`、`SandboxedProcessService17`、`SandboxedProcessService18`、`SandboxedProcessService19`、`SandboxedProcessService20`、`SandboxedProcessService21`、`SandboxedProcessService22`、`SandboxedProcessService23`、`SandboxedProcessService24`、`SandboxedProcessService25`、`SandboxedProcessService26`、`SandboxedProcessService27`、`SandboxedProcessService28`、`SandboxedProcessService29`、`SandboxedProcessService30`、`SandboxedProcessService31`、`SandboxedProcessService32`、`SandboxedProcessService33`、`SandboxedProcessService34`、`SandboxedProcessService35`、`SandboxedProcessService36`、`SandboxedProcessService37`、`SandboxedProcessService38`、`SandboxedProcessService39`、`PrivilegedProcessService0`、`PrivilegedProcessService1`、`PrivilegedProcessService2`、`PrivilegedProcessService3`、`PrivilegedProcessService4`、`MediaNotificationService`、`ReconnectionService`。
- 广播接收器（已列出 24）：`PasswordProtectionBroadcastReceiver`、`DisclosureAcceptanceBroadcastReceiver`、`WebApkInstallBroadcastReceiver`、`PackageBroadcastReceiver`、`QuickActionSearchWidgetProvider$QuickActionSearchWidgetProviderSearch`、`QuickActionSearchWidgetProvider$QuickActionSearchWidgetProviderDino`、`FBNSNotificationsReceiver`、`PackageSettingsReceiver`、`NotificationService$Receiver`、`NotificationServiceImpl$Receiver`、`NotificationPermissionChangeReceiver`、`BackgroundTaskBroadcastReceiver`、`DataSharingNotificationManager$Receiver`、`LHQAMReceiver`、`TheaterViewStatusReceiver`、`PackageReplacedBroadcastReceiver`、`AnnouncementNotificationManager$Receiver`、`NotificationIntentInterceptor$Receiver`、`DisplayAgent$Receiver`、`AutoFetchNotifier$CompleteNotificationReceiver`、`AutoFetchNotifier$InProgressCancelReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`MediaIntentReceiver`。
- 请求的权限：62。敏感子集（9）：`ACCESS_COARSE_LOCATION`、`ACCESS_FINE_LOCATION`、`POST_NOTIFICATIONS`、`READ_EXTERNAL_STORAGE`、`RECORD_AUDIO`、`RECORD_AUDIO_BACKGROUND`、`SYSTEM_ALERT_WINDOW`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（7）：`com.oculus.browser.permission.CHILD_SERVICE`、`com.oculus.browser.permission.READ_WRITE_BOOKMARK_FOLDERS`、`com.oculus.browser.TOS_ACKED`、`com.oculus.browser.permission.C2D_MESSAGE`、`com.oculus.browser.permission.DEBUG`、`com.oculus.browser.LHQAM_BROADCAST_PERMISSION`、`com.oculus.browser.THEATER_VIEW_STATUS_BROADCAST_PERMISSION`。
- AIDL 风格的接口名（11）：`android.content.pm.IPackageManager`、`android.os.IMessenger`、`android.widget.directwriting.IDirectWritingService`、`android.widget.directwriting.IDirectWritingServiceCallback`、`com.oculus.assistant.api.IOculusAssistantService`、`com.oculus.assistant.api.IOculusAssistantSubscriber`、`com.oculus.vrshell.privateipc.IBrowserDuiReplacementCallback`、`com.oculus.vrshell.privateipc.IBrowserDuiReplacementService`、`com.oculus.vrshell.privateipc.IEnvironmentIPCService`、`com.oculus.vrshell.privateipc.ITheaterViewCallback`、`com.oculus.vrshell.privateipc.ITheaterViewService`。
- 类属性字符串（10）；前十个：`debug.oc_shell.dbg_watchdog`、`debug.oculus.graphtier`、`debug.oculus.mqd.off`、`persist.ovr.deviceconfig.service`、`ro.build.type`、`ro.build.version.oneui`、`ro.product.first_api_level`、`ro.vros.build.version`、`sys.hmt.connected`、`sys.hmt.mounted`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：96。内容 authority（6）：`content://com.facebook.mobileconfigservice.contentprovider`、`content://com.google.android.gms.phenotype/`、`content://com.google.android.gsf.gservices`、`content://com.google.android.gsf.gservices/prefix`、`content://com.oculus.horizon.sso/cookies?fb_app_id=1562231310457908`、`content://com.oculus.horizon/profile?app_id=1916519981771802`。

## Assistant（助手）

包名 `com.oculus.assistant`，版本 172.0.0.0.61（代码 334600368），minSdk 29，targetSdk 29，无共享 UID。来源 `system_ext_a/priv-app/Assistant/Assistant.apk`。

助手应用（包名 com.oculus.assistant）。其服务包括 AssistantService、CaptionService、PlatformDictationService 和 MRVIService（类名）。其 LAUNCHER 活动为 AssistantHelpActivity。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：6305 个，位于 2 个 dex 文件中。原生库：138。资源（assets）：144。
- 组件：活动 7（导出 5）；服务 24（导出 10）；广播接收器 16（导出 7）；内容提供者 5（导出 2）。
- MAIN 动作的活动（1）：`AssistantHelpActivity`。
- LAUNCHER 活动（1）：`AssistantHelpActivity`。
- 服务（24）：`ExecutionJobService`、`DataDeletionJobService`、`ScheduleFederatedJobsService`、`PackageFinderService`、`AssistantE2ELoggingService`、`CaptionService`、`AssistantService`、`IVCService`、`PlatformLoggerService`、`PlatformDictationService`、`MRVIService`、`AssistantDumpService`、`AssistantIntentService`、`AssistantUserEdService`、`DwellClickAccessibilityService`、`FaDummyRecordWriterService`、`LollipopUploadSafeService`、`LollipopUploadService`、`AlarmBasedUploadService`、`AssistantSettingsJobService`、`SystemAlarmService`、`SystemJobService`、`SystemForegroundService`、`MultiInstanceInvalidationService`。
- 广播接收器（已列出 16）：`WakeWordStartupReceiver`、`DoubleTapSettingStartupReciever`、`AssistantBroadcastReceiver`、`AssistantNotifBroadcastReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`HighPriUploadRetryReceiver`、`ProfileInstallReceiver`、`ForceStopRunnable$BroadcastReceiver`、`ConstraintProxy$BatteryChargingProxy`、`ConstraintProxy$BatteryNotLowProxy`、`ConstraintProxy$StorageNotLowProxy`、`ConstraintProxy$NetworkStateProxy`、`RescheduleReceiver`、`ConstraintProxyUpdateReceiver`、`DiagnosticsReceiver`。
- 请求的权限：50。敏感子集（7）：`MEDIA_CONTENT_CONTROL`、`READ_EXTERNAL_STORAGE`、`RECORD_AUDIO`、`RECORD_AUDIO_BACKGROUND`、`SYSTEM_ALERT_WINDOW`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（4）：`com.oculus.assistant.PANEL`、`com.oculus.assistant.BROADCAST_PERMISSION`、`com.oculus.assistant.permission.CROSS_PROCESS_BROADCAST_MANAGER`、`com.oculus.assistant.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（16）：`com.oculus.aidl.IAuthService2`、`com.oculus.aidl.IAuthenticationCallback`、`com.oculus.assistant.api.IOculusAssistantService`、`com.oculus.assistant.api.IOculusAssistantSubscriber`、`com.oculus.assistant.service.api.attention.IOculusAssistantAttentionListener`、`com.oculus.captionservice.producer.IDictationCaptureServiceInterface`、`com.oculus.captionservice.subscriber.IDictationSubscriber`、`com.oculus.metacam.ipc.ICaptureServiceBinder`、`com.oculus.metacam.ipc.ICastStateListener`、`com.oculus.metacam.ipc.ILivestreamStateListenerV2`、`com.oculus.metacam.ipc.ILivestreamingServiceBinder`、`com.oculus.metacam.ipc.IMetaCamServiceBinder`、`com.oculus.metacam.ipc.IOnCaptureStoppedListener`、`com.oculus.metacam.ipc.IScreenshotStateListener`、`com.oculus.metacam.ipc.IVirtualCameraServiceBinder`、`com.oculus.metacam.ipc.IVirtualCameraStateListener`。
- 类属性字符串（26）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`、`debug.meta.federatedcomputing.client.allow_all_federated_events`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：376。内容 authority（6）：`content://com.facebook.mobileconfigservice.contentprovider`、`content://com.oculus.common.navigation/PREF_ACTIVE_APP_KEY`、`content://com.oculus.common.navigation/PREF_ACTIVE_PANEL_KEY`、`content://com.oculus.common.navigation/primaryPackage`、`content://com.oculus.timer.provider`、`content://com.oculus.vrshell.assistant/status`。

## Horizon（主页）

包名 `com.oculus.horizon`，版本 71.0.0.894.253（代码 665345962），minSdk 23，targetSdk 23，共享 UID `com.oculus.uid`。来源 `system_ext_a/app/Horizon/Horizon.apk`。

主页应用（包名 com.oculus.horizon，共享 UID com.oculus.uid）。清单中没有 MAIN 或 LAUNCHER 活动。按类名看，其服务涵盖 Gk 配置（GkService）、MobileConfig、身份验证（AuthService2 及身份验证器服务）、媒体上传（MediaUploaderService）、云存储（CloudStorageIntentService）和下载（OculusDownloadService）。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：13037 个，位于 2 个 dex 文件中。原生库：137。资源（assets）：330。
- 组件：活动 3（导出 1）；服务 53（导出 17）；广播接收器 37（导出 16）；内容提供者 15（导出 12）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（53）：`PackageFinderService`、`OculusDownloadService`、`GkService`、`NotificationReceiver`、`SocialJobService`、`CloudStorageIntentService`、`CloudStorageBackgroundService`、`MediaUploaderService`、`MediaUploaderServiceInternal`、`MediaUploaderStoreFBGamingTokenService`、`MediaChangeObserverService`、`MediaUploaderSyncService`、`HorizonFbnsService`、`OVRService`、`SDKLoggingService`、`ODHInterfaceService`、`OVRMediaService`、`AccountSyncJobService`、`SessionPingJobService`、`F3UpgradeJobService`、`AccountUpdateJobService`、`AccountFixJobService`、`RebootJobService`、`AuthControlDumpsysService`、`DeviceTokenRefreshService`、`AuthService2`、`AuthService`、`AuthServicePermissionProtected`、`MobileConfigService`、`LoginHandlersService`、`MetaAuthenticatorService`、`VRLifecycleJobService`、`DurableIAPJobService`、`HandsTrackingConsentSyncIntentService`、`DevicePrivacyConsentSyncIntentService`、`DevicePrivacyConsentSyncJobService`、`FacebookAuthenticatorService`、`FacebookSsoAuthenticatorService`、`HorizonWorldsPlatformAuthenticatorService`、`InstagramSsoAuthenticatorService`、`OculusAuthenticatorService`、`WorkAuthenticatorService`、`EnsureDeviceOwnershipJobService`、`LollipopUploadSafeService`、`LollipopUploadService`、`Analytics2UploadService`、`AlarmBasedUploadService`、`FirebaseMessagingService`、`SystemAlarmService`、`SystemJobService`、`SystemForegroundService`、`MultiInstanceInvalidationService`、`ComponentDiscoveryService`。
- 广播接收器（已列出 37）：`OculusDownloadManagerReceiver`、`DeviceConfigDebugReceiver`、`StatusRequestReceiver`、`DeviceAuthTokenRefreshBroadcastReceiver`、`OculusFbnsCallbackReceiver`、`FBConnectReceiver`、`PresenceBootReceiver`、`RegisteredMqttSystemBroadcastReceiver`、`RegisteredSystemBroadcastReceiver`、`SharingEducationReceiver`、`PackagesCacheReceiver`、`AccountSyncTriggerReceiver`、`SessionPingTriggerReceiver`、`F3UpgradeTriggerReceiver`、`AccountUpdateTriggerReceiver`、`AccountFixTriggerReceiver`、`HeartbeatMonitorTriggerReceiver`、`SnapshotTriggerReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`AuthBroadcastReceiver`、`VRLifecycleStartUpBootReceiver`、`HandsTrackingConsentSyncBroadcastReceiver`、`EnsureDeviceOwnershipBootReceiver`、`RLCPDataChangeReceiver`、`RLCPSalsaReceiver`、`HighPriUploadRetryReceiver`、`ForceStopRunnable$BroadcastReceiver`、`ConstraintProxy$BatteryChargingProxy`、`ConstraintProxy$BatteryNotLowProxy`、`ConstraintProxy$StorageNotLowProxy`、`ConstraintProxy$NetworkStateProxy`、`RescheduleReceiver`、`ConstraintProxyUpdateReceiver`、`DiagnosticsReceiver`、`FirebaseInstanceIdReceiver`、`ProfileInstallReceiver`。
- 请求的权限：57。敏感子集（6）：`CAMERA`、`MANAGE_EXTERNAL_STORAGE`、`RECORD_AUDIO_BACKGROUND`、`SYSTEM_ALERT_WINDOW`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（20）：`com.oculus.horizon.permission.ACCESS`、`com.oculus.permission.DEVICE_CONFIG_PUSH_TO_CLIENT`、`com.oculus.permission.REQUEST_METADEVICECONFIG_STATUS`、`com.oculus.horizon.notifications.receivers.RECEIVE_NOTIFICATION_DESTINATIONS`、`com.oculus.horizon.notifications.receivers.TEST_FBNS_NOTIFICATION`、`com.oculus.horizon.notifications.receivers.RECEIVE_SILENT_NOTIFS`、`com.oculus.horizon.receivers.permission.REQUEST_TO_JOIN_ACTION`、`com.oculus.horizon.receivers.permission.APPLICATION_INVITE_ACTION`、`com.oculus.panelapp.people.permission.SOCIAL_REQUEST_ACTION`、`com.oculus.panelapp.parties.permission.SOCIAL_PARTY_END_NOTIF_ACTION`、`com.oculus.vrshell.RECEIVE_NOTIFICATION_DESTINATIONS`、`com.oculus.panelapp.messenger.permission.SOCIAL_NOTIF_UPSELL_ACTION`、`com.oculus.panelapp.parties.permission.CONCURRENT_NOTIF_QUICK_ACTION`、`com.oculus.horizon.FBCONNECT_CONTENT_PROVIDER_WRITE_ACCESS`、`com.oculus.horizon.FBCONNECT_CONTENT_PROVIDER_READ_ACCESS`、`com.oculus.horizon.LINKED_ACCOUNTS_READ_PERMISSION`、`com.oculus.horizon.LINKED_ACCOUNTS_WRITE_PERMISSION`、`com.oculus.horizon.permission.OVR_PLATFORM_BROADCAST`、`com.oculus.horizon.auth`、`com.oculus.horizon.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（27）：`android.os.IMessenger`、`com.oculus.aidl.IAuthService2`、`com.oculus.aidl.IAuthenticationCallback`、`com.oculus.aidl.IGkService`、`com.oculus.iap.IAPResultPermission`、`com.oculus.metacam.ipc.ICaptureServiceBinder`、`com.oculus.metacam.ipc.ICastStateListener`、`com.oculus.metacam.ipc.ILivestreamCheckFbAccountLinkedQueryListener`、`com.oculus.metacam.ipc.ILivestreamCommentListener`、`com.oculus.metacam.ipc.ILivestreamSettingQueryListener`、`com.oculus.metacam.ipc.ILivestreamStateListener`、`com.oculus.metacam.ipc.ILivestreamStateListenerV2`、`com.oculus.metacam.ipc.ILivestreamingServiceBinder`、`com.oculus.metacam.ipc.IMetaCamServiceBinder`、`com.oculus.metacam.ipc.IOnCaptureStoppedListener`、`com.oculus.metacam.ipc.IScreenshotStateListener`、`com.oculus.metacam.ipc.IVirtualCameraServiceBinder`、`com.oculus.metacam.ipc.IVirtualCameraStateListener`、`com.oculus.presence.hyperspace.IHyperspaceApiProvider`、`com.oculus.presence.hyperspace.IHyperspaceApiStatusListener`、`com.oculus.presence.hyperspace.IHyperspaceServiceVideoApi`、`com.oculus.presence.hyperspace.IOutgoingVideoObserver`、`com.oculus.presence.hyperspace.ISurfaceVideoObserver`、`com.oculus.userserver.managerservice.IOculusUserManager`、`com.oculus.vrshell.notifications.IVrNotificationWaypointMetricService`、`vendor.meta.parfait.IConfigQuery`、`vendor.meta.parfait.ITrigger`。
- 类属性字符串（124）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`、`debug.meta.federatedcomputing.client`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：558。内容 authority（17）：`content://com.facebook.app.FacebookContentProvider`、`content://com.facebook.katana.provider.AttributionIdProvider`、`content://com.facebook.mobileconfigservice.contentprovider`、`content://com.facebook.wakizashi.provider.AttributionIdProvider`、`content://com.oculus.horizon`、`content://com.oculus.horizon.gatekeeper/fetch`、`content://com.oculus.horizon.linkedaccounts/accounts`、`content://com.oculus.horizon/profile`、`content://com.oculus.identitymanagement.capabilities.contentprovider/account`、`content://com.oculus.identitymanagement.capabilities.contentprovider/account/activeuser/capabilities`、`content://com.oculus.identitymanagement.capabilities.contentprovider/activeuser`、`content://com.oculus.identitymanagement.capabilities.contentprovider/capabilities`、`content://com.oculus.messenger.service.provider`、`content://com.oculus.socialplatform.messenger.service.provider`、`content://com.oculus.socialplatform.messengervraccounts/v1`、`content://com.oculus.socialplatform.partiesapi`、`content://downloads/my_downloads`。

## Metacam（采集）

包名 `com.oculus.metacam`，版本 118.0.0.251.161（代码 511400824），minSdk 28，targetSdk 28，无共享 UID。来源 `system_ext_a/app/Metacam/Metacam.apk`。

采集与相机应用（包名 com.oculus.metacam）。其服务为 CaptureService、CameraSdkService、CastingService、LivestreamingService 和 VirtualCameraService（类名）。其 LAUNCHER 活动为 SharingPanelActivity 和 LivestreamYoutubeChatActivity。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：8627 个，位于 2 个 dex 文件中。原生库：15。资源（assets）：10。
- 组件：活动 3（导出 2）；服务 12（导出 7）；广播接收器 19（导出 10）；内容提供者 2（导出 0）。
- MAIN 动作的活动（1）：`SharingPanelActivity`。
- LAUNCHER 活动（2）：`LivestreamYoutubeChatActivity`、`SharingPanelActivity`。
- 服务（12）：`SharingPanelService`、`SimpleMediaSyncObserverImmersiveService`、`CameraSdkService`、`CaptureService`、`CastingService`、`DeviceBootJobService`、`LivestreamingService`、`VirtualCameraService`、`SystemAlarmService`、`SystemJobService`、`SystemForegroundService`、`MultiInstanceInvalidationService`。
- 广播接收器（已列出 19）：`SimpleMediaSyncJobReceiver`、`SyncCapturedMediaReceiver`、`MediaSyncJobReceiver`、`VRRuntimeBroadcastReceiver`、`UserManagerBroadcastReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`CompanionServerBroadcastReceiver`、`DeviceBootBroadcastReceiver`、`SilentNotificationReceiver`、`ForceStopRunnable$BroadcastReceiver`、`ConstraintProxy$BatteryChargingProxy`、`ConstraintProxy$BatteryNotLowProxy`、`ConstraintProxy$StorageNotLowProxy`、`ConstraintProxy$NetworkStateProxy`、`RescheduleReceiver`、`ConstraintProxyUpdateReceiver`、`DiagnosticsReceiver`、`ProfileInstallReceiver`。
- 请求的权限：49。敏感子集（11）：`ACCESS_BACKGROUND_LOCATION`、`ACCESS_COARSE_LOCATION`、`ACCESS_FINE_LOCATION`、`MANAGE_EXTERNAL_STORAGE`、`NEARBY_WIFI_DEVICES`、`POST_NOTIFICATIONS`、`READ_EXTERNAL_STORAGE`、`RECORD_AUDIO`、`RECORD_AUDIO_BACKGROUND`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（1）：`com.oculus.metacam.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（18）：`com.oculus.aidl.IScreenCaptureService`、`com.oculus.captionservice.producer.IHighQualityAudioCaptureServiceInterface`、`com.oculus.captionservice.subscriber.IHighQualityAudioSubscriber`、`com.oculus.metacam.camerasdkservice.ICameraSdkServiceBinder`、`com.oculus.metacam.ipc.ICaptureServiceBinder`、`com.oculus.metacam.ipc.ICastStateListener`、`com.oculus.metacam.ipc.ILivestreamCheckFbAccountLinkedQueryListener`、`com.oculus.metacam.ipc.ILivestreamCommentListener`、`com.oculus.metacam.ipc.ILivestreamSettingQueryListener`、`com.oculus.metacam.ipc.ILivestreamStateListener`、`com.oculus.metacam.ipc.ILivestreamStateListenerV2`、`com.oculus.metacam.ipc.ILivestreamingServiceBinder`、`com.oculus.metacam.ipc.IMetaCamServiceBinder`、`com.oculus.metacam.ipc.IOnCaptureStoppedListener`、`com.oculus.metacam.ipc.IScreenshotStateListener`、`com.oculus.metacam.ipc.IVirtualCameraServiceBinder`、`com.oculus.metacam.ipc.IVirtualCameraStateListener`、`com.oculus.vrshell.privateipc.ILayerIPCService`。
- 类属性字符串（32）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`、`debug.oculus.capture.always_up`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：401。内容 authority（2）：`content://com.facebook.mobileconfigservice.contentprovider`、`content://com.oculus.horizon.fbconnect/app_scoped_access_token`。

## SystemUX（系统界面）

包名 `com.oculus.systemux`，版本 71.0.0.605.522（代码 665334819），minSdk 32，targetSdk 32，无共享 UID。来源 `system_ext_a/priv-app/SystemUX/SystemUX.apk`。

系统用户界面应用（包名 com.oculus.systemux）。其面板服务（类名）涵盖对话框（AndroidDialogPanelService）、助手（AssistantPanelService）、随时界面（AnytimeUIPanelService）、资料库和通知。全部 13 个活动均为导出。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：14060 个，位于 2 个 dex 文件中。原生库：36。资源（assets）：79。
- 组件：活动 13（导出 13）；服务 33（导出 27）；广播接收器 21（导出 12）；内容提供者 2（导出 0）。
- MAIN 动作的活动（6）：`ShortcutCreationDialogActivity`、`AuiInternalSettingsActivity`、`AutofillTrampolineHelperActivity`、`BugReporterActivity`、`BugReporterReprojectedDialogActivity`、`DevicePairingActivity`。
- LAUNCHER 活动（5）：`AuiInternalSettingsActivity`、`AutofillTrampolineHelperActivity`、`BugReporterActivity`、`BugReporterReprojectedDialogActivity`、`DevicePairingActivity`。
- 服务（33）：`AndroidDialogPanelService`、`AndroidDialogStandalonePanelService`、`AssistantPanelService`、`NavigatorItemPinningService`、`AnytimeUIPanelService`、`LibraryPanelService`、`LibraryStandalonePanelService`、`AppCacheJobService`、`DestinationUiPanelService`、`DestinationUiStandalonePanelService`、`AppMenuPanelService`、`AuiInternalSettingsPanelService`、`AuiInternalSettingsStandalonePanelService`、`NotificationsPanelService`、`NotificationsStandalonePanelService`、`ControllerSettingsPanelService`、`ControllerSettingsStandalonePanelService`、`FileViewerPanelService`、`FileViewerStandalonePanelService`、`VirtualObjectsPanelService`、`VirtualObjectsStandalonePanelService`、`MixedRealityObjectsPanelService`、`MixedRealityObjectsStandalonePanelService`、`CaptionPanelService`、`DevicePairingPanelService`、`ThirdPartyNotificationService`、`SystemUXDumpService`、`AnytimeUIInCallService`、`BugReporterPanelService`、`SystemAlarmService`、`SystemJobService`、`SystemForegroundService`、`MultiInstanceInvalidationService`。
- 广播接收器（已列出 21）：`SurveyReceiver`、`AppCacheReceiver`、`NotificationSharingQuickActionReceiver`、`SharingUpsellReceiver`、`PinnedAppRequestReceiver`、`StartTwilightCastingDialogCompanionServerReceiver`、`StartTwilightCastingDialogFBPermissionReceiver`、`BugReporterScreenshotBroadcastReceiver`、`RemoteLaunchAppRequestReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`VRCallingCaptionReceiver`、`ProfileInstallReceiver`、`ForceStopRunnable$BroadcastReceiver`、`ConstraintProxy$BatteryChargingProxy`、`ConstraintProxy$BatteryNotLowProxy`、`ConstraintProxy$StorageNotLowProxy`、`ConstraintProxy$NetworkStateProxy`、`RescheduleReceiver`、`ConstraintProxyUpdateReceiver`、`DiagnosticsReceiver`。
- 请求的权限：108。敏感子集（19）：`ACCESS_BACKGROUND_LOCATION`、`ACCESS_COARSE_LOCATION`、`ACCESS_FINE_LOCATION`、`BLUETOOTH_ADVERTISE`、`BLUETOOTH_CONNECT`、`BLUETOOTH_PRIVILEGED`、`BLUETOOTH_SCAN`、`FACE_EYE_INTERNAL_API`、`HYPERSPACE_CALL`、`MANAGE_EXTERNAL_STORAGE`、`MEDIA_CONTENT_CONTROL`、`MODIFY_PHONE_STATE`、`PACKAGE_USAGE_STATS`、`POST_NOTIFICATIONS`、`READ_EXTERNAL_STORAGE`、`RECORD_AUDIO`、`SYSTEM_ALERT_WINDOW`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（6）：`com.oculus.panelapp.androiddialog.permission.DISMISS_DIALOG`、`com.oculus.permission.CODEC_AVATAR_REAUTH`、`com.oculus.panelapp.bugreporter.permission.LaunchBugReporterActivity`、`com.oculus.panelapp.bugreporter.permission.LaunchBugReporterReprojectedDialogActivity`、`com.oculus.common.passcode.permission.LAUNCH_PASSCODE`、`com.oculus.systemux.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（27）：`com.oculus.assistant.api.IOculusAssistantService`、`com.oculus.assistant.api.IOculusAssistantSubscriber`、`com.oculus.metacam.ipc.ICaptureServiceBinder`、`com.oculus.metacam.ipc.ICastStateListener`、`com.oculus.metacam.ipc.ILivestreamCommentListener`、`com.oculus.metacam.ipc.ILivestreamSettingQueryListener`、`com.oculus.metacam.ipc.ILivestreamStateListenerV2`、`com.oculus.metacam.ipc.ILivestreamingServiceBinder`、`com.oculus.metacam.ipc.IMetaCamServiceBinder`、`com.oculus.metacam.ipc.IOnCaptureStoppedListener`、`com.oculus.metacam.ipc.IScreenshotStateListener`、`com.oculus.metacam.ipc.IVirtualCameraServiceBinder`、`com.oculus.metacam.ipc.IVirtualCameraStateListener`、`com.oculus.presence.hyperspace.IHyperspaceApiProvider`、`com.oculus.presence.hyperspace.IHyperspaceApiStatusListener`、`com.oculus.presence.hyperspace.IHyperspaceServiceVideoApi`、`com.oculus.userserver.managerservice.IOculusUserManager`、`com.oculus.vrshell.notifications.INotificationProviderV2`、`com.oculus.vrshell.privateipc.IDestinationUIIPCService`、`com.oculus.vrshell.privateipc.IDragDropIPCService`、`com.oculus.vrshell.privateipc.IDragStateEventCallback`、`com.oculus.vrshell.privateipc.IEnvDownloadIPCService`、`com.oculus.vrshell.privateipc.ILayerIPCService`、`com.oculus.vrshell.privateipc.IObjectDragEventCallback`、`com.oculus.vrshell.privateipc.IShellEntityIPCCallback`、`com.oculus.vrshell.privateipc.IShellEntityIPCService`、`com.oculus.vrshell.privateipc.ISystemActionIPCService`。
- 类属性字符串（25）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`、`debug.layout`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：551。内容 authority（8）：`content://com.facebook.mobileconfigservice.contentprovider`、`content://com.oculus.common.socialtablet.communicator.mlvprovider.CommunicatorMLVProvider`、`content://com.oculus.socialplatform.partiesapi`、`content://com.oculus.socialplatform.partiesapi/get_internal_debug_info`、`content://com.oculus.socialplatform.partiesapi/get_last_connected_funnel_extra_params`、`content://com.oculus.socialplatform.partiesapi/get_party_graphql_data`、`content://com.oculus.socialplatform.partiesapi/get_party_voip_data`、`content://com.oculus.socialplatform.partiesapi/get_voip_connection_status`。

## VrShell（VR 外壳）

包名 `com.oculus.vrshell`，版本 71.0.0.612.529（代码 665334768），minSdk 29，targetSdk 29，无共享 UID。来源 `system_ext_a/priv-app/VrShell/VrShell.apk`。

VR 外壳应用（包名 com.oculus.vrshell）。其服务包括 ShellSpatialWindowManagerService、ShellService、DisplayResolverService、KeyboardInputMethodService 和 VrNotificationService（类名）。其大部分 LAUNCHER 活动带有 LaunchCheck 前缀；从名称看，这些是在应用启动前执行的检查（根据名称推断）。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：9752 个，位于 2 个 dex 文件中。原生库：143。资源（assets）：103。
- 组件：活动 41（导出 4）；服务 52（导出 30）；广播接收器 21（导出 10）；内容提供者 7（导出 3）。
- MAIN 动作的活动（36）：`LaunchCheckAppDisabledDialogActivity`、`LaunchCheckAppDegradedDialogActivity`、`LaunchCheckTrialAppOutOfSyncDialogActivity`、`LaunchCheckTrialAppStillUpcomingDialogActivity`、`LaunchCheckRequires6dofDialogActivity`、`LaunchCheckTrialAppNoNetworkFirstStartDialogActivity`、`LaunchCheckTrialAppNoPlayTimeDialogActivity`、`LaunchCheckPassthroughPermissionDialogActivity`、`LaunchCheckRequiresSetupOrthofitDialogActivity`、`LaunchCheckRequiresSocialPresenceSetupDialogActivity`、`LaunchCheckCaptionPrivacyConsentDialogActivity`、`LaunchCheckSpatialDataSharingNoticeDialogActivity`、`LaunchCheckEntitlementSharingDialogActivity`、`LaunchCheckExitMediaAppDialogActivity`、`LaunchCheckNonCompliantDialogActivity`、`DeveloperErrorDialogActivity`、`AppModeIncompatibleDialogActivity`、`GuardianAdjustDialogActivity`、`GuardianQuickBoundaryAdjustRoomScaleDialogActivity`、`SystemFailureDialogActivity`、`GuardianQuickBoundaryAdjustStationaryDialogActivity`、`LocalStreamPrivacyCheckDialogActivity`、`QuitAndLaunchDialogActivity`、`UnOfficailAppDialogActivity`、`MvanLinkRestrictedDialogActivity`、`MvanSocialRestrictedDialogActivity`、`MvanSideLoadingRestrictedDialogActivity`、`SalsaRestrictedDialogActivity`、`DownloadFailureLowStorageDialogActivity`、`CloudStorageDialogActivity`、`MainActivity`、`DogfoodMainActivity`、`GauntletTestPanelActivity`、`QuickActionsMenuActivity`、`ShellHomeUXActivity`、`CastingPanelActivity`。
- LAUNCHER 活动（34）：`LaunchCheckAppDisabledDialogActivity`、`LaunchCheckAppDegradedDialogActivity`、`LaunchCheckTrialAppOutOfSyncDialogActivity`、`LaunchCheckTrialAppStillUpcomingDialogActivity`、`LaunchCheckRequires6dofDialogActivity`、`LaunchCheckTrialAppNoNetworkFirstStartDialogActivity`、`LaunchCheckTrialAppNoPlayTimeDialogActivity`、`LaunchCheckPassthroughPermissionDialogActivity`、`LaunchCheckRequiresSetupOrthofitDialogActivity`、`LaunchCheckRequiresSocialPresenceSetupDialogActivity`、`LaunchCheckCaptionPrivacyConsentDialogActivity`、`LaunchCheckSpatialDataSharingNoticeDialogActivity`、`LaunchCheckEntitlementSharingDialogActivity`、`LaunchCheckExitMediaAppDialogActivity`、`LaunchCheckNonCompliantDialogActivity`、`DeveloperErrorDialogActivity`、`AppModeIncompatibleDialogActivity`、`GuardianAdjustDialogActivity`、`GuardianQuickBoundaryAdjustRoomScaleDialogActivity`、`SystemFailureDialogActivity`、`GuardianQuickBoundaryAdjustStationaryDialogActivity`、`LocalStreamPrivacyCheckDialogActivity`、`QuitAndLaunchDialogActivity`、`UnOfficailAppDialogActivity`、`MvanLinkRestrictedDialogActivity`、`MvanSocialRestrictedDialogActivity`、`MvanSideLoadingRestrictedDialogActivity`、`SalsaRestrictedDialogActivity`、`DownloadFailureLowStorageDialogActivity`、`CloudStorageDialogActivity`、`MainActivity`、`QuickActionsMenuActivity`、`ShellHomeUXActivity`、`CastingPanelActivity`。
- 服务（52）：`ShellSpatialWindowManagerService`、`ShellService`、`ShellAccessibilityService`、`VrNotificationService`、`VrNotificationWaypointMetricService`、`KeyboardPanelService`、`QuickActionsMenuPanelService`、`KeyboardInputMethodService`、`KeyboardInputMethodService$KeyboardInputMethodCallbackService`、`ShellDebugPanelService`、`TheaterScreenPanelService`、`ToastsPanelService`、`ControlBarPanelService`、`ShellDumpService`、`VrNotificationsDumpService`、`ShellClayDumpService`、`DisplayResolverService`、`PanelActivityService`、`InternalPrivateIPCService`、`MrIPCService`、`ShellEntityIPCService`、`SpatialEntityIPCService`、`SystemActionIPCService`、`LifecycleActionIPCService`、`EnvDownloadIPCService`、`DestinationUIIPCService`、`QuickSettingsIPCService`、`ResizeIPCService`、`AdaptabilityModeIPCService`、`DragDropIPCService`、`AvatarIPCService`、`SecureScreenIPCService`、`BrowserDuiReplacementService`、`TheaterViewService`、`EnvironmentIPCService`、`LayerIPCService`、`ModalClosedIPCService`、`MultiLayerService`、`VrShellTooltipLegacyIpcService`、`VrFbnsService`、`VrFbnsCallbackHandler`、`TransportFileService`、`GauntletTestPanelService`、`ShellHomeUXPanelService`、`TheaterScreenMediaRouteProviderService`、`FirebaseMessagingService`、`MultiInstanceInvalidationService`、`ComponentDiscoveryService`、`SystemAlarmService`、`SystemJobService`、`SystemForegroundService`、`HeroKeepAliveService`。
- 广播接收器（已列出 21）：`ShellControlBroadcastReceiver`、`ShellWidgetBroadcastReceiver`、`AssistantBroadcastReceiver`、`CaptionReceiver`、`AUINotificationReceiver`、`SystemKeyDisplayModeSwitchReceiver`、`VrFbnsCallbackReceiver`、`RegisteredMqttSystemBroadcastReceiver`、`RegisteredSystemBroadcastReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`ProfileInstallReceiver`、`FirebaseInstanceIdReceiver`、`ForceStopRunnable$BroadcastReceiver`、`ConstraintProxy$BatteryChargingProxy`、`ConstraintProxy$BatteryNotLowProxy`、`ConstraintProxy$StorageNotLowProxy`、`ConstraintProxy$NetworkStateProxy`、`RescheduleReceiver`、`ConstraintProxyUpdateReceiver`、`DiagnosticsReceiver`。
- 请求的权限：115。敏感子集（9）：`MEDIA_CONTENT_CONTROL`、`MODIFY_PHONE_STATE`、`POST_NOTIFICATIONS`、`RECORD_AUDIO`、`SET_SPATIAL_MEDIA_LISTENER`、`SPATIAL_MEDIA_PLAYBACK`、`SYSTEM_ALERT_WINDOW`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（16）：`com.oculus.vrshell.OVERLAY_HOST`、`com.oculus.vrshell.ShellWidgetBroadcastReceiverPermission`、`com.oculus.vrshell.VrNotificationServicePermission`、`com.oculus.vrshell.notifications.VrNotificationWaypointMetricServicePermission`、`com.oculus.common.navigation.PERMISSION`、`com.oculus.vrshell.permission.FOREGROUND_DESKTOP`、`com.oculus.vrshell.permission.ACCESS_PANEL_ACTIVITY_FEATURES`、`com.oculus.vrshell.permission.ACCESS_PRIVATE_IPC`、`com.oculus.vrshell.permission.ACCESS_SPATIAL_ENTITIES`、`com.oculus.vrshell.permission.ACCESS_MULTILAYER_IPC`、`com.oculus.vrshell.permission.THEATER_VIEW_SERVICE`、`com.oculus.vrshell.permission.REPLACE_DUI`、`com.oculus.vrshell.permission.BROWSER_DUI_REPLACEMENT`、`com.oculus.vrshell.permission.SEND_VRRUNTIME_SYSTEM_BROADCAST`、`com.oculus.vrshell.permission.SEND_GUIDING_TOAST`、`com.oculus.vrshell.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（45）：`android.os.IMessenger`、`com.oculus.assistant.api.IOculusAssistantService`、`com.oculus.assistant.api.IOculusAssistantSubscriber`、`com.oculus.identitymanagement.common.contract.ILocalAccountModeService`、`com.oculus.metacam.ipc.ICaptureServiceBinder`、`com.oculus.metacam.ipc.ICastStateListener`、`com.oculus.metacam.ipc.ILivestreamStateListenerV2`、`com.oculus.metacam.ipc.ILivestreamingServiceBinder`、`com.oculus.metacam.ipc.IMetaCamServiceBinder`、`com.oculus.metacam.ipc.IOnCaptureStoppedListener`、`com.oculus.metacam.ipc.IScreenshotStateListener`、`com.oculus.metacam.ipc.IVirtualCameraServiceBinder`、`com.oculus.metacam.ipc.IVirtualCameraStateListener`、`com.oculus.vrshell.notifications.INotificationProviderV2`、`com.oculus.vrshell.notifications.IVrNotificationWaypointMetricService`、`com.oculus.vrshell.privateipc.IAdaptabilityModeCallback`、`com.oculus.vrshell.privateipc.IAdaptabilityModeIPCService`、`com.oculus.vrshell.privateipc.IAvatarIPCService`、`com.oculus.vrshell.privateipc.IBrowserDuiReplacementCallback`、`com.oculus.vrshell.privateipc.IBrowserDuiReplacementService`、`com.oculus.vrshell.privateipc.IDestinationUIIPCService`、`com.oculus.vrshell.privateipc.IDragDropIPCService`、`com.oculus.vrshell.privateipc.IDragStateEventCallback`、`com.oculus.vrshell.privateipc.IEnvDownloadIPCService`、`com.oculus.vrshell.privateipc.IEnvironmentIPCService`、`com.oculus.vrshell.privateipc.IInternalPrivateIPCService`、`com.oculus.vrshell.privateipc.ILayerIPCService`、`com.oculus.vrshell.privateipc.ILifecycleActionIPCService`、`com.oculus.vrshell.privateipc.IModalClosedCallback`、`com.oculus.vrshell.privateipc.IModalClosedIPCService`、`com.oculus.vrshell.privateipc.IMrIPCService`、`com.oculus.vrshell.privateipc.IObjectDragEventCallback`、`com.oculus.vrshell.privateipc.IQuickSettingsIPCService`、`com.oculus.vrshell.privateipc.IResizeIPCService`、`com.oculus.vrshell.privateipc.ISecureScreenIPCService`、`com.oculus.vrshell.privateipc.IShellEntityIPCCallback`、`com.oculus.vrshell.privateipc.IShellEntityIPCService`、`com.oculus.vrshell.privateipc.ISystemActionIPCService`、`com.oculus.vrshell.privateipc.ITheaterViewCallback`、`com.oculus.vrshell.privateipc.ITheaterViewService`、`com.oculus.vrshell.privateipc.multilayer.IMultiLayerService`、`com.oculus.vrshell.privateipc.multilayer.IOnLayerAcquiredCallback`、`com.oculus.vrshell.privateipc.multilayer.IOnLayerChangedCallback`、`vendor.meta.parfait.IConfigQuery`、`vendor.meta.parfait.ITrigger`。
- 类属性字符串（30）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`、`debug.layout`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：467。内容 authority（1）：`content://com.facebook.mobileconfigservice.contentprovider`。

## VrDriver（VR 驱动）

包名 `com.oculus.systemdriver`，版本 71.0.0.601.498（代码 665335459），minSdk 23，targetSdk 24，无共享 UID。来源 `system_a/system/priv-app/VrDriver/VrDriver.apk`。

位于 `system/priv-app` 的 VR 驱动应用（包名 com.oculus.systemdriver）。其服务为 VrRuntimeService、ScreenCaptureService、RuntimeDumpService 和 PackageFinderService（类名）。它声明了 SEND_FACE_EYE_TRACKING_COMMAND 和 SEND_PROTECTED_CAPTURE_COMMAND。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：984 个，位于 1 个 dex 文件中。原生库：41。资源（assets）：11。
- 组件：活动 1（导出 1）；服务 4（导出 3）；广播接收器 4（导出 2）；内容提供者 5（导出 2）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（4）：`PackageFinderService`、`VrRuntimeService`、`RuntimeDumpService`、`ScreenCaptureService`。
- 广播接收器（已列出 4）：`LyingDownModeActivityTransitionReceiver`、`ProfileInstallReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`。
- 请求的权限：55。敏感子集（4）：`ACCESS_CAMERA_DATA`、`FACE_EYE_INTERNAL_API`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（4）：`oculus.render_notification`、`com.oculus.vrruntimeservice.SEND_FACE_EYE_TRACKING_COMMAND`、`com.oculus.vrruntimeservice.permission.SEND_PROTECTED_CAPTURE_COMMAND`、`com.oculus.systemdriver.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（8）：`android.app.IActivityManager`、`android.app.IApplicationThread`、`android.app.IServiceConnection`、`com.meta.ar.captureservice.IARCaptureService`、`com.oculus.aidl.IScreenCaptureService`、`com.oculus.assistant.api.IOculusAssistantService`、`com.oculus.assistant.api.IOculusAssistantSubscriber`、`com.oculus.metacam.ipc.IAudioCaptureServiceBinder`。
- 类属性字符串（9）；前十个：`debug.oculus.mqd.off`、`persist.debug.oculus.ldm.posture_detection.consecutive_lying_down_event_threshold`、`persist.debug.oculus.ldm.posture_detection.consecutive_not_lying_down_event_threshold`、`persist.debug.oculus.ldm.posture_detection.detection_interval_ms`、`persist.oculus.guardian.package`、`persist.ovr.deviceconfig.svc`、`ro.build.fingerprint`、`sys.hmt.mounted`、`vendor.qti.hardware.cacert.server`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：336。内容 authority（2）：`content://com.facebook.mobileconfigservice.contentprovider`、`content://com.oculus.ocms.publiclibrary/apps/`。

## xrstreamingclient（XR 串流客户端）

包名 `com.oculus.xrstreamingclient`，版本 71.0.0.601.524（代码 665335378），minSdk 24，targetSdk 29，无共享 UID。来源 `system_a/system/app/xrstreamingclient/xrstreamingclient.apk`。

XR 串流客户端（包名 com.oculus.xrstreamingclient）。其 LAUNCHER 活动为 MainActivity。其服务为 SpeedTestService 和 HighwindService，BootReceiver 是开机广播接收器（类名）。

- 签名证书 SHA-256：`c8a2e9bccf597c2fb6dc66bee293fc13f2fc47ec77bc6b2b0d52c11f51192ab8`。
- 类：4299 个，位于 3 个 dex 文件中。原生库：36。资源（assets）：7。
- 组件：活动 2（导出 1）；服务 2（导出 2）；广播接收器 4（导出 2）；内容提供者 0（导出 0）。
- MAIN 动作的活动（2）：`MainActivity`、`NetworkInterruptionActivity`。
- LAUNCHER 活动（1）：`MainActivity`。
- 服务（2）：`SpeedTestService`、`HighwindService`。
- 广播接收器（已列出 4）：`BootReceiver`、`audioDebugReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`。
- 请求的权限：27。敏感子集（6）：`ACCESS_CAMERA_DATA`、`READ_EXTERNAL_STORAGE`、`READ_PHONE_STATE`、`RECORD_AUDIO`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（0）：无。
- AIDL 风格的接口名（21）：`android.accounts.IAccountManager`、`android.app.IActivityClientController`、`android.app.IActivityManager`、`android.app.IActivityTaskManager`、`android.app.IAlarmManager`、`android.app.INotificationManager`、`android.app.job.IJobScheduler`、`android.app.usage.IStorageStatsManager`、`android.content.IClipboard`、`android.content.pm.IPackageManager`、`android.hardware.display.IDisplayManager`、`android.location.ILocationManager`、`android.media.IAudioService`、`android.net.IConnectivityManager`、`android.net.wifi.IWifiManager`、`android.os.IPowerManager`、`android.os.IUserManager`、`android.os.storage.IMountService`、`android.os.storage.IStorageManager`、`android.view.IWindowManager`、`android.view.accessibility.IAccessibilityManager`。
- 类属性字符串（21）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.app_cmdlines`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：269。内容 authority（1）：`content://com.facebook.mobileconfigservice.contentprovider`。

## BodyApiService（身体 API）

包名 `com.oculus.bodyapiservice`，版本 71.0.0.541.758（代码 665334810），minSdk 23，targetSdk 29，无共享 UID。来源 `system_ext_a/priv-app/BodyApiService/BodyApiService.apk`。

身体 API 服务（包名 com.oculus.bodyapiservice）。其服务为 BodyApiService 和 EyeTrackingService（类名）。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：1205 个，位于 1 个 dex 文件中。原生库：43。资源（assets）：0。
- 组件：活动 1（导出 1）；服务 2（导出 2）；广播接收器 0（导出 0）；内容提供者 1（导出 0）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（2）：`BodyApiService`、`EyeTrackingService`。
- 广播接收器（已列出 0）：无。
- 请求的权限：10。敏感子集（1）：`RECORD_AUDIO`。
- 声明的权限（0）：无。
- AIDL 风格的接口名（0）：无。
- 类属性字符串（1）；前十个：`debug.oculus.mqd.off`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：5。内容 authority（0）：无。

## SpatialPersistenceService（空间锚点持久化）

包名 `com.facebook.spatial_persistence_service`，版本 71.0.0.605.527（代码 665334781），minSdk 28，targetSdk 29，无共享 UID。来源 `system_ext_a/priv-app/SpatialPersistenceService/SpatialPersistenceService.apk`。

空间锚点持久化服务（包名 com.facebook.spatial_persistence_service）。其服务为 AnchorPersistenceService、AnchorPersistenceDurableService 和 AnchorPersistenceDumpService。AnchorPersistenceServiceSweeper 是广播接收器。它请求下文列出的 COLOCATION_API 权限。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：1707 个，位于 1 个 dex 文件中。原生库：86。资源（assets）：5。
- 组件：活动 0（导出 0）；服务 3（导出 2）；广播接收器 2（导出 1）；内容提供者 0（导出 0）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（3）：`AnchorPersistenceService`、`AnchorPersistenceDurableService`、`AnchorPersistenceDumpService`。
- 广播接收器（已列出 2）：`AnchorPersistenceBroadcastReceiver`、`AnchorPersistenceServiceSweeper`。
- 请求的权限：20。敏感子集（4）：`COLOCATION_API_GET_MAP_UUID`、`COLOCATION_API_READ_MAP`、`COLOCATION_API_WRITE_MAP`、`WRITE_EXTERNAL_STORAGE`。
- 声明的权限（1）：`com.facebook.spatial_persistence_service.permission.MANAGE_FEATURE_STATE`。
- AIDL 风格的接口名（0）：无。
- 类属性字符串（6）；前十个：`debug.atrace.app_cmdlines`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`、`debug.oculus.mqd.off`、`vendor.qti.hardware.cacert.server`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：260。内容 authority（1）：`content://com.oculus.horizon.gatekeeper/fetch?name=`。

## WearableInputService（可穿戴输入）

包名 `com.oculus.wearableinputservice`，版本 80.0.0.0.0（代码 637286716），minSdk 26，targetSdk 26，无共享 UID。来源 `system_ext_a/priv-app/WearableInputService/WearableInputService.apk`。

可穿戴输入应用（包名 com.oculus.wearableinputservice）。其服务包括 EMGService、WearableInputService 和 WearableInputCLIService2，以及两个测试服务（类名）。名称 EMGService 暗示肌电输入（仅根据名称推断）。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：15190 个，位于 10 个 dex 文件中。原生库：28。资源（assets）：3。
- 组件：活动 1（导出 1）；服务 5（导出 4）；广播接收器 1（导出 1）；内容提供者 0（导出 0）。
- MAIN 动作的活动（1）：`WearableInputTestServiceLauncher`。
- LAUNCHER 活动（1）：`WearableInputTestServiceLauncher`。
- 服务（5）：`EMGService`、`WearableInputCLIService2`、`WearableInputIntegrationTestService`、`WearableInputService`、`WearableInputTestService`。
- 广播接收器（已列出 1）：`PrepareLogsReceiver`。
- 请求的权限：10。敏感子集（5）：`ACCESS_BACKGROUND_LOCATION`、`ACCESS_FINE_LOCATION`、`BLUETOOTH_CONNECT`、`BLUETOOTH_PRIVILEGED`、`BLUETOOTH_SCAN`。
- 声明的权限（1）：`com.oculus.permission.WEARABLE_INPUT_SERVICE`。
- AIDL 风格的接口名（1）：`android.os.IBinder`。
- 类属性字符串（32）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.app_cmdlines`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：274。内容 authority（0）：无。

## FirstTimeNuxEureka（首次设置）

包名 `com.oculus.firsttimenux`，版本 71.2.0.117.949（代码 665345994），minSdk 28，targetSdk 28，无共享 UID。来源 `system_a/system/app/FirstTimeNuxEureka/FirstTimeNuxEureka.apk`。

首次设置应用。其 LAUNCHER 活动为 FirstTimeNuxActivity。ReloginService 和 MiniNuxBroadcastReceiver（类名）分别是重新登录和迷你设置的入口。上传服务（Lollipop、Analytics2 和基于闹钟的）暗示设置后会上传数据（根据名称推断）。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：7639 个，位于 2 个 dex 文件中。原生库：15。资源（assets）：594。
- 组件：活动 3（导出 2）；服务 11（导出 3）；广播接收器 14（导出 4）；内容提供者 1（导出 0）。
- MAIN 动作的活动（1）：`FirstTimeNuxActivity`。
- LAUNCHER 活动（1）：`FirstTimeNuxActivity`。
- 服务（11）：`PanelService`、`ReloginService`、`LollipopUploadSafeService`、`LollipopUploadService`、`Analytics2UploadService`、`AlarmBasedUploadService`、`SystemAlarmService`、`SystemJobService`、`SystemForegroundService`、`HeroKeepAliveService`、`MultiInstanceInvalidationService`。
- 广播接收器（已列出 14）：`FirstTimeNuxBroadcastReceiver`、`MiniNuxBroadcastReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`HighPriUploadRetryReceiver`、`ProfileInstallReceiver`、`ForceStopRunnable$BroadcastReceiver`、`ConstraintProxy$BatteryChargingProxy`、`ConstraintProxy$BatteryNotLowProxy`、`ConstraintProxy$StorageNotLowProxy`、`ConstraintProxy$NetworkStateProxy`、`RescheduleReceiver`、`ConstraintProxyUpdateReceiver`、`DiagnosticsReceiver`。
- 请求的权限：38。敏感子集（4）：`GET_ACCOUNTS`、`READ_EXTERNAL_STORAGE`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（1）：`com.oculus.firsttimenux.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（9）：`com.oculus.assistant.api.IOculusAssistantService`、`com.oculus.assistant.api.IOculusAssistantSubscriber`、`com.oculus.userserver.managerservice.IOculusUserManager`、`com.oculus.vrshell.privateipc.ILifecycleActionIPCService`、`com.oculus.vrshell.privateipc.IModalClosedCallback`、`com.oculus.vrshell.privateipc.IModalClosedIPCService`、`com.oculus.vrshell.privateipc.ISecureScreenIPCService`、`vendor.meta.parfait.IConfigQuery`、`vendor.meta.parfait.ITrigger`。
- 类属性字符串（28）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`、`debug.oculus.graphtier`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：301。内容 authority（4）：`content://com.facebook.mobileconfigservice.contentprovider`、`content://com.oculus.horizon.fbconnect/account`、`content://com.oculus.horizon.parties`、`content://com.oculus.horizon.social`。

## SystemWebView（WebView）

包名 `com.android.webview`，版本 126.0.6478.122（代码 647812201），minSdk 26，targetSdk 34，无共享 UID。来源 `product_a/app/SystemWebView/SystemWebView.apk`。

包名为 com.android.webview（版本 126.0.6478.122）的应用似为系统 WebView 应用（根据包名和文件名 SystemWebView.apk 推断），其原生库 libwebviewchromium.so 和 org.chromium.content 类前缀暗示其以 Chromium 为基础（根据名称推断）。其三个活动中有两个导出：org.chromium.android_webview.devui.MainActivity（动作为 com.android.webview.SHOW_DEV_UI）和 org.chromium.android_webview.nonembedded.LicenseActivity（动作为 android.settings.WEBVIEW_LICENSE）；名称暗示开发者界面和许可证页面（根据名称推断）。其声明了 52 个服务，其中 40 个为 org.chromium.content.app 中的 SandboxedProcessService0 至 SandboxedProcessService39（类名），而 VariationsSeedServer、CrashReceiverService 和 MetricsUploadService 等名称暗示 Variations 种子、崩溃报告和指标（根据名称推断）。其请求 6 项权限，包括 android.permission.INTERNET 和 android.permission.ACCESS_NETWORK_STATE，且未声明任何权限；它还声明了最低 SDK 版本 26 和目标 SDK 版本 34。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：3480 个，位于 1 个 dex 文件中。原生库：4。资源（assets）：86。
- 组件：活动 3（导出 2）；服务 52（导出 50）；广播接收器 0（导出 0）；内容提供者 4（导出 4）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（52）：`StartupFeatureMetadataHolder`、`VariationsSeedServer`、`AwVariationsSeedFetcher`、`CrashReceiverService`、`AwMinidumpUploadJobService`、`DeveloperUiService`、`SafeModeService`、`MetricsBridgeService`、`MetricsUploadService`、`AwComponentUpdateService`、`ComponentsProviderService`、`JsSandboxService0`、`SandboxedProcessService0`、`SandboxedProcessService1`、`SandboxedProcessService2`、`SandboxedProcessService3`、`SandboxedProcessService4`、`SandboxedProcessService5`、`SandboxedProcessService6`、`SandboxedProcessService7`、`SandboxedProcessService8`、`SandboxedProcessService9`、`SandboxedProcessService10`、`SandboxedProcessService11`、`SandboxedProcessService12`、`SandboxedProcessService13`、`SandboxedProcessService14`、`SandboxedProcessService15`、`SandboxedProcessService16`、`SandboxedProcessService17`、`SandboxedProcessService18`、`SandboxedProcessService19`、`SandboxedProcessService20`、`SandboxedProcessService21`、`SandboxedProcessService22`、`SandboxedProcessService23`、`SandboxedProcessService24`、`SandboxedProcessService25`、`SandboxedProcessService26`、`SandboxedProcessService27`、`SandboxedProcessService28`、`SandboxedProcessService29`、`SandboxedProcessService30`、`SandboxedProcessService31`、`SandboxedProcessService32`、`SandboxedProcessService33`、`SandboxedProcessService34`、`SandboxedProcessService35`、`SandboxedProcessService36`、`SandboxedProcessService37`、`SandboxedProcessService38`、`SandboxedProcessService39`。
- 广播接收器（已列出 0）：无。
- 请求的权限：6。敏感子集（1）：`POST_NOTIFICATIONS`。
- 声明的权限（0）：无。
- AIDL 风格的接口名（3）：`android.content.pm.IPackageManager`、`android.widget.directwriting.IDirectWritingService`、`android.widget.directwriting.IDirectWritingServiceCallback`。
- 类属性字符串（1）；前十个：`ro.product.first_api_level`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：19。内容 authority（0）：无。

## Q4BMdm（设备管理）

包名 `com.oculus.q4b.mdm`，版本 103.0.0.0.166（代码 551600503），minSdk 31，targetSdk 31，无共享 UID。来源 `product_a/app/Q4BMdm/Q4BMdm.apk`。

包名 `com.oculus.q4b.mdm` 的应用声明了元数据项 `android.app.device_admin`，其名称暗示设备管理（依据包名中 `mdm` 段的名称读法，推断）。活动 `AdminPolicyComplianceActivity` 和 `GetProvisioningModeActivity`、服务 `DeviceAdminService` 需要 `android.permission.BIND_DEVICE_ADMIN`，广播接收器 `Q4BAdminReceiver` 同样受该权限保护（类名）。已导出的活动 `com.oculus.q4b.mdm.kiosk.KioskSessionSetupActivity` 需要声明的权限 `com.oculus.q4b.mdm.permission.Q4B_KIOSK_SESSION`；同名类也由 `com.oculus.q4bservice` 声明，在该包中需要 `com.oculus.q4bservice.permission.Q4B_KIOSK_SESSION`。服务 `AttestationJobService`、`ConfigJobService`、`RemoteWipeJobService` 和 `KioskResetJobService` 需要 `android.permission.BIND_JOB_SERVICE`（类名；证明、配置、远程擦除和信息亭（kiosk）重置的职能属于名称读法，因此为推断）。该应用请求 `android.permission.INSTALL_PACKAGES`、`android.permission.MANAGE_USERS` 和 `android.permission.USES_POLICY_WIPE_DATA`。已导出的服务 `com.oculus.q4bservice.e2etestsupport.E2EService` 和已导出的内容提供者 `com.oculus.q4b.e2etestsupport.service.E2EContentProvider`（类名）未设置权限属性，名称暗示端到端测试支持（根据名称推断）。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：11131 个，位于 2 个 dex 文件中。原生库：14。资源（assets）：12。
- 组件：活动 3（导出 3）；服务 21（导出 4）；广播接收器 24（导出 7）；内容提供者 1（导出 1）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（21）：`PackageFinderService`、`E2EService`、`DeviceAdminService`、`AttestationJobService`、`AttestationVerdictJobService`、`FbnsService`、`FbnsService`、`KioskResetJobService`、`RemoteConfigService`、`RemoteWipeJobService`、`Q4BDumpService`、`ConfigJobService`、`CertificateRenewalService`、`KioskRemoteSessionResetJobService`、`StateReporterJobService`、`IntegrationUpdateJobService`、`VirtualEnvironmentSetupService`、`SystemAlarmService`、`SystemJobService`、`SystemForegroundService`、`MultiInstanceInvalidationService`。
- 广播接收器（已列出 24）：`Q4BAdminReceiver`、`BootReceiver`、`FbnsBroadcastReceiver`、`MqttSystemBroadcastReceiver`、`KioskCleanupBootReceiver`、`KioskHomeUpdateBootReceiver`、`RemoteConfigOcmsReceiver`、`StateReporterReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`InstallReceiver`、`MixedModeBootReceiver`、`VirtualEnvironmentSetupReceiver`、`VrNotificationActionReceiver`、`PackagesCacheReceiver`、`ForceStopRunnable$BroadcastReceiver`、`ConstraintProxy$BatteryChargingProxy`、`ConstraintProxy$BatteryNotLowProxy`、`ConstraintProxy$StorageNotLowProxy`、`ConstraintProxy$NetworkStateProxy`、`RescheduleReceiver`、`ConstraintProxyUpdateReceiver`、`DiagnosticsReceiver`、`ProfileInstallReceiver`。
- 请求的权限：30。敏感子集（4）：`ACCESS_COARSE_LOCATION`、`ACCESS_FINE_LOCATION`、`REQUEST_INSTALL_PACKAGES`、`WRITE_SETTINGS`。
- 声明的权限（5）：`com.oculus.q4b.mdm.permission.Q4B_KIOSK_SESSION`、`com.oculus.q4b.mdm.permission.GET_Q4B_REMOTE_CONFIG`、`com.oculus.q4b.mdm.permission.Q4B_CONFIG_PROCESSED`、`com.oculus.q4b.mdm.permission.Q4B_STATUS`、`com.oculus.q4b.mdm.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（55）：`com.oculus.aidl.IAuthService2`、`com.oculus.aidl.IAuthenticationCallback`、`com.oculus.oemconfig.aidl.IOemConfigService`、`com.oculus.presence.IApiStatusListener`、`com.oculus.presence.IDebugApi`、`com.oculus.presence.IHomeProvider`、`com.oculus.presence.IHomeProviderListener`、`com.oculus.presence.IHomeProviderShell`、`com.oculus.presence.IIdAnonymizer`、`com.oculus.presence.ITravelManager`、`com.oculus.presence.channels.IAugmentProviderApi`、`com.oculus.presence.channels.IEnvironmentProviderApi`、`com.oculus.presence.channels.IMetaEnvironmentManager`、`com.oculus.presence.channels.IRenderingProviderApi`、`com.oculus.presence.cowatching.ICowatchingEventListener`、`com.oculus.presence.cowatching.ICowatchingManager`、`com.oculus.presence.datasync.IDataListener`、`com.oculus.presence.datasync.IDataSyncManager`、`com.oculus.presence.grouping.IProviderEventListener`、`com.oculus.presence.grouping.capabilities.audiocall.IAudioCallCapability`、`com.oculus.presence.grouping.capabilities.audiocall.IMicStateListener`、`com.oculus.presence.grouping.capabilities.datasync.IDataSyncCapability`、`com.oculus.presence.grouping.capabilities.datasync.IUserUpdateListener`、`com.oculus.presence.hyperspace.ICallsInfoUpdateListener`、`com.oculus.presence.hyperspace.IClientConnection`、`com.oculus.presence.hyperspace.IHyperspaceApiProvider`、`com.oculus.presence.hyperspace.IHyperspaceApiStatusListener`、`com.oculus.presence.hyperspace.IHyperspaceNativeApiProvider`、`com.oculus.presence.hyperspace.IHyperspaceServiceApi`、`com.oculus.presence.hyperspace.IHyperspaceServiceVideoApi`、`com.oculus.presence.hyperspace.IOutgoingVideoObserver`、`com.oculus.presence.hyperspace.IOvrDataMessageCallback`、`com.oculus.presence.hyperspace.IOvrDataReadyCallback`、`com.oculus.presence.hyperspace.IRealtimeSessionInitTransportCallback`、`com.oculus.presence.hyperspace.IStateAgreementObserver`、`com.oculus.presence.presencestate.IPresenceStateListener`、`com.oculus.presence.presencestate.IRichPresenceManager`、`com.oculus.presence.realtimeinfra.IRequestStreamService`、`com.oculus.presence.realtimeinfra.IStream`、`com.oculus.presence.realtimeinfra.IStreamEventHandler`、`com.oculus.presence.stable.IPresenceServiceApi`、`com.oculus.presence.stable.grouping.IClientEventListener`、`com.oculus.presence.stable.grouping.IClientManager`、`com.oculus.presence.stable.grouping.IGroup`、`com.oculus.presence.stable.grouping.IProviderManager`、`com.oculus.presence.stable.party.ICallStateListener`、`com.oculus.presence.stable.party.IParty`、`com.oculus.presence.stable.party.IPartyInviteListener`、`com.oculus.presence.stable.party.IPartyManager`、`com.oculus.presence.stable.party.IStreamListener`、`com.oculus.presence.travel.ITravelCallback`、`com.oculus.shellenv.IApiStatusListener`、`com.oculus.shellenv.IAvatarServiceManager`、`vendor.meta.parfait.IConfigQuery`、`vendor.meta.parfait.ITrigger`。
- 类属性字符串（86）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`、`debug.oculus.force_sideload_apps_can_update`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：439。内容 authority（8）：`content://com.facebook.mobileconfigservice.contentprovider`、`content://com.oculus.identitymanagement.capabilities.contentprovider/account`、`content://com.oculus.identitymanagement.capabilities.contentprovider/account/activeuser/capabilities`、`content://com.oculus.identitymanagement.capabilities.contentprovider/activeuser`、`content://com.oculus.identitymanagement.capabilities.contentprovider/capabilities`、`content://com.oculus.messenger.service.provider`、`content://com.oculus.q4bservice.status`、`content://com.oculus.socialplatform.messenger.service.provider`。

## Q4BService（配置与信息亭）

包名 `com.oculus.q4bservice`，版本 71.0.0.896.252（代码 665348098），minSdk 31，targetSdk 31，无共享 UID。来源 `product_a/priv-app/Q4BService/Q4BService.apk`。

Q4B 服务应用（包名 com.oculus.q4bservice）似乎负责设备配置（provisioning）、信息亭（kiosk）模式和移动设备管理（MDM），其类名如 DeviceOwnerProvisioningActivity、com.oculus.q4b.mdm.kiosk.KioskSessionSetupActivity 和 RemoteWipeJobService（根据类名推断）。其声明了 4 个活动（3 个导出）、17 个服务（4 个导出）、22 个广播接收器（6 个导出）和 5 个内容提供者（4 个导出）。DeviceOwnerProvisioningActivity 需要权限 com.oculus.permission.LAUNCH_PROVISION。SharedModeSessionSetupActivity 需要权限 com.oculus.q4bservice.permission.Q4B_KIOSK_SESSION，同样需要该权限的还有本包声明的 com.oculus.q4b.mdm.kiosk.KioskSessionSetupActivity；该类名也由 com.oculus.q4b.mdm 声明，但其权限为 com.oculus.q4b.mdm.permission.Q4B_KIOSK_SESSION。其请求 40 项权限，包括 android.permission.MANAGE_USERS、android.permission.INSTALL_PACKAGES 和 android.permission.WRITE_SECURE_SETTINGS。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：8314 个，位于 2 个 dex 文件中。原生库：10。资源（assets）：9。
- 组件：活动 4（导出 3）；服务 17（导出 4）；广播接收器 22（导出 6）；内容提供者 5（导出 4）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（17）：`NotificationJobService`、`PackageFinderService`、`E2EService`、`OemConfigService`、`Q4BDumpService`、`IntegrationUpdateJobService`、`ConfigJobService`、`DeviceProvisionedJobService`、`ProcessJobService`、`VirtualEnvironmentSetupService`、`RemoteWipeJobService`、`StateReporterJobService`、`KioskResetService`、`SystemAlarmService`、`SystemJobService`、`SystemForegroundService`、`MultiInstanceInvalidationService`。
- 广播接收器（已列出 22）：`NotificationBootReceiver`、`InstallReceiver`、`MixedModeBootReceiver`、`SharedModeSessionReceiver`、`HomeUpdateBootReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`VirtualEnvironmentSetupReceiver`、`BootReceiver`、`RequiredAppsBootReceiver`、`KioskCleanupBootReceiver`、`KioskHomeUpdateBootReceiver`、`PackagesCacheReceiver`、`ForceStopRunnable$BroadcastReceiver`、`ConstraintProxy$BatteryChargingProxy`、`ConstraintProxy$BatteryNotLowProxy`、`ConstraintProxy$StorageNotLowProxy`、`ConstraintProxy$NetworkStateProxy`、`RescheduleReceiver`、`ConstraintProxyUpdateReceiver`、`DiagnosticsReceiver`、`ProfileInstallReceiver`。
- 请求的权限：40。敏感子集（5）：`ACCESS_COARSE_LOCATION`、`ACCESS_FINE_LOCATION`、`MANAGE_EXTERNAL_STORAGE`、`REQUEST_INSTALL_PACKAGES`、`WRITE_SETTINGS`。
- 声明的权限（5）：`com.oculus.permission.LAUNCH_PROVISION`、`com.oculus.q4bservice.permission.Q4B_KIOSK_SESSION`、`com.oculus.q4bservice.status.ACCESS_PERMISSION`、`com.oculus.q4bservice.permission.Q4B_READ_IMAGE`、`com.oculus.q4bservice.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（3）：`com.oculus.aidl.IAuthService2`、`com.oculus.aidl.IAuthenticationCallback`、`com.oculus.oemconfig.aidl.IOemConfigService`。
- 类属性字符串（34）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`、`debug.oculus.force_sideload_apps_can_update`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：391。内容 authority（2）：`content://com.facebook.mobileconfigservice.contentprovider`、`content://com.oculus.q4bservice.status`。

## OCMS（应用安装）

包名 `com.oculus.ocms`，版本 71.0.0.884.253（代码 560606331），minSdk 32，targetSdk 32，无共享 UID。来源 `system_ext_a/priv-app/OCMS/OCMS.apk`。

OCMS 应用（包名 com.oculus.ocms，位于 system_ext_a/priv-app/OCMS/OCMS.apk）没有声明任何活动，其组件名称暗示与应用安装、更新和应用资料库有关（根据名称推断）。其服务包括 AppManagerService、LocationService 和 MauiAppManagerService，这三者均为导出，且各自指定了以 com.oculus.ocms 开头的权限；其广播接收器包括 Installer2StatusReceiver 和 OCMSPushReceiver（类名）。其内容提供者中有三个（LibraryProvider、PublicLibraryProvider 和 TestProvider）均为导出且未指定权限，并请求了 63 项权限，其中包括 android.permission.INSTALL_PACKAGES、android.permission.DELETE_PACKAGES 和 android.permission.QUERY_ALL_PACKAGES，这些权限名称暗示与包管理有关（根据名称推断）。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：20160 个，位于 11 个 dex 文件中。原生库：39。资源（assets）：11。
- 组件：活动 0（导出 0）；服务 29（导出 9）；广播接收器 37（导出 20）；内容提供者 6（导出 3）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（29）：`PackageFinderService`、`OculusDownloadService`、`LoginHandlersService`、`AppManagerService`、`AntiPiracyIntentService`、`AntiPiracyJobService`、`LowBatteryNotificationJobService`、`CloudProfilesUploadJobService`、`DefaultAppsHighPriorityJobService`、`DefaultAppsSetupJobService`、`DefaultAppsPostNUXInstallJobService`、`OCMSDumpService`、`ExtractShader`、`BinaryCheckUpdateIntentService`、`BinaryRemovalIntentService`、`ThirdPartyPackageWarnOrBlockIntentService`、`LocationService`、`MauiAppManagerService`、`PushFGService`、`AutoUpdateJobService`、`AutoUpdateWhenChargingJobService`、`InstallerService`、`ConsistencyJobService`、`DozeDelayService`、`UpdatePreShutdownFGService`、`SystemAlarmService`、`SystemJobService`、`SystemForegroundService`、`MultiInstanceInvalidationService`。
- 广播接收器（已列出 37）：`CloudBackupPreShutdownTask$QueryTaskBeforeShutdownReceiver`、`CloudBackupPreShutdownTask$RunTaskReceiver`、`ForegroundAppListener$ImmersiveAppChangedReceiver`、`OculusDownloadManagerReceiver`、`AuthBroadcastReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`Q4BRemoteConfigReceiver`、`Q4BActionReceiver`、`LowBatteryUnmountAlarmReceiver`、`HighPowerUsageNotificationReceiver`、`CloudProfilesBroadcastReceiver`、`DefaultAppsBroadcastReceiver`、`Installer2StatusReceiver`、`PackageRemovedReceiver`、`ThirdPartyPackageStartedReceiver`、`InstallExistingReceiver`、`Q4BAppActionReceiver`、`OCMSPushReceiver`、`AutoUpdateOnScreenOffAlarmReceiver`、`PackagesCacheReceiver`、`SharedModeSessionReceiver`、`GrantAppUserConsentReceiver`、`VrNotificationActionReceiver`、`UninstallBroadcastReceiver`、`UpdatePreShutdownTask$QueryTaskBeforeShutdownReceiver`、`UpdatePreShutdownTask$RunTaskReceiver`、`ForceStopRunnable$BroadcastReceiver`、`ConstraintProxy$BatteryChargingProxy`、`ConstraintProxy$BatteryNotLowProxy`、`ConstraintProxy$StorageNotLowProxy`、`ConstraintProxy$NetworkStateProxy`、`RescheduleReceiver`、`ConstraintProxyUpdateReceiver`、`DiagnosticsReceiver`、`InstallerServiceBootReceiver`、`ProfileInstallReceiver`。
- 请求的权限：63。敏感子集（6）：`PACKAGE_USAGE_STATS`、`POST_NOTIFICATIONS`、`READ_EXTERNAL_STORAGE`、`SYSTEM_ALERT_WINDOW`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（12）：`com.oculus.permission.UPDATE_Q4B_REMOTE_CONFIG`、`com.oculus.ocms.am.service.permission.ACCESS`、`com.oculus.ocms.antipiracy.internal.REQUEST_DEVELOPER_STATUS`、`com.oculus.ocms.installer2.INSTALL_EVENT_PERMISSION`、`com.oculus.ocms.installer2.READ_EXTRACTED_FILES`、`com.oculus.ocms.BINARY_CHECK_UPDATE`、`com.oculus.ocms.BINARY_REMOVAL`、`com.oculus.ocms.locationservices.permission.ACCESS`、`com.oculus.ocms.maui.permission.ACCESS`、`com.oculus.horizon.notifications.receivers.RECEIVE_SILENT_NOTIFS`、`com.oculus.ocms.updates.permission.UPDATE_NOW`、`com.oculus.ocms.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（11）：`com.oculus.aidl.IAuthService2`、`com.oculus.aidl.IAuthenticationCallback`、`com.oculus.aidl.ILocationCallback`、`com.oculus.aidl.ILocationServiceInterface`、`com.oculus.aidl.backuptransportservice.IExtractedArchiveStager`、`com.oculus.aidl.backuptransportservice.IStageForRestoreCallback`、`com.oculus.assistant.api.IOculusAssistantService`、`com.oculus.assistant.api.IOculusAssistantSubscriber`、`com.oculus.userserver.managerservice.IOculusUserManager`、`vendor.meta.parfait.IConfigQuery`、`vendor.meta.parfait.ITrigger`。
- 类属性字符串（114）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.app_cmdlines`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：531。内容 authority（2）：`content://com.facebook.mobileconfigservice.contentprovider`、`content://downloads/my_downloads`。

## Guardian（边界）

包名 `com.oculus.guardian`，版本 71.0.0.613.530（代码 665334697），minSdk 24，targetSdk 23，无共享 UID。来源 `system_ext_a/priv-app/Guardian/Guardian.apk`。

仅根据名称推断（Guardian 名称、类前缀 com.oculus.vrguardianservice 和 BOUNDARY_VISIBILITY 权限），Guardian 应用（包名 com.oculus.guardian，版本 71.0.0.613.530，版本代码 665334697，最低 SDK 24，目标 SDK 23）似为 VR guardian（边界）应用。它未声明任何活动；其服务为 VrGuardianService、VrGuardianUserService 和 VrGuardianVideoCachingJobService（类名），前两个为导出。其广播接收器包括 VrGuardianUserBroadcastReceiver 和 DeviceConfigClientReceiver（类名），DeviceConfigClientReceiver 为导出，并带有权限 com.oculus.permission.DEVICE_CONFIG_PUSH_TO_CLIENT。它声明了三个权限：com.oculus.permission.GUARDIAN_USER_CONTROL、com.oculus.permission.BOUNDARY_VISIBILITY 和 com.oculus.guardian.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：751 个，位于 1 个 dex 文件中。原生库：4。资源（assets）：8。
- 组件：活动 0（导出 0）；服务 3（导出 2）；广播接收器 5（导出 2）；内容提供者 2（导出 0）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（3）：`VrGuardianService`、`VrGuardianUserService`、`VrGuardianVideoCachingJobService`。
- 广播接收器（已列出 5）：`VrGuardianUserBroadcastReceiver`、`VrGuardianVideoCachingBroadcastReceiver`、`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`ProfileInstallReceiver`。
- 请求的权限：47。敏感子集（4）：`ACCESS_CAMERA_DATA`、`POST_NOTIFICATIONS`、`SYSTEM_ALERT_WINDOW`、`WRITE_SETTINGS`。
- 声明的权限（3）：`com.oculus.permission.GUARDIAN_USER_CONTROL`、`com.oculus.permission.BOUNDARY_VISIBILITY`、`com.oculus.guardian.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（2）：`com.oculus.vrguardianservice.IVrGuardianService`、`com.oculus.vrguardianservice.IVrGuardianUserController`。
- 类属性字符串（4）；前十个：`debug.oculus.mqd.off`、`persist.oculus.guardian_json_cmds_user_build`、`persist.ovr.deviceconfig.svc`、`ro.build.fingerprint`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：46。内容 authority（2）：`content://com.facebook.mobileconfigservice.contentprovider`、`content://com.oculus.ocms.publiclibrary/apps/`。

## SkyguardService

包名 `com.meta.internal.skyguard`，版本 None（代码 None），minSdk 29，targetSdk 29，共享 UID `android.uid.system`。来源 `system_ext_a/priv-app/SkyguardService/SkyguardService.apk`。

SkyguardService 应用（包名 `com.meta.internal.skyguard`）的 APK 位于 `system_ext_a/priv-app/SkyguardService/SkyguardService.apk`，声明共享 UID `android.uid.system`，并声明最低 SDK 级别为 29、目标 SDK 级别为 29，未记录版本名称和版本号（两者均为空值）。其原生库名称（例如其 34 个原生库中的 `libskyguardservice_native.so` 和 `libandroid_skyguard_database.so`）提示它是一项带有原生代码和数据库的服务（仅根据名称推断）。其清单声明了一个导出的服务 `com.meta.internal.skyguard.SkyguardService`，带有 MAIN 动作和 LAUNCHER 类别的 intent 过滤器；另有一个未导出的广播接收器 `com.meta.internal.skyguard.BootReceiver`，带有针对 `android.intent.action.BOOT_COMPLETED` 和 `android.intent.action.LOCKED_BOOT_COMPLETED` 的 intent 过滤器；它未声明任何活动，并请求一项权限 `android.permission.RECEIVE_BOOT_COMPLETED`。其签名证书文件为 `META-INF/PLATFORM.RSA`，SHA-256 摘要以 `c8a2e9bccf597c2f` 开头，可能是平台证书（仅根据共享 UID 推断；不以文件名为据，因为 Telecom 的相同摘要存放在 `META-INF/CERT.RSA` 中）。

- 签名证书 SHA-256：`c8a2e9bccf597c2fb6dc66bee293fc13f2fc47ec77bc6b2b0d52c11f51192ab8`。
- 类：89 个，位于 1 个 dex 文件中。原生库：34。资源（assets）：0。
- 组件：活动 0（导出 0）；服务 1（导出 1）；广播接收器 1（导出 0）；内容提供者 0（导出 0）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（1）：`SkyguardService`。
- 广播接收器（已列出 1）：`BootReceiver`。
- 请求的权限：1。敏感子集（0）：无。
- 声明的权限（0）：无。
- AIDL 风格的接口名（0）：无。
- 类属性字符串（0）；前十个：无。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：0。内容 authority（0）：无。

## Integrity（滥用举报）

包名 `com.oculus.integrity`，版本 71.0.0.618.530（代码 665335042），minSdk 28，targetSdk 28，无共享 UID。来源 `system_ext_a/priv-app/Integrity/Integrity.apk`。

Integrity 应用（包名 `com.oculus.integrity`）的类名暗示其与滥用举报和视频处理有关（根据名称推断）。其服务 `PanelService` 已导出、未设置权限属性，声明进程为 `:abuse`，并带有 `com.oculus.vrshell.SHELL_MAIN` 和 `com.facebook.secure.packagefinder.intent.ACTION_QUERY_PACKAGES` 的 Intent 过滤器。其导出的服务 `RecordingService` 要求调用方持有 `com.oculus.horizon.permission.OVR_PLATFORM_BROADCAST`；本 APK 声明的是 `com.oculus.integrity.permission.OVR_PLATFORM_BROADCAST`。服务 `VideoUploaderService`、`VideoFileToCacheCopierService` 和 `VideoURIFetcherService` 均位于 `abusereport` 包中且未导出，提供者 `AbuseReportFileProvider` 同样未导出。其请求 `android.permission.READ_EXTERNAL_STORAGE` 和 `android.permission.WRITE_EXTERNAL_STORAGE` 权限，这与视频文件相关的类名相符（根据名称推断）。

- 签名证书 SHA-256：`fac61745dc0903786fb9ede62a962b399f7348f0bb6f899b8332667591033b9c`。
- 类：7503 个，位于 2 个 dex 文件中。原生库：21。资源（assets）：18。
- 组件：活动 1（导出 0）；服务 15（导出 3）；广播接收器 12（导出 3）；内容提供者 2（导出 0）。
- MAIN 动作的活动（0）：无。
- LAUNCHER 活动（0）：无。
- 服务（15）：`PanelService`、`VideoFileToCacheCopierService`、`VideoUploaderCleanerService`、`RecordingService`、`VideoURIFetcherService`、`VideoUploaderService`、`LollipopUploadSafeService`、`LollipopUploadService`、`Analytics2UploadService`、`AlarmBasedUploadService`、`SystemAlarmService`、`SystemJobService`、`SystemForegroundService`、`HeroKeepAliveService`、`MultiInstanceInvalidationService`。
- 广播接收器（已列出 12）：`DeviceConfigClientReceiver`、`DeviceConfigClientDebugReceiver`、`HighPriUploadRetryReceiver`、`ProfileInstallReceiver`、`ForceStopRunnable$BroadcastReceiver`、`ConstraintProxy$BatteryChargingProxy`、`ConstraintProxy$BatteryNotLowProxy`、`ConstraintProxy$StorageNotLowProxy`、`ConstraintProxy$NetworkStateProxy`、`RescheduleReceiver`、`ConstraintProxyUpdateReceiver`、`DiagnosticsReceiver`。
- 请求的权限：33。敏感子集（4）：`GET_ACCOUNTS`、`READ_EXTERNAL_STORAGE`、`WRITE_EXTERNAL_STORAGE`、`WRITE_SETTINGS`。
- 声明的权限（2）：`com.oculus.integrity.permission.OVR_PLATFORM_BROADCAST`、`com.oculus.integrity.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`。
- AIDL 风格的接口名（16）：`com.oculus.assistant.api.IOculusAssistantService`、`com.oculus.assistant.api.IOculusAssistantSubscriber`、`com.oculus.metacam.ipc.ICaptureServiceBinder`、`com.oculus.metacam.ipc.ICastStateListener`、`com.oculus.metacam.ipc.ILivestreamStateListenerV2`、`com.oculus.metacam.ipc.ILivestreamingServiceBinder`、`com.oculus.metacam.ipc.IMetaCamServiceBinder`、`com.oculus.metacam.ipc.IOnCaptureStoppedListener`、`com.oculus.metacam.ipc.IScreenshotStateListener`、`com.oculus.metacam.ipc.IVirtualCameraServiceBinder`、`com.oculus.metacam.ipc.IVirtualCameraStateListener`、`com.oculus.userserver.managerservice.IOculusUserManager`、`com.oculus.vrshell.privateipc.ILifecycleActionIPCService`、`com.oculus.vrshell.privateipc.ISecureScreenIPCService`、`vendor.meta.parfait.IConfigQuery`、`vendor.meta.parfait.ITrigger`。
- 类属性字符串（27）；前十个：`dalvik.vm.heapgrowthlimit`、`dalvik.vm.heapmaxfree`、`dalvik.vm.heapminfree`、`dalvik.vm.heapstartsize`、`dalvik.vm.heaptargetutilization`、`debug.atrace.tags.enableflags`、`debug.fbsystrace.force_java`、`debug.fbsystrace.tags`、`debug.fbsystrace.trace_load`、`debug.oculus.graphtier`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：295。内容 authority（1）：`content://com.facebook.mobileconfigservice.contentprovider`。

## Telecom（电话）

包名 `com.android.server.telecom`，版本 12（代码 32），minSdk 32，targetSdk 32，共享 UID `android.uid.system`。来源 `system_a/system/priv-app/Telecom/Telecom.apk`。

Telecom 应用（包名 `com.android.server.telecom`，版本 12，共享 UID `android.uid.system`）是一款电话应用（根据名称推断）。其两个服务 TelecomService（过滤动作 `android.telecom.ITelecomService`）和 BluetoothPhoneService（过滤动作 `android.bluetooth.IBluetoothHeadsetPhone`）均为导出。其 9 个活动中有 4 个导出，其中包括 UserCallActivity，它要求调用方持有权限 `android.permission.CALL_PHONE`，并声明过滤动作 `android.intent.action.CALL`。在其请求的 28 项权限中，名称与通话相关的有若干项，包括 `android.permission.CALL_PRIVILEGED`、`android.permission.MODIFY_PHONE_STATE`、`android.permission.BIND_INCALL_SERVICE`、`android.permission.BIND_CONNECTION_SERVICE` 和 `android.permission.HANDLE_CALL_INTENT`（根据名称推断）。

- 签名证书 SHA-256：`c8a2e9bccf597c2fb6dc66bee293fc13f2fc47ec77bc6b2b0d52c11f51192ab8`。
- 类：3092 个，位于 1 个 dex 文件中。原生库：0。资源（assets）：0。
- 组件：活动 9（导出 4）；服务 2（导出 2）；广播接收器 2（导出 1）；内容提供者 0（导出 0）。
- MAIN 动作的活动（2）：`RespondViaSmsSettings`、`EnableAccountPreferenceActivity`。
- LAUNCHER 活动（0）：无。
- 服务（2）：`BluetoothPhoneService`、`TelecomService`。
- 广播接收器（已列出 2）：`TelecomBroadcastReceiver`、`AppUninstallBroadcastReceiver`。
- 请求的权限：28。敏感子集（6）：`CALL_PRIVILEGED`、`MODIFY_PHONE_STATE`、`READ_CALL_LOG`、`READ_EXTERNAL_STORAGE`、`SEND_SMS`、`WRITE_CALL_LOG`。
- 声明的权限（5）：`android.permission.BROADCAST_CALLLOG_INFO`、`android.permission.PROCESS_CALLLOG_INFO`、`android.permission.BROADCAST_PHONE_ACCOUNT_REGISTRATION`、`android.permission.PROCESS_PHONE_ACCOUNT_REGISTRATION`、`android.permission.HANDLE_CALL_INTENT`。
- AIDL 风格的接口名（0）：无。
- 类属性字符串（6）；前十个：`debug.enable_enhanced_calling`、`persist.log.tag`、`persist.logd.logpersistd`、`persist.logd.logpersistd.buffer`、`ro.boot.revision`、`ro.storage_manager.enabled`。这些是字符串池中的匹配，不是已确认的读取。
- Intent 动作字符串：39。内容 authority（1）：`content://icc/adn/`。
