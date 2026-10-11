# 24 - Native tools and HAL binaries

[中文版](24-native-tools-and-hal-binaries.zh-CN.md)

## Summary

The group holds 346 rows: 157 regular files (132 PIE executables and 17 shared-object-format executables, plus 8 scripts) and 189 symlinks, spread over vendor_a (241), system_ext_a (55), odm_a (38), system_a (10) and product_a (2). The core is a toybox multi-call binary (172 applet symlinks) and a toolbox multi-call binary (6 symlinks), an mksh shell, a vendor binder service manager, and 14 HAL service executables (10 in vendor_a/bin/hw, 4 in odm_a/bin/hw) that register HIDL or AIDL interfaces and are started by init services. The odm partition carries the SyncBoss sensor-controller toolset: fw_init, a oneshot firmware updater that reads /odm/etc/fw_init.cfg, plus 17 shared-object-format tools that each export main. system_ext_a carries 50 platform service and command-line executables and 4 shell wrappers; product_a carries 2 crash-uploader executables. All 157 files match the manifest sha256 and byte size, and init or SELinux text references 62 of them by full path.
Role table: `data/subsystems/native-tools-and-hal-binaries/roles.tsv` (346 files: 190 verified, 139 observed, 16 inferred, 1 unverified).

## Verified claims

- **G01** (verified): The group extract holds 346 rows, partitioned as odm_a 38, product_a 2, system_a 10, system_ext_a 55 and vendor_a 241.
- **G02** (verified): 157 rows are regular files (132 application/x-pie-executable, 17 application/x-sharedlib, 7 text/plain, 1 text/x-shellscript) and 189 rows are symlinks.
- **G03** (verified): All 157 regular files match the manifest sha256 and byte size on disk.
- **G04** (verified): All 189 symlink rows exist as symbolic links on the mounted partitions.
- **G05** (verified): Symlink targets are 172 links to toybox_vendor, 6 to toolbox, 1 to meta_screencap, and 10 absolute links from system_a/odm/* to /vendor/odm/<name> (app, bin, etc, firmware, framework, lib, lib64, overlay, priv-app, usr).
- **G06** (verified): 171 of the 172 toybox applet link names appear as exact strings in the toybox_vendor string table; the name '[' does not.
- **G07** (observed): toybox_vendor identifies its version as 0.8.4-android and its top-level usage line is 'usage: toybox [--long | --help | --version | [COMMAND] [ARGUMENTS...]]'.
- **G08** (verified): toybox_vendor is an AArch64 position-independent executable, stripped, with interpreter /system/bin/linker64 and NEEDED libcrypto.so, libz.so, liblog.so, libprocessgroup.so, libselinux.so, libc.so, libm.so and libdl.so; it has no SONAME.
- **G09** (verified): toolbox provides six applet links (getprop, setprop, start, stop, getevent, modprobe), and each applet name appears once in the toolbox string table.
- **G11** (observed): The vendor shell is a MirBSD KSH R59 (mksh) build whose default ENV file is /vendor/etc/mkshrc and whose default PATH lists /odm/bin and /vendor/bin.
- **G12** (verified): /vendor/etc/mkshrc exists as a 389-byte file in vendor_a/etc.
- **G13** (observed): The vendor binder service manager is started by an init service on /dev/vndbinder in class core, runs as system with group system readproc, and restarts class main, hal and early_hal when it restarts.
- **G14** (observed): vndservicemanager is labelled vndservicemanager_exec by vendor file contexts, and the vendor vndservice contexts file maps the manager entry to service_manager_vndservice.
- **G15** (observed): vndservice is a command-line client for the vendor service manager with subcommands list, check SERVICE and call SERVICE CODE followed by typed arguments.
- **G16** (verified): hal_proxy_daemon has NEEDED libnl.so, libcutils.so, liblog.so, libwifi-hal-qcom.so, libhardware_legacy.so, libc++.so, libc.so, libm.so and libdl.so.
- **G18** (verified): hal_proxy_daemon has no full-path or basename reference in the mounted init or SELinux text.
- **G19** (observed): The light HAL is started by init service light-hal-2-0 in class hal, runs as system, and registers android.hardware.light@2.0::ILight and vendor.oculus.hardware.light@2.0::ILight; an early-boot block sets owner and mode on /sys/class/leds/{blue,green,red,white:indicator} nodes.
- **G20** (observed): The light HAL is labelled hal_light_default_exec in vendor file contexts, and the vendor hwservice contexts map vendor.oculus.hardware.light::ILight to hal_light_hwservice.
- **G22** (observed): The KeyMint service is started by 'on init' (start vendor.keymint-qti), runs as service vendor.keymint-qti in class early_hal with group system drmrpc, and is labelled vendor_hal_keymint_qti_exec at vendor file contexts line 489.
- **G24** (verified): The KeyMint binary has NEEDED entries for libqtikeymint.so, libcrypto.so, android.hardware.keymaster@4.0.so, libhidlbase.so and three android.hardware.security.*-V1-ndk_platform.so libraries, in addition to the common C and C++ runtime.
- **G27** (observed): fw_init is a oneshot, disabled-at-boot init service of class main that runs as system with groups system, graphics and input, and writes to /dev/kmsg.
- **G28** (observed): The vendor init script sets vendor.syncbosshal.wait_for_service to true and then starts fw_init, with a comment that fw_init suppresses SyncBoss reset until sensorservice starts.
- **G29** (verified): The fw_init binary matches the manifest sha256 62380bdbfd4280475b8ad8deddeed1589804e6d8b89e960ae6fafe21b1b59be7.
- **G30** (verified): fw_init is a PIE with interpreter /system/bin/linker64 and NEEDED libsyncboss.so, libovrfwupdate.so, libiad-utils.so, libpng.so, libdeviceid.so, vendor.oculus.hardware.telemetry@1.0.so and the common runtime libraries; 157 dynamic symbols are defined.
- **G31** (observed): fw_init carries literal strings for the SyncBoss device node, the lock file pattern, the fw_init.cfg path, the boot-duration property and its SWD and SPL update and nRF version messages.
- **G32** (observed): /odm/etc/fw_init.cfg has three sections: [syncboss] (path /odm/firmware/syncboss.bin, id 1, busy_path under /sys/devices/virtual/misc/syncboss0), [secondary] (path /odm/firmware/santana_manager.bin, id 0) and [blu] (path /vendor/firmware/blu.bin, version_path /vendor/firmware/blu.toml, part bb51-8kb, uart /dev/ttyHS13, bootloader and reset sysfs paths, bootloader_update_delay_s 5, app_update_delay_ms 500).
- **G33** (verified): Two of the firmware images named by fw_init.cfg are present under odm_a/firmware: syncboss.bin (366076 bytes) and santana_manager.bin (60240 bytes), each an ARM Cortex-M image; /vendor/firmware/blu.bin and blu.toml are absent from vendor_a/firmware.
- **G34** (observed): fw_init is labelled syncboss_exec through the odm path in vendor file contexts (line 97); the SyncBoss device nodes /dev/syncboss.* carry syncboss_device (line 221).
- **G35** (observed): A property trigger on vendor.fw_init.duration_ms writes the FwInitDuration value to bootstat.
- **G36** (verified): All 17 shared-object-format executables under odm_a/bin define a main symbol and use interpreter /system/bin/linker64, so they are launched as programs although their ELF type is shared object.
- **G37** (verified): syncboss_getstatus has NEEDED liblog.so, libm.so, libdl.so and libc.so only (libsyncboss code is not a NEEDED dependency) and defines main as a global function at 0x5ed4 of 304 bytes.
- **G38** (observed): The SyncBoss tools reference a shared set of property keys: persist.vendor.syncbosshal.disable_fw_version_check, persist.vendor.syncbosshal.force_single_pulsar_tx_slot, persist.vendor.syncbosshal.sensor_controller.imu, vendor.syncbosshal.pulsar_afh and vendor.syncbosshal.pulsar_blocklist.
- **G39** (observed): vendor property contexts assign the syncbosshal keys to vendor_syncboss_prop and the fw_init keys to vendor_fw_init_prop.
- **G40** (observed): The SyncBoss tool options are: print_fw_version (--check-fw-version, --check-fw-flavor, --short, --build-flavor, --help); powerstate (--get, --watch); prox (--get, --get-raw, --get-raw-interactive, --watch, --watch-raw, --run-calibration and others).
- **G41** (observed): syncboss_consumers_ctl is a shell script that starts, stops or restarts a fixed list of seven services in order, honours --exclude, takes an flock on /data/local/tmp/fw-init-syncboss.lck during start, and treats vendor.oculus.sensors-mock-hal-1-0 as an extra service on stop.
- **G42** (verified): syncboss_consumers_ctl is not referenced by any init rc or SELinux text in the mounted partitions; its only textual reference is in its own file.
- **G43** (observed): board_init is a oneshot init service of class core that runs as system, invoked as board_init --setprops, and its option text says it sets properties rather than reading and dumping them.
- **G44** (verified): board_init has NEEDED libbase.so, libcutils.so, libdeviceid.so, libgflags.so, liblog.so, libosutils.so, libc++.so, libc.so, libm.so and libdl.so, and its property names are ro.product.device, ro.vendor.hw.deviceid and ro.vendor.product.model_extended.
- **G45** (observed): perf_service is started by init service perfservice with --schedutil and advertises IPower 1.0 to 1.3, IThermal 1.0 and 2.0 and IClocks 1.0 as default instances; it is labelled hal_perfservice_exec at vendor file contexts line 102, and vendor.scheduler. properties carry vendor_scheduler_prop.
- **G46** (verified): perf_service has NEEDED android.hardware.power@1.0 to 1.3, android.hardware.thermal@1.0 and 2.0, vendor.oculus.hardware.clocks@1.0.so, libthermalhalutils.so and libmros_vendor_sysprops.so, among others.
- **G47** (observed): The USB HAL 1.2 service (odm) is init service vendor.usb-hal-1-2 in class hal running as root, registers IUsb 1.0 to 1.2 and IUsbGadget 1.0 to 1.2, has a 20-second delayed shutdown, and is labelled hal_usb_oculus_exec at vendor file contexts line 101; its configfs strings refer to /config/usb_gadget/g1.
- **G48** (observed): vrdevicemanagerserver is init service vrdevice in class late_start running as system with SYS_NICE, labelled vrdevicemanagerserver_exec at system_ext file contexts line 82, and it reads /system/etc/device_props.json and /system/etc/display.conf.
- **G49** (verified): vrdevicemanagerserver has NEEDED android.hardware.power@1.0 and 1.1, android.hardware.thermal@2.0, vendor.oculus.hardware.clocks@1.0, vendor.oculus.hardware.graphics.composer@1.1, libjsoncpp.so and libprocessgroup.so, among others.
- **G50** (verified): The 32-bit executables in the group use interpreter /system/bin/linker, and the 64-bit ones use /system/bin/linker64; boringssl_self_test32 is one of the 32-bit ones.
- **G52** (verified): Of the 157 regular files, 94 have no full-path or basename reference in the mounted init or SELinux text, 62 have a full-path reference, and 1 has a basename-only reference.
- **G53** (verified): odm_a/etc/selinux holds only binary precompiled policy and two checksum files, so odm binary labels are taken from vendor file contexts entries written with the (odm|vendor/odm) prefix.
- **G54** (verified): No explicit file-context entry exists for /vendor/bin/toolbox, /vendor/bin/sh or /vendor/bin/toybox_vendor in vendor or system_ext file contexts.
- **G55** (verified): roles.tsv holds one header and 346 data rows with unique partition and path pairs; confidence totals are 190 verified, 139 observed, 16 inferred and 1 unverified.

## Claims with corrected wording

- **G10** (observed): toolbox carries the getprop and setprop usage strings, imports __system_property_foreach and __system_property_read_callback from libc, and its string table contains the path /dev/__properties__/property_info. Reading through that file is inferred from the path string; no import or open call shows it.
- **G17** (observed): hal_proxy_daemon imports wifi_* functions (for example wifi_initialize, wifi_get_ifaces and wifi_start_gscan, which are undefined dynamic symbols resolved from its NEEDED library libwifi-hal-qcom.so), and its string table holds WifiLogger command descriptions numbered 1 to 13.
- **G21** (observed): The light HAL binary references, as undefined dynamic-symbol imports, the method symbols of android.hardware.light.V2_0.ILight, vendor.oculus.hardware.light.V2_0.ILight and android.hidl.base.V1_0.IBase (31 symbols, none defined in the binary).
- **G23** (observed): The KeyMint binary carries symbols in the android::hardware::security::keymint namespace for IKeyMintDevice, IKeyMintOperation and IRemotelyProvisionedComponent. Their ::hash members are defined (weak), and the IKeyMintDevice and IRemotelyProvisionedComponent ::descriptor members are undefined imports.
- **G25** (verified): Within the *_contexts files, keymint appears only in vendor_file_contexts (lines 70 and 489), and the vendor service and hardware-service context files have no keymint entry. Across the whole etc/selinux directory, keymint also appears as type names in vendor_sepolicy.cil (51 matches) and plat_pub_versioned.cil (43 matches).
- **G26** (observed): The health HAL 2.1 service health-hal-2-1 (classes hal and charger; user and group system; capabilities WAKE_ALARM and BLOCK_SUSPEND) is labelled hal_health_default_exec at vendor_file_contexts line 41. Its binary references, as undefined imports, android.hardware.health.V2_0.IHealth and android.hardware.health.V2_0.IHealthInfoCallback.

## Excluded claims

| ID | Claim | Reason |
|---|---|---|
| G51 | None of the 149 ELF images in the group is unstripped; each is reported as stripped. | Two rows report 'not stripped': odm_a/bin/peripheralconfig-tool (has debug_info sections) and system_ext_a/bin/bpftrace (has a .symtab section). The claimed zero comes from matching the hyphenated 'not-stripped', which the type-detection output never prints (it prints 'not stripped'). The work-directory summary records all 149 as 'stripped', which is wrong for these two rows. |

## Gaps

- Four toybox applet links in vendor_a/bin (groups, hwclock, tee, touch) are classified under config-files, display-and-composer, security-keystore-crypto and sensors-mcu-meta-hal; they are not among this group's 172 toybox rows.
- The toybox applet '[' has a verified link target, but its name is not present as an exact string in toybox_vendor; its applet behaviour was not confirmed.
- Purposes of most SyncBoss shared-object-format tools (vendor_input_tool, syncboss_rftest, syncboss_streammag, syncboss_streamimu, syncboss_per_tool, syncboss_perf_tool, syncboss_input_tool, syncboss_catty, syncboss_rf_blocklist_tool) rest on names and shared strings only; syncboss_per_tool's purpose is not determined.
- 94 of 157 files have no full-path or basename reference in the mounted init or SELinux text, so their invocation paths (other binaries, scripts or manual use) were not established; this includes hal_proxy_daemon and most SyncBoss tools.
- odm_a/etc/selinux contains only precompiled binary policy, which was not decoded; odm binary labels come from vendor_file_contexts regex entries and are not confirmed against the compiled policy.
- No executable was run; startup order, boot-time property values and the effect of configuration keys (for example persist.vendor.syncbosshal.disable_fw_version_check) were not observed and are reported only as key names.
- The consumer of syncboss_consumers_ctl is not found in mounted init or SELinux text; the lock-file name suggests coupling with fw_init but this is unverified.
- Firmware images named by fw_init.cfg (syncboss.bin, santana_manager.bin) are classified under dsp-and-remote-processors; their contents were not analysed. /vendor/firmware/blu.bin and blu.toml are absent from the mount.
- The shared libraries in the NEEDED lists (libsyncboss.so, libqtikeymint.so, libwifi-hal-qcom.so, libovrfwupdate.so, libmros_vendor_sysprops.so and others) were not checked against other subsystems and their own contents were not analysed.
- Not deep-dived beyond the role line in roles.tsv: set_hint, flashswap.sh, set_usb_cdc_mac.sh, virtualcontroller_ctl, trackingservice_ctl, killtop.sh, btcontrol.sh, pdfu_test, bpftrace (51.9 MB), and the remaining system_ext_a service executables.
- system_ext_a/bin/screencap is a symlink to meta_screencap, which is outside this group and was not analysed.
- The 16 inferred and 1 unverified roles in roles.tsv are name-based (for example syncboss_streammag, dramstress, xrframeconverter, syncboss_per_tool) and were not confirmed from content.
