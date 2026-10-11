# Credits and third-party notices

[中文版](NOTICE.zh-CN.md)

## Author

Original analysis and documentation: **gdrci**.

This covers the documentation under `docs/` and the README files. Those are licensed under [CC BY 4.0](LICENSE).

## Not covered by the CC BY 4.0 grant

The following are excluded from the license in [LICENSE](LICENSE). Their rights remain with their owners.

| Material | Where it is | Owner | Basis |
|---|---|---|---|
| Eureka userdebug image (Meta Quest, `meta-quest_eureka_userdebug_v71_super.img.xz`): excerpts, strings, symbol names, library and module metadata, manifest and certificate facts, and the file inventory with SHA-256 values | `data/` | Meta Platforms, Inc., and the owners of the third-party components in the image | Included as evidence for the documentation. Firmware binaries are not included. |

## Third-party components named in the evidence

The image contains components from other owners. The evidence records their names, metadata and symbols. Their code is not copied into this repository.

- **Android platform components** (Android Open Source Project, Apache License 2.0). The init lines in `data/behaviour/` are excerpts from the image's `.rc` files. Some of those scripts derive from Android platform sources.
- **Chromium-based WebView** (`com.android.webview`, Chromium project, BSD-style license). Its package, class and string names are in `data/apks/`.
- **Linux kernel modules** (GPL-2.0). `data/kernel/` records each module's name, licence string, author line, parameters and symbol names. The module binaries are not included.
- **Qualcomm components** (vendor drivers and libraries, Qualcomm Technologies, Inc.). Their names and metadata are in `data/elf/` and `data/kernel/`.

## Trademarks

Meta, Meta Quest, Qualcomm, Snapdragon, Android, Chromium and Google are trademarks of their respective owners. They are used only to identify the software, devices and vendors analysed.

## Affiliation

This repository is an independent analysis. It is not affiliated with, sponsored by or endorsed by Meta Platforms, Inc., Qualcomm, or any other vendor named in it.
