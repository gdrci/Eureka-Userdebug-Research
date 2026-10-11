# Credits and third-party notices

[中文版](NOTICE.zh-CN.md)

## Author

Original analysis and documentation: **gdrci**.

This covers the documentation under `docs/`, the README files, and the helper tools under `tools/`. Those are licensed under [CC BY 4.0](LICENSE).

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

## Tools and projects used

These tools produced the evidence. Their code is not copied into this repository.

- **Go 1.24.7** standard library (BSD-style license, The Go Authors). Used by the programs under `tools/`: `superunpack`, `zapinspect`, `hashcheck`, `fwparse`, `rcscan`, `vintfscan` and `elfindex`.
- **androguard 4.1.4** (Apache License 2.0), with its dependency **apkInspector 1.3.7** (Apache License 2.0) and **loguru 0.7.3** (MIT license). Used by `tools/apk/` to read APK manifests, certificates and dex strings.
- **Python** standard library (`csv`, `json`, `zipfile`, `hashlib`, `re` and others), used by `tools/`.
- **Linux kernel** (GPL-2.0). The module metadata in `data/kernel/` describes modules from the image. The kernel itself is not in the image.
- **GNU binutils** (`readelf`, `nm`, GPL) and **OpenSSL 3.0.13** (Apache License 2.0). Used only to check results. They are not distributed with this repository.

## Trademarks

Meta, Meta Quest, Qualcomm, Snapdragon, Android, Chromium and Google are trademarks of their respective owners. They are used only to identify the software, devices and vendors analysed.

## Affiliation

This repository is an independent analysis. It is not affiliated with, sponsored by or endorsed by Meta Platforms, Inc., Qualcomm, or any other vendor named in it.
