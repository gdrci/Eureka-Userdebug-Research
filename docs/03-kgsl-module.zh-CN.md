# 03 - kgsl 内核模块

[English](03-kgsl-module.md)

kgsl 模块是内核驱动，负责加载 GPU 固件并提供 GPU 设备。只有两项结论通过了两项检查，因此本页较短。

## 已验证的结论

- **F1**（verified）：ELF 类型与机器：64 位小端 AArch64 可重定位目标（ET_REL，EM_AARCH64，ELFOSABI_NONE，EV_CURRENT）。未剥离（存在 .symtab）。无程序头。文件大小 2868976 字节。SHA-256 e5d17bec61d20020709578cac263396988524e5e08b95f27e71555011c95d21c。SHA-1 构建 ID a09092d7b3e78c37956e0b71ad84ba4c4b59e21f。
- **F16**（verified）：导入：506 个未定义符号（507 条 __versions 记录，多出的一条是 module_layout）。其中 35 个由本分区中的同级模块导出：qcom-scm 17 个，llcc-qcom 5 个，qcom_iommu_util 3 个，mem_buf_dev 2 个，mdt_loader 2 个，cmd-db 2 个，clk-qcom 2 个，secure_buffer 1 个，log_gpu_error 1 个。这九个模块与 F2 的 depends 列表完全一致。其余 471 个未定义名称的归属见 F17。

## 开放项

下表每行给出复现结果与声明的完整文字。

| ID | 标注 | 复现 / 状态 | 声明 |
|---|---|---|---|
| F2 | verified | 成立 / 已更正 | 模块元数据：description '3D Graphics driver'；license 'GPL v2'；name 'msm_kgsl'；intree=Y；softdep 'pre: arm_smmu nvmem_qfprom'；vermagic '5.10.205-01655-g5c5a360e9c21 SMP preempt mod_unload modversions aarch64'；depends 'qcom_iommu_util,qcom-scm,cmd-db,llcc-qcom,clk-qcom,log_gpu_error,mdt_loader,secure_buffer,mem_buf_dev'；alias 'of:N*T*Cqcom,kgsl-3d0' 以及 'of:N*T*Cqcom,kgsl-3d0C*'。 |
| F3 | verified | 已更正 / 已更正 | 模块未声明任何模块参数（没有 parm= 条目），也没有 author 字段。只存在 F2 所列的 modinfo 键。 |
| F4 | verified | 已更正 / 已更正 | 设备、类与驱动的命名字符串：设备名 kgsl-3d0（kgsl-3d 为相关字符串）；裸字符串 kgsl 出现三次，分别在 kgsl_core_init 中用作类名、字符设备区域名以及传给 dev_set_name 的名称，并用作 kgsl_core_procfs_init 与 kgsl_core_debugfs_init 中创建的 procfs 和 debugfs 目录的名称，同时也是 83 个名称以 event_class_ 开头的对象的第一个字段，例如 event_class_kgsl_gpubusy。 |
| F6 | verified | 已更正 / 已更正 | 设备树匹配与相关兼容字符串：qcom,kgsl-3d0（见 F2 的 alias）；Adreno GPU 兼容字符串 qcom,adreno-gpu-a619-holi、qcom,adreno-gpu-a621、qcom,adreno-gpu-a660-shima、qcom,adreno-gpu-a662、qcom,adreno-gpu-gen6-3-26-0、qcom,adreno-gpu-gen7-0-0、qcom,adreno-gpu-gen7-0-1、qcom,adreno-gpu-gen7-3-0、qcom,adreno-gpu-gen7-4-0 以及 qcom,adreno-gpu-gen7-6-0。 |
| F7 | verified | 成立 / 已更正 | 以字面字符串出现的固件文件名（26 个具体名称）：a300_pfp.fw、a300_pm4.fw、a530_gpmu.fw2、a530_pfp.fw、a530_pm4.fw、a530v2_seq.fw2、a530v3_gpmu.fw2、a530v3_seq.fw2、a540_gpmu.fw2、a612_rgmu.bin、a619_gmu.bin、a621_gmu.bin、a630_gmu.bin、a630_sqe.fw、a640_gmu.bin、a650_gmu.bin、a650_sqe.fw、a660_gmu.bin、a660_sqe.fw、a662_gmu.bin、a710_sqe.fw、a730_sqe.fw、a740v3_sqe.fw、c500_gmu.bin、gmu_gen70000.bin、gmu_gen70200.bin。 |
| F9 | verified | 成立 / 已更正 | 不同的大写 KGSL_ 标记：13 个。KGSL_EVENT_CANCELLED、KGSL_EVENT_RETIRED、KGSL_FENCE、KGSL_FT_DISABLE、KGSL_FT_OFF、KGSL_FT_REPLAY、KGSL_FT_SKIPCMD、KGSL_FT_SKIPFRAME、KGSL_FT_SKIPIB、KGSL_FT_TEMP_DISABLE、KGSL_FT_THROTTLE、KGSL_USER_MEM_TYPE_ADDR、KGSL_USER_MEM_TYPE_ION。标识符 IOCTL_KGSL_DEVICE_WAITTIMESTAMP 不计入，因为它以 IOCTL_ 开头。 |
| F11 | verified | 已更正 / 已更正 | 去除 .cfi_jt、.llvm 与哈希后缀后，不同的小写 kgsl_ 符号名共 489 个。前缀计数：kgsl_ioctl_ 61，kgsl_iommu 37，kgsl_mmu 30。 |
| F12 | verified | 成立 / 已更正 | ioctl 处理函数符号：符号表中共 61 个不同的 kgsl_ioctl_* 名称，不计裸符号 kgsl_ioctl（已去除 .cfi_jt 与 .llvm 后缀）。其中包括 kgsl_ioctl_funcs（局部只读对象），以及全局函数 kgsl_ioctl_helper、kgsl_ioctl_copy_in 和 kgsl_ioctl_copy_out。 |
| F14 | verified | 成立 / 已更正 | 导出符号：来自 __ksymtab 节的六个非 GPL 导出：kgsl_add_rcu_notifier、kgsl_del_rcu_notifier、kgsl_get_stats、kgsl_gpu_frame_count、kgsl_gpu_num_freqs、kgsl_gpu_stat。 |
| F15 | verified | 已更正 / 已更正 | vendor_dlkm_a 分区中（共 262 个 .ko 文件）没有其他模块导入 kgsl_ 导出的六个符号中的任何一个。该分区中的导出链接是单向的：msm_kgsl.ko 从九个同级模块导入其 506 个未定义符号中的 35 个，并且没有模块导入 msm_kgsl.ko 定义的任何符号。 |
| F17 | verified | 成立 / 已更正 | 本分区中无导出者的未定义符号：471 个名称。其中包括 icc_put、icc_set_bw、icc_set_tag 和 of_icc_get，vendor_dlkm_a 中没有任何模块导出这些符号。 |
| F18 | verified | 已更正 / 已更正 | 节大小：.text 716632 字节，.rodata 148096，.data 108880，.bss 17800（NOBITS），__versions 32448，.modinfo 353，.symtab 482784，.strtab 254333。 |
| F19 | verified | 已更正 / 不成立 | kgsl 事件的 ftrace 事件类与跟踪点存在，例如 event_class_kgsl_gpubusy、event_class_kgsl_mem_alloc、event_class_kgsl_readtimestamp。 |
