# 00 - 概述

[English](00-overview.md)

本仓库记录了 Eureka userdebug 镜像（Android 12，平台 `anorak`）的八个逻辑分区。每个文件归属于一个子系统，每个子系统都有各自的页面。

## 来源

| 字段 | 值 |
|---|---|
| 镜像 | `meta-quest_eureka_userdebug_v71_super.img.xz`，1,731,727,192 字节 |
| 解压后 | `super.img`，4,917,264,792 字节（稀疏格式） |
| 逻辑分区 | `odm_a`、`odm_dlkm_a`、`product_a`、`system_a`、`system_ext_a`、`vendor_a`、`vendor_dlkm_a`、`scratch`（`data/gpu/partitions.tsv`） |

`boot`、`vendor_boot` 和内核镜像不在 super 镜像中。设备树和引导加载程序未被覆盖。

## 清单与覆盖范围

`data/inventory/files.tsv` 列出了 odm_a、odm_dlkm_a、product_a、system_a、system_ext_a、vendor_a 和 vendor_dlkm_a 这七个分区中的全部 5,685 条记录（常规文件和符号链接），每个常规文件都附有大小、MIME 类型和 SHA-256。超级镜像的第八个分区 scratch 在清单中没有记录。每条记录恰好归属于一个子系统。`data/inventory/subsystem_counts.txt` 给出了计数。

对于每个子系统，`data/subsystems/<subsystem>/roles.tsv` 为每个文件保存一行。行数等于该子系统的清单行数，对全部 21 个子系统成立：角色表合计 5,685 行。`scratch` 分区只包含空的 overlay 目录。

## 子系统页面

| 页面 | 子系统 | 文件数 |
|---|---|---:|
| [06](06-kernel-modules.md) | 内核模块（vendor 和 ODM dlkm） | 306 |
| [07](07-selinux-policy.md) | SELinux 策略和文件上下文 | 51 |
| [08](08-hal-manifests.md) | HAL 清单（VINTF） | 62 |
| [09](09-init-and-properties.md) | Init 脚本和构建属性 | 183 |
| [10](10-display-and-composer.md) | 显示与合成器 | 132 |
| [11](11-camera.md) | 相机 | 83 |
| [12](12-dsp-and-remote-processors.md) | DSP 和远程处理器 | 243 |
| [13](13-wifi-bluetooth.md) | Wi-Fi 和蓝牙 | 104 |
| [14](14-audio.md) | 音频 | 239 |
| [15](15-sensors-mcu-meta-hal.md) | 传感器、MCU 和 Meta HAL | 263 |
| [16](16-power-thermal-charging.md) | 电源、热管理和充电 | 21 |
| [17](17-security-keystore-crypto.md) | Keystore、keymaster 和加密 | 60 |
| [18](18-media-codecs.md) | 媒体和编解码器 | 163 |
| [19](19-framework-and-system-services.md) | 框架和系统服务 | 707 |
| [20](20-apps.md) | 应用 | 431 |
| [21](21-native-libraries.md) | 原生共享库 | 1,371 |
| [22](22-fonts-locale-input-resources.md) | 字体、语言区域、输入和资源 | 448 |
| [23](23-config-files.md) | 配置文件 | 392 |
| [24](24-native-tools-and-hal-binaries.md) | 原生工具和 HAL 二进制文件 | 346 |
| [25](25-unclassified.md) | 杂项根文件和小文件 | 13 |

GPU 子系统有两套页面。第 [01](01-zap-firmware.md) 至 [05](05-init-selinux-and-hal.md) 页涵盖 zap 镜像、SQE 和 GMU 固件、kgsl 模块、用户空间 GPU 库以及 GPU init 条目。GPU 分组（`data/subsystems/gpu/`，角色表中列出 67 个文件）有其自身的声明，记录在 `data/subsystems/gpu/claims.json` 中。这些声明未并入第 01 至 05 页。

## 行为与应用

[26](26-init-boot-behaviour.zh-CN.md) 和 [27](27-important-apps.zh-CN.md) 不属于清单中的子系统。26 将 init 分组（第 09 页）中的 `.rc` 文件解析为服务、动作、导入和启动阶段；其表格位于 `data/behaviour/`。27 描述应用分组（第 20 页）中的 22 个应用。另有三个子系统页包含由数据文件生成的小节：第 06 页（模块元数据与符号依赖，`data/kernel/`）、第 20 页（全部 146 个 `.apk` 文件的索引，`data/apks/`）和第 21 页（1,361 个原生库文件的索引及其 NEEDED 依赖图，`data/elf/`）。

## 声明与证据

每条声明都记录在 [`data/subsystems/<子系统>/claims.json`](../data/subsystems/) 中，包括其证据、结论与原因。角色表位于 `data/subsystems/<子系统>/roles.tsv`。第 01 至 05 页的每项发现都有复现记录，位于 [`data/gpu/confirmed_findings.json`](../data/gpu/confirmed_findings.json) 中。21 个子系统共 835 条声明：653 条成立，160 条已更正，22 条不成立。

## 限制

- 镜像的引导链、TrustZone、虚拟机监控器和内核都不在 super 镜像中，因此未决的引导链项目（对象 `0x91`、OEM 密钥读取器和 SBL1 跳转）无法通过这些分区重新核查。
- 这些文档未确立固件签名及其来源。页面描述的是挂载后的字节内容。
- 行为描述基于名称、字符串、配置和静态结构。未使用任何设备。
