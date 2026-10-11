# 00 - Overview

[中文版](00-overview.zh-CN.md)

This repository documents the Eureka userdebug image (Android 12, platform `anorak`) from its eight logical partitions. Each file is assigned to one subsystem, and each subsystem has its own page.

## Source

| Field | Value |
|---|---|
| Image | `meta-quest_eureka_userdebug_v71_super.img.xz`, 1,731,727,192 bytes |
| Decompressed | `super.img`, 4,917,264,792 bytes (sparse) |
| Logical partitions | `odm_a`, `odm_dlkm_a`, `product_a`, `system_a`, `system_ext_a`, `vendor_a`, `vendor_dlkm_a`, `scratch` (`data/gpu/partitions.tsv`) |

`boot`, `vendor_boot` and the kernel image are not in the super image. The device tree and the bootloaders are not covered.

## Inventory and coverage

`data/inventory/files.tsv` lists all 5,685 entries in the eight partitions (regular files and symbolic links), with size, MIME type and SHA-256 for each regular file. Each entry is assigned to exactly one subsystem. `data/inventory/subsystem_counts.txt` gives the counts.

For each subsystem, `data/subsystems/<subsystem>/roles.tsv` has one row per file. The row count equals the number of inventory rows for that subsystem, which holds for all 21 subsystems: the role tables total 5,685 rows. The `scratch` partition holds only empty overlay directories.

## Subsystem pages

| Page | Subsystem | Files |
|---|---|---:|
| [06](06-kernel-modules.md) | Kernel modules (vendor and ODM dlkm) | 306 |
| [07](07-selinux-policy.md) | SELinux policy and file contexts | 51 |
| [08](08-hal-manifests.md) | HAL manifests (VINTF) | 62 |
| [09](09-init-and-properties.md) | Init scripts and build properties | 183 |
| [10](10-display-and-composer.md) | Display and composer | 132 |
| [11](11-camera.md) | Camera | 83 |
| [12](12-dsp-and-remote-processors.md) | DSP and remote processors | 243 |
| [13](13-wifi-bluetooth.md) | Wi-Fi and Bluetooth | 104 |
| [14](14-audio.md) | Audio | 239 |
| [15](15-sensors-mcu-meta-hal.md) | Sensors, MCU and Meta HALs | 263 |
| [16](16-power-thermal-charging.md) | Power, thermal and charging | 21 |
| [17](17-security-keystore-crypto.md) | Keystore, keymaster and crypto | 60 |
| [18](18-media-codecs.md) | Media and codecs | 163 |
| [19](19-framework-and-system-services.md) | Framework and system services | 707 |
| [20](20-apps.md) | Apps | 431 |
| [21](21-native-libraries.md) | Native shared libraries | 1,371 |
| [22](22-fonts-locale-input-resources.md) | Fonts, locale, input and resources | 448 |
| [23](23-config-files.md) | Configuration files | 392 |
| [24](24-native-tools-and-hal-binaries.md) | Native tools and HAL binaries | 346 |
| [25](25-unclassified.md) | Miscellaneous root and small files | 13 |

The GPU subsystem has two sets of pages. Pages [01](01-zap-firmware.md) to [05](05-init-selinux-and-hal.md) cover the zap image, the SQE and GMU firmware, the kgsl module, the userspace GPU libraries and the GPU init entries. The subsystem run also produced a GPU group (`data/subsystems/gpu/`, 67 files) with its own claims, in `data/subsystems/gpu/claims.json`. Those claims are not merged into pages 01 to 05.

## Behaviour and applications

[26](26-init-boot-behaviour.md) and [27](27-important-apps.md) are not inventory subsystems. 26 parses the `.rc` files of the init group (page 09) into services, actions, imports and boot phases; its tables are in `data/behaviour/`. 27 decodes 22 applications from the apps group (page 20) with androguard. Three subsystem pages also carry generated sections built from the data files: page 06 (module metadata and symbol dependencies, `data/kernel/`), page 20 (the index of all 146 `.apk` files, `data/apks/`) and page 21 (the index of the 1,361 native-library files and their NEEDED graph, `data/elf/`). The readers and generators are in `tools/apk/` and `tools/elfindex/`.

## Claims and their evidence

Each claim is recorded in [`data/subsystems/<subsystem>/claims.json`](../data/subsystems/), with its evidence, verdict and reason. The role tables are in `data/subsystems/<subsystem>/roles.tsv`. The GPU evidence is in [`data/gpu/confirmed_findings.json`](../data/gpu/confirmed_findings.json). Across the 21 subsystems there are 835 claims: 653 confirmed, 160 with corrected wording and 22 refuted.

## Reproducing the evidence

`tools/` holds the unpacker, the helper programs, the APK extraction scripts (`tools/apk/`) and the ELF and module indexer (`tools/elfindex/`). `tools/README.md` gives the build steps and the variables used by the evidence commands.

## Limits

- The image's boot chain, TrustZone, hypervisor and kernel are not in the super image, so the stalled items from the Greatwhite research (object `0x91`, the OEM key reader and the SBL1 jump) cannot be rechecked from these partitions.
- The firmware signatures and their provenance are not established by these documents. The pages describe the bytes as mounted.
- Behaviour is described from names, strings, configuration and static structure. No device was used.
