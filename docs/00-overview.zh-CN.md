# 00 - 概览

[English](00-overview.md)

本仓库记录 Eureka userdebug 镜像的 GPU 子系统。该子系统分为四部分：GPU 运行的固件、加载固件的内核驱动、应用调用的用户空间库，以及启动它们的 init 与 SELinux 条目。

## 解包内容

super 镜像包含八个逻辑分区（`data/gpu/partitions.tsv`）。GPU 相关内容位于其中两个：

| 分区 | 大小（字节） | 内容 |
|---|---:|---|
| `vendor_a` | 426,274,816 | `firmware/` 下的 GPU 固件、用户空间库、init 脚本与厂商 SELinux 策略 |
| `vendor_dlkm_a` | 63,754,240 | 内核模块 `lib/modules/msm_kgsl.ko`（kgsl） |

其他分区中的 GPU 相关文件尚未由第一轮解码覆盖（见下文清单）。

## 已解码的内容

已解码五组目标：zap 镜像（`01`）、SQE 与 GMU 固件（`02`）、kgsl 模块（`03`）、用户空间库（`04`），以及 init 与 SELinux 条目（`05`）。每项结论先用一项检查复现，再用第二项检查尝试推翻。只有两项检查都通过的结论才收入本文档。

- 31 项结论通过了两项检查，列于 `data/gpu/confirmed_findings.json`。
- 部分结论在实质上成立，但措辞或标注有误。它们尚未收入本文档，每节列出了开放项。

## 尚未解码的内容

对已挂载分区的完整性检查返回了 143 个名称与 GPU 术语相符的文件（`data/gpu/uncovered_candidates.json`）。这些文件不在 `01` 至 `05` 节覆盖范围内。主要分组如下：

- **`system_a`**：`gpuservice` 守护进程（`system/bin/gpuservice`）及其 init 文件，GPU 内存的 eBPF 对象（`system/etc/bpf/gpu_mem.o`），以及框架图形的启动镜像。
- **`odm_a`**：Oculus 合成器 HAL（`vendor.oculus.hardware.graphics.composer@1.0`、`@1.1`）与 Oculus GPU 控制 HAL（`vendor.oculus.hardware.graphics.gpucontrol@1.0`），含 32 位与 64 位版本及其 VINTF 清单条目。
- **`product_a`**：OpenGL ES 与 Vulkan 的功能声明文件（`etc/permissions/android.hardware.opengles.aep.xml` 及 Vulkan 级别文件）。

这些需要第二轮解码后才能对其作出结论。

## 复现证据

`data/gpu/confirmed_findings.json` 中的命令调用 `tools/` 中的工具，并使用 shell 变量表示已挂载的路径。`tools/README.md` 给出构建步骤与变量取值，因此每条命令都可以在本仓库中运行。

## 硬件

固件将 GPU 标注为 `a740v3`（Adreno 740 世代）。内核模块为 `msm_kgsl`，设备节点为 `kgsl-3d0`。镜像中没有设备树，因此 GPU 的基址与时钟表不在本文范围内。

## 本构建不包含的内容

- 启动链、TrustZone、虚拟化层与内核镜像不在 super 镜像中。Greatwhite 研究中停滞的条目（对象 `0x91`、OEM 密钥读取者与 SBL1 跳转）无法根据这些分区在本构建上重新检查。
- 固件签名及其来源未由本文档确立。本文档描述的是挂载时读取到的字节。
