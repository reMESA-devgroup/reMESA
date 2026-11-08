# SGX544-reMESA — Round 13 (2026-11-07, English): kick back-channel decoded; "PDS - 1src" generator identified

> NEW incremental file. Prior rounds: 2026-11-07 files in this folder.
> Source of record: `/home/gama/sgx544-re/docs/INVESTIGACAO-LIBGLSLCOMPILER-SYSTEM-P5200.md`
> (round-13 addendum) + handoff round-13 block.

## 1. `misc6w` — SGXKickTA arg3 back-channel decoded (CONFIRMED writes; labels INFERRED)

GLESv2 passes `&local_c8` (6-dword stack buffer at ESP+0x54) **uninitialized** as arg3 of
`SGXKickTA(hDev, kick, local_c8, 0, 0)` (round-12 asm evidence). libsrv_um SGXKickTA
(Ghidra 0x23b40, hash-verifiable blob `ccafe90d…`) then:

| dword | value | note |
| --- | --- | --- |
| [0] | 0 → 1 when pending-row found | `iVar22 = puVar2[0x1b] != 0` and the 0x60-stride scan of `*(puVar2[0x1a] + i*0x60 + 0x14)` (flag bit 0x80) hits before count |
| [1] | `(*(iVar4+8) − 0x20) − *(*piVar18+0x14)` | remaining command-space delta (INFERRED); `iVar4 = *(*(*(iVar20+4))+0xc)`, `iVar20 = param_2[0xce]` |
| [2] | `iVar22*2` | pending counter ×2 (INFERRED) |
| [3] | `puVar2[0x12]*2` | pending counter ×2 (INFERRED) |
| [4],[5] | 0 | only touched by entry `PVRSRVMemSet(param_3, 0, 0x18)` |

- GLESv2 consumes ONLY `[0] != 0` (CONFIRMED absence of [1..5] reads in DoKickTA pseudo).
- `puVar2 = param_2[0xcf]` — mutex-guarded kick-tracking object (entry `PVRSRVLockMutex(*puVar2)`,
  early-return 3 if null). Tracking-object field-identity = next target.

## 2. `FUN_0002a340` = `sgxtransfer_buffer.c` allocator (CONFIRMED)

39-line function (evidence `ev_885aae7a…`), source-file asserts `0x47`/`0x4a`:

- Requires `param_4` power-of-two `param_4 <= 0x1000`, `param_3` compatible
  (`((param_4|param_3) & param_4-1)==0`).
- Builds **0x130-byte descriptor**: `+0xc` size (param_3), `+0x10` granularity (param_4),
  `+0x14` param_5, `+0x18..` devmem result slot `PVRSRVAllocDeviceMem(*param_2, param_7,
  class, size, 0x1000, &desc+0x18)`, `+0x128` name tag ("PDS - 1src" / "texture - SB"),
  `+0x12c` ctx back-ref; class `0xD` if param_6==0 else `0xF`.
- TROVE CHAIN CLOSED: SGXCreateTransferContext's "PDS - 1src" block = 0x400-B transfer
  buffer (granularity 0x10, class 0xD) whose resident device content is **template
  `DAT_00049480` (ELF .rodata 0x38480; words 07000185 / 07040002 / 07081004 / af000000)
  replicated 10×** (0x40-strided loop into `**(desc+0x18)`).

## Classification / next

- CONFIRMED: §1–§2 writes and struct relations (evidence ids + blob hashes in source repo).
- INFERRED: remaining-space / pending-count semantics; 0x60-stride row meaning.
- UNRESOLVED: `FUN_00026200` + tracking-object fields; GLESv2 obj+0x3c writer arm (factory
  kind map); third-word firmware consumer after kernel `PVRSRVLookupHandle` type-5
  resolution — STATIC LIMIT (CCB copy visible in kernel; PDS/USSE read-out not observable
  in statically analyzable code).
- NEXT ACTION: REA libsrv_um — `FUN_00026200(param_1, iVar20, uVar14)` body + tracking
  object identity; then GLESv2 PIC jumptable resolution for the factory kind map.
