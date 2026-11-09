# [2026-10-31] unity-il2cpp-scrambled-metadata-resource-key

## 场景分类 / Scenario

iOS IPA reverse (Unity IL2CPP). Static analysis of a protected IL2CPP build to recover how a downloaded resource file format is decrypted.

## 目标概述 / Goal

Given an IPA and encrypted resource files (`*.bytes`) extracted from the game's content packs, find the container format and decryption scheme, using static analysis only, then decode the plaintext (which turned out to be a hotfix patch in InjectFix/IFix format).

## Scope 摘要（脱敏） / Scope (anonymized)

- auth_basis: own_system (user-supplied local files)
- network_profile: offline
- asset_types: [ipa, resource-bytes]

## 角色 / Roles

- lead_role: lead
- specialists: []

## 完整执行链路 / Execution chain

1. Extract the IPA entries (`{binary}`, the IL2CPP framework, `{metadata}`); confirm the Mach-O is not FairPlay-encrypted (`LC_ENCRYPTION_INFO_64.cryptid == 0`).
2. Inspect the resource files: constant 4-byte magic, `u32` plaintext length, ciphertext whose length minus 8 is a multiple of 16 and equals `length + PKCS7 pad`. Two variants of the same file share exactly the first N ciphertext blocks and then diverge completely, so the mode is chained (CBC), not ECB.
3. First attempt (negative): brute-force every 16-byte window of the binary, the metadata, and all frameworks as an AES-128 key, validating with the PKCS7 padding of the last block (works for ECB and for CBC because the previous ciphertext block is known). 0 hits: the key was derived at runtime, not stored.
4. Try to parse the metadata with a dumper: fails. The sanity word is changed and header fields are scrambled.
5. Find the metadata loader by searching the code for the modified sanity constant as `mov/movk` immediates. The loader copies each header word through `(v - c_i) ^ (K + d_i)` into permuted slots.
6. Emulate that block with a small Python interpreter over the exact instructions (`ldr/sub/add/orr/eor/str`) to rebuild the real header. Validate: sections are contiguous and the last section ends exactly at the file size.
7. Rewrite the header (and the sanity word) to get a standard v29 metadata file. Parse type/method/image tables and build a method-name to address map: each image has an `Il2CppCodeGenModule` whose first field points to the module name string; `methodPointers[(token & 0xFFFFFF) - 1]`.
8. Locate the resource reader from names in the metadata (reader/encoder classes, `GetMixedKey`-style methods) and list their direct callees by scanning `bl` instructions.
9. Read the key-mixing method: key = static `byte[16]` XOR a per-name hash; hash = `h*31 + toupper(c)` over the resource name; IV = zero array.
10. Recover the static `byte[16]`: the class constructor calls `RuntimeHelpers.InitializeArray` with a metadata-usage slot whose encoded value gives a `FieldRef` index (top 3 bits = usage kind, `FieldInfo`); `fieldRefs[i]` gives (type index, relative field index); the typedef whose `byvalTypeIndex` equals that type index is the holder class; read the field's default value data.
11. Validate: PKCS7 check on all sample files, then `encrypt(decrypt(file)) == file` byte-for-byte for every sample.
12. Parse the plaintext: it is an IFix patch. Rebuild the format from the public InjectFix loader order, parse every file to its exact last byte, and infer opcode meaning from data because opcodes are shuffled per build.

## Evidence 链摘要（脱敏） / Evidence summary

| E-id | severity | status | source_type | Reusable command pattern | Related finding |
|------|----------|--------|-------------|--------------------------|-----------------|
| E-001 | info | validated | command | `python decode_metadata_header.py` then check last section end == file size | F-001 |
| E-002 | info | validated | command | `python verify_roundtrip.py` (re-encrypt equals original, all files) | F-001 |
| E-003 | info | validated | command | `python ifix_parse.py <plaintext>` (parser ends exactly at EOF) | F-001 |

## Finding / Path 摘要

- top_finding: the resource key is derived per file name from a static array; the plaintext is a hotfix patch container.
- path_type: callflow
- path_one_liner: scrambled metadata header -> loader emulation -> method map -> resource reader -> key derivation -> byte-exact round trip.

## 踩坑记录 / Pitfalls

| Problem | Cause | Fix | Cost |
|---|---|---|---|
| Raw 16-byte key scan found nothing | Key derived at runtime (static array XOR name hash) | Read the reader class instead of scanning | medium |
| Metadata dumper gives no output | Header scrambled in the file, restored by the loader | Find loader via sanity-constant immediates; emulate the block | high |
| GUI dumper builds do not accept CLI args | Interactive only | Write the method map yourself from the code-gen modules | medium |
| `IndexError` resolving nested type names | `declaringTypeIndex` is an `Il2CppType` index, not a typedef index | Use `byvalTypeIndex` to map type index to typedef; skip nesting until the type table is parsed | low |
| Wrong record size for methods | v29 method definition is 32 bytes | Check `section_size % stride == 0` for every table | low |
| Enum constants in metadata look random | Constants are obfuscated per build | Infer opcode meaning from data (first/last instruction, operand ranges, extern signatures) | medium |
| `python -I` ignored encoding and script directory | Isolated mode ignores env vars and drops the script dir from `sys.path` | Use `python -X utf8 -I` and insert the script dir into `sys.path` | low |

## 工具链发现 / Toolchain findings

- Capstone for ARM64 disassembly and numpy for bulk scans are enough; no decompiler was required.
- A vectorized numpy AES block decrypt (key expansion for many keys at once) scans about 140k keys/s per core; fine for 240 MB in minutes across cores.
- `cryptography` is enough for AES-CBC verification.
- Public InjectFix source (loader order, `Code` enum, shuffle generator) was the key reference for the patch format.

## 关键代码/命令 / Key snippets

```text
# last-block PKCS7 key test (CBC: XOR with previous ciphertext block first)
pad = 16 - plain_len % 16
P   = AES_decrypt(key, C_last) [XOR C_prev for CBC]
ok  = P[-pad:] == bytes([pad]) * pad
```

## 改进建议 / Improvement suggestions

- Add a `references/il2cpp-protected-metadata.md` how-to (patterns below) to `mobile-reverse`.
- In the `mobile-reverse` Phase 2 checklist, add: "if the dumper fails, compare the metadata sanity/version with the standard values and search the code for the modified constant".

## 可复用模式 / Reusable patterns

1. **Modified sanity constant**: the loader compares the first word with a constant; search for it as `mov w, #lo` + `movk w, #hi, lsl #16` (or a literal) to find the loader quickly.
2. **Header unscramble by emulation**: a straight-line block of `ldr/sub/add/orr/eor/str` can be emulated with a 40-line interpreter over capstone output; validate by section contiguity and `last_end == file_size`.
3. **Code-gen module map**: find `"<image>.dll\0"` in the binary, find the 8-byte pointer to it, read `methodPointerCount` (+8) and `methodPointers` (+16); `methodPointers[(token & 0xFFFFFF) - 1]`.
4. **Static `byte[]` constants**: class constructor -> `InitializeArray` -> metadata-usage slot (`(kind << 29) | index*2+1`-style encoding, kind 4 = `FieldInfo`) -> `fieldRefs` -> typedef by `byvalTypeIndex` -> field default data.
5. **Key validation without plaintext knowledge**: PKCS7 padding of the last block using the length from the file header; add a second check on other files before trusting a hit.
6. **Round trip as proof**: re-encrypt the decrypted plaintext and compare to the original bytes; this also confirms IV, padding and mode.
7. **Parser as oracle**: a format parser that must end exactly at EOF is a strong correctness check for a reconstructed format.

## 进化动作 / Evolution actions

- [x] Entry added to `field-journal` and indexed in `_index.md`
- [x] Reusable patterns also published as `skills/mobile-reverse/references/il2cpp-protected-metadata.md`
- [ ] Optional: link the new reference from `mobile-reverse/SKILL.md` (Reference list) in a later change
