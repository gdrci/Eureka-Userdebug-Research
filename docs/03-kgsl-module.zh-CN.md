# 03 - kgsl 内核模块

[English](03-kgsl-module.md)

kgsl 模块是内核驱动，负责加载 GPU 固件并提供 GPU 设备。只有两项结论通过了两项检查，因此本页较短。

## 已验证的结论

- **F1**（verified）：ELF 类型与机器：64 位小端 AArch64 可重定位目标（ET_REL，EM_AARCH64，ELFOSABI_NONE，EV_CURRENT）。未剥离（存在 .symtab）。无程序头。文件大小 2868976 字节。SHA-256 e5d17bec61d20020709578cac263396988524e5e08b95f27e71555011c95d21c。SHA-1 构建 ID a09092d7b3e78c37956e0b71ad84ba4c4b59e21f。
- **F16**（verified）：导入：506 个未定义符号（507 条 __versions 记录，多出的一条是 module_layout）。其中 35 个由本分区中的同级模块导出：qcom-scm 17 个，llcc-qcom 5 个，qcom_iommu_util 3 个，mem_buf_dev 2 个，mdt_loader 2 个，cmd-db 2 个，clk-qcom 2 个，secure_buffer 1 个，log_gpu_error 1 个。这九个模块与 F2 的 depends 列表完全一致。其余 471 个未定义名称的归属见 F17。

## 开放项

下方声明已排除或重新表述；每条都给出了原因。

| ID | 标注 | 复现 / 推翻 | 结论（原文） |
|---|---|---|---|
| F2 | verified | confirmed / needs_change | Module metadata: description '3D Graphics driver'; license 'GPL v2'; name 'msm_kgsl'; intree=Y; softdep 'pre: arm_smmu nvmem_qfprom'; vermagic '5.10.2 |
| F3 | verified | needs_change / needs_change | The module declares no module parameters (no parm= entries) and no author field. Only the modinfo keys listed in F2 are present. |
| F4 | verified | needs_change / needs_change | Device, class and driver naming strings: device name kgsl-3d0 (with kgsl-3d as a related string); the bare string kgsl appears three times and is used |
| F6 | verified | needs_change / needs_change | Device-tree match and related compatible strings: qcom,kgsl-3d0 (alias in F2); Adreno GPU compatibles qcom,adreno-gpu-a619-holi, qcom,adreno-gpu-a621, |
| F7 | verified | confirmed / needs_change | Firmware file names present as literal strings (26 concrete names): a300_pfp.fw, a300_pm4.fw, a530_gpmu.fw2, a530_pfp.fw, a530_pm4.fw, a530v2_seq.fw2, |
| F9 | verified | confirmed / needs_change | Distinct uppercase KGSL_ tokens: 13. KGSL_EVENT_CANCELLED, KGSL_EVENT_RETIRED, KGSL_FENCE, KGSL_FT_DISABLE, KGSL_FT_OFF, KGSL_FT_REPLAY, KGSL_FT_SKIPC |
| F11 | verified | needs_change / needs_change | Unique lowercase kgsl_ symbol names: 473 after removing .cfi_jt, hash, .llvm and local suffixes. Prefix counts: kgsl_ioctl 61, kgsl_iommu 37, kgsl_mmu |
| F12 | verified | confirmed / needs_change | Ioctl handler symbols: 46 distinct kgsl_ioctl_* names (including kgsl_ioctl_funcs, kgsl_ioctl_helper, kgsl_ioctl_copy_in and kgsl_ioctl_copy_out). Com |
| F14 | verified | confirmed / needs_change | Exported symbols: six non-GPL exports from section __ksymtab: kgsl_add_rcu_notifier, kgsl_del_rcu_notifier, kgsl_get_stats, kgsl_gpu_frame_count, kgsl |
| F15 | verified | needs_change / needs_change | No other module in the vendor_dlkm_a partition (262 .ko files) imports any of the six kgsl_ exports. The export linkage in this partition is one-direc |
| F17 | verified | confirmed / needs_change | Undefined symbols with no exporter in this partition: 471 names. Among them, vendor-style names with no exporter in the partition are icc_put, icc_set |
| F18 | verified | needs_change / needs_change | Section sizes: .text 716632 bytes, .rodata 148096, .data 108880, .bss 17800 (NOBITS), __versions 32448, .modinfo 353, .symtab 482784, .strtab 254333.  |
| F19 | verified | needs_change / refuted | Ftrace event classes and tracepoints for kgsl events are present, for example event_class_kgsl_gpubusy, event_class_kgsl_mem_alloc, event_class_kgsl_r |
