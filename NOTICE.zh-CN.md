# 署名与第三方声明

[English](NOTICE.md)

## 作者

原创分析与文档：**gdrci**。

这包括 `docs/` 下的文档、README 文件与 `tools/` 下的辅助工具。这些内容按 [CC BY 4.0](LICENSE) 授权。

## 不在 CC BY 4.0 授权范围内的内容

以下内容被排除在 [LICENSE](LICENSE) 的授权之外，其权利仍归原权利人所有。

| 材料 | 位置 | 权利人 | 依据 |
|---|---|---|---|
| Eureka userdebug 镜像（Meta Quest，`meta-quest_eureka_userdebug_v71_super.img.xz`）：摘录、字符串、符号名、库与模块元数据、清单与证书事实，以及带 SHA-256 的文件清单 | `data/` | Meta Platforms, Inc. 及镜像中第三方组件的权利人 | 作为文档的证据收录。固件二进制文件不包含在内。 |

## 证据中提及的第三方组件

镜像包含其他权利人的组件。证据记录它们的名称、元数据与符号；它们的代码未复制到本仓库。

- **Android 平台组件**（Android Open Source Project，Apache License 2.0）。`data/behaviour/` 中的 init 脚本行摘录自镜像中的 `.rc` 文件，其中部分脚本源自 Android 平台源码。
- **基于 Chromium 的 WebView**（`com.android.webview`，Chromium 项目，BSD 式许可证）。其包名、类名与字符串位于 `data/apks/`。
- **Linux 内核模块**（GPL-2.0）。`data/kernel/` 记录每个模块的名称、许可证字符串、作者行、参数与符号名。模块二进制文件未包含在内。
- **Qualcomm 组件**（厂商驱动与库，Qualcomm Technologies, Inc.）。其名称与元数据位于 `data/elf/` 与 `data/kernel/`。

## 使用的工具与项目

这些工具产生了证据。它们的代码未复制到本仓库。

- **Go 1.24.7** 标准库（BSD 式许可证，The Go Authors）。由 `tools/` 下的程序使用：`superunpack`、`zapinspect`、`hashcheck`、`fwparse`、`rcscan`、`vintfscan` 与 `elfindex`。
- **androguard 4.1.4**（Apache License 2.0），及其依赖 **apkInspector 1.3.7**（Apache License 2.0）与 **loguru 0.7.3**（MIT 许可证）。由 `tools/apk/` 用于读取 APK 清单、证书与 dex 字符串。
- **Python** 标准库（`csv`、`json`、`zipfile`、`hashlib`、`re` 等），由 `tools/` 使用。
- **Linux 内核**（GPL-2.0）。`data/kernel/` 中的模块元数据描述镜像中的模块。内核本身不在镜像中。
- **GNU binutils**（`readelf`、`nm`，GPL）与 **OpenSSL 3.0.13**（Apache License 2.0）。仅用于核对结果，不随本仓库分发。

## 商标

Meta、Meta Quest、Qualcomm、Snapdragon、Android、Chromium 与 Google 是其各自权利人的商标。这些名称仅用于标识所分析的软件、设备与厂商。

## 关联声明

本仓库是独立分析。它不隶属于、不受赞助于、也未获得 Meta Platforms, Inc.、Qualcomm 或文中提及的任何其他厂商的认可。
