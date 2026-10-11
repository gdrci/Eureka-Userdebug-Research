# 12 - DSP and remote processors

[中文版](12-dsp-and-remote-processors.zh-CN.md)

## Summary

The group has 243 rows (237 regular files and 6 symlinks): 20 in odm_a and 223 in vendor_a. It holds the remote-processor firmware and runtime set: Hexagon DSP6 root images for ADSP and CDSP (split .mdt plus .bNN segments), the FastRPC user daemons (adsprpcd, cdsprpcd) and DSP-side shell and skel libraries, the XRA audio-pipeline CAPI modules with JSON configs and tuning blobs, an EVA Xtensa image, an SPSS ARM image, the QCA6490 WLAN firmware with board-data ELFs, message catalogs and an ini config, AArch64 secure-app images with servreg JSON configs, ODM microcontroller images (SyncBoss, SantanaManager, MCU tar bundles with unwind ELFs), PDFU update packages and an EFM8 image. The mounted ueventd.rc, vendor_file_contexts, vendor_sepolicy.cil and init .rc files confirm the firmware search paths, the labels for /vendor/bin/adsprpcd and /vendor/bin/cdsprpcd, the three remoteproc instances (spss, adsp, cdsp) and the adsprpcd and cdsprpcd init services. Most firmware image names are not referenced by name in any mounted init or SELinux text file.

Role table: `data/subsystems/dsp-and-remote-processors/roles.tsv` (243 files: 153 verified, 79 observed, 11 inferred, 0 unverified).

## Verified claims

- **G1** (verified): The group has 243 input rows: 237 regular files and 6 symlinks, with 20 rows in odm_a and 223 in vendor_a.
- **G2** (verified): All 237 regular input files exist under the mount root and match their recorded sha256 values.
- **G3** (verified): The six symlinks point outside the mounted set (four to /persist calibration paths from odm_a, two WLAN links from vendor_a). The WLAN config target exists inside vendor_a/etc/wifi; the /persist targets are not in the mount root.
- **G4** (verified): Nine firmware stems are split images: an ELF program-header file (.mdt) plus numbered segment files (.bNN). For every stem, each present segment file size equals its program-header p_filesz, except adsp.b31 (see D3).
- **G5** (observed): Image identity strings present in the group: DSP root images, EVA, video firmware and QCA6490 WLAN each carry a QC_IMAGE_VERSION_STRING and IMAGE_VARIANT_STRING.
- **G6** (observed): The mounted configuration names the firmware search paths and three remoteproc instances used by the images in this group.
- **G7** (verified): The roles table covers all 243 rows once each, with confidence counts of 153 verified, 79 observed and 11 inferred.
- **D1** (verified): bin/adsprpcd is an AArch64 PIE FastRPC user-side daemon for the ADSP that dynamically links liblog, libc++, libc, libm and libdl.
- **D1b** (observed): adsprpcd log strings describe its loop: it starts the 'adsp daemon', loads libadsp_default_listener.so through a listener start function, and restarts after 25 ms when an error occurs.
- **D1c** (observed): adsprpcd is started by a main-class init service running as user system in group media, and its executable is labeled vendor_adsprpcd_exec with init transition and device rules defined.
- **D2** (verified): bin/cdsprpcd has the same ELF type and NEEDED list as adsprpcd and differs from it in 1226 byte positions; its log strings describe a CDSP listener loop with a 100 ms restart delay.
- **D2b** (observed): cdsprpcd init service is disabled by default and started on boot for listed SKUs and devices, and stopped by the property vendor.fastrpc.disable.cdsprpcd.daemon=1. Its executable is labeled vendor_cdsprpcd_exec.
- **D3** (verified): adsp.mdt is an ELF32 Hexagon DSP6 EXEC with entry 0x87a00000 and 47 program headers; 46 segment files exist. Segment 31 has p_filesz 0 but a 7528-byte file, and segment 45 has p_filesz 0 with no file. The segment sizes total 13,393,180 bytes.
- **D3b** (observed): ADSP image labels in configuration: /vendor/firmware is labeled firmware_file; the ADSP remoteproc1 SSR sysfs nodes are labeled vendor_sysfs_ssr; the adsp image carries DSP.XT.1.1-00646-HALLIDAY-1 with variant halliday.adsp.prodQ.
- **D4** (verified): cdsp.mdt is an ELF32 Hexagon DSP6 EXEC with entry 0x8b500000 and 16 program headers; 15 segment files exist (none for index 14, whose p_filesz is 0) and all present sizes match.
- **D4b** (observed): The CDSP image is identified as DSP.XT.1.1-00646-HALLIDAY-1 (variant halliday.cdsp.prodQ) and is paired with servreg domain cdsp/root_pd (QMI instance 76).
- **D5** (verified): evass.mdt is an ELF32 Tensilica Xtensa image with entry 0xf500000 and 20 program headers; 20 segment files exist, eight of them zero-length (b10 to b17). evass.mbn is a separate single-file Xtensa ELF of 2,333,480 bytes that does not begin with the .mdt bytes.
- **D5b** (observed): The EVA image is identified as EVA.FIRMWARE.3.6-11 (variant PROD). A vendor property prefix vendor.runtime.eva. is labeled vendor_adsprpc_prop, tying EVA runtime properties to the FastRPC property label.
- **D6** (verified): spss1p and spss1t are ARM EABI5 split images (machine 0x28, entry 0x1000, three program headers each), each with three matching segment files. Their single-file .mbn forms are also ARM ELF images.
- **D7** (verified): hdcp1.mdt, hdcp2p2.mdt, hdcpsrm.mdt and loadalgota64.mdt are AArch64 ELF64 split images with nine program headers each and matching segment files. Their strings name a root certificate 'Eureka_FW_Root1' and the subject 'Meta Platforms, Inc.'. widevine.mbn links libcmnlib.so.
- **D7b** (observed): Other trusted-application images in this group carry identifying strings: eyetz64.mbn has tz_app_init and 'calling command test', and sp_license.mbn prints 'SP license app Init'. No mounted init or SELinux text file names these .mbn or .mdt images (grep count 0), apart from a Widevine HAL service entry that starts a separate HAL binary.
- **D8** (verified): amss.bin is the QCA6490 WLAN firmware image: ARM ELF32 EXEC with entry 0x143a800 and size 5,523,568 bytes. amss20.bin is a second build of the same WLAN.HSP.2.0 build (eval_v2) at 5,603,328 bytes. m3.bin is a WLAN ucode image with an ELF32 header whose e_machine field is 3 (EM_386).
- **D10** (verified): Data.msc and Data20.msc are text WLAN firmware message catalogs. Each has a VERSION:536936572 header and id-coded records of the form id,args,[ file : line ] text: 10,298 records in Data.msc and 10,579 in Data20.msc.
- **D10b** (inferred): regdb.bin is a WLAN regulatory database in binary form; its record layout was not decoded.
- **D11** (verified): xra_config_render and xra_config_capture are JSON configs for the XRA audio pipeline. They name resource files under /vendor/lib/rfsa/adsp/xra_resources, pin mappings and sampling (48 kHz, sample types), and debug output under /data/data_collection.
- **D12** (observed): xra_render_capi.so and xra_capture_capi.so are DSP6 CAPI modules for XRA. Their strings include xra_capi_get_static_properties, 13 capi_utils_props_* helper names and C++ symbols in the facebook2xr5audio namespace. The dynamic symbol table does not list xra_capi_get_static_properties.
- **D13** (observed): The DSP-side FastRPC shell libraries (fastrpc_shell_0, fastrpc_shell_3, fastrpc_shell_unsigned_3) are DSP6 DYN objects with no NEEDED entries and about 1,550 to 1,600 defined global functions. Their exports include the HAP and qurt APIs. The synx library exports the synx_* sync-object API.
- **D13b** (observed): Several DSP skel libraries carry QAIC-generated invoke entry points (for example loadalgo_skel_handle_invoke_qaic_version, sysmonquery_skel_handle_invoke_qaic_version), indicating the FastRPC IDL skel pattern.
- **D14** (verified): The five servreg JSON files (adspr, cdspr, adsps, adspua, battmgr) declare an sr_domain and one or two sr_service entries with QMI instance IDs 74 (adsp) and 76 (cdsp). adspua.jsn adds an avs audio service entry.
- **D15b** (observed): The syncboss-app.bin image has a dAeH-prefixed header with a 12-hex-digit build identifier, and the MCU tar bundles hold per-device app and SPL-updater images with dated sets (2020 to 2024).
- **D15c** (observed): The unwind ELF images (unwind-elk-app.bin, unwind-coulee-app.bin, syncboss-unwind.bin) hold only .debug_frame and symbol sections and no code section.
- **D15d** (observed): The ODM odm_dlkm modules list includes syncboss kernel modules (syncboss_spi.ko, syncboss_miscfifo.ko, syncboss_powerstate.ko, syncboss_direct_channel.ko, syncboss_nsync.ko), and vendor init sets syncboss sysfs permissions and waits for the firmware update node.
- **D16** (verified): The PDFU update packages begin with a 46-byte ASCII hex header terminated by CRLF at offset 0x2e. The payload begins with words 0x20003000 and a reset vector, the layout of a Cortex-M vector table.
- **D16b** (observed): The PDFU packages are dispatched through the Meta PDFU path: the ueventd external handler matches 2833-*.pdfu (Meta VID 0x2833), the PDFU HAL is labeled hal_pdfu_default_exec, and /odm/bin/pdfu_depot is labeled pdfu_depot_exec. The stored file names begin with product names (Lehua-, Moku-), not 2833-.
- **D17** (observed): The EFM8 image Eureka_Dev1.efm8 is a raw binary (not ELF) with a device tag and LED label strings. No mounted init or SELinux text names it.
- **D19** (observed): The ODM fw_init.cfg [blu] section names /vendor/firmware/blu.bin and blu.toml, but no such file is in vendor_a/firmware in this group.

## Claims with corrected wording

- **D6b** (observed): The variant suffix p or t also appears on the signature containers crypt1p.sig and crypt1t.sig. Both carry the organizationalUnit value 'Qualcomm Cryptographic Operations'. Their commonName values differ: 'SPU MBNv7 Image Signing Root CA 6 SubCA 1' (crypt1p.sig) and 'SPU MBNv7 Image Signing Test Root CA 6 SubCA 1' (crypt1t.sig). The digit strings 1200 and 1705 that appear after the organizational unit are DER header bytes, not part of a subject string.
- **D8b** (observed): The QCA6490 WLAN config WCNSS_qcom_cfg.ini has 126 lines: 99 key=value lines, 14 comment lines, 12 blank lines and a final END line. amss.bin embeds the names of its sub-images, and WLAN config and label references appear in the mounted SELinux file contexts.
- **D9** (verified): The fourteen qca6490 eureka_*.elf and bdwlan*.elf files are board-data ELF wrappers of 60,028 bytes each, in four content groups. Each group embeds its own 58,180-byte .data payload, so the four groups carry four distinct payloads.
- **D9b** (observed): The board-data ELFs embed their payload through linker symbols named _binary_tempDir_d1Xday120bdwlan_bin_start, _end and _size (bdwlan group) or _binary_tempDir_d1Xday120bdwlang_bin_start, _end and _size (bdwlang group), where X depends on the group: d10 for bdwlan.elf and bdwlang.elf, d11 for eureka_evt2.1_bdwlan.elf and eureka_pvt1.1_bdwlang.elf.
- **D11b** (observed): The XRA tuning resource blobs carry text profile headers naming Stinson_EVT2v1.4 (resources_render.dat), stinsonWola (resources_capture.dat) and stinsonExperimentalWola (resources_capture_ex.dat).
- **D15** (verified): The ODM syncboss.bin and santana_manager.bin images are ARM Cortex-M firmware with vector tables (SyncBoss SP 0x2001e000, reset 0x000006e8; SantanaManager SP 0x20008000, reset 0x080005e0). fw_init.cfg names their paths, and catty.cfg names their serial devices (/dev/ttyHS14 and /dev/ttyHS12). The fw_init binary references /odm/etc/fw_init.cfg; the component that reads catty.cfg was not identified, so the launch relationship is inferred.
- **D18** (observed): The PROX_PS calibration link targets under /persist/calibration/prox/min_ipd/ are covered only by the generic /persist/calibration label (vendor_file_contexts line 252, persist_cal_file). The /persist/PROX[A-Z_]+ label (line 224) does not match these targets. No mounted init or ODM config names PROX_PS or min_ipd (grep count 0).

## Gaps

- Several DSP libraries have no defined symbols in their export tables: libdspCV_skel.so, libhexagon_skel.so, libdfs_skel.so, libboltEva_skel.so, libQnnBoltnnOpPackageV69.so, the rfsa copy of libevadsp_3_0.so, and the XRA CAPI modules. Their roles rest on names, sonames and strings.
- Hexagon, Xtensa, ARM and AArch64 image bodies were not disassembled. The functions of the EVA, SPSS, VPU, attestation, sensor-lock, OVR and trusted-application images are inferred from names and strings only.
- regdb.bin layout was not decoded. The Eureka_Dev1.efm8 format was not decoded.
- The bodies of the adsp.b31 segment (p_filesz 0 in the program header, 7528-byte file) and of the zero-length segments were not interpreted.
- cdsp.b09 is reported by file(1) as a Matlab v4 mat-file. It is treated as a segment because its size matches program header 9, but its contents were not decoded.
- The evass.mbn layout relative to the .mdt and .bNN files was not established beyond showing that the .mbn does not start with the .mdt bytes.
- The PROX_PS symlink targets under /persist and the WLAN MAC file /persist/wlan_mac.bin are outside the mounted set, so their contents and sizes are unknown.
- The ODM firmware labels (odm_a/etc/selinux holds only precompiled_sepolicy) could not be checked against text file_contexts. The /odm/firmware and /odm/etc/firmware file labels are unverified.
- No mounted init or SELinux text names the ADSP, CDSP, SPSS, EVA, VPU or QCA6490 image files. Their load path is inferred from ueventd firmware_directories, the /vendor/firmware file label and the remoteproc sysfs labels. The loading code is in binaries outside this group.
- The ueventd glob 2833-*.pdfu does not match the stored PDFU file names, which start with Lehua- or Moku-. The staging step that would produce matching names is not in the mounted partitions.
- The odm_dlkm and vendor lib64 host-side libraries that consume these firmware and DSP files (for example libadsprpc.so, libadsp_default_listener.so, libcdsp_default_listener.so) are outside this group's rows and were not analysed.
- The MCU tar bundle images (elk-app.bin, deerfly-app.bin, jedi.bin, coulee-app.bin, jade-app.bin and the SPL updater images) are identified only by file(1) output; their internal formats and version fields were not decoded.
- The PDFU payload beyond the first 8 bytes and the dAeH header fields beyond magic, length and build identifier were not decoded.
- The OEM_IMAGE_VERSION_STRING values found in several images include a build-host identifier; they were not reproduced and their meaning was not analysed.
