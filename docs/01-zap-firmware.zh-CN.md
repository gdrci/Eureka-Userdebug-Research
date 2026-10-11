# 01 - Zap 安全固件（a740v3）

[English](01-zap-firmware.md)

Zap 是安全世界加载的 GPU 固件。镜像分为三部分，同一个代码段同时出现在 ELF 与签名封装中。

## 已验证的结论

- **F1**（verified）：文件大小：.mdt 6284 字节（0x188c），.b00 148（0x94），.b01 1072（0x430），.b02 6136（0x17f8），.elf 5168（0x1430），.mbn 14328（0x37f8）。完整 SHA-256 见证据文件。
- **F2**（verified）：.mdt 以 ELF32 小端格式头部开头：EI_CLASS 1，EI_DATA 1，EI_VERSION 1，OS/ABI 0。字段：e_type 2（EXEC），e_machine 164（0xa4），e_version 1，e_entry 0x1000，e_phoff 0x34，e_shoff 0（无节头表），e_flags 0x3，e_ehsize 52，e_phentsize 32，e_phnum 3，e_shentsize 0，e_shnum 0，e_shstrndx 0。
- **F3**（verified）：.mdt 中位于 0x34 的程序头表有三个 32 字节条目。[0]：p_type 0，p_offset 0x0，p_filesz 0x94（148），p_flags 0x07000000。[1]：p_type 1（PT_LOAD），p_offset 0x1000，p_vaddr 0x1000，p_filesz 0x430（1072），p_flags 0x08000007，p_align 0x100000。[2]：p_type 0，p_offset 0x2000，p_filesz 0x17f8（6136），p_flags 0x02000000，p_align 0x1000。.mbn 的表中有相同的三个条目。
- **F5**（verified）：.b00 与 .mdt 的 0x0 至 0x93 字节逐字节相同（148 字节）。.b02 与 .mdt 从 0x94 到末尾的字节逐字节相同（6136 字节）。.b01 的字节在 .mdt 中不出现。
- **F9**（verified）：.elf 与 .mbn 的头部字节不同：.elf 的 e_phnum 为 1、只有一个程序头，.mbn 的 e_phnum 为 3、有三条表项。在重叠前缀 0x0 至 0x1430 中有 30 个字节不同，首个在 0x2c，末个在 0x91。0x1000 至 0x142f 的代码字节相同。
- **F13**（verified）：.b01、.b02、.mdt 或 .mbn 的 32 字节 SHA-256 摘要，均未以原始字节序列出现在哈希段中。
- **F16**（verified）：.elf、.b00 与 .b01 中没有长度 4 或以上的可打印字符串。.b02、.mdt 与 .mbn 各含 83 段此类字符串，均位于哈希段中，即 F14 列出的证书名、有效期字符串和类随机片段。未找到构建 ID、版本字符串或消息名。
- **F17**（verified）：代码段（.b01，在 .elf 与 .mbn 中的文件偏移为 0x1000，虚拟地址 0x1000）从 e_entry 0x1000 开始。第一个小端字为 0x01511014。
- **F19**（verified）：对完整的 1072 字节 .b01 使用同一解码器与默认 CPU 反汇编，得到 202 个由大括号界定的数据包块，以及 60 条“无效指令编码”警告。警告的原因未确定。

## 开放项

下方声明已排除或重新表述；每条都给出了原因。

| ID | 标注 | 复现 / 推翻 | 结论（原文） |
|---|---|---|---|
| F7 | verified | confirmed / needs_change | .elf (5168 bytes) is an ELF32 EXEC file with e_machine 164, named Qualcomm Hexagon in the ELF header dump. e_entry is 0x1000, e_phnum is 1 and there a |
| F8 | verified | confirmed / needs_change | .mbn is 14328 bytes (0x37f8) and is the same ELF image with the three-entry table. Bytes 0x0 to 0x93 equal .b00, bytes 0x1000 to 0x142f equal .b01, and bytes 0x2000 to 0x37f7 equal .b02. |
| F10 | verified | confirmed / needs_change | The hash segment (b02, mbn file offset 0x2000) begins with little-endian u32 words at segment offsets: +0x00 0x0, +0x04 0x7, +0x08 0x18, +0x0c 0x0, +0x10 0xe0, +0x14 0x90, +0x18 0x0, +0x1c 0x0. |
| F11 | verified | needs_change / needs_change | Hash segment (b02, 6136 bytes) layout: [0x00,0x68) header words, 13 nonzero bytes. [0x68,0x218) a 432-byte block with 205 nonzero bytes. [0x218,0x10a8) holds five X.509-style certificate structures placed back to back, C1 to C5, of 645, 772, 773, 772 and 766 bytes. [0x10a8,0x17f8) is 1872 bytes, all equal to 0xff. |
| F14 | verified | needs_change / needs_change | The hash segment contains five X.509-style certificate structures with these name and validity strings at hash-segment offsets (hex). C1 at 0x218 (645 bytes) has subject CN Eureka Qcom FW Signing Certificate at 0x338, notBefore 230516210606Z at 0x2b4 and notAfter 430511210606Z at 0x2c3. C2 at 0x49d (772 bytes), C3 at 0x7a1 (773 bytes), C4 at 0xaa6 (772 bytes) and C5 at 0xdaa (766 bytes) each have identical issuer and subject CN values Eureka_FW_Root_0, Eureka_FW_Root_1, Eureka_FW_Root_2 and Eureka_FW_Root, with notBefore strings at 0x53b, 0x83f, 0xb44 and 0xe46. |
| F18 | verified | needs_change / needs_change | Disassembling the first 64 bytes of .b01 with the hexagon triple (default CPU) yields ten brace-delimited packets: { r1 = memw(r5+#4) ; r4 = memub(r1+ |
