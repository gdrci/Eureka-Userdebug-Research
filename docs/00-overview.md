# 00 - Overview

[中文版](00-overview.zh-CN.md)

This repository documents the GPU subsystem of the Eureka userdebug image. The subsystem has four parts: firmware that the GPU runs, the kernel driver that loads it, the userspace libraries that applications call, and the init and SELinux entries that start them.

## What was unpacked

The super image has eight logical partitions (`data/gpu/partitions.tsv`). The GPU material is in two of them:

| Partition | Size (bytes) | Holds |
|---|---:|---|
| `vendor_a` | 426,274,816 | GPU firmware under `firmware/`, the userspace libraries, the init scripts and vendor SELinux policy |
| `vendor_dlkm_a` | 63,754,240 | the kgsl kernel module `lib/modules/msm_kgsl.ko` |

The other partitions hold GPU-related files that the first pass did not decode (section list below).

## Decoded so far

Five target groups were decoded: the zap image (`01`), the SQE and GMU firmware (`02`), the kgsl module (`03`), the userspace libraries (`04`), and the init and SELinux entries (`05`). Each claim was reproduced by one check and then tested by a second check that tried to refute it. Only claims that passed both are in these documents.

- 31 claims passed both checks. They are in `data/gpu/confirmed_findings.json`.
- Some claims were judged right in substance but worded or labelled wrongly. They are not in these documents yet. Each section lists the open items.

## Not yet decoded

A completeness pass over the mounted partitions returned 143 files whose names match GPU terms (`data/gpu/uncovered_candidates.json`). They are not covered by sections `01` to `05`. The main groups:

- **`system_a`**: the `gpuservice` daemon (`system/bin/gpuservice`), its init file, an eBPF object for GPU memory (`system/etc/bpf/gpu_mem.o`), and the framework-graphics boot images.
- **`odm_a`**: the Oculus composer HAL (`vendor.oculus.hardware.graphics.composer@1.0`, `@1.1`) and the Oculus GPU-control HAL (`vendor.oculus.hardware.graphics.gpucontrol@1.0`), in 32-bit and 64-bit builds, with their VINTF manifest entries.
- **`product_a`**: the OpenGL ES and Vulkan feature declarations (`etc/permissions/android.hardware.opengles.aep.xml` and the Vulkan level files).

These need a second decoding run before any claim is made about them.

## Reproducing the evidence

The commands in `data/gpu/confirmed_findings.json` call the helpers in `tools/` and use shell variables for the mounted paths. `tools/README.md` gives the build steps and the variable values, so each command can be run from the repository.

## Hardware

The firmware names the GPU as `a740v3` (Adreno 740 generation). The kernel module is `msm_kgsl`, with the device node `kgsl-3d0`. The device tree is not in the image, so the GPU's base address and clock tables are not covered.

## What this build does not contain

- The boot chain, TrustZone, hypervisor and kernel images are not in the super image. The stalled items from the Greatwhite research (object `0x91`, the OEM key reader and the SBL1 jump) cannot be rechecked on this build from these partitions.
- The firmware signatures and their provenance are not established by these documents. The documents describe the bytes as mounted.
