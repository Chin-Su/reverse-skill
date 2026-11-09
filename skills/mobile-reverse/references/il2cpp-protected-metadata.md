# IL2CPP protected metadata and runtime-derived resource keys

Applies to Unity IL2CPP builds (Android `libil2cpp.so`, iOS `UnityFramework`) where a standard dumper fails or where downloaded resource files are encrypted. Static analysis only. Use on materials you are authorized to analyze.

Worked example: [field-journal entry](../../field-journal/2026-10-31_unity-il2cpp-scrambled-metadata-resource-key.md).

## When to use

- The metadata dumper reports a bad sanity/version, or finds no tables, although `global-metadata.dat` is present.
- The first word of the metadata is not `0xFAB11BAF`, or header offsets are odd or not multiples of the record stride.
- Resource files have a constant magic, a length field, and a ciphertext length that is a multiple of 16.

## Workflow

1. Confirm the binary is decrypted (`LC_ENCRYPTION_INFO_64.cryptid == 0` on iOS).
2. Compare the metadata sanity and version with the standard values. If the sanity differs, search the binary code for the modified constant (as `mov`/`movk` immediates or a literal) to find the loader.
3. Emulate the loader's header transform (straight-line `ldr/sub/add/orr/eor/str`) and rebuild the real header. Check that sections are contiguous and the last one ends at the file size.
4. Write the real header (and the standard sanity) into a copy of the metadata. Parse v29 tables: record sizes are typedef 88, method 32, image 40, field 12, fieldDefaultValue 12 bytes. Check `section_size % stride == 0`.
5. Build a method-name to address map from the code-gen modules (pattern 3 below).
6. Find the resource reader from class and method names, list direct callees, read the key derivation.
7. Recover static `byte[]` constants (pattern 4), derive keys, validate (patterns 5 and 6).

## Reusable patterns

1. **Modified sanity constant**: the loader compares the first word with a constant; search for it as `mov w, #lo` + `movk w, #hi, lsl #16` (or a literal) to find the loader quickly.
2. **Header unscramble by emulation**: a straight-line block of `ldr/sub/add/orr/eor/str` can be emulated with a 40-line interpreter over capstone output; validate by section contiguity and `last_end == file_size`.
3. **Code-gen module map**: find `"<image>.dll\0"` in the binary, find the 8-byte pointer to it, read `methodPointerCount` (+8) and `methodPointers` (+16); `methodPointers[(token & 0xFFFFFF) - 1]`.
4. **Static `byte[]` constants**: class constructor -> `InitializeArray` -> metadata-usage slot (`(kind << 29) | index*2+1`-style encoding, kind 4 = `FieldInfo`) -> `fieldRefs` -> typedef by `byvalTypeIndex` -> field default data.
5. **Key validation without plaintext knowledge**: PKCS7 padding of the last block using the length from the file header; add a second check on other files before trusting a hit.
6. **Round trip as proof**: re-encrypt the decrypted plaintext and compare to the original bytes; this also confirms IV, padding and mode.
7. **Parser as oracle**: a format parser that must end exactly at EOF is a strong correctness check for a reconstructed format.


## Pitfalls

| Problem | Cause | Fix | Cost |
|---|---|---|---|
| Raw 16-byte key scan found nothing | Key derived at runtime (static array XOR name hash) | Read the reader class instead of scanning | medium |
| Metadata dumper gives no output | Header scrambled in the file, restored by the loader | Find loader via sanity-constant immediates; emulate the block | high |
| GUI dumper builds do not accept CLI args | Interactive only | Write the method map yourself from the code-gen modules | medium |
| `IndexError` resolving nested type names | `declaringTypeIndex` is an `Il2CppType` index, not a typedef index | Use `byvalTypeIndex` to map type index to typedef; skip nesting until the type table is parsed | low |
| Wrong record size for methods | v29 method definition is 32 bytes | Check `section_size % stride == 0` for every table | low |
| Enum constants in metadata look random | Constants are obfuscated per build | Infer opcode meaning from data (first/last instruction, operand ranges, extern signatures) | medium |
| `python -I` ignored encoding and script directory | Isolated mode ignores env vars and drops the script dir from `sys.path` | Use `python -X utf8 -I` and insert the script dir into `sys.path` | low |


## Notes

- Constants, keys, magic values, and opcode orders are specific to each build and each title. Do not reuse them across projects without verification.
- Keep project data (binaries, decrypted contents, keys) in the project `work/` directory, not in this repository.
