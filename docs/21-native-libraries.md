# 21 - Native shared libraries

[中文版](21-native-libraries.zh-CN.md)

## Summary

The native-libraries group is 1371 rows across five partitions: system_a 787, vendor_a 368, system_ext_a 169, odm_a 27 and product_a 20. It contains 1361 regular files and 10 symlinks. The regular files are 1351 ELF shared objects (854 ELF64 AArch64, 497 ELF32 ARM, all ET_DYN) and 10 LLVM bitcode files (libclc built-in libraries under system/lib and system/lib64). The symlinks are 8 bionic links (libc, libdl, libdl_android, libm in system/lib and system/lib64) that point into /apex/com.android.runtime, which is not in the mount, plus odm_a and vendor_a lib/modules links to the dlkm partitions. system_a holds the AOSP platform and NDK-public libraries plus HIDL/AIDL interface libraries and Rust dylibs. vendor_a holds Qualcomm HIDL/AIDL clients, HAL modules (lib/hw, lib64/hw), FastRPC, audio (AGM/PAL) and camera/display libraries. system_ext_a, product_a and odm_a hold device-specific libraries (Oculus/Meta telemetry, tracking, XR loader, maintenance boot, Vulkan layers). The configuration that governs the group is system/etc/public.libraries.txt (29 entries) and vendor/etc/public.libraries.txt (8 entries), plus SELinux labels in plat_file_contexts line 121 and vendor_file_contexts lines 142-153, 196. The role table is at roles.tsv with 1371 rows: 453 verified, 857 inferred, 61 unverified. `<image>/mnt` is the mount point of the image's partitions.

The role table has one row for each of the 1371 files in this group (453 verified, 0 observed, 857 inferred, 61 unverified). It is at `data/subsystems/native-libraries/roles.tsv`.

## Verified claims

- **G1** (verified): The group has 1371 rows: 1361 regular files and 10 symlinks. Partition counts are system_a 787, vendor_a 368, system_ext_a 169, odm_a 27, product_a 20 (proof in `data/subsystems/native-libraries/claims.json`, id G1).
- **G2** (verified): All 1361 regular files match the size and sha256 recorded in the inventory; the mounted copies are byte-identical to the classified rows (proof in `data/subsystems/native-libraries/claims.json`, id G2).
- **G3** (verified): Format split: 1351 ELF ET_DYN shared objects (854 ELF64 EM_AARCH64, 497 ELF32 EM_ARM) and 10 LLVM bitcode files, with no other file formats (proof in `data/subsystems/native-libraries/claims.json`, id G3).
- **G4** (verified): The 10 symlinks are 8 bionic links (libc.so, libdl.so, libdl_android.so, libm.so in system/lib and system/lib64) into /apex/com.android.runtime, and 2 kernel-module directory links (odm_a and vendor_a lib/modules) (proof in `data/subsystems/native-libraries/claims.json`, id G4).
- **G6** (verified): The role table covers each input row exactly once: 1371 rows, 453 verified, 857 inferred, 61 unverified. Roles marked unverified are name-only guesses (proof in `data/subsystems/native-libraries/claims.json`, id G6).
- **G7** (verified): There are 810 unique basenames across the 1371 rows; many names recur in lib and lib64 and in two or more partitions (proof in `data/subsystems/native-libraries/claims.json`, id G7).
- **G8** (verified): The system SELinux mapping labels every system/lib and system/lib64 path as system_lib_file, with bootstrap copies labelled system_bootstrap_lib_file (proof in `data/subsystems/native-libraries/claims.json`, id G8).
- **D2.1** (verified): system/lib64/libbinder.so is an ELF64 AArch64 DYN object with SONAME libbinder.so, a build-id note and 28 sections, stripped. It has 1392 defined dynamic symbols and 245 undefined entries, including the null entry (proof in `data/subsystems/native-libraries/claims.json`, id D2.1).
- **D2.2** (verified): libbinder.so depends on liblog, libcutils, libutils, libc++, libc, libm and libdl. Its exports include the C++ Binder API, for example _ZN7android12ProcessState15startThreadPoolEv (ProcessState::startThreadPool). It is not listed in public.libraries.txt (proof in `data/subsystems/native-libraries/claims.json`, id D2.2).
- **D3.1** (verified): system/lib64/libutils.so is an AArch64 DYN object with SONAME libutils.so and 389 defined dynamic symbols. Its exports include Looper and RefBase, and it depends on libcutils, liblog, libprocessgroup, libvndksupport, libc++, libc, libm and libdl (proof in `data/subsystems/native-libraries/claims.json`, id D3.1).
- **D4.1** (verified): system/lib64/liblog.so is the Android logging library. It exports __android_log_print, __android_log_write and android_log_* symbols with version tags such as @@LIBLOG, and depends only on libc++, libc, libm and libdl. It is listed in the NDK public-library file (proof in `data/subsystems/native-libraries/claims.json`, id D4.1).
- **D5.1** (verified): system/lib64/libcutils.so is the Android C utilities library. It exports property_get, ashmem_create_region, android_get_control_socket and socket_local_client, has 120 defined symbols, and depends on liblog, libbase, libc++, libc, libm and libdl (proof in `data/subsystems/native-libraries/claims.json`, id D5.1).
- **D6.2** (verified): The bitcode is consumed by libRSCpuRef.so, which contains the literal paths /system/lib64/libclcore.bc, libclcore_debug.bc and libclcore_debug_g.bc (proof in `data/subsystems/native-libraries/claims.json`, id D6.2).
- **D7.1** (verified): system/lib64/hw/android.hidl.memory@1.0-impl.so is a passthrough HIDL implementation with SONAME android.hidl.memory@1.0-impl.so. It exports the AshmemMapper and AshmemMemory classes in android::hidl::memory::V1_0::implementation, and depends on android.hidl.memory@1.0.so and libhidlbase.so (proof in `data/subsystems/native-libraries/claims.json`, id D7.1).
- **D7.2** (verified): The system VINTF manifest declares android.hidl.memory 1.0 as a passthrough HAL on 32+64-bit architectures, so the implementation is loaded into the client process (proof in `data/subsystems/native-libraries/claims.json`, id D7.2).
- **D8.1** (verified): vendor/lib64/hw/bootctrl.anorak.so is an AArch64 HAL module with SONAME bootctrl.anorak.so. It exports the HMI module-info object (568 bytes), boot_control_init, set_active_boot_slot, slot_suffix_arr, gpt_disk_alloc and gpt_utils_is_ufs_device, and has 28 defined dynamic symbols (proof in `data/subsystems/native-libraries/claims.json`, id D8.1).
- **D8.2** (verified): bootctrl.anorak.so depends on libcutils, liblog, libz, libc++, libc, libm and libdl. Its string table holds the boot-slot API names get_current_slot, get_number_slots, is_slot_bootable, is_slot_marked_successful, set_active_boot_slot and set_slot_as_unbootable (proof in `data/subsystems/native-libraries/claims.json`, id D8.2).
- **D8.3** (inferred): The boot HAL service that consumes the module is declared in init as vendor.boot-hal-1-2, running /vendor/bin/hw/android.hardware.boot@1.2-service with interfaces IBootControl 1.0, 1.1 and 1.2 in the early_hal class. That the service loads bootctrl.anorak.so is inferred from the HMI export and the shared slot API names, not from an import in the service binary (proof in `data/subsystems/native-libraries/claims.json`, id D8.3).
- **D9.1** (verified): vendor/lib64/vendor.qti.hardware.display.mapper@4.0.so is a passthrough HIDL interface library for IQtiMapper v4.0. It depends on graphics common 1.0/1.1/1.2, graphics mapper 4.0, the mapperextensions 1.0 interface and libhidlbase, and exports 101 defined symbols including IQtiMapper::getService and BnHwQtiMapper (proof in `data/subsystems/native-libraries/claims.json`, id D9.1).
- **D9.2** (verified): The vendor VINTF manifest declares vendor.qti.hardware.display.mapper 4.0 as passthrough with interface IQtiMapper, in a fragment named android.hardware.graphics.mapper-impl-qti-display.xml (proof in `data/subsystems/native-libraries/claims.json`, id D9.2).
- **D10.1** (verified): system_ext/lib64/libopenxr_loader.so is an OpenXR loader: SONAME libopenxr_loader.so, 55 defined symbols and 162 undefined, exporting xr* entry points such as xrCreateInstance, xrDestroyInstance and xrAcquireSwapchainImage. It depends on libandroid, libEGL, liblog, libnativewindow and libvulkan (proof in `data/subsystems/native-libraries/claims.json`, id D10.1).
- **D11.1** (verified): system_ext/lib64/libVkLayer_khronos_validation.so is a Vulkan layer. It is an AArch64 DYN object with no SONAME and 21482200 bytes. It exports 7 vk* loader-interface symbols, including vkNegotiateLoaderLayerInterfaceVersion, and depends on libdl, liblog, libandroid, libm and libc (proof in `data/subsystems/native-libraries/claims.json`, id D11.1).
- **D12.2** (verified): libadsprpc.so is listed in vendor/etc/public.libraries.txt and is labelled same_process_hal_file in vendor_file_contexts. Its strings include fastrpc_mmap, fastrpc_munmap and fastrpc_async_get_status (proof in `data/subsystems/native-libraries/claims.json`, id D12.2).
- **D13.1** (verified): odm/lib64/libmaintenanceboot.so is a maintenance-boot library with SONAME libmaintenanceboot.so and 122 defined symbols. It exports OVR::OS MiscWriter, ReadMiscVirtualAbMessage, update_bootloader_message_in_struct, clear_bootloader_message and android::fs_mgr fstab readers. It depends on android.hardware.health@2.1.so, libosutils, libutils, libbase, libcutils and liblog (proof in `data/subsystems/native-libraries/claims.json`, id D13.1).
- **D13.2** (verified): The maintenance-boot HAL service is declared in odm init as vendor.oculus.hardware.maintenanceboot.maintenanceboot-service, interface aidl vendor.oculus.hardware.maintenanceboot.IMaintenanceBoot/default, class hal. No init or SELinux file names libmaintenanceboot.so directly; the service binary is labelled hal_maintenanceboot_default_exec (proof in `data/subsystems/native-libraries/claims.json`, id D13.2).
- **D15.1** (verified): vendor/etc/public.libraries.txt lists 8 vendor entries: libadsprpc.so, libcdsprpc.so, libeva.so, libhexagon.so, libsdsprpc.so, libsynx.so (all six present in this group), plus libdrm.so and libOpenCL.so (present in display-and-composer and gpu subsystems) (proof in `data/subsystems/native-libraries/claims.json`, id D15.1).

## Claims with corrected wording

The claims below are restated with the wording that the data supports.

- **G5** (verified): 1300 ELF objects carry a DT_SONAME. The 51 without one are 50 .dylib.so objects and libVkLayer_khronos_validation.so. The 50 .dylib.so objects follow the Rust crate-dylib naming; 40 list libstd.dylib.so in NEEDED, and the Rust origin of the nine small crates without that entry (for example libbitflags.dylib.so) is inferred from their names (proof in `data/subsystems/native-libraries/claims.json`, id G5).
- **G9** (verified): The vendor SELinux mapping labels vendor libadsprpc.so and the vendor.qti.hardware.display.mapper@2.0, @3.0 and @4.0 libraries and the vendor.qti.hardware.display.mapperextensions@1.0, @1.1 and @1.2 libraries in this group as same_process_hal_file (vendor_file_contexts lines 161-166 and 196). vendor_file_contexts line 17 labels the boot HAL service executable as hal_bootctl_default_exec, and line 108 labels the odm maintenance-boot service executable as hal_maintenanceboot_default_exec (proof in `data/subsystems/native-libraries/claims.json`, id G9).
- **D1.1** (verified): system/lib64/libc.so is a 46-byte symbolic link to /apex/com.android.runtime/lib64/bionic/libc.so. That path is not materialized in the mount, and system_a/apex is empty. The APEX package itself is in the mount at system/apex/com.android.runtime.apex, and its apex_payload.img contains /lib64/bionic/libc.so (1322360 bytes) (proof in `data/subsystems/native-libraries/claims.json`, id D1.1).
- **D6.1** (verified): system/lib64/libclcore.bc (and the _debug, _debug_g, _g and _neon variants) are LLVM IR bitcode files that begin with the magic bytes 42 43 c0 de (verified). Sizes in lib64 are 330432, 353076, 1077160, 1124700 and 333936 bytes respectively (verified). Their purpose (libclc built-in library for OpenCL and RenderScript) is inferred from the name (inferred) (proof in `data/subsystems/native-libraries/claims.json`, id D6.1).
- **D10.2** (verified): The system OpenXR runtime manifests under system/etc/openxr/1 (active_runtime.aarch64.json, active_runtime.armv7a-vfp.json, active_runtime.x86_64.json) each name libopenxr_forwardloader.so inside VrDriver.apk; none names libopenxr_loader.so. No text file in the mount (init, SELinux, XML, JSON or other) references libopenxr_loader.so. Binary files do: system/lib64/libxrapp.so and system/lib64/libvrui_jni.so list libopenxr_loader.so in NEEDED (proof in `data/subsystems/native-libraries/claims.json`, id D10.2).
- **D11.2** (verified): No layer manifest or configuration text file in the mount names libVkLayer_khronos_validation.so. The name does appear as a string in the binaries system/lib64/libvulkan.so and system/lib/libvulkan.so, which also contain the debug.vulkan.layers and debug.vulkan.layer.<priority> strings; no text file in the mount sets those properties, so the registration path is not determined from the mount (proof in `data/subsystems/native-libraries/claims.json`, id D11.2).
- **D12.1** (verified): vendor/lib64/libadsprpc.so is a FastRPC user-space library (SONAME libadsprpc.so, 44 defined symbols, exporting remote_handle_control, remote_session_control, remote_mem_map, remote_munmap64 and rpcmem_to_fd). Its association with the audio DSP is inferred from the adsp name prefix (inferred) (proof in `data/subsystems/native-libraries/claims.json`, id D12.1).
- **D14.1** (verified): system/etc/public.libraries.txt lists 29 entries after a comment header. The entry libneuralnetworks.so carries a trailing nopreload token. Five listed names (libicu.so, libicui18n.so, libicuuc.so, libnativehelper.so and libneuralnetworks.so) have no file or classified row under those exact names and are absent from the mount. The substring libneuralnetworks matches only libneuralnetworks_packageinfo.so (native-libraries, system_a) (proof in `data/subsystems/native-libraries/claims.json`, id D14.1).

## Excluded claims

The claims below are excluded; the reason for each is given.

| ID | Claim | Reason |
|---|---|---|
| D14.2 | Of the 29 public entries, 23 are present as files or links in the group rows. Entries such as libaaudio.so, libEGL.so, libGLESv2.so and libvulkan.so are present in other subsystems (audio, gpu, camera), not in this group. Meaning of the nopreload token is inferred to mark libraries excluded from preloading. | Not reproduced. The five absent names of D14.1 leave 24 of the 29 entries, not 23; the file that accounts for the difference was not identified. |

<!-- generated: libraries start -->
## Library index (generated)

The native-libraries group has 1,361 files. Source: `data/elf/libraries.tsv` (headers and dynamic symbols). Every file's SHA-256 matches the inventory (1,361 of 1,361).

- 1,351 files are ELF shared objects (ET_DYN): 854 ELFCLASS64 with machine EM_AARCH64, 497 ELFCLASS32 with machine EM_ARM.
- 10 files are LLVM bitcode, not ELF: `system_a/system/lib/libclcore.bc`, `system_a/system/lib/libclcore_debug.bc`, `system_a/system/lib/libclcore_debug_g.bc`, `system_a/system/lib/libclcore_g.bc`, `system_a/system/lib/libclcore_neon.bc`, `system_a/system/lib64/libclcore.bc`, `system_a/system/lib64/libclcore_debug.bc`, `system_a/system/lib64/libclcore_debug_g.bc`, `system_a/system/lib64/libclcore_g.bc`, `system_a/system/lib64/libclcore_neon.bc`.
- 12 files keep a .symtab section; the other 1,339 are stripped to the dynamic symbol table.
- 1,343 of the 1,351 ELF files carry a GNU build-id note; 8 do not. 51 files have no SONAME.
- The first .comment string of 12 files is `Android (7485623, based on r416183b1) clang version 12.0.7 (https://android.googlesource.com/toolchain/llvm-project c935d99d7cf2016289302412d708641d52d2f7ee)`.

### By partition

| Partition | Files | ELF files |
|---|---|---|
| system_a | 779 | 769 |
| vendor_a | 367 | 367 |
| system_ext_a | 169 | 169 |
| odm_a | 26 | 26 |
| product_a | 20 | 20 |

### NEEDED graph

The 1,351 ELF files list 11,630 NEEDED edges naming 613 distinct libraries. 10,845 edges name a library that is the SONAME of a file in this group (453 distinct names). The other 785 edges name 160 distinct libraries that are not in this group. The most common NEEDED entries:

| NEEDED | Libraries |
|---|---|
| `libc.so` | 1,343 |
| `libdl.so` | 1,284 |
| `libm.so` | 1,267 |
| `libc++.so` | 1,224 |
| `liblog.so` | 921 |
| `libutils.so` | 677 |
| `libcutils.so` | 574 |
| `libhidlbase.so` | 348 |
| `libbase.so` | 288 |
| `libbinder.so` | 246 |
| `libbinder_ndk.so` | 123 |
| `libosutils.so` | 115 |
| `libui.so` | 62 |
| `libz.so` | 58 |
| `libjsoncpp.so` | 52 |

NEEDED names with no SONAME in this group (the most common):

| NEEDED | Edges | File of this name elsewhere in the inventory |
|---|---|---|
| `libcrypto.so` | 45 | security-keystore-crypto |
| `libstd.dylib.so` | 40 | native-libraries |
| `libEGL.so` | 31 | gpu |
| `libGLESv2.so` | 25 | gpu |
| `android.hardware.graphics.mapper@4.0.so` | 25 | display-and-composer |
| `libdisplaydebug.so` | 23 | display-and-composer |
| `libaudioclient.so` | 21 | audio |
| `libnativehelper.so` | 18 | none |
| `libstagefright_foundation.so` | 16 | media-codecs |
| `libaudioutils.so` | 15 | audio |
| `libvrsensors-hidlwrapper.so` | 15 | sensors-mcu-meta-hal |
| `libstatssocket.so` | 14 | none |

Of the 160 names with no SONAME match in this group, 152 have a file of the same name somewhere in the inventory. The match is by file name only, so it does not confirm a SONAME. A subsystem of `native-libraries` in the column means the file is in this group but carries a different SONAME. The other names have no file of that name in the image.

### Dynamic symbols

Across the ELF files, the dynamic symbol tables hold 572,803 defined and 137,597 undefined symbols. The counts exclude the null entry at index 0. The libraries with the most undefined symbols:

| Library | Machine | Undefined | Defined |
|---|---|---|---|
| `system_a/system/lib/libandroid_runtime.so` | EM_ARM | 2400 | 1878 |
| `system_a/system/lib64/libandroid_runtime.so` | EM_AARCH64 | 2398 | 1856 |
| `system_a/system/lib/libandroid_servers.so` | EM_ARM | 815 | 908 |
| `system_a/system/lib64/libandroid_servers.so` | EM_AARCH64 | 815 | 908 |
| `system_a/system/lib/libhwui.so` | EM_ARM | 804 | 121 |
| `system_a/system/lib64/libhwui.so` | EM_AARCH64 | 794 | 121 |
| `system_a/system/lib/libmediaplayerservice.so` | EM_ARM | 784 | 1638 |
| `system_a/system/lib/libmedia_jni.so` | EM_ARM | 679 | 542 |
| `system_a/system/lib64/libmedia_jni.so` | EM_AARCH64 | 677 | 542 |
| `vendor_a/lib/libfastcvopt.so` | EM_ARM | 584 | 5950 |
| `vendor_a/lib64/libfastcvopt.so` | EM_AARCH64 | 576 | 6000 |
| `vendor_a/lib64/libwvhidl.so` | EM_AARCH64 | 535 | 3421 |

Bitcode files and the stripped-library limits are listed in the table; symbol-level matching between libraries is not computed here.

<!-- generated: libraries end -->

## Gaps

- The bionic targets under /apex/com.android.runtime (libc, libdl, libm, libdl_android) are not materialized in the mount: system_a/apex is empty, so the 8 bionic symlinks could not be followed to a file. The APEX package system/apex/com.android.runtime.apex is in the mount, and its payload contains these libraries (claim D1.1).
- libicu.so, libicui18n.so, libicuuc.so, libnativehelper.so and libneuralnetworks.so are named in system/etc/public.libraries.txt but do not appear in any subsystem inventory; whether APEX modules supply them was not determined.
- The roles for 857 rows are inferred from file names and the 453 verified roles from exported symbols or NEEDED entries; no per-file behaviour was checked. The 61 unverified rows are name-only guesses (for example com.qti.stats.common.so, libgipp.so, libkll.so, libfruit.so) and are marked as such in roles.tsv.
- The NEEDED edges of all 1,351 ELF objects are in the generated section above. Edges are matched to SONAMEs in this group. A NEEDED name outside the group was matched to files by file name only, and the SONAME of such a file was not read; a name with no file in the image may come from an APEX module that is not materialized in the mount.
- The linker configuration (system/linkerconfig, system/etc/linker.config.pb) is binary and was not decoded. A literal grep for libadsprpc.so, libopenxr_loader.so and libmaintenanceboot.so found no matches in linker.config.pb, but the search rules and namespace search paths were not determined.
- The HMI module id string for bootctrl.anorak.so was not extracted, and no import linkage between android.hardware.boot@1.0-impl-1.2-qti.so and bootctrl.anorak.so was found. The consumer relationship is inferred only.
- The Vulkan layer manifest for libVkLayer_khronos_validation.so was not found in the mount, so its registration and enablement path is unknown.
- The OpenXR loader's consumer apps were not identified. The runtime manifest points at a different library inside VrDriver.apk.
- The 50 Rust .dylib.so crate objects and the 10 bitcode files were classified from names and format only, apart from the Rust NEEDED check and the bitcode magic bytes.
- The 15 deep-dive items are a selection of the most important files by name and role. The remaining binaries were not deep-dived beyond the role table.
- Undefined-symbol counts in the claims include the null symbol-table entry, so they are one higher than counts that exclude it. The claims cite the higher counts.
