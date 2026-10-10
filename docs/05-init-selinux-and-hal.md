# 05 - Init, SELinux and HAL entries

[中文版](05-init-selinux-and-hal.zh-CN.md)

These are the init scripts, SELinux labels and HAL service declarations that start the GPU stack on this build.

## Verified claims

- **svc-composer-hw** (verified): Service vendor.qti.hardware.display.composer runs /vendor/bin/hw/vendor.qti.hardware.display.composer-service with class 'hal animation', user system, group 'graphics drmrpc', capabilities SYS_NICE, no seclabel, socket pps (stream 0660 system system), onrestart 'class_restart strata' and 'restart surfaceflinger', task_profiles ServiceCapacityLow ServicePerformance, flags override and disabled. Exec label of the binary is hal_graphics_composer_default_exec.
- **svc-oculus-gpucontrol** (verified): hal_oculus_gpucontrol_default has exec type hal_oculus_gpucontrol_default_exec (vendor_sepolicy.cil:1185) with init exec and transition rules at lines 3338-3343. Vendor file contexts bind only odm library paths for the gpucontrol HAL (lines 116 and 118); no binary path is bound. No service for this HAL appears in the vendor rc set. Its hwservice type hal_oculus_gpucontrol_hwservice is declared at line 1239.
- **init-kgsl-writes** (verified): Init writes to kgsl sysfs at boot. init.anorak.rc: 'on boot' (trigger line 129) contains 'write /sys/class/kgsl/kgsl-3d0/perfcounter 1' at line 189. init.anorak.power.rc: 'on boot' (trigger line 16) contains 'write /sys/class/kgsl/kgsl-3d0/bcl 0' at line 22.
- **init-gpu-config-files** (verified): On post-fs-data, init removes /data/vendor/gpu/esx_config.txt and /data/vendor/gpu/qgl_config.txt (lines 426-427). On sys.boot_completed=1 with debug.vendor.gpu.esx_config set (trigger line 429), init creates /data/vendor/gpu (mode 0755, owner system, group system) and writes esx_config.txt, then chmod 664 (lines 430-432). The qgl_config.txt path follows the same pattern at lines 434-437. The file contexts label these paths gpu_config_override_file (vendor_file_contexts line 241, declared vendor_sepolicy.cil:1123).
- **label-kgsl-sysfs-proc** (verified): genfscon labels: /sys/gpu_error is proc_gpu_error (line 12); /kgsl/proc is proc_kgsl_dbg (line 15); /devices/platform/soc/3d00000.qcom,kgsl-3d0/devfreq/3d00000.qcom,kgsl-3d0 is vendor_sysfs_kgsl (line 62); /devices/platform/soc/3d00000.qcom,kgsl-3d0/kgsl/kgsl-3d0 is vendor_sysfs_kgsl (line 130); /devices/virtual/kgsl/kgsl/pagetables is vendor_sysfs_kgsl_proc (line 162); /devices/virtual/kgsl/kgsl/thread is vendor_sysfs_kgsl (line 171); /devices/virtual/kgsl/kgsl/proc is vendor_sysfs_kgsl_proc (line 172); /devices/virtual/kgsl is vendor_sysfs_kgsl (line 192). Types are declared at lines 849 (proc_gpu_error), 853 (proc_kgsl_dbg), 927 (vendor_sysfs_kgsl) and 929 (vendor_sysfs_kgsl_proc).
- **vintf-compat-matrix** (verified): The compatibility matrix (root type=device version=4.0) requires android.hidl.allocator@1.0 IAllocator/ashmem (line 14, optional false) and android.hidl.memory@1.0 IMapper/ashmem (line 30, optional false). These are the only memory-allocation and mapping HAL requirements in the matrix. No graphics composer or memtrack requirement is present.

## Open items

These claims were reproduced but judged wrong in wording or label, or they were refuted. They are not stated above until the wording is corrected.

| ID | Label | Reproduce / refute | Claim (as produced) |
|---|---|---|---|
| svc-composer-dup | verified | needs_change / needs_change | The same service name vendor.qti.hardware.display.composer is defined a second time at etc/init/vendor.qti.hardware.display.composer-service.rc:1 with |
| svc-allocator | verified | needs_change / needs_change | Service vendor.qti.hardware.display.allocator runs /vendor/bin/hw/vendor.qti.hardware.display.allocator-service with class 'hal animation', user syste |
| svc-memtrack | verified | needs_change / needs_change | Service vendor.qti.hardware.memtrack runs /vendor/bin/hw/vendor.qti.hardware.memtrack-service with class hal, user nobody, group system, no capabiliti |
| svc-health-dup | verified | needs_change / needs_change | Service health-hal-2-1 runs /vendor/bin/hw/android.hardware.health@2.1-service (exec label hal_health_default_exec) with class 'hal charger', user sys |
| svc-surfaceflinger | verified | confirmed / needs_change | Service surfaceflinger runs /system/bin/surfaceflinger with class 'core animation', user system, group 'graphics drmrpc readproc', capabilities SYS_NI |
| svc-display-property-handler | verified | confirmed / needs_change | Service display_property_handler runs /vendor/bin/display_property_handler (exec label disp_prop_handler_exec, vendor_file_contexts line 468) with cla |
| svc-charger-graphics-group | verified | needs_change / needs_change | Service charger (/system/bin/charger, class charger, seclabel u:r:charger:s0, user system, group 'system graphics input', capabilities SYS_BOOT, disab |
| svc-perfservice-policy | verified | needs_change / refuted | No service in the scanned vendor rc set starts hal_perfservice. Its executable path is /odm/bin/hw/perf_service (file contexts line 102, label hal_per |
| label-kgsl-node | verified | needs_change / refuted | /dev/kgsl-3d0 is labelled gpu_device (vendor_file_contexts line 381). The type gpu_device is declared at plat_pub_versioned.cil:888 and its versioned  |
| label-dri-graphics-device | verified | needs_change / needs_change | The graphics_device label covers /dev/dri/card[0-9] (line 219), /dev/spec_sync (line 387), /dev/dri/controlD64 (line 399) and /dev/dri/renderD128 (lin |
| label-gpu-firmware | verified | confirmed / needs_change | The /vendor/firmware(/.*)? path is labelled firmware_file (vendor_file_contexts line 231; type declared at vendor_sepolicy.cil:843). The GPU-related f |
| label-adreno-libs | verified | needs_change / needs_change | Adreno user-space libraries are labelled same_process_hal_file by vendor file contexts at lines 147, 168, 170-175, 181, 185-188, 191-192 and 195, cove |
| types-gpu-declared | verified | confirmed / needs_change | GPU, graphics-HAL and related types declared in vendor_sepolicy.cil: sysfs_graphics (865), proc_gpu_error (849), proc_kgsl_dbg (853), vendor_sysfs_kgs |
| allow-kgsl-sysfs-read | verified | needs_change / needs_change | Allow rules granting domains access to vendor_sysfs_kgsl or vendor_sysfs_kgsl_proc, by line: bootanim_32_0 (1919-1921); base_typeattr_546_32_0 (1969-1 |
| allow-gpu-proc-and-config | verified | confirmed / needs_change | proc_gpu_error access: hal_health_default (2164 dir search; 2168 file ioctl read write ...). proc_kgsl_dbg access: shell_32_0 (2687-2689). gpu_config_ |
| allow-gpu-device-ioctl | verified | confirmed / needs_change | gpu_device_32_0 chr_file rules: mediacodec read, write, getattr, lock, append, map, open, watch (1755). allowx ioctl command-number ranges on chr_file |
| allow-gpucontrol-policy | verified | needs_change / needs_change | hal_oculus_gpucontrol policy: hwservice find for client (3327), add and find for server (3328), hwservice_manager add for hidl_base (3329), neverallow |
| allow-graphics-domain-rules | verified | confirmed / refuted | The rules that name a graphics, memtrack, health, syncboss, disp_prop_handler, surfaceflinger, hal_perfservice or graphics_device type total 324 lines |
| vintf-gfx-hals | verified | confirmed / needs_change | Graphics HAL entries declared by the vintf manifests: vendor.qti.hardware.display.composer@3.2 IQtiComposer/default (hwbinder, composer-service.xml:72 |
| rc-scan-summary | verified | needs_change / needs_change | The init rc set contains 39 .rc files with 60 service definitions and 79 action blocks. No service has a direct GPU term (gpu, kgsl, adreno, /dev/dri, |
