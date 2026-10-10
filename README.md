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
| Init, SELinux and HAL entries | [05](docs/05-init-selinux-and-hal.md) | [05](docs/05-init-selinux-and-hal.zh-CN.md) |

## Evidence

- `data/gpu/confirmed_findings.json`: every claim in these documents, with the command and output that reproduce it.
- `data/gpu/partitions.tsv`: the logical partitions in the super image, with sizes and SHA-256 prefixes.
- `tools/superunpack.go`: the unpacker used to split the super image. It streams the logical partitions and checks them against the LP metadata.

## Evidence labels

- **Verified**: reproduced with a command in the evidence file.
- **Observed**: read from a header, table, string or configuration line, without reproduction.
- **Inferred**: a conclusion drawn from names or context.
- **Unverified**: a guess or background knowledge.

## License

The documentation is licensed under [CC BY 4.0](LICENSE). Copyright (c) 2026 gdrci. The contents of `data/` are excerpts and extractions included as evidence, and they are not covered by this license. The rights in that material remain with their owners. Credits and third-party notices are in [NOTICE.md](NOTICE.md).
