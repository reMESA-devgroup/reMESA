# SGX544-reMESA — Round 15 (2026-11-07, English): devmem handle identity confirmed; List-A word chain closed to CCB

> NEW incremental file. Prior rounds: 2026-11-07 files above in this folder.
> Source of record: `/home/gama/sgx544-re/docs/INVESTIGACAO-LIBGLSLCOMPILER-SYSTEM-P5200.md`
> (round-15 addendum) + handoff round-15 block. Evidence: libsrv_um blob SHA `ccafe90d…`.

## 1. devmem out-struct (0x34 B) field map — CONFIRMED

`PVRSRVAllocDeviceMem` (export ELF 0xb4f0) forwards to impl `FUN_0001a430` which:
- rounds size to 0x1000 when flags bit 0x10000000 set; sends flags masked `0xf7ffffff`;
- performs bridge ioctl **`0xc01c6707`** (in 0x30 B / out **0x54 B**, `resp[0]` = error);
- builds the **0x34-byte user struct** (`__ptr[i] = resp dword[i+?]`, frame-relative):

| user offset | content | consumer |
| --- | --- | --- |
| +0x00 (`__ptr[0]`) | resp@0x08 — CPU mapping slot (0 when flags bit 0x8000000) | CBUF obj+0x10 ← GLES2CreateGC |
| +0x08 (`__ptr[2]`) | resp@0x10 — **DEVICE ADDRESS** | CBUF obj+0x0c/0x28; **List-A record word 0** |
| +0x20 (`__ptr[8]`) | resp@0x28 — **KERNEL MEMORY HANDLE** | **List-A record word 2** → kernel `PVRSRVLookupHandle` |
| +0x04..0x1c, +0x24..0x30 | resp dwords (sync/side info) | (detail UNRESOLVED) |

- `FUN_00022970` = **PVRPMapKMem** (string evidence) — with the handle as arg5:
  bridge **`0xc01c670c` = MHANDLE_TO_MMAP_DATA** (in 8 B / out 0x14 B) then
  `__mmap2(0, len, PROT, MAP_SHARED, conn, off)`; success writes
  `*param_4 = cpuVA` and `*param_2 = cpuVA + end`; failure strings
  `"PVRPMapKMem"`, `"PVRPMapKMem: MHANDLE_TO_MMAP_DATA failed: handle=%p, error %d"`,
  `"PVRPMapKMem: mmap(2) failed: Mapping handle=%p: %s"`.
  ⇒ **`*(devmem+0x20)` is a kernel memory handle — CONFIRMED.**

## 2. List-A record word chain — CLOSED to the CCB boundary (static)

| word | GLESv2 producer | transport | kernel | firmware consumption |
| --- | --- | --- | --- | --- |
| 0 (`+12i`) | `*(devmem+8)` device address (CBUF arm 0x22f2f) | kick+0x184+12i → KM_arg+0x38+12i | copy to `CCB+0x348+8i` | **GPU address directly** |
| 1 (`+12i`) | per-draw packed tag (FUN_000481d0, record 0 only) | kick+0x188+12i | copy | type tags (semantics INFERRED) |
| 2 (`+12i`) | `*(devmem+0x20)` handle (CBUF arm 0x22f26) | kick+0x18c+12i | **resolved in place via `PVRSRVLookupHandle` type 5** | GPU address (post-resolution) |

List-B (kick+0x300..0x30c, ≤ 3) follows the same `CCB+0x448+8i` path. Beyond CCB,
the PDS/USSE fetch of these addresses is firmware-runtime behavior — STATIC LIMIT.

## 3. CBUF size fields — writers check (CONFIRMED absence in-blob)

GLESv2 READS `ctx+0x1b80..0x1b9c` at 14 sites, ZERO stores; generated
`artifacts/dis/libIMGegl.dis` (20,199 lines, SHA `6a28beb7…`) contains zero
`0x1b??(` displacement hits. ⇒ sizes zero-default (`size=4` arms → 0x40/0x20 align
rounding); a Samsung-glue writer remains UNRESOLVED.

## Classification / next

- CONFIRMED: §1–§3 (evidence ids + blob hashes in source repo).
- INFERRED: resp-offset↔frame-local mapping precision; sync/side dword meanings.
- UNRESOLVED: FUN_00031bb0 (GLESv2) identity; List-B producer sweep beyond DoKickTA
  staging; extra 0x18-B struct in the allocator.
- NEXT ACTION: kernel re-verif of third-word resolution in pvrsgx.ko
  (`artifacts/firmware/rd-lib-modules/pvrsgx.ko`) — snapshot libsrv_um, open kernel blob;
  then checklist §4 refresh.
