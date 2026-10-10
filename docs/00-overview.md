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

For each subsystem, `data/subsystems/<subsystem>/roles.tsv` has one row per file. The row count equals the number of inventory rows for that subsystem, which was checked for all 21 subsystems: the role tables total 5,685 rows. The `scratch` partition holds only empty overlay directories.

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

The GPU subsystem has two sets of pages. Pages [01](01-zap-firmware.md) to [05](05-init-selinux-and-hal.md) come from the first GPU run and cover the zap image, the SQE and GMU firmware, the kgsl module, the userspace GPU libraries and the GPU init entries. The subsystem run also produced a GPU group (`data/subsystems/gpu/`, 67 files) with its own claims, in `data/subsystems/gpu/claims.json`. Those claims are not merged into pages 01 to 05.

## Behaviour and applications

Two pages are built from the image's files but are not inventory subsystems. [26](26-init-boot-behaviour.md) parses the `.rc` files of the init group (page 09) into services, actions, imports and boot phases; its tables are in `data/behaviour/`. [27](27-important-apps.md) decodes 22 applications from the apps group (page 20) with androguard; its facts are in `data/apks/`, and the extraction and page-generation scripts are in `tools/apk/`.

## How the claims were checked

Each subsystem has a role table and a claims file. A separate pass re-ran each claim's command and tried to refute it. The pages state the verified claims. Claims whose wording was corrected show the corrected wording. Claims that did not hold are listed as excluded, with the reason. The totals across the 21 subsystems are 835 claims: 653 confirmed, 160 needing a wording change, and 22 refuted.

## Reproducing the evidence

`tools/` holds the unpacker, the helper programs and the APK extraction scripts (`tools/apk/`). `tools/README.md` gives the build steps and the variables used by the evidence commands.

## Limits

- The image's boot chain, TrustZone, hypervisor and kernel are not in the super image, so the stalled items from the Greatwhite research (object `0x91`, the OEM key reader and the SBL1 jump) cannot be rechecked from these partitions.
- The firmware signatures and their provenance are not established by these documents. The pages describe the bytes as mounted.
- Behaviour is described from names, strings, configuration and static structure. Commands were not executed on a device.
