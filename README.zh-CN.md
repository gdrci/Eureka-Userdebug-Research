# Eureka userdebug（全部子系统）

[English](README.md)

本仓库记录 Eureka userdebug 镜像（Android 12，平台 `anorak`）的各个子系统：先列出镜像中的每个文件，再为每个子系统给出角色表与经过复现检查的结论，并包括 Adreno GPU 子系统的详细分析。

Discord：[discord.gg/oculus](https://discord.gg/oculus) · gdrci（Discord）

## 范围

本仓库描述该镜像的构建与组成方式，不是漏洞研究。不含利用代码，也不含关于缺陷的结论。固件二进制不包含在仓库中；`data/` 保存哈希、表格与提取出的结构性事实。

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
| Init、SELinux 与 HAL 条目（GPU） | [05](docs/05-init-selinux-and-hal.md) | [05](docs/05-init-selinux-and-hal.zh-CN.md) |
| 内核模块 | [06](docs/06-kernel-modules.md) | [06](docs/06-kernel-modules.zh-CN.md) |
| SELinux 策略与文件标签 | [07](docs/07-selinux-policy.md) | [07](docs/07-selinux-policy.zh-CN.md) |
| HAL 清单（VINTF） | [08](docs/08-hal-manifests.md) | [08](docs/08-hal-manifests.zh-CN.md) |
| Init 脚本与构建属性 | [09](docs/09-init-and-properties.md) | [09](docs/09-init-and-properties.zh-CN.md) |
| 显示与合成器 | [10](docs/10-display-and-composer.md) | [10](docs/10-display-and-composer.zh-CN.md) |
| 摄像头 | [11](docs/11-camera.md) | [11](docs/11-camera.zh-CN.md) |
| DSP 与远程处理器 | [12](docs/12-dsp-and-remote-processors.md) | [12](docs/12-dsp-and-remote-processors.zh-CN.md) |
| Wi-Fi 与蓝牙 | [13](docs/13-wifi-bluetooth.md) | [13](docs/13-wifi-bluetooth.zh-CN.md) |
| 音频 | [14](docs/14-audio.md) | [14](docs/14-audio.zh-CN.md) |
| 传感器、MCU 与 Meta HAL | [15](docs/15-sensors-mcu-meta-hal.md) | [15](docs/15-sensors-mcu-meta-hal.zh-CN.md) |
| 电源、温控与充电 | [16](docs/16-power-thermal-charging.md) | [16](docs/16-power-thermal-charging.zh-CN.md) |
| 密钥库、keymaster 与加密 | [17](docs/17-security-keystore-crypto.md) | [17](docs/17-security-keystore-crypto.zh-CN.md) |
| 媒体与编解码器 | [18](docs/18-media-codecs.md) | [18](docs/18-media-codecs.zh-CN.md) |
| 框架与系统服务 | [19](docs/19-framework-and-system-services.md) | [19](docs/19-framework-and-system-services.zh-CN.md) |
| 应用 | [20](docs/20-apps.md) | [20](docs/20-apps.zh-CN.md) |
| 原生共享库 | [21](docs/21-native-libraries.md) | [21](docs/21-native-libraries.zh-CN.md) |
| 字体、区域设置、输入与资源 | [22](docs/22-fonts-locale-input-resources.md) | [22](docs/22-fonts-locale-input-resources.zh-CN.md) |
| 配置文件 | [23](docs/23-config-files.md) | [23](docs/23-config-files.zh-CN.md) |
| 原生工具与 HAL 二进制文件 | [24](docs/24-native-tools-and-hal-binaries.md) | [24](docs/24-native-tools-and-hal-binaries.zh-CN.md) |
| 其他根目录与小文件 | [25](docs/25-unclassified.md) | [25](docs/25-unclassified.zh-CN.md) |
| 初始化与启动行为 | [26](docs/26-init-boot-behaviour.md) | [26](docs/26-init-boot-behaviour.zh-CN.md) |
| 重要应用 | [27](docs/27-important-apps.md) | [27](docs/27-important-apps.zh-CN.md) |

## 证据

- `data/gpu/confirmed_findings.json`：GPU 首轮分析中每项结论的复现命令与输出。
- `data/gpu/partitions.tsv`：super 镜像中的逻辑分区，附大小与 SHA-256 前缀。
- `data/inventory/files.tsv`：八个分区中的每个文件与符号链接，附子系统、大小、MIME 类型与 SHA-256。
- `data/behaviour/`：从 `.rc` 文件解析出的服务、动作、导入、启动阶段与服务类，是第 26 页的依据。
- `data/apks/`：全部 146 个 `.apk` 文件的清单与 dex 事实，附汇总表 `summary.tsv`。第 27 页详细描述其中 22 个。
- `data/elf/libraries.tsv`：原生库分组中 1,361 个文件的 ELF 头、NEEDED、动态符号计数与 build-id，是第 21 页的依据。
- `data/kernel/`：289 个内核模块的 `.modinfo` 字段（`modules.tsv`）与全局符号（`symbols.tsv`），是第 06 页的依据。
- `data/subsystems/<子系统>/`：每个子系统的角色表（`roles.tsv`）与附证据的结论（`claims.json`）。
- `data/gpu/uncovered_candidates.json`：首轮 GPU 完整性检查返回的文件清单。
- `tools/superunpack.go`：用于拆分 super 镜像的解包器。它以流式方式输出各逻辑分区，并依据 LP 元数据进行校验。
- `tools/`：解包与检查所用的 Go 工具，构建方法见 `tools/README.md`。

## 证据标注

- **已验证**：用证据文件中的命令复现。
- **已观察**：从头部、表格、字符串或配置行读取，未复现。
- **推断**：根据名称或上下文得出的结论。
- **未验证**：猜测或背景知识。

## 许可证

文档采用 [CC BY 4.0](LICENSE) 许可。版权所有 (c) 2026 gdrci。`data/` 中的内容是作为证据收录的摘录、哈希、表格与提取出的事实，不适用本许可证，其权利归各自的权利人所有。署名与第三方声明见 [NOTICE.zh-CN.md](NOTICE.zh-CN.md)。
