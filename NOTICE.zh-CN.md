# 署名与第三方声明

[English](NOTICE.md)

## 作者

原创分析与文档：**gdrci**。

这包括 `docs/` 下的文档、README 文件与 `tools/` 下的辅助工具。这些内容按 [CC BY 4.0](LICENSE) 授权。

## 不在 CC BY 4.0 授权范围内的内容

以下内容被排除在 [LICENSE](LICENSE) 的授权之外，其权利仍归原权利人所有。

| 材料 | 位置 | 权利人 | 依据 |
|---|---|---|---|
| Ray-Ban Display 固件：摘录、字符串、符号表、反汇编与反编译输出、十六进制提取物与元数据 | `data/` | Meta Platforms, Inc. 及固件中第三方组件的权利人 | 作为分析的证据收录 |

## 使用的工具与项目

- **Capstone** 5.0.7（反汇编），BSD 许可。
- **Python** 标准库（解析 ELF、FDT 与 AVB 结构）。
- **Linux 内核**（GPL-2.0）。`data/android/` 中的模块名与元数据来自固件中的内核模块。
- **OP-TEE** 文档（*Hoya architecture* 页面），用于第 10 节的参考。其文档的许可证请参阅 OP-TEE 仓库。

## 商标


## 关联声明

本仓库是独立分析。它不隶属于、不受赞助于、也未获得 Meta Platforms, Inc.、Qualcomm 或文中提及的任何其他厂商的认可。
