# 21 - Native shared libraries

[中文版](21-native-libraries.zh-CN.md)

## Summary

The native-libraries group is 1371 rows across five partitions: system_a 787, vendor_a 368, system_ext_a 169, odm_a 27 and product_a 20. It contains 1361 regular files and 10 symlinks. The regular files are 1351 ELF shared objects (854 ELF64 AArch64, 497 ELF32 ARM, all ET_DYN) and 10 LLVM bitcode files (libclc built-in libraries under system/lib and system/lib64). The symlinks are 8 bionic links (libc, libdl, libdl_android, libm in system/lib and system/lib64) that point into /apex/com.android.runtime, which is not in the mount, plus odm_a and vendor_a lib/modules links to the dlkm partitions. system_a holds the AOSP platform and NDK-public libraries plus HIDL/AIDL interface libraries and Rust dylibs. vendor_a holds Qualcomm HIDL/AIDL clients, HAL modules (lib/hw, lib64/hw), FastRPC, audio (AGM/PAL) and camera/display libraries. system_ext_a, product_a and odm_a hold device-specific libraries (Oculus/Meta telemetry, tracking, XR loader, maintenance boot, Vulkan layers). The configuration that governs the group is system/etc/public.libraries.txt (29 entries) and vendor/etc/public.libraries.txt (8 entries), plus SELinux labels in plat_file_contexts line 121 and vendor_file_contexts lines 142-153, 196. The role table is at roles.tsv with 1371 rows: 453 verified, 857 inferred, 61 unverified. In the evidence, <image>/mnt means <image>/mnt.

The role table has one row for each of the 1371 files in this group (453 verified, 0 observed, 857 inferred, 61 unverified). It is at `data/subsystems/native-libraries/roles.tsv`.

## Verified claims

- **G1** (verified): The group has 1371 rows: 1361 regular files and 10 symlinks. Partition counts are system_a 787, vendor_a 368, system_ext_a 169, odm_a 27, product_a 20.
- **G2** (verified): All 1361 regular files match the size and sha256 recorded in the inventory; the mounted copies are byte-identical to the classified rows.
- **G3** (verified): Format split: 1351 ELF ET_DYN shared objects (854 ELF64 EM_AARCH64, 497 ELF32 EM_ARM) and 10 LLVM bitcode files, with no other file formats.
- **G4** (verified): The 10 symlinks are 8 bionic links (libc.so, libdl.so, libdl_android.so, libm.so in system/lib and system/lib64) into /apex/com.android.runtime, and 2 kernel-module directory links (odm_a and vendor_a lib/modules).
- **G6** (verified): The role table covers each input row exactly once: 1371 rows, 453 verified, 857 inferred, 61 unverified. Roles marked unverified are name-only guesses.
- **G7** (verified): There are 810 unique basenames across the 1371 rows; many names recur in lib and lib64 and in two or more partitions.
- **G8** (verified): The system SELinux mapping labels every system/lib and system/lib64 path as system_lib_file, with bootstrap copies labelled system_bootstrap_lib_file.
- **D2.1** (verified): system/lib64/libbinder.so is an ELF64 AArch64 DYN object with SONAME libbinder.so, a build-id note and 28 sections, stripped. It has 1392 defined dynamic symbols and 245 undefined entries, including the null entry.
- **D2.2** (verified): libbinder.so depends on liblog, libcutils, libutils, libc++, libc, libm and libdl. Its exports include the C++ Binder API, for example _ZN7android12ProcessState15startThreadPoolEv (ProcessState::startThreadPool). It is not listed in public.libraries.txt.
- **D3.1** (verified): system/lib64/libutils.so is an AArch64 DYN object with SONAME libutils.so and 389 defined dynamic symbols. Its exports include Looper and RefBase, and it depends on libcutils, liblog, libprocessgroup, libvndksupport, libc++, libc, libm and libdl.
- **D4.1** (verified): system/lib64/liblog.so is the Android logging library. It exports __android_log_print, __android_log_write and android_log_* symbols with version tags such as @@LIBLOG, and depends only on libc++, libc, libm and libdl. It is listed in the NDK public-library file.
- **D5.1** (verified): system/lib64/libcutils.so is the Android C utilities library. It exports property_get, ashmem_create_region, android_get_control_socket and socket_local_client, has 120 defined symbols, and depends on liblog, libbase, libc++, libc, libm and libdl.
- **D6.2** (verified): The bitcode is consumed by libRSCpuRef.so, which contains the literal paths /system/lib64/libclcore.bc, libclcore_debug.bc and libclcore_debug_g.bc.
- **D7.1** (verified): system/lib64/hw/android.hidl.memory@1.0-impl.so is a passthrough HIDL implementation with SONAME android.hidl.memory@1.0-impl.so. It exports the AshmemMapper and AshmemMemory classes in android::hidl::memory::V1_0::implementation, and depends on android.hidl.memory@1.0.so and libhidlbase.so.
- **D7.2** (verified): The system VINTF manifest declares android.hidl.memory 1.0 as a passthrough HAL on 32+64-bit architectures, so the implementation is loaded into the client process.
- **D8.1** (verified): vendor/lib64/hw/bootctrl.anorak.so is an AArch64 HAL module with SONAME bootctrl.anorak.so. It exports the HMI module-info object (568 bytes), boot_control_init, set_active_boot_slot, slot_suffix_arr, gpt_disk_alloc and gpt_utils_is_ufs_device, and has 28 defined dynamic symbols.
- **D8.2** (verified): bootctrl.anorak.so depends on libcutils, liblog, libz, libc++, libc, libm and libdl. Its string table holds the boot-slot API names get_current_slot, get_number_slots, is_slot_bootable, is_slot_marked_successful, set_active_boot_slot and set_slot_as_unbootable.
- **D8.3** (inferred): The boot HAL service that consumes the module is declared in init as vendor.boot-hal-1-2, running /vendor/bin/hw/android.hardware.boot@1.2-service with interfaces IBootControl 1.0, 1.1 and 1.2 in the early_hal class. That the service loads bootctrl.anorak.so is inferred from the HMI export and the shared slot API names, not from an import in the service binary.
- **D9.1** (verified): vendor/lib64/vendor.qti.hardware.display.mapper@4.0.so is a passthrough HIDL interface library for IQtiMapper v4.0. It depends on graphics common 1.0/1.1/1.2, graphics mapper 4.0, the mapperextensions 1.0 interface and libhidlbase, and exports 101 defined symbols including IQtiMapper::getService and BnHwQtiMapper.
- **D9.2** (verified): The vendor VINTF manifest declares vendor.qti.hardware.display.mapper 4.0 as passthrough with interface IQtiMapper, in a fragment named android.hardware.graphics.mapper-impl-qti-display.xml.
- **D10.1** (verified): system_ext/lib64/libopenxr_loader.so is an OpenXR loader: SONAME libopenxr_loader.so, 55 defined symbols and 162 undefined, exporting xr* entry points such as xrCreateInstance, xrDestroyInstance and xrAcquireSwapchainImage. It depends on libandroid, libEGL, liblog, libnativewindow and libvulkan.
- **D11.1** (verified): system_ext/lib64/libVkLayer_khronos_validation.so is a Vulkan layer. It is an AArch64 DYN object with no SONAME and 21482200 bytes. It exports 7 vk* loader-interface symbols, including vkNegotiateLoaderLayerInterfaceVersion, and depends on libdl, liblog, libandroid, libm and libc.
- **D12.2** (verified): libadsprpc.so is listed in vendor/etc/public.libraries.txt and is labelled same_process_hal_file in vendor_file_contexts. Its strings include fastrpc_mmap, fastrpc_munmap and fastrpc_async_get_status.
- **D13.1** (verified): odm/lib64/libmaintenanceboot.so is a maintenance-boot library with SONAME libmaintenanceboot.so and 122 defined symbols. It exports OVR::OS MiscWriter, ReadMiscVirtualAbMessage, update_bootloader_message_in_struct, clear_bootloader_message and android::fs_mgr fstab readers. It depends on android.hardware.health@2.1.so, libosutils, libutils, libbase, libcutils and liblog.
- **D13.2** (verified): The maintenance-boot HAL service is declared in odm init as vendor.oculus.hardware.maintenanceboot.maintenanceboot-service, interface aidl vendor.oculus.hardware.maintenanceboot.IMaintenanceBoot/default, class hal. No init or SELinux file names libmaintenanceboot.so directly; the service binary is labelled hal_maintenanceboot_default_exec.
- **D15.1** (verified): vendor/etc/public.libraries.txt lists 8 vendor entries: libadsprpc.so, libcdsprpc.so, libeva.so, libhexagon.so, libsdsprpc.so, libsynx.so (all six present in this group), plus libdrm.so and libOpenCL.so (present in display-and-composer and gpu subsystems).

## Claims with corrected wording

These claims held in substance. The wording below replaces the original.

- **G5** (verified): 1300 ELF objects carry a DT_SONAME. The 51 without one are 50 .dylib.so objects and libVkLayer_khronos_validation.so. The 50 .dylib.so objects follow the Rust crate-dylib naming; 40 list libstd.dylib.so in NEEDED, and the Rust origin of the nine small crates without that entry (for example libbitflags.dylib.so) is inferred from their names.
- **G9** (verified): The vendor SELinux mapping labels vendor libadsprpc.so and the vendor.qti.hardware.display.mapper@2.0, @3.0 and @4.0 libraries and the vendor.qti.hardware.display.mapperextensions@1.0, @1.1 and @1.2 libraries in this group as same_process_hal_file (vendor_file_contexts lines 161-166 and 196). vendor_file_contexts line 17 labels the boot HAL service executable as hal_bootctl_default_exec, and line 108 labels the odm maintenance-boot service executable as hal_maintenanceboot_default_exec.
- **D1.1** (verified): system/lib64/libc.so is a 46-byte symbolic link to /apex/com.android.runtime/lib64/bionic/libc.so. That path is not materialized in the mount, and system_a/apex is empty. The APEX package itself is in the mount at system/apex/com.android.runtime.apex, and its apex_payload.img contains /lib64/bionic/libc.so (1322360 bytes).
- **D6.1** (verified): system/lib64/libclcore.bc (and the _debug, _debug_g, _g and _neon variants) are LLVM IR bitcode files that begin with the magic bytes 42 43 c0 de (verified). Sizes in lib64 are 330432, 353076, 1077160, 1124700 and 333936 bytes respectively (verified). Their purpose (libclc built-in library for OpenCL and RenderScript) is inferred from the name (inferred).
- **D10.2** (verified): The system OpenXR runtime manifests under system/etc/openxr/1 (active_runtime.aarch64.json, active_runtime.armv7a-vfp.json, active_runtime.x86_64.json) each name libopenxr_forwardloader.so inside VrDriver.apk; none names libopenxr_loader.so. No text file in the mount (init, SELinux, XML, JSON or other) references libopenxr_loader.so. Binary files do: system/lib64/libxrapp.so and system/lib64/libvrui_jni.so list libopenxr_loader.so in NEEDED.
- **D11.2** (verified): No layer manifest or configuration text file in the mount names libVkLayer_khronos_validation.so. The name does appear as a string in the binaries system/lib64/libvulkan.so and system/lib/libvulkan.so, which also contain the debug.vulkan.layers and debug.vulkan.layer.<priority> strings; no text file in the mount sets those properties, so the registration path is not determined from the mount.
- **D12.1** (verified): vendor/lib64/libadsprpc.so is a FastRPC user-space library (SONAME libadsprpc.so, 44 defined symbols, exporting remote_handle_control, remote_session_control, remote_mem_map, remote_munmap64 and rpcmem_to_fd). Its association with the audio DSP is inferred from the adsp name prefix (inferred).
- **D14.1** (verified): system/etc/public.libraries.txt lists 29 entries after a comment header. The entry libneuralnetworks.so carries a trailing nopreload token. Five listed names (libicu.so, libicui18n.so, libicuuc.so, libnativehelper.so and libneuralnetworks.so) have no file or classified row under those exact names and are absent from the mount. The substring libneuralnetworks matches only libneuralnetworks_packageinfo.so (native-libraries, system_a).

## Excluded claims

These claims did not hold on reproduction.

| ID | Claim | Reason |
|---|---|---|
| D14.2 | Of the 29 public entries, 23 are present as files or links in the group rows. Entries such as libaaudio.so, libEGL.so, libGLESv2.so and libvulkan.so are present |  |

## Gaps

- The bionic targets under /apex/com.android.runtime (libc, libdl, libm, libdl_android) are not in the mount; system_a/apex is empty, so the 8 bionic symlinks could not be followed to content.
- libicu.so, libicui18n.so, libicuuc.so, libnativehelper.so and libneuralnetworks.so are named in system/etc/public.libraries.txt but do not appear in any subsystem inventory; whether APEX modules supply them was not determined.
- The roles for 857 rows are inferred from file names and the 453 verified roles from exported symbols or NEEDED entries; no per-file behaviour was checked. The 61 unverified rows are name-only guesses (for example com.qti.stats.common.so, libgipp.so, libkll.so, libfruit.so) and are marked as such in roles.tsv.
- Reverse dependencies were computed only for the 15 deep-dive items. A full NEEDED graph across the 1351 ELF objects was not built.
- The linker configuration (system/linkerconfig, system/etc/linker.config.pb) is binary and was not decoded. A literal grep for libadsprpc.so, libopenxr_loader.so and libmaintenanceboot.so found no matches in linker.config.pb, but the search rules and namespace search paths were not determined.
- The HMI module id string for bootctrl.anorak.so was not extracted, and no import linkage between android.hardware.boot@1.0-impl-1.2-qti.so and bootctrl.anorak.so was found. The consumer relationship is inferred only.
- The Vulkan layer manifest for libVkLayer_khronos_validation.so was not found in the mount, so its registration and enablement path is unknown.
- The OpenXR loader's consumer apps were not identified. The runtime manifest points at a different library inside VrDriver.apk.
- The 50 Rust .dylib.so crate objects and the 10 bitcode files were classified from names and format only, apart from the Rust NEEDED check and the bitcode magic bytes.
- The 15 deep-dive items are a selection of the most important files by name and role. The remaining binaries were not deep-dived beyond the role table.
- Symbol counts from pyelftools (in facts.jsonl) differ from readelf by one for undefined symbols because readelf counts the null entry. Claims cite readelf counts.
