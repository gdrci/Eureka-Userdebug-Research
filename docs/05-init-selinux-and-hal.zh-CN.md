# 05 - Init、SELinux 与 HAL 条目

[English](05-init-selinux-and-hal.md)

以下是在本构建上启动 GPU 栈的 init 脚本、SELinux 标签与 HAL 服务声明。

## 已验证的结论

- **svc-composer-hw**（verified）：服务 vendor.qti.hardware.display.composer 运行 /vendor/bin/hw/vendor.qti.hardware.display.composer-service，类别为 hal animation，用户 system，组 graphics drmrpc，能力 SYS_NICE，无 seclabel，socket pps（stream 0660 system system），onrestart 为 class_restart strata 与 restart surfaceflinger，task_profiles 为 ServiceCapacityLow 与 ServicePerformance，标志 override 与 disabled。二进制的 exec 标签为 hal_graphics_composer_default_exec。
- **svc-oculus-gpucontrol**（verified）：hal_oculus_gpucontrol_default 的 exec 类型为 hal_oculus_gpucontrol_default_exec（vendor_sepolicy.cil:1185），init exec 与 transition 规则位于第 3338 至 3343 行。厂商 file_contexts 仅为该 gpucontrol HAL 绑定 odm 库路径（第 116 与 118 行），未绑定二进制路径。厂商 rc 集中没有该 HAL 的服务。其 hwservice 类型 hal_oculus_gpucontrol_hwservice 声明于第 1239 行。
- **init-kgsl-writes**（verified）：启动时 init 写入 kgsl 的 sysfs 节点。init.anorak.rc：on boot（触发行第 129 行）包含第 189 行的 write /sys/class/kgsl/kgsl-3d0/perfcounter 1。init.anorak.power.rc：on boot（触发行第 16 行）包含第 22 行的 write /sys/class/kgsl/kgsl-3d0/bcl 0。
- **init-gpu-config-files**（verified）：在 post-fs-data 阶段，init 删除 /data/vendor/gpu/esx_config.txt 与 /data/vendor/gpu/qgl_config.txt（第 426 至 427 行）。当 sys.boot_completed=1 且设置了 debug.vendor.gpu.esx_config 时（触发行第 429 行），init 创建 /data/vendor/gpu（模式 0755，属主与属组均为 system），写入 esx_config.txt，并 chmod 664（第 430 至 432 行）。qgl_config.txt 路径按相同模式在第 434 至 437 行。file_contexts 将这些路径标为 gpu_config_override_file（vendor_file_contexts 第 241 行，声明于 vendor_sepolicy.cil:1123）。
- **label-kgsl-sysfs-proc**（verified）：genfscon 标签：/sys/gpu_error 为 proc_gpu_error（第 12 行）；/kgsl/proc 为 proc_kgsl_dbg（第 15 行）；/devices/platform/soc/3d00000.qcom,kgsl-3d0/devfreq/3d00000.qcom,kgsl-3d0 为 vendor_sysfs_kgsl（第 62 行）；/devices/platform/soc/3d00000.qcom,kgsl-3d0/kgsl/kgsl-3d0 为 vendor_sysfs_kgsl（第 130 行）；/devices/virtual/kgsl/kgsl/pagetables 为 vendor_sysfs_kgsl_proc（第 162 行）；/devices/virtual/kgsl/kgsl/thread 为 vendor_sysfs_kgsl（第 171 行）；/devices/virtual/kgsl/kgsl/proc 为 vendor_sysfs_kgsl_proc（第 172 行）；/devices/virtual/kgsl 为 vendor_sysfs_kgsl（第 192 行）。类型声明位于第 849 行（proc_gpu_error）、853 行（proc_kgsl_dbg）、927 行（vendor_sysfs_kgsl）与 929 行（vendor_sysfs_kgsl_proc）。
- **vintf-compat-matrix**（verified）：兼容性矩阵（根节点 type=device version=4.0）要求 android.hidl.allocator@1.0 的 IAllocator/ashmem（第 14 行，optional false）与 android.hidl.memory@1.0 的 IMapper/ashmem（第 30 行，optional false）。这是矩阵中仅有的内存分配与映射 HAL 要求。矩阵中没有图形合成器或 memtrack 要求。

## 开放项

以下结论已复现，但措辞或标注有误，或被推翻。在措辞修正之前，它们不在上文陈述。表格中的结论文本保留英文原文。

| ID | 标注 | 复现 / 推翻 | 结论（原文） |
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
