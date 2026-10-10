# 01 - Zap secure firmware (a740v3)

[中文版](01-zap-firmware.zh-CN.md)

Zap is the GPU firmware that the secure world loads. The image is split into three parts, and the same code segment appears in the ELF and in the signed wrapper.

## Verified claims

- **F1** (verified): File sizes: .mdt 6284 bytes (0x188c), .b00 148 (0x94), .b01 1072 (0x430), .b02 6136 (0x17f8), .elf 5168 (0x1430), .mbn 14328 (0x37f8). Full SHA-256 values are listed in the evidence.
- **F2** (verified): The .mdt begins with an ELF32 little-endian header: EI_CLASS 1, EI_DATA 1, EI_VERSION 1, OS/ABI 0. Fields: e_type 2 (EXEC), e_machine 164 (0xa4), e_version 1, e_entry 0x1000, e_phoff 0x34, e_shoff 0 (no section header table), e_flags 0x3, e_ehsize 52, e_phentsize 32, e_phnum 3, e_shentsize 0, e_shnum 0, e_shstrndx 0.
- **F3** (verified): The .mdt program header table at 0x34 has three 32-byte entries. [0]: p_type 0, p_offset 0x0, p_vaddr 0x0, p_paddr 0x0, p_filesz 0x94 (148), p_memsz 0, p_flags 0x07000000, p_align 0. [1]: p_type 1 (PT_LOAD), p_offset 0x1000, p_vaddr 0x1000, p_paddr 0x1000, p_filesz 0x430 (1072), p_memsz 0x430, p_flags 0x08000007, p_align 0x100000. [2]: p_type 0, p_offset 0x2000, p_vaddr 0, p_paddr 0, p_filesz 0x17f8 (6136), p_memsz 0x17f8, p_flags 0x02000000, p_align 0x1000. The .mbn table contains the same three entries.
- **F5** (verified): .b00 is byte-identical to .mdt bytes 0x0 to 0x93 (148 bytes). .b02 is byte-identical to .mdt bytes from 0x94 to end (6136 bytes). The .b01 bytes do not occur anywhere in the .mdt.
- **F9** (verified): Header bytes differ between .elf and .mbn: .elf has e_phnum 1 and one program header, .mbn has e_phnum 3 and the three-entry table. Over the overlapping prefix 0x0 to 0x1430 there are 30 differing bytes, the first at 0x2c and the last at 0x91. The code bytes at 0x1000 to 0x142f are identical.
- **F13** (verified): None of the 32-byte SHA-256 digests of .b01, .b02, .mdt or .mbn appears as a raw byte sequence in the hash segment.
- **F16** (verified): The .elf, .b00 and .b01 files contain no printable run of 4 or more characters. The .b02, .mdt and .mbn files each contain 83 such runs, all located in the hash segment, and these are the certificate names, validity strings and random-looking fragments listed in F14. No build ID, version string or message name was found.
- **F17** (verified): The code segment (.b01, file offset 0x1000 in .elf and .mbn, virtual address 0x1000) starts at e_entry 0x1000. The first little-endian word is 0x01511014.
- **F19** (verified): Disassembling the whole 1072-byte .b01 with the same decoder and default CPU yields 202 brace-delimited packet blocks and 60 'invalid instruction encoding' warnings. The cause of the warnings is not determined.

## Open items

These claims were reproduced but judged wrong in wording or label, or they were refuted. They are not stated above until the wording is corrected.

| ID | Label | Reproduce / refute | Claim (as produced) |
|---|---|---|---|
| F7 | verified | confirmed / needs_change | .elf (5168 bytes) is an ELF32 EXEC file with e_machine 164, named Qualcomm Hexagon in the ELF header dump. e_entry is 0x1000, e_phnum is 1 and there a |
| F8 | verified | confirmed / needs_change | .mbn is 14328 bytes (0x37f8) and is the same ELF image with the three-entry table. Bytes 0x0 to 0x93 equal .b00, bytes 0x1000 to 0x142f equal .b01, an |
| F10 | verified | confirmed / needs_change | The hash segment (b02, mbn file offset 0x2000) begins with little-endian u32 words at segment offsets: +0x00 0x0, +0x04 0x7, +0x08 0x18, +0x0c 0x0, +0 |
| F11 | verified | needs_change / needs_change | Hash segment (b02, 6136 bytes) layout: [0x00,0x68) header words, 13 nonzero bytes. [0x68,0x218) a 432-byte block with 205 nonzero bytes. [0x218,0x10a8 |
| F14 | verified | needs_change / needs_change | The hash segment contains five X.509-style certificate structures with these name and validity strings at hash-segment offsets (hex). C1 at 0x218 (645 |
| F18 | verified | needs_change / needs_change | Disassembling the first 64 bytes of .b01 with the hexagon triple (default CPU) yields ten brace-delimited packets: { r1 = memw(r5+#4) ; r4 = memub(r1+ |
