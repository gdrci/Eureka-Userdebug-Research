# 26 - Init and boot behaviour

[中文版](26-init-boot-behaviour.zh-CN.md)

This page describes what init does when the system starts, derived from the `.rc` files in the image. It is static: it lists which services each trigger starts, under which property conditions, and does not show a run on a device. The runtime property values are not in the image, so every condition is given as written.

## Sources

- `data/behaviour/services.tsv`: 190 service definitions, with binary, arguments, class, user, group, seclabel, flags, `onrestart` and the source file and line.
- `data/behaviour/actions.tsv`: 371 `on` blocks and 2,326 commands, each with its trigger, command, arguments and source line.
- `data/behaviour/imports.tsv`: the 13 `import` lines.
- `data/behaviour/boot_phases.tsv`: the start, class-start, enable, stop, restart, trigger and setprop commands for the boot triggers.
- `data/behaviour/service_classes.tsv`: the services in each class.

The 170 `.rc` files are in the `init-and-properties` group (page 09).

## Import chain

The main file is `system/etc/init/hw/init.rc` in `system_a`. It imports:

- `/init.environ.rc` (`system_a`);
- `/system/etc/init/hw/init.usb.rc` and `/system/etc/init/hw/init.usb.configfs.rc`;
- `/init.${ro.hardware}.rc` and `/vendor/etc/init/hw/init.${ro.hardware}.rc`, where the platform name comes from the `ro.hardware` property;
- `/system/etc/init/hw/init.${ro.zygote}.rc`, where the zygote variant comes from the `ro.zygote` property.

The vendor file `vendor_a:etc/init/hw/init.eureka.rc` imports `init.anorak.rc`, which imports the `debug`, `cgroup`, `power` and `usb` scripts for the platform. `system/etc/ueventd.rc` imports the vendor and ODM `ueventd` files.

The two `${...}` imports depend on properties that are not in the image. Which branch runs at boot is therefore not established from these files.

## Boot phases

The table gives the triggers in the order init runs them. Commands are listed in the order of the source files.

| Trigger | What it starts or does |
|---|---|
| `early-init` | Starts `ueventd`. Sets `sys.usb.controller` from `ro.boot.usbcontroller`. Starts `set_secnvm_perms_sh`. |
| `init` | Starts `logd`, `lmkd`, `servicemanager`, `hwservicemanager` and `vndservicemanager`. Starts `vendor.keymaster-4-1`, `vendor.keymint-qti`, `vendor.qseecomd` and `qseecom-service`. |
| `early-fs` | Starts `vold`. |
| `late-fs` | Class-starts `early_hal`. |
| `post-fs-data` | Starts `tombstoned`, `odsign` and `settingsserver`. Restarts `apexd`. Sets the `keystore.boot_level` property, and `persist.sys.fuse`. |
| `zygote-start`, by encryption state | Starts `statsd`, `netd`, `zygote` and `zygote_secondary`. The branch depends on `ro.crypto.state`: `unencrypted`, `unsupported`, or `encrypted` with `ro.crypto.type=file`. |
| `boot` | Class-starts `hal` and `core`. Sets the TCP buffer sizes (`net.tcp.buffersize.*`). |
| `nonencrypted` | Class-starts `main` and `late_start`. |
| `charger` | Class-starts `charger`. |

The `late-init` trigger issues these triggers in turn: `early-fs`, `fs`, `post-fs`, `late-fs`, `post-fs-data`, `load_persist_props_action`, `load_bpf_programs`, `zygote-start`, `firmware_mounts_complete`, `early-boot` and `boot` in turn.

## Services by class

| Class | Count | Services |
|---|---:|---|
| `main` | 59 | See `data/behaviour/service_classes.tsv` |
| `hal` | 35 | Listed in `data/behaviour/service_classes.tsv` |
| `core` | 30 | Listed in `data/behaviour/service_classes.tsv` |
| `late_start` | 28 | Listed in `data/behaviour/service_classes.tsv` |
| (no class) | 26 | Started by a named trigger or by another service |
| `early_hal` | 9 | `gatekeeper-1-0`, `keystore2`, `system_suspend`, `vendor.atrace-hal-1-0`, `vendor.boot-hal-1-2`, `vendor.keymaster-4-1`, `vendor.keymint-qti`, `vendor.oculus.externalbattery-hal-1-0`, `vendor.oculus.iad-hal-1-0` |
| `animation` | 8 | `bootanim`, `hwservicemanager`, `servicemanager`, `surfaceflinger` (twice), `vendor.qti.hardware.display.allocator`, `vendor.qti.hardware.display.composer`, `vendor.qti.hardware.display.composer` |
| `charger` | 5 | `charger`, `health-hal-2-1` (twice), `insmod_charger_sh`, `vendor.dock-1-0` |
| `graphics` | 1 | `vendor.oculus.hardware.composer-service` |
| `strata` | 1 | `vendor.oculus.hardware.composer-service` |

Some class lists repeat a name because the service is defined in more than one `.rc` file.

The users of the 190 services are: `system` 83, `root` 36, none set 24, `nobody` 8, `media` 8, `logd` 4, `wifi` 3 and `audioserver` 3. Of the services, 56 are `oneshot` and 82 are `disabled` at definition, so they start only when a trigger or another service starts them.

## Property-driven behaviour

Many of the 371 action blocks depend on properties. Three groups are the most consequential:

- **USB gadget.** 149 triggers test `sys.usb.config` and `sys.usb.configfs`, with values such as `adb`, `mtp,adb`, `accessory,audio_source,adb`, `rndis,adb` and `none`. Each value selects a configfs function set.
- **Audio HAL.** When `init.svc.audioserver` is `stopped` or `running`, or `sys.audio.restart.hal=1`, init starts the audio HAL services (`vendor.audio-hal`, `vendor.audio-hal-2-0`, `audio-hal-2-0`, `vendor.audio-hal-4-0-msd`, and `audio_proxy_service`).
- **Sensor implementation.** At `boot`, the property `persist.vendor.oculus.sensor.service.impl` chooses one of three sensor services: `sensor-emulator`, `sensoremu` or `sensorservice` (`vendor.oculus.sensors-hal-1-0`).

Two triggers also start debug services: `persist.debug.atrace.boottrace=1` starts `boottrace`, and `persist.heapprofd.enable=1` or `traced.lazy.heapprofd=1` starts `heapprofd`.

## What is not established

- The actual boot order on a device. The conditions are listed as written, and the runtime property values are not in the image.
- Which of the two `${...}` import branches is taken on this build.
- The timing of each service. The files give only the order of triggers.
