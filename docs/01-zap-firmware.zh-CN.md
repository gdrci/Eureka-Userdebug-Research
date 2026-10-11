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

下表每行给出复现或推翻的结论，以及声明的完整文字。

| ID | 标注 | 复现 / 推翻 | 声明 |
|---|---|---|---|
| F7 | verified | confirmed / needs_change | .elf（5168 字节）是 ELF32 EXEC 文件，e_machine 为 164，在 ELF 头部转储中被命名为 Qualcomm Hexagon。e_entry 为 0x1000，e_phnum 为 1，且不存在节头表（e_shoff 为 0，e_shnum 为 0）。同一机器值在另一种 ELF 转储中被命名为 QUALCOMM DSP6 Processor，因此名称取决于所用的转储。 |
| F8 | verified | confirmed / needs_change | .mbn 为 14328 字节（0x37f8），与 .elf 为同一 ELF 映像，带有三项程序头表。字节 0x0 至 0x93 与 .b00 相同，字节 0x1000 至 0x142f 与 .b01 相同，字节 0x2000 至 0x37f7 与 .b02 相同。 |
| F10 | verified | confirmed / needs_change | 哈希段（b02，mbn 文件偏移 0x2000）以小端 u32 字开头，段内偏移为：+0x00 0x0，+0x04 0x7，+0x08 0x18，+0x0c 0x0，+0x10 0xe0，+0x14 0x90，+0x18 0x0，+0x1c 0x0。 |
| F11 | verified | needs_change / needs_change | 哈希段（b02，6136 字节）布局：[0x00,0x68) 为头部字，其中 13 个字节非零。[0x68,0x218) 为 432 字节的块，其中 205 个字节非零。[0x218,0x10a8) 依次排列五个 X.509 风格的证书结构 C1 至 C5，长度分别为 645、772、773、772 和 766 字节。[0x10a8,0x17f8) 为 1872 字节，全部等于 0xff。 |
| F14 | verified | needs_change / needs_change | 哈希段包含五个 X.509 风格的证书结构，其名称与有效期字符串位于以下哈希段偏移（十六进制）处。C1 位于 0x218（645 字节），其主题 CN 为 Eureka Qcom FW Signing Certificate，位于 0x338；notBefore 230516210606Z 位于 0x2b4，notAfter 430511210606Z 位于 0x2c3。C2 位于 0x49d（772 字节）、C3 位于 0x7a1（773 字节）、C4 位于 0xaa6（772 字节）、C5 位于 0xdaa（766 字节），各自的颁发者与主题 CN 值相同，依次为 Eureka_FW_Root_0、Eureka_FW_Root_1、Eureka_FW_Root_2 和 Eureka_FW_Root，notBefore 字符串位于 0x53b、0x83f、0xb44 和 0xe46。 |
| F18 | verified | needs_change / needs_change | 对 .b01 的前 64 字节以 hexagon 三元组（默认 CPU）反汇编，得到 10 个大括号分隔的包：{ r1 = memw(r5+#4) ; r4 = memub(r1+#0) }。计数不成立：同样的 64 字节解码为 13 个大括号分隔的包，且文件偏移 0x2c 与 0x3c 处的字不能解码。 |
