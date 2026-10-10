# Eureka userdebug（Adreno GPU 子系统）

[English](README.md)

本仓库记录 Eureka userdebug 构建（Android 12，平台 `anorak`）中的 GPU 子系统：Adreno 740 固件、kgsl 内核驱动、用户空间 GPU 库，以及启动它们的 init 与 SELinux 条目。

Discord：[discord.gg/oculus](https://discord.gg/oculus) · gdrci（Discord）

## 范围

本仓库描述该子系统的构建与组成方式，不是漏洞研究。不含利用代码，也不含关于缺陷的结论。固件二进制不包含在仓库中；`data/` 保存哈希、表格与提取出的结构性事实。

## 源镜像

| 字段 | 值 |
|---|---|
| 镜像 | `meta-quest_eureka_userdebug_v71_super.img.xz`，1,731,727,192 字节 |
| 解压后 | `super.img`，4,917,264,792 字节（稀疏格式） |
| 构建 | `ro.build.type=userdebug`，`ro.build.version.release=12`，`ro.build.version.sdk=32`，`ro.board.platform=anorak` |

super 镜像包含动态分区。其中没有 `boot`、`vendor_boot` 与内核镜像，因此设备树与引导程序不在本文范围内。

## 文档

| 文档 | English | 中文 |
|---|---|---|
| 概览 | [00](docs/00-overview.md) | [00](docs/00-overview.zh-CN.md) |
| Zap 安全固件（a740v3） | [01](docs/01-zap-firmware.md) | [01](docs/01-zap-firmware.zh-CN.md) |
| SQE 与 GMU 固件 | [02](docs/02-sqe-and-gmu-firmware.md) | [02](docs/02-sqe-and-gmu-firmware.zh-CN.md) |
| kgsl 内核模块 | [03](docs/03-kgsl-module.md) | [03](docs/03-kgsl-module.zh-CN.md) |
| 用户空间 GPU 库 | [04](docs/04-userspace-libraries.md) | [04](docs/04-userspace-libraries.zh-CN.md) |
| Init、SELinux 与 HAL 条目 | [05](docs/05-init-selinux-and-hal.md) | [05](docs/05-init-selinux-and-hal.zh-CN.md) |

## 证据

- `data/gpu/confirmed_findings.json`：文档中的每项结论，附复现命令与输出。
- `data/gpu/partitions.tsv`：super 镜像中的逻辑分区，附大小与 SHA-256 前缀。
- `data/gpu/uncovered_candidates.json`：尚未解码的 GPU 相关文件清单。
- `tools/`：解包与检查所用的 Go 工具，构建方法见 `tools/README.md`。

## 证据标注

- **已验证**：用证据文件中的命令复现。
- **已观察**：从头部、表格、字符串或配置行读取，未复现。
- **推断**：根据名称或上下文得出的结论。
- **未验证**：猜测或背景知识。

## 许可证

文档采用 [CC BY 4.0](LICENSE) 许可。版权所有 (c) 2026 gdrci。`data/` 中的内容是作为证据收录的摘录与提取物，不适用本许可证，其权利归各自的权利人所有。署名与第三方声明见 [NOTICE.zh-CN.md](NOTICE.zh-CN.md)。
