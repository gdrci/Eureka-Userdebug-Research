# Eureka userdebug (Adreno GPU subsystem)

Documentation of the GPU subsystem in the Eureka userdebug build (Android 12, platform `anorak`): the Adreno 740 firmware, the kgsl kernel driver, the userspace GPU libraries, and the init and SELinux entries that start them.

Chinese version: [README.zh-CN.md](README.zh-CN.md)

## Scope

This repository describes how the subsystem is built and laid out. It is not vulnerability research. It contains no exploit material and no findings about weaknesses. Firmware binaries are not included; `data/` holds hashes, tables and extracted structural facts.

## Source image

| Field | Value |
|---|---|
| Image | `meta-quest_eureka_userdebug_v71_super.img.xz`, 1,731,727,192 bytes |
| Decompressed | `super.img`, 4,917,264,792 bytes (sparse) |
| Build | `ro.build.type=userdebug`, `ro.build.version.release=12`, `ro.build.version.sdk=32`, `ro.board.platform=anorak` |

The super image holds the dynamic partitions. `boot`, `vendor_boot` and the kernel image are not in it, so the device tree and the bootloaders are not covered here.

## Documents

| Document | English | 中文 |
|---|---|---|
| Overview | [00](docs/00-overview.md) | [00](docs/00-overview.zh-CN.md) |
| Zap secure firmware (a740v3) | [01](docs/01-zap-firmware.md) | [01](docs/01-zap-firmware.zh-CN.md) |
| SQE and GMU firmware | [02](docs/02-sqe-and-gmu-firmware.md) | [02](docs/02-sqe-and-gmu-firmware.zh-CN.md) |
| kgsl kernel module | [03](docs/03-kgsl-module.md) | [03](docs/03-kgsl-module.zh-CN.md) |
| Userspace GPU libraries | [04](docs/04-userspace-libraries.md) | [04](docs/04-userspace-libraries.zh-CN.md) |
| Init, SELinux and HAL entries (GPU) | [05](docs/05-init-selinux-and-hal.md) | [05](docs/05-init-selinux-and-hal.zh-CN.md) |
| Kernel modules | [06](docs/06-kernel-modules.md) | [06](docs/06-kernel-modules.zh-CN.md) |
| SELinux policy and file contexts | [07](docs/07-selinux-policy.md) | [07](docs/07-selinux-policy.zh-CN.md) |
| HAL manifests (VINTF) | [08](docs/08-hal-manifests.md) | [08](docs/08-hal-manifests.zh-CN.md) |
| Init scripts and build properties | [09](docs/09-init-and-properties.md) | [09](docs/09-init-and-properties.zh-CN.md) |
| Display and composer | [10](docs/10-display-and-composer.md) | [10](docs/10-display-and-composer.zh-CN.md) |
| Camera | [11](docs/11-camera.md) | [11](docs/11-camera.zh-CN.md) |
| DSP and remote processors | [12](docs/12-dsp-and-remote-processors.md) | [12](docs/12-dsp-and-remote-processors.zh-CN.md) |
| Wi-Fi and Bluetooth | [13](docs/13-wifi-bluetooth.md) | [13](docs/13-wifi-bluetooth.zh-CN.md) |
| Audio | [14](docs/14-audio.md) | [14](docs/14-audio.zh-CN.md) |
| Sensors, MCU and Meta HALs | [15](docs/15-sensors-mcu-meta-hal.md) | [15](docs/15-sensors-mcu-meta-hal.zh-CN.md) |
| Power, thermal and charging | [16](docs/16-power-thermal-charging.md) | [16](docs/16-power-thermal-charging.zh-CN.md) |
| Keystore, keymaster and crypto | [17](docs/17-security-keystore-crypto.md) | [17](docs/17-security-keystore-crypto.zh-CN.md) |
| Media and codecs | [18](docs/18-media-codecs.md) | [18](docs/18-media-codecs.zh-CN.md) |
| Framework and system services | [19](docs/19-framework-and-system-services.md) | [19](docs/19-framework-and-system-services.zh-CN.md) |
| Apps | [20](docs/20-apps.md) | [20](docs/20-apps.zh-CN.md) |
| Native shared libraries | [21](docs/21-native-libraries.md) | [21](docs/21-native-libraries.zh-CN.md) |
| Fonts, locale, input and resources | [22](docs/22-fonts-locale-input-resources.md) | [22](docs/22-fonts-locale-input-resources.zh-CN.md) |
| Configuration files | [23](docs/23-config-files.md) | [23](docs/23-config-files.zh-CN.md) |
| Native tools and HAL binaries | [24](docs/24-native-tools-and-hal-binaries.md) | [24](docs/24-native-tools-and-hal-binaries.zh-CN.md) |
| Miscellaneous root and small files | [25](docs/25-unclassified.md) | [25](docs/25-unclassified.zh-CN.md) |
| Init and boot behaviour | [26](docs/26-init-boot-behaviour.md) | [26](docs/26-init-boot-behaviour.zh-CN.md) |
| Important applications | [27](docs/27-important-apps.md) | [27](docs/27-important-apps.zh-CN.md) |

## Evidence

- `data/gpu/confirmed_findings.json`: every claim in these documents, with the command and output that reproduce it.
- `data/gpu/partitions.tsv`: the logical partitions in the super image, with sizes and SHA-256 prefixes.
- `data/inventory/files.tsv`: every file and symlink in the eight partitions, with its subsystem, size, MIME type and SHA-256.
- `data/behaviour/`: the services, actions, imports, boot phases and service classes parsed from the `.rc` files, behind document 26.
- `data/apks/`: the manifest and dex facts for all 146 `.apk` files, with `summary.tsv`. Document 27 describes 22 of them in detail.
- `data/elf/libraries.tsv`: the ELF headers, NEEDED and dynamic symbol counts and build-ids of the 1,361 native-library files, behind document 21.
- `data/kernel/`: the `.modinfo` fields (`modules.tsv`) and the global symbols (`symbols.tsv`) of the 289 kernel modules, behind document 06.
- `data/subsystems/<subsystem>/`: the role table (`roles.tsv`) and the claims with their evidence (`claims.json`) for each subsystem.
- `tools/superunpack.go`: the unpacker used to split the super image. It streams the logical partitions and checks them against the LP metadata.

## Evidence labels

- **Verified**: reproduced with a command in the evidence file.
- **Observed**: read from a header, table, string or configuration line, without reproduction.
- **Inferred**: a conclusion drawn from names or context.
- **Unverified**: a guess or background knowledge.

## License

The documentation is licensed under [CC BY 4.0](LICENSE). Copyright (c) 2026 gdrci. The contents of `data/` are excerpts and extractions included as evidence, and they are not covered by this license. The rights in that material remain with their owners. Credits and third-party notices are in [NOTICE.md](NOTICE.md).
