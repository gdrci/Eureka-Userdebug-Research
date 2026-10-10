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

`data/inventory/files.tsv` 列出了八个分区中的全部 5,685 条记录（常规文件和符号链接），每个常规文件都附有大小、MIME 类型和 SHA-256。每条记录恰好归属于一个子系统。`data/inventory/subsystem_counts.txt` 给出了计数。

对于每个子系统，`data/subsystems/<subsystem>/roles.tsv` 为每个文件保存一行。行数等于该子系统的清单行数，已对全部 21 个子系统核对：角色表合计 5,685 行。`scratch` 分区只包含空的 overlay 目录。

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

GPU 子系统有两套页面。第 [01](01-zap-firmware.md) 至 [05](05-init-selinux-and-hal.md) 页来自第一次 GPU 运行，涵盖 zap 镜像、SQE 和 GMU 固件、kgsl 模块、用户空间 GPU 库以及 GPU init 条目。子系统运行还生成了一个 GPU 分组（`data/subsystems/gpu/`，67 个文件），其声明记录在 `data/subsystems/gpu/claims.json` 中。这些声明未并入第 01 至 05 页。

## 行为与应用

有两页是由镜像文件生成的，但不属于清单中的子系统。[26](26-init-boot-behaviour.zh-CN.md) 将 init 分组（第 09 页）中的 `.rc` 文件解析为服务、动作、导入和启动阶段；其表格位于 `data/behaviour/`。[27](27-important-apps.zh-CN.md) 使用 androguard 解码应用分组（第 20 页）中的 22 个应用；其事实位于 `data/apks/`，提取与页面生成脚本位于 `tools/apk/`。

## 声明的核查方式

每个子系统都有一张角色表和一份声明文件。另一轮独立核查会重新运行每条声明的命令，并尝试反驳它。各页面陈述的是已核实的声明。措辞经过修正的声明会显示修正后的措辞。未成立的声明会被列为已排除，并注明原因。21 个子系统的声明总数为 835 条：653 条已确认，160 条需修改措辞，22 条被反驳。

## 复现证据

`tools/` 包含解包器、辅助程序和 APK 提取脚本（`tools/apk/`）。`tools/README.md` 给出了构建步骤以及证据命令所用的变量。

## 限制

- 镜像的引导链、TrustZone、虚拟机监控器和内核都不在 super 镜像中，因此 Greatwhite 研究中搁置的项目（对象 `0x91`、OEM 密钥读取器和 SBL1 跳转）无法通过这些分区重新核查。
- 这些文档未确立固件签名及其来源。页面描述的是挂载后的字节内容。
- 行为描述基于名称、字符串、配置和静态结构。未在设备上执行任何命令。
