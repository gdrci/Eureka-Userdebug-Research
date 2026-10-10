# 03 - kgsl kernel module

[中文版](03-kgsl-module.zh-CN.md)

The kgsl module is the kernel driver that loads GPU firmware and exposes the GPU device. Only two of its claims passed both checks, so this page is short.

## Verified claims

- **F1** (verified): ELF type and machine: 64-bit little-endian AArch64 relocatable object (ET_REL, EM_AARCH64, ELFOSABI_NONE, EV_CURRENT). Not stripped (.symtab present). No program headers. File size 2868976 bytes. SHA-256 e5d17bec61d20020709578cac263396988524e5e08b95f27e71555011c95d21c. SHA-1 build ID a09092d7b3e78c37956e0b71ad84ba4c4b59e21f.
- **F16** (verified): Imports: 506 undefined symbols (507 __versions entries; the extra one is module_layout). 35 imports are exported by sibling modules in this partition: qcom-scm 17 (qcom_scm_dcvs_ca_available, qcom_scm_dcvs_core_available, qcom_scm_dcvs_init_ca_v2, qcom_scm_dcvs_init_v2, qcom_scm_dcvs_reset, qcom_scm_dcvs_update, qcom_scm_dcvs_update_ca_v2, qcom_scm_dcvs_update_v2, qcom_scm_io_reset, qcom_scm_kgsl_set_smmu_aperture, qcom_scm_kgsl_set_smmu_lpac_aperture, qcom_scm_pas_auth_and_reset, qcom_scm_pas_shutdown_retry, qcom_scm_set_remote_state, qtee_shmbridge_allocate_shm, qtee_shmbridge_free_shm, qtee_shmbridge_is_enabled); llcc-qcom 5 (llcc_get_slice_id, llcc_slice_activate, llcc_slice_deactivate, llcc_slice_getd, llcc_slice_putd); qcom_iommu_util 3 (qcom_alloc_io_pgtable_ops, qcom_free_io_pgtable_ops, qcom_iommu_get_asid_nr); mem_buf_dev 2 (mem_buf_dma_buf_copy_vmperm, mem_buf_dma_buf_exclusive_owner); mdt_loader 2 (qcom_mdt_get_size, qcom_mdt_load); cmd-db 2 (cmd_db_read_addr, cmd_db_read_aux_data); clk-qcom 2 (qcom_clk_dump, qcom_clk_set_flags); secure_buffer 1 (hyp_assign_table); log_gpu_error 1 (log_gpu_error). These nine modules are exactly the depends= list in F2. The remaining 471 undefined names are attributed in F17.

## Open items

These claims were reproduced but judged wrong in wording or label, or they were refuted. They are not stated above until the wording is corrected.

| ID | Label | Reproduce / refute | Claim (as produced) |
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
