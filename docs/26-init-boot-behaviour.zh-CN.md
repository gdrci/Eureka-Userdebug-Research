# 26 - 初始化与启动行为

[English](26-init-boot-behaviour.md)

本页说明系统启动时 init 做了什么，依据是镜像中的 `.rc` 文件。它是静态分析：列出每个触发器启动哪些服务、在哪些属性条件下启动，但不代表设备上的实际运行。镜像中不包含运行时属性值，因此每个条件均按原文给出。

## 来源

- `data/behaviour/services.tsv`：190 条服务定义，包含二进制文件、参数、类、用户、组、seclabel、标志、`onrestart`，以及来源文件和行号。
- `data/behaviour/actions.tsv`：371 个 `on` 块和 2,326 条命令，每条包含其触发器、命令、参数、来源文件和行号。
- `data/behaviour/imports.tsv`：13 条 `import` 语句。
- `data/behaviour/boot_phases.tsv`：启动触发器中的 start、class-start、enable、stop、restart、trigger 和 setprop 命令。
- `data/behaviour/service_classes.tsv`：每个类中的服务。

镜像中的 170 个 `.rc` 文件属于 `init-and-properties` 组（第 09 页）。

## 导入链

（取自 `data/behaviour/imports.tsv`）

主文件是 `system_a` 中的 `system/etc/init/hw/init.rc`。它导入：

- `/init.environ.rc`（`system_a`）；
- `/system/etc/init/hw/init.usb.rc` 和 `/system/etc/init/hw/init.usb.configfs.rc`；
- `/init.${ro.hardware}.rc` 和 `/vendor/etc/init/hw/init.${ro.hardware}.rc`，其中平台名来自 `ro.hardware` 属性；
- `/system/etc/init/hw/init.${ro.zygote}.rc`，其中 zygote 变体来自 `ro.zygote` 属性。

vendor 文件 `vendor_a:etc/init/hw/init.eureka.rc` 导入 `init.anorak.rc`，后者导入该平台的 `debug`、`cgroup`、`power` 和 `usb` 脚本。`system/etc/ueventd.rc` 导入 vendor 和 ODM 的 `ueventd` 文件。

两个 `${...}` 导入依赖镜像中不存在的属性，因此仅凭这些文件无法确定本构建在启动时执行哪个分支。

## 启动阶段

（取自 `data/behaviour/boot_phases.tsv` 与 `data/behaviour/actions.tsv`）

下表按 init 执行的顺序列出各触发器。命令按源文件的顺序排列。

| 触发器 | 启动或执行的内容 |
|---|---|
| `early-init` | 启动 `ueventd`。从 `ro.boot.usbcontroller` 设置 `sys.usb.controller`。启动 `set_secnvm_perms_sh`。 |
| `init` | 启动 `logd`、`lmkd`、`servicemanager`、`hwservicemanager` 和 `vndservicemanager`。启动 `vendor.keymaster-4-1`、`vendor.keymint-qti`、`vendor.qseecomd` 和 `qseecom-service`。 |
| `early-fs` | 启动 `vold`。 |
| `late-fs` | 按类启动 `early_hal`。 |
| `post-fs-data` | 启动 `tombstoned`、`odsign` 和 `settingsserver`。重启 `apexd`。设置 `keystore.boot_level` 属性和 `persist.sys.fuse`。 |
| `zygote-start`，按加密状态 | 启动 `statsd`、`netd`、`zygote` 和 `zygote_secondary`。分支取决于 `ro.crypto.state`：`unencrypted`、`unsupported`，或 `encrypted` 且 `ro.crypto.type=file`。 |
| `boot` | 按类启动 `hal` 和 `core`。设置 TCP 缓冲区大小（`net.tcp.buffersize.*`）。 |
| `nonencrypted` | 按类启动 `main` 和 `late_start`。 |
| `charger` | 按类启动 `charger`。 |

`late-init` 触发器依次发出这些触发器：`early-fs`、`fs`、`post-fs`、`late-fs`、`post-fs-data`、`load_persist_props_action`、`load_bpf_programs`、`zygote-start`、`firmware_mounts_complete`、`early-boot` 和 `boot`。

## 按类列出的服务

（取自 `data/behaviour/service_classes.tsv` 与 `data/behaviour/services.tsv`）

| 类 | 数量 | 服务 |
|---|---:|---|
| `main` | 59 | 见 `data/behaviour/service_classes.tsv` |
| `hal` | 35 | 列在 `data/behaviour/service_classes.tsv` 中 |
| `core` | 30 | 列在 `data/behaviour/service_classes.tsv` 中 |
| `late_start` | 28 | 列在 `data/behaviour/service_classes.tsv` 中 |
| （无类） | 26 | 由具名触发器或其他服务启动 |
| `early_hal` | 9 | `gatekeeper-1-0`、`keystore2`、`system_suspend`、`vendor.atrace-hal-1-0`、`vendor.boot-hal-1-2`、`vendor.keymaster-4-1`、`vendor.keymint-qti`、`vendor.oculus.externalbattery-hal-1-0`、`vendor.oculus.iad-hal-1-0` |
| `animation` | 8 | `bootanim`、`hwservicemanager`、`servicemanager`、`surfaceflinger`（两次）、`vendor.qti.hardware.display.allocator`、`vendor.qti.hardware.display.composer`、`vendor.qti.hardware.display.composer` |
| `charger` | 5 | `charger`、`health-hal-2-1`（两次）、`insmod_charger_sh`、`vendor.dock-1-0` |
| `graphics` | 1 | `vendor.oculus.hardware.composer-service` |
| `strata` | 1 | `vendor.oculus.hardware.composer-service` |

有些类的列表中名称会重复，因为该服务在多个 `.rc` 文件中定义。

190 个服务的用户分别为：`system` 83 个、`root` 36 个、未设置 24 个、`nobody` 8 个、`media` 8 个、`logd` 4 个、`wifi` 3 个和 `audioserver` 3 个。在这些服务中，56 个为 `oneshot`，82 个在定义时即为 `disabled`，因此它们只在某个触发器或其他服务启动它们时才会启动。

## 属性驱动的行为

（取自 `data/behaviour/actions.tsv`）

371 个动作块中有很多依赖属性。其中最重要的三组如下：

- **USB 外设（gadget）。** 149 个触发器检查 `sys.usb.config` 和 `sys.usb.configfs`，取值如 `adb`、`mtp,adb`、`accessory,audio_source,adb`、`rndis,adb` 和 `none`。每个取值选择一组 configfs 功能。
- **音频 HAL。** 当 `init.svc.audioserver` 为 `stopped` 或 `running`，或 `sys.audio.restart.hal=1` 时，init 启动音频 HAL 服务（`vendor.audio-hal`、`vendor.audio-hal-2-0`、`audio-hal-2-0`、`vendor.audio-hal-4-0-msd` 和 `audio_proxy_service`）。
- **传感器实现。** 在 `boot` 时，属性 `persist.vendor.oculus.sensor.service.impl` 从三种传感器服务中选择一种：`sensor-emulator`、`sensoremu` 或 `sensorservice`（`vendor.oculus.sensors-hal-1-0`）。

另有两个触发器会启动调试服务：`persist.debug.atrace.boottrace=1` 启动 `boottrace`，`persist.heapprofd.enable=1` 或 `traced.lazy.heapprofd=1` 启动 `heapprofd`。

## 未确定的内容

- 设备上实际的启动顺序。条件按原文列出，镜像中不含运行时属性值。
- 本构建会走两个 `${...}` 导入分支中的哪一个。
- 每个服务的启动时机。文件只给出触发器的顺序。
