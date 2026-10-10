# Credits and third-party notices

[中文版](NOTICE.zh-CN.md)

## Author

Original analysis and documentation: **gdrci**.

This covers the documentation under `docs/`, the README files, and the helper tools under `tools/`. Those are licensed under [CC BY 4.0](LICENSE).

## Not covered by the CC BY 4.0 grant

The following are excluded from the license in [LICENSE](LICENSE). Their rights remain with their owners.

| Material | Where it is | Owner | Basis |
|---|---|---|---|
| Ray-Ban Display firmware: excerpts, strings, symbol tables, disassembly and decompiler output, hex extracts and metadata | `data/` | Meta Platforms, Inc. and the owners of the third-party components in the firmware | Included as evidence for the analysis |

## Tools and projects used

- **Capstone** 5.0.7 (disassembly), BSD license.
- **Python** standard library (parsing of ELF, FDT and AVB structures).
- **Linux kernel** (GPL-2.0). Module names and metadata in `data/android/` come from the kernel modules in the firmware.
- **OP-TEE** documentation (the *Hoya architecture* page) for the reference in section 10. Consult the OP-TEE repository for its documentation license.

## Trademarks


## Affiliation

This repository is an independent analysis. It is not affiliated with, sponsored by or endorsed by Meta Platforms, Inc., Qualcomm, or any other vendor named in it.
