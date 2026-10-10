# 02 - SQE and GMU firmware

[中文版](02-sqe-and-gmu-firmware.zh-CN.md)

The SQE is the command-processor microcode. The GMU is the power and clock controller firmware. Both have their own binary layouts, decoded here from the files.

## Verified claims

- **G-01** (verified): gmu_gen70200.bin is 67,240 bytes (0x106a8), a multiple of 4. It holds 11 descriptor records at offsets 0x000-0x0af, then four block records whose headers sit at 0x0b0, 0x29c, 0x10648 and 0x10680, each followed by its payload. The walk consumes the file exactly (0 unconsumed bytes). Payload bytes total 67,000, and 11x16 + 4x16 + 67,000 = 67,240.
- **G-02** (verified): Descriptor table: 11 records of 16 bytes at 0x00-0xaf. Each record is four little-endian dwords {addr, zero word, type, size}. Every zero word is 0. Type is 1 for records 0-8 and 7 for records 9-10. Values: 0x00000000/0x1dc, 0x00004000/0x1039c, 0x00000200/0x3838, 0x00100000/0x92ac, 0x00200000/0x12c80, 0x00300000/0x1b240, 0x10004000/0x2d50, 0x10007000/0xf84, 0x10007fdc/0x18 (type 1); 0x60000000/0x1000 and 0x60001000/0x1000 (type 7).
- **S-01** (verified): a740v3_sqe.fw is 76,148 bytes (0x12974), a multiple of 4, giving 19,037 dwords. dword0 is 0x00000000.
- **S-03** (verified): Applying the driver's version rule to this file gives version 0x01511175. The low nibble of dword1 is 0x5, so the special case (nibble 0xa) does not apply, and the version is dword1 itself. dword3 is not used for this file.
- **S-06** (verified): dword2 is 0x01004a5c. Its low 24 bits, 0x4a5c = 19,036, equal both the dword count the loader copies (size-4)/4 and the index of the last dword in the file.

## Open items

These claims were reproduced but judged wrong in wording or label, or they were refuted. They are not stated above until the wording is corrected.

| ID | Label | Reproduce / refute | Claim (as produced) |
|---|---|---|---|
| G-03 | verified | confirmed / needs_change | Block records start at 0x0b0 and sit between payloads. Header = {addr, size, dword +0x8, dword +0xc}; payload begins at header+0x10 and is size bytes. |
| G-04 | verified | confirmed / needs_change | Descriptor versus block cross-match: descriptors at 0x00000000 (size 0x1dc), 0x00004000 (size 0x1039c) and 0x10007fdc (size 0x18) have payload blocks  |
| G-05 | verified | needs_change / needs_change | The first payload (file offset 0xc0) starts with four words 0x10007f80, 0x00000105, 0x00000145, 0x00000159. Words 1-3 have bit 0 set, which is the usu |
| G-07 | observed | confirmed / needs_change | Descriptor parser gen7_gmu_parse_fw reads each 16-byte record as: +0x00 addr; +0x04 must be 0, and the first record with a non-zero word here ends the |
| G-08 | observed | confirmed / needs_change | Loader gen7_gmu_load_fw reads block headers differently from the parser: +0x00 addr; +0x04 size. A zero size means skip 16 bytes. Otherwise the payloa |
| G-09 | observed | needs_change / refuted | The driver's own names for the block fields are addr and size, used in GMU block messages. No module string names the zero word, the type word, or the |
| G-10 | verified | confirmed / needs_change | gmu_gen70200.bin contains no version or build identifier in any printable run of 5 characters or more. Its printable text is firmware routine names an |
| G-11 | observed | needs_change / needs_change | The driver obtains the GMU firmware version at runtime through HFI and checks it against a minimum, rather than reading it from the image. Strings: 'G |
| G-12 | observed | confirmed / needs_change | The GMU payload contains named firmware routines and diagnostics, for example HfiMemReply, HfiFeatureCtrl, GmuPwrRequestPowerOn, EcpSetupPreemptBuffer |
| S-02 | verified | confirmed / needs_change | The leading dwords are dword1=0x01511175, dword2=0x01004a5c, dword3=0x01511146, dword4=0x010024c8, dword5=0x010e4e50, dword6=0xb8020000, dword7=0xb803 |
| S-04 | observed | needs_change / needs_change | The SQE loader adreno_get_firmware copies file bytes from offset 4 to the end (size-4 bytes, 76,144 bytes = 19,036 dwords) into a kernel-global buffer |
| S-05 | observed | confirmed / needs_change | The SQE firmware structure sits at device offset 0x1510 (gen7_microcode_read passes x0+0x1510). Its version field is at struct offset 0x10 (absolute 0 |
| S-07 | verified | confirmed / needs_change | dword4 is 0x010024c8: low 24 bits 0x24c8 = 9416, which indexes dword 0x01000000 (same top-byte pattern as dword4). dword5 is 0x010e4e50: low 24 bits 0 |
| S-08 | verified | needs_change / needs_change | The SQE file ends with ASCII '01504202' at file offset 0x12964, followed by 'ESU$' and 'R' with three zero bytes. The last dword is 0x00000052. This i |
| X-01 | verified | confirmed / needs_change | msm_kgsl.ko contains both target file names as literal strings. The SQE name a740v3_sqe.fw is a candidate in the SQE table, and gmu_gen70200.bin is th |
| X-02 | observed | confirmed / needs_change | The GMU file name is read from device config +0x60 (fallback at +0x68) inside gen7_gmu_parse_fw, and the GMU firmware is requested through firmware_re |
| T-01 | verified | needs_change / confirmed | A standalone dumper for both formats was built and run. It reproduces the record tables, block walk, descriptor cross-match, SQE version rule and dwor |
