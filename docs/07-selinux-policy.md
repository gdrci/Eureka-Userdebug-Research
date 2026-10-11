# 07 - SELinux policy and file contexts

[中文版](07-selinux-policy.zh-CN.md)

## Summary

The selinux-policy group is the Android SELinux policy set carried in this image: 51 files across system_a (22), system_ext_a (15), vendor_a (11) and odm_a (3). It holds the platform policy source in CIL (plat_sepolicy.cil, 28,911 lines, mls true, 27 sids, 4 policycaps), vendor and system_ext policy sources (vendor_sepolicy.cil, system_ext_sepolicy.cil), a versioned platform public policy (plat_pub_versioned.cil, 2,077 names suffixed _32_0) that vendor policy builds on, one precompiled binary policy on odm (header reports SE Linux policy v30), per-version mapping files (26.0 to 32.0) with compat stubs, label and lookup tables (file_contexts, seapp_contexts, property_contexts, service_contexts, hwservice_contexts, keystore2_key_contexts), signature-to-seinfo XML tables, a denial-metadata table, and a platform version file reading 32.0. Two checksum records per partition bind the precompiled policy to its inputs; the platform and system_ext records were recomputed and match the concatenation of policy source plus mapping/32.0.cil. Init and libselinux reference these paths by name.
The role table has one row for each of the 51 files in this group (46 verified, 2 observed, 3 inferred, 0 unverified). It is at `data/subsystems/selinux-policy/roles.tsv`.

## Verified claims

- **C01** (verified): The selinux-policy group has 51 rows: system_a 22, system_ext_a 15, vendor_a 11, odm_a 3. All 51 are kind file: 41 text/plain, 6 inode/x-empty, 3 text/xml, 1 application/octet-stream.
- **C02** (verified): All 51 mapped files exist under the mount, and the recomputed SHA-256 and byte size of each match columns 7 and 5 of the inventory row.
- **C03** (verified): The platform checksum record equals SHA-256 of plat_sepolicy.cil followed by mapping/32.0.cil. Plat CIL alone does not match.
- **C04** (verified): The system_ext checksum record equals SHA-256 of system_ext_sepolicy.cil followed by the system_ext copy of mapping/32.0.cil. The system_a copy of mapping/32.0.cil gives a different digest, and the two 32.0.cil files differ.
- **C05** (verified): The two odm checksum files named precompiled_sepolicy.<name> are byte-identical to the platform and system_ext records respectively.
- **C06** (verified): odm precompiled_sepolicy is a binary SELinux policy of 1,006,142 bytes. Its first 64 bytes give magic 0xf97cff8c, a length-prefixed string 'SE Linux', version 30, config word 0x80000001, 8 symbol tables and 7 ocontext records. The field names are inferred from the usual policy header order.
- **C07** (observed): The binary embeds identifiers named after the context families (for example file_contexts_file, seapp_contexts_file, service_contexts_file, hwservice_contexts_file, property_contexts_file, keystore2_key_contexts_file, vendor_service_contexts_file, vndservice_contexts_file, nonplat_service_contexts_file). Their use is inferred.
- **C09** (verified): Top-level form counts in plat_sepolicy.cil: 9,891 allow, 3,773 neverallow, 1,483 roletype, 1,482 type, 1,212 typeattribute, 1,068 typeattributeset, 1,024 category, 562 dontaudit, 365 genfscon, 261 typetransition, 102 class, 89 mlsconstrain, 74 classcommon, 27 sid, 27 sidcontext, 20 fsuse, 5 common, 4 policycap.
- **C10** (verified): mapping/32.0.cil (system_a) is 189,038 bytes and 3,714 lines. It holds 1,238 each of typeattribute, typeattributeset and expandtypeattribute forms, each attribute suffixed _32_0. The system_ext copy is 49,528 bytes with 299 such triples.
- **C11** (verified): Mapping set for versions 26.0 to 32.0: .cil files of 44,069 / 80,908 / 95,919 / 111,009 / 124,230 / 140,511 / 189,038 bytes (lines 754 / 1476 / 1726 / 1977 / 2188 / 2470 / 3714). Compat files are 987, 987, 987, 846, 902 bytes for 26.0 to 30.0, and 29 bytes for 31.0. The 31.0 compat file reads ';; This file can\'t be empty.' In system_ext, every compat file for 26.0 to 31.0 is 0 bytes. The content of the 26.0, 28.0 and 29.0 compat files was not read.
- **C12** (verified): plat_pub_versioned.cil (1,028,727 bytes) is a versioned copy of the platform public policy. It has 2,077 distinct names with the _32_0 suffix, 4,695 allow rules, 1,675 neverallow rules and 2,463 typeattribute forms. Its first declarations include typeattribute dev_type, bdev_type and domain.
- **C13** (verified): vendor_sepolicy.cil refers to 1,589 distinct _32_0 names. Of these, 54 are not defined in plat_pub_versioned.cil. All 54 are the generated form base_typeattr_N_32_0 and are declared in vendor_sepolicy.cil itself as typeattribute forms. No other reference is unresolved.
- **C15** (verified): system_ext_sepolicy.cil is 400,885 bytes and 4,724 lines, with 3,159 allow, 357 type, 230 neverallow, 228 typeattributeset, 222 typeattribute and 93 typetransition forms.
- **C18** (verified): plat_file_contexts is a two-column table of path regex and SELinux context. 627 entries have two fields and 17 have a third field, '--', that restricts to regular files. Patterns use (vendor|system/vendor) alternations and (/.*)? suffixes.
- **C19** (verified): vendor_file_contexts (492 entries) and system_ext_file_contexts (86 entries) are two-column path-regex to context tables with no file-type field.
- **C20** (verified): plat_seapp_contexts (34 lines) maps app identity to a domain and data type. Selector keys are user, seinfo, name, isPrivApp, isSystemServer, isEphemeralApp, minTargetSdkVersion and fromRunAs. Output keys are domain, type and levelFrom. The first rule maps isSystemServer=true to system_server_startup.
- **C24** (observed): The services framework code refers to the mac_permissions files by path (plat, odm and nonplat variants). The services.jar archive itself is compressed, so the path strings were read from the compiled oat image.
- **C26** (verified): plat_service_contexts maps binder service names to labels: 312 two-field entries over 272 distinct labels, all declared in the policy. The last entry is the '*' default mapping to default_android_service.
- **C27** (verified): hwservice contexts map 'package::Interface' names to labels: 85 entries over 74 distinct labels for plat, 50 for vendor (31 labels), 3 for system_ext. All labels resolve.
- **C29** (verified): plat_keystore2_key_contexts maps integer Keystore2 namespaces to key labels: 7 entries. Namespaces must be integers in [0, 2^31). Examples are 0 to su_key, 1 to shell_key, 100 to vold_key.
- **C31** (verified): plat_sepolicy_vers.txt holds the platform policy version '32.0' (5 bytes). This matches mapping/32.0.cil, which the platform checksum chain uses.
- **C33** (verified): vendor_service_contexts maps three services (two drm entries and one display config entry). The label hal_drm_service is not declared in any CIL file in the mounted partitions, so that label does not resolve in this set.
- **C34** (verified): The only precompiled policy files in the inventory are the three odm_a files. The vendor_a partition has no precompiled entry, and product_a has no etc/selinux directory in the mount.
- **C35** (observed): The init binary carries the path strings for the policy inputs: the platform, system_ext, vendor, odm and product policy CIL files, the platform property contexts, precompiled policy paths for odm and vendor, the mapping directories, the platform version file, and the checksum suffixes. The binary is stripped, so the load order is not established.
- **C36** (verified): Seven context files begin with a '#line 1' provenance directive that gives the build-tree source path. Examples are system/sepolicy/private/property_contexts, vendor/meta/sepolicy/sepolicy-common/private/property_contexts and device/meta/anorak-common/sepolicy/vendor/service_contexts.

## Claims with corrected wording

These claims held in substance. The wording below replaces the original.

- **C08** (verified): plat_sepolicy.cil is 28,911 lines and 1,897,754 bytes of CIL. It opens with three role declarations and two cil_gen_require lines, then (handleunknown deny) and (mls true), four policycaps, and 27 sids with their sidcontext forms. The user u range runs from s0 with no categories to s0 with the category range c0 through c1023.
- **C16** (verified): Labels in plat_file_contexts (644 lines), vendor_file_contexts (492) and system_ext_file_contexts (86) all resolve to a type declared in one of the five CIL files. The five files together declare 2,275 distinct type names. The distinct labels per file are 423 (plat), 248 (vendor) and 80 (system_ext), and none is unresolved.
- **C17** (verified): plat_file_contexts labels policy files with sepolicy_file in 11 rules (lines 36, 66, 333, 371, 445, 495, 496, 623, 624, 642, 644). The platform checksum name matches only line 88 (system_file). The odm checksum name matches line 23 (vendor_file) and line 28 (vendor_configs_file), and does not match line 644.
- **C22** (observed): system_ext seapp rules select on the seinfo values oculus_apps, oculus_apps_v2 and fbandroid_debug, and map to domains such as asr_app, mrds_service_app and spatial_persistence. The Oculus and Facebook attribution of these values is inferred from their names.
- **C23** (verified): The mac_permissions XML files are signature-to-seinfo tables. plat has 3 signer elements (seinfo platform, media, network_stack), vendor has 1 (platform), and system_ext has 11 (echovr, fbandroid_debug, oculus_apps, oculus_apps_v2, oculus_core, oculus_inhouse, oculus_loneechoavalanche, oculus_privapps, oculus_prod, oculus_workrooms, oculus_workrooms_dev). Each signature is hex beginning with 3082, a DER SEQUENCE header. In plat, two begin 308204a8 and one begins 308205dc.
- **C25** (verified): plat_property_contexts has 932 data lines: 197 two-field lines (name and label) and 735 with a match keyword and type (exact 726, prefix 9). Types are string 376, int 164, bool 177, enum 11, uint 6 and double 1. 108 names end with a dot (99 of them on two-field lines) and 824 do not. The characterization of the dot-suffixed names as an older style is inferred.
- **C28** (observed): The service manager binaries carry context-file names: hwservicemanager contains the error text 'No match for interface %s in hwservice_contexts', and servicemanager contains ' in service_contexts.'. hwservicemanager has no separate service_contexts reference; its match is a substring of hwservice_contexts. That these files are read is inferred from the strings.
- **C30** (verified): selinux_denial_metadata is a 35-row annotation table with four whitespace-separated fields: source domain, target type, object class and a bug reference of the form b/ followed by 8 or 9 digits (27 rows with 8 digits, 8 rows with 9). The logd binary references this path.
- **C32** (observed): The precompiled binary reports policy version 30, while the platform version file reads 32.0 and the mapping directory includes mapping/32.0.cil (the directory spans 26.0 to 32.0). The cause of the difference is not determined from these files.

## Excluded claims

These claims did not hold on reproduction.

| ID | Claim | Reason |
|---|---|---|
| C14 | vendor_sepolicy.cil is 388,964 bytes and 1,190 lines of CIL forms, with 2,150 allow, 436 type, 200 genfscon, 146 dontaudit, 130 typetransition, 86 neverallow an | wc -l gives 3951 lines (grep -c '' also 3951; 3950 top-level forms). The claimed 1,190 lines does not hold. Byte size 388964, allow 2150, type 436, genfscon 200, dontaudit 146, typetransition 130, nev |
| C21 | The vendor seapp table has one rule and system_ext_seapp_contexts (39 lines) has the rest, 40 rules in total. Each rule carries user, seinfo, name, domain, type | 40 rules in total (vendor 1, system_ext 39). user, seinfo, name, domain and type are present on all 40. levelFrom is present on 36. The four rules without it are system_ext rules for com.meta.internal |

## Gaps

- The precompiled binary was parsed only as far as its 64-byte header. Its symbol tables, class and permission lists, avtab and ocontext records were not decoded, so the rule content of the binary is not known.
- No policy was compiled or validated. secilc (system_a/system/bin/secilc, aarch64 ELF) was not executed, so the build flags, the target policy version and the compile output were not reproduced.
- Init and libselinux reference paths that are not in the mounted partitions: /odm/etc/selinux/odm_sepolicy.cil and odm_*_contexts, /product/etc/selinux/product_sepolicy.cil and product_*_contexts, /vendor/etc/selinux/nonplat_sepolicy.cil and nonplat_*_contexts, /vendor/etc/selinux/precompiled_sepolicy, /debug_ramdisk/userdebug_plat_sepolicy.cil and /mfgmode_plat_sepolicy.cil. Their contents and existence were not checked.
- The label hal_drm_service (two entries in vendor_service_contexts) is declared in no mounted CIL file. The partition that declares it was not in the mount.
- The file_contexts regexes at lines 642 and 644 name checksum files differently from the files on disk. The on-disk checksum files are labeled by the broader rules at lines 88 and 28. The intent behind the naming was not established.
- Regex precedence in file_contexts (which of several matching entries the consumer applies) was not determined. Only the sets of matching entries were listed.
- The certificates in the mac_permissions signature attributes were not decoded (subject, issuer, validity). Only the hex prefix was confirmed.
- The embedded identifiers in the precompiled binary (for example vendor_service_contexts_file) were read from strings only. Their role in the binary is inferred.
- The init binary is stripped. The path references come from strings, with no disassembly, so the order and conditions under which init loads each file were not established.
- The contents of the compat stubs for 26.0, 28.0 and 29.0 (system_a) were not read. Their roles are inferred from file names and sizes.
- The source of the version mismatch between the binary (v30) and the platform version (32.0) was not determined.
- The bug references in selinux_denial_metadata (b/NNNNNNNN) were not resolved against any external source.
- Declared-type counts (2,275) come from grepping '(type NAME' forms in five CIL files. They are not the output of a policy compile and do not include typealias or generated names.
- Only the 51 group rows were examined. Other rows of classified.tsv that reference SELinux paths were not in this group and were not evaluated.
