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

下表每行给出复现或推翻的结论，以及声明的完整文字。

| ID | 标注 | 复现 / 推翻 | 声明 |
|---|---|---|---|
| G-03 | verified | confirmed / needs_change | 块记录从 0x0b0 开始，位于各负载之间。头部为 {addr, size, dword +0x8, dword +0xc}；负载从头部 +0x10 处开始，长度为 size 字节。 |
| G-04 | verified | confirmed / needs_change | 描述符与块的交叉匹配：描述符位于 0x00000000（size 0x1dc）、0x00004000（size 0x1039c）和 0x10007fdc（size 0x18），它们分别与块头 0x0b0、0x29c 和 0x10680 处的负载块具有相同的 addr 与 size。 |
| G-05 | verified | needs_change / needs_change | 第一个负载（文件偏移 0xc0）以四个字开头：0x10007f80、0x00000105、0x00000145、0x00000159。第 1 至 3 个字的 bit 0 被置位，即常见的 Thumb 模式位。清除 bit 0 后，这些字分别给出负载起点之后 0x104、0x144 和 0x158 字节的位置；文件偏移 0x1c4、0x204 和 0x218 处的代码解码为 Thumb 指令，每处都以 ldr、ldr、lsl.w、add.w 的序列开始。 |
| G-07 | observed | confirmed / needs_change | 描述符解析函数 gen7_gmu_parse_fw 将每条 16 字节记录读作：+0x00 为 addr；+0x04 必须为 0，第一条 +0x04 字为非零的记录会结束描述符遍历，且函数返回 0。类型字位于 +0x08，类型为 1 或 7 的记录被处理，其余记录被跳过。该函数位于 vendor_dlkm_a 的 msm_kgsl.ko 中，不在 gmu_gen70200.bin 中。 |
| G-08 | observed | confirmed / needs_change | 加载函数 gen7_gmu_load_fw 读取块头的方式与解析函数不同：+0x00 为 addr；+0x04 为 size。size 为零时跳过 16 字节。否则负载从头部 +0x10 处开始，长度为 size 字节，下一个头部紧接其后。 |
| G-09 | observed | needs_change / refuted | 驱动对块字段的命名为 addr 与 size，并用于 GMU 块消息。模块字符串并未命名零字、类型字或其余两个字段，这一说法不成立：块消息命名了 addr 和 size。在模块中，只有零字和类型字没有名称。 |
| G-10 | verified | confirmed / needs_change | gmu_gen70200.bin 中，在任何长度不少于 5 个字符的可打印串中都不含版本或构建标识符：其 434 个这样的串中，没有一个含有 version 或 build 一词，也没有带小数点的数字。其中 164 个串含有四个连续字母，包括 HfiMemReply、GmuPwrRequestPowerOn、IPCC_Attach Error 和 Invalid size 等例程名与诊断消息。其余 270 个串不含四个连续字母。 |
| G-11 | observed | needs_change / needs_change | 驱动在运行时通过 HFI 获得 GMU 固件版本，并将其与最低版本比较，而不是从镜像中读取。字符串 'GMU FW version 0x%x does not satisfy the required minimum (0x%x)' 由 gen7_gmu_version_info 打印，它是唯一引用该字符串的函数。该函数通过 kgsl_regmap_read 从五个 GMU 寄存器读取版本，不进行 HFI 调用，并将第一个寄存器值与设备偏移 0x1508 所指结构中偏移 0x50 处存储的最低版本比较。因此，关于该函数通过 HFI 获得版本的说法不成立。 |
| G-12 | observed | confirmed / needs_change | GMU 负载包含命名的固件例程与诊断信息，例如 HfiMemReply、HfiFeatureCtrl、GmuPwrRequestPowerOn、EcpSetupPreemptBuffers。负载中还包含诊断消息，例如 IPCC_Register Failed Arg。 |
| S-02 | verified | confirmed / needs_change | 前导的双字为 dword1=0x01511175、dword2=0x01004a5c、dword3=0x01511146、dword4=0x010024c8、dword5=0x010e4e50、dword6=0xb8020000、dword7=0xb80300ef。 |
| S-04 | observed | needs_change / needs_change | SQE 加载函数 adreno_get_firmware 将文件从偏移 4 到末尾的字节（size-4 字节，即 76,144 字节 = 19,036 个双字）复制到内核全局缓冲区中。该缓冲区由 kgsl_allocate_global 分配，其第二个参数为 size-4，第六个参数为字符串 'ucode'。复制操作写入所返回描述符偏移 0x8 处保存的地址，且仅当返回值不是错误指针时执行。 |
| S-05 | observed | confirmed / needs_change | SQE 固件结构位于设备偏移 0x1510 处（gen7_microcode_read 传递 x0+0x1510）。其版本字段位于结构偏移 0x10（绝对偏移 0x1520），adreno_get_firmware 将版本存放在该处，取值依据版本规则从文件的 dword1 得出。 |
| S-07 | verified | confirmed / needs_change | dword4 为 0x010024c8：低 24 位 0x24c8 = 9416，它索引到双字 0x01000000（与 dword4 的最高字节模式相同）。dword5 为 0x010e4e50：低 24 位 0x0e4e50 = 937552，超出文件末尾（最后一个双字索引为 19036）。 |
| S-08 | verified | needs_change / needs_change | SQE 文件以 ASCII '01504202' 结尾，位于文件偏移 0x12964，其后为 'ESU$' 和带三个零字节的 'R'。最后一个双字为 0x00000052。这是文件的最后一个双字，位于索引 19,036（0x4a5c）和文件偏移 0x12970，即 19,037 个双字中的最后一个。文件并不以上述字符串结尾：该字符串始于文件末尾之前 16 字节处。 |
| X-01 | verified | confirmed / needs_change | msm_kgsl.ko 中包含两个目标文件名作为字面字符串。gmu_gen70200.bin 是 adreno_gpu_core_gen7_6_0 偏移 0x60 处的 GMU 名称，该结构是 adreno_gpulist 的 43 个条目中的最后一个。SQE 名称 a740v3_sqe.fw 同样是模块中的字面字符串，但不在该条目中。 |
| X-02 | observed | confirmed / needs_change | GMU 文件名从 gen7_gmu_parse_fw 内的设备配置 +0x60 读取（回退位置为 +0x68），GMU 固件通过 firmware_request_nowarn 请求。 |
| T-01 | verified | needs_change / confirmed | 对两种格式：记录表、块遍历、描述符交叉匹配、SQE 版本规则与双字检查均已测试。对 gmu_gen70200.bin，记录、块遍历与描述符交叉匹配检查均成立（G-01、G-02、G-04）。对 a740v3_sqe.fw，这些检查不适用：其第一条 16 字节记录在 +0x04 处有非零字，因此不属于 GMU 描述符格式；在偏移 0 处的块头会结束于 0x1511185，超出 0x12974 字节文件的末尾。SQE 版本规则与双字检查成立（S-03、S-06、S-07）。 |
