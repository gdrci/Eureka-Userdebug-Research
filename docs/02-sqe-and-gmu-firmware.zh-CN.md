# 02 - SQE 与 GMU 固件

[English](02-sqe-and-gmu-firmware.md)

SQE 是命令处理器微码，GMU 是电源与时钟控制器固件。两者各有自己的二进制布局，以下根据文件解码得出。

## 已验证的结论

- **G-01**（verified）：gmu_gen70200.bin 为 67,240 字节（0x106a8），是 4 的倍数。它含 11 条描述符记录（偏移 0x000 至 0x0af），之后是四条块记录，其头部位于 0x0b0、0x29c、0x10648 与 0x10680，每条后跟其载荷。遍历恰好消耗整个文件（未消耗字节为 0）。载荷合计 67,000 字节，11×16 + 4×16 + 67,000 = 67,240。
- **G-02**（verified）：描述符表：偏移 0x00 至 0xaf 共 11 条记录，每条 16 字节。每条记录为四个小端双字 {addr, 零字, type, size}。所有零字均为 0。记录 0 至 8 的 type 为 1，记录 9 至 10 的 type 为 7。取值：0x00000000/0x1dc，0x00004000/0x1039c，0x00000200/0x3838，0x00100000/0x92ac，0x00200000/0x12c80，0x00300000/0x1b240，0x10004000/0x2d50，0x10007000/0xf84，0x10007fdc/0x18（type 1）；0x60000000/0x1000 与 0x60001000/0x1000（type 7）。
- **S-01**（verified）：a740v3_sqe.fw 为 76,148 字节（0x12974），是 4 的倍数，共 19,037 个双字。dword0 为 0x00000000。
- **S-03**（verified）：按驱动的版本规则，该文件的版本为 0x01511175。dword1 的低 4 位为 0x5，因此不适用特殊情况（低 4 位为 0xa），版本即 dword1 本身。该文件不使用 dword3。
- **S-06**（verified）：dword2 为 0x01004a5c。其低 24 位 0x4a5c = 19,036，既等于加载器复制的双字数 (size-4)/4，也等于文件最后一个双字的索引。

## 开放项

下方声明已排除或重新表述；每条都给出了原因。

| ID | 标注 | 复现 / 推翻 | 结论（原文） |
|---|---|---|---|
| G-03 | verified | confirmed / needs_change | Block records start at 0x0b0 and sit between payloads. Header = {addr, size, dword +0x8, dword +0xc}; payload begins at header+0x10 and is size bytes. |
| G-04 | verified | confirmed / needs_change | Descriptor versus block cross-match: descriptors at 0x00000000 (size 0x1dc), 0x00004000 (size 0x1039c) and 0x10007fdc (size 0x18) have payload blocks with the same addr and size, at block headers 0x0b0, 0x29c and 0x10680. |
| G-05 | verified | needs_change / needs_change | The first payload (file offset 0xc0) starts with four words 0x10007f80, 0x00000105, 0x00000145, 0x00000159. Words 1-3 have bit 0 set, which is the usual Thumb-mode bit. With bit 0 cleared, the words give 0x104, 0x144 and 0x158 bytes past the payload start, and the code at file offsets 0x1c4, 0x204 and 0x218 decodes as Thumb instructions, each opening with an ldr, ldr, lsl.w and add.w sequence. |
| G-07 | observed | confirmed / needs_change | Descriptor parser gen7_gmu_parse_fw reads each 16-byte record as: +0x00 addr; +0x04 must be 0, and the first record with a non-zero word here ends the |
| G-08 | observed | confirmed / needs_change | Loader gen7_gmu_load_fw reads block headers differently from the parser: +0x00 addr; +0x04 size. A zero size means skip 16 bytes. Otherwise the payload begins at header+0x10 and is size bytes long, and the next header starts directly after it. |
| G-09 | observed | needs_change / refuted | The driver's own names for the block fields are addr and size, used in GMU block messages. No module string names the zero word, the type word, or the |
| G-10 | verified | confirmed / needs_change | gmu_gen70200.bin contains no version or build identifier in any printable run of 5 characters or more. Its printable text is firmware routine names and diagnostic messages in 125 of its 447 runs of 5 characters or more, for example HfiMemReply, GmuPwrRequestPowerOn, IPCC_Attach Error and Invalid size, and byte sequences that contain no name or message in the other 322 runs. Most of the printable runs contain no run of four letters. |
| G-11 | observed | needs_change / needs_change | The driver obtains the GMU firmware version at runtime through HFI and checks it against a minimum, rather than reading it from the image. Strings: 'G |
| G-12 | observed | confirmed / needs_change | The GMU payload contains named firmware routines and diagnostics, for example HfiMemReply, HfiFeatureCtrl, GmuPwrRequestPowerOn, EcpSetupPreemptBuffers. The payload also holds diagnostic messages such as IPCC_Register Failed Arg. |
| S-02 | verified | confirmed / needs_change | The leading dwords are dword1=0x01511175, dword2=0x01004a5c, dword3=0x01511146, dword4=0x010024c8, dword5=0x010e4e50, dword6=0xb8020000, dword7=0xb80300ef. |
| S-04 | observed | needs_change / needs_change | The SQE loader adreno_get_firmware copies file bytes from offset 4 to the end (size-4 bytes, 76,144 bytes = 19,036 dwords) into a kernel-global buffer. The buffer is allocated by kgsl_allocate_global, called with size-4 as its second argument and the string 'ucode' as its sixth argument. The copy writes to the address stored at offset 0x8 of the returned descriptor, and it runs only when the returned value is not an error pointer. |
| S-05 | observed | confirmed / needs_change | The SQE firmware structure sits at device offset 0x1510 (gen7_microcode_read passes x0+0x1510). Its version field is at struct offset 0x10 (absolute 0 |
| S-07 | verified | confirmed / needs_change | dword4 is 0x010024c8: low 24 bits 0x24c8 = 9416, which indexes dword 0x01000000 (same top-byte pattern as dword4). dword5 is 0x010e4e50: low 24 bits 0x0e4e50 = 937552, which is past the end of the file (last dword index 19036). |
| S-08 | verified | needs_change / needs_change | The SQE file ends with ASCII '01504202' at file offset 0x12964, followed by 'ESU$' and 'R' with three zero bytes. The last dword is 0x00000052. This i |
| X-01 | verified | confirmed / needs_change | msm_kgsl.ko contains both target file names as literal strings. The SQE name a740v3_sqe.fw is a candidate in the SQE table, and gmu_gen70200.bin is the GMU name in the same entry, adreno_gpu_core_gen7_6_0, at offset 0x60. That entry is the last of the 43 entries in adreno_gpulist. |
| X-02 | observed | confirmed / needs_change | The GMU file name is read from device config +0x60 (fallback at +0x68) inside gen7_gmu_parse_fw, and the GMU firmware is requested through firmware_request_nowarn. |
| T-01 | verified | needs_change / confirmed | For both formats: the record tables, block walk, descriptor cross-match, SQE version rule and dwor |
