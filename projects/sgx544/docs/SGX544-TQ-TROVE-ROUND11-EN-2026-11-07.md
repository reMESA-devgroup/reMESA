# SGX544-reMESA — Round 11 (2026-11-07, English): TQ template trove closed to its consumer

> NEW incremental file (round-11 findings only). Prior state: see `SGX544-ADDENDA-EN-2026-11-07.md`,
> `SGX544-HANDOFF-UPDATES-EN-2026-11-07.md`, `SGX544-STATUS-EXEC-EN-2026-11-07.md` in this folder.
> Source of record: `/home/gama/sgx544-re/docs/INVESTIGACAO-LIBGLSLCOMPILER-SYSTEM-P5200.md`
> (round-11 addendum) and `/home/gama/sgx544-re/docs/HANDOFF-AGENTES-OPEN-MODEL-SGX544.md`
> (round-11 block). No binaries or disassemblies copied here.

## What closed this round

The "firmware-consumed addresses" question for the transfer-queue hit its static endpoint:
the sole code reference to trove base ELF `.rodata` 0x398c0 (`0x1b329: addl $0x398c0, %ebp`,
the only `398c0/498c0` literal in the entire libsrv_um disassembly) resolves, in Ghidra
Golds, to **`SGXCreateTransferContext`** (hash/line evidence and IDs on the source repo):

- Evidence id: `SGXCreateTransferContext` disassembly slot captured as `ev_eef51b55…` (short form); full 814-line decompile.
- Mechanism: for every staged template kind a fresh device-memory buffer is allocated
  (`PVRSRVAllocDeviceMem(class 0xb, size, align 8)`), the raw template words are
  `PVRSRVMemCopy`-ed **from `DAT_000498c0 + iVar16`** directly into that device buffer,
  and the buffer's data pointer is anchored into a per-kind 0x24-byte object (field +0x10).
- This makes the trove bytes **device-memory-resident content destined for firmware
  consumption** (the last-hop consumer beyond the device store is the PDS/USSE firmware; INFERRED).

## Case table (staging loop 1, kinds 1..0x38)

Per kind: [field+0x18, field+0x1c] pair, copy size, trove offset (`DAT_000498c0+offset`).
Kind 2 is special: object +0x8 receives a **direct .rodata pointer** `DAT_00049b38`, size 0x10.

| kind | [f18,f1c] | size | trove off |
| --- | --- | --- | --- |
| 1 | [0,1] | 0x10 | 0x0 |
| 2 | special | 0x10 | direct ptr 0x49b38 |
| 3 | [1,2] | 0x58 | 0x1d8 |
| 4 | [1,2] | 0x48 | 0x230 |
| 5 | [1,2] | 0x18 | 0x358 |
| 6..0xc | [0,*] | 0x10..0x18 | 0x370..0x3d8 |
| 0xd..0x12 | [0,*] | 0x10 | 0x3e8..0x438 |
| 0x13..0x18 | [0/1,*] | 0x10..0x18 | 0x170..0x1c0 (re-entrant region) |
| 0x19..0x24 | [0,*] | 0x10..0x18 | 0x288..0x348 |
| 0x25 | [1,2] | 0x28 | 0x468 |
| 0x26 | [1,2] | 0x28 | 0x490 |
| 0x27 | [1,3] | 0x38 | 0x4c0 |
| 0x28 | [1,2] | 0x40 | 0x500 |
| 0x29 | [0,1] | 0x40 | 0x540 |
| 0x2a | [10,2] | 0xd0 | 0xa0 |
| 0x2b | [10,2] | 0x90 | 0x10 |
| 0x2c | [0,0] | 0x10 | 0x448 |
| 0x2d | [0,0] | 0x10 | 0x458 |
| 0x2e | [0,1] | 0x10 | 0x580 |
| 0x2f | [0,1] | 0x20 | 0x590 |
| 0x30 | [0,0] | 0x10 | 0x5b0 |
| 0x31 | [0,3] | 0x40 | 0x5c0 |
| 0x32 | [1,3] | 0x38 | 0x600 |
| 0x33 | [1,2] | 0x38 | 0x640 |
| 0x34 | [1,2] | 0x18 | 0x6a0 |
| 0x35 | [0,1] | 0x18 | 0x680 |
| 0x36 | [0,1] | 0x20 | 0x6c0 |
| 0x37 | [0,1] | 0x20 | 0x6e0 |
| 0x38 | [0,1] | 0x28 | 0x700 |

Semantics of field pairs [f18,f1c]: **UNKNOWN** (INFERRED: per-template config words; the
`10` value appears only for the two largest blocks 0x2a/0x2b).

## Direct-pointer loops (staging loops 2/3) and the PDS "1src" fill

- Loop 2 (objects at `piVar4[iVar5+0x15]`, template ptr at obj+8, size at piVar4[X]+0xc):
  kind0: `DAT_00049338` sz 0x18; kind1: builder `FUN_0002a340(...,0x400,0x10,1,0,…,"PDS - 1src",piVar4+0x14)`
  then a **0x40-strided loop copying template `DAT_00049480` ten times into a 0x400-byte
  device block** — the "PDS - 1src" program content is literally this 0x40-byte template
  (`07000185 07040002 07081004 af000000`) replicated 10x (CONFIRMED);
  kind2: `DAT_000493a0` sz 0x58; kind3: `DAT_00049400` sz 0x80; kind4: `DAT_00049360` sz 0x2c.
- Loop 3 (objects at `piVar4[iVar5+0x1a]`, `*(obj+0x14)=1`): `DAT_000494e0` (0x2c),
  `DAT_00049520` (0x2c), `DAT_00049560` (0x54), `DAT_000495c0` (0x68), `DAT_00049640` (0x7c),
  `DAT_000496c0` (0x90), `DAT_00049760` (0x90), `DAT_00049800` (0xa4), default `DAT_000494c0` (0x18).
- Structural observation (INFERRED): loop-2/3 field+0x18 values (0x10..0x80) track the
  template source-size class; field+0x1c looks like a destination allocation class.

## Rest of TQ assembly in `SGXCreateTransferContext` (CONFIRMED call inventory)

1. "Failed to allocate memory for **End of Render program**!" — pre-loop alloc path.
2. Two **pixel-event program** objects (`piVar4[0x24..0x26]`), obj+0xc from `FUN_0003ec80`/`FUN_0003ec90`; obj+0x1c flags 0/1.
3. **Fence ID**: 4-byte devmem (alloc class 0x4f), zero-initialized on allocation.
4. **HWTransferContext**: 0xe4-byte user alloc; fields {+0:1, +4:1, +0xc/+0x10: fetch from `piVar4[9]` object +8s, +0x14:0, +0x37/+0x38:0, +0xd8:`PVRSRVGetCurrentProcessID()`}; registered via `FUN_0001ed80(param_1, piVar4+3, obj, 0xe4, 8, *param_3, piVar4+4)`.
5. `FUN_0003bd80(param_1, 0x10000, 0x40, 0x400, uVar17, piVar4+0xc)` — per-ctx buffer (semantics UNRESOLVED).
6. **HW2DContext**: 0x24-byte object registered via `FUN_0001f170(param_1, piVar4+10, obj, 0x24, 8, *param_3, piVar4+0xb)`.
7. Mutex + `piVar4[2]=param_3[1]; *param_4=piVar4; return 0`.

**hTQ closure**: the object returned via `*param_4` IS the TQ whose handle later reaches
HMI+0x1e0 (round-9 chain: getter at gralloc HMI+0x110 → IMGegl `(*(module+0x110))(module,
&queue[0xfd])` → queue byte 0x3f4). Round 9's hTQ producer now has a fully mapped body.

Alternate entry path in the same function: `FUN_0002a340(param_1, piVar4, param_2(+4&~3), 4, 1, 1, uVar12, "texture - SB", piVar4+0x6a)` — same builder, explicit name **"texture - SB"**.

## Artifact

- `/home/gama/sgx544-re/artifacts/imgeng/libsrv_um_tq_template_trove_0x39338-0x39b80.txt`
  — SHA-256 `7237b1af8378c952d8e2f337473e285b7cc8aa8d4b5d0cbfb52825fee35ca6b7`, 552-line
  word dump of ELF `.rodata` 0x39338..0x39b80 (VA == file offset in this blob), block
  boundaries matching every template pointer above. Descriptor ladder
  `07000185 / 07040002 / 07081004 / 07040402 / 070c2004 / 07100002 / 07143004` with
  `af000000` terminators; all blocks zero-prefixed.

## Classification

- CONFIRMED: trove byte content, case table, direct ptr destinations, 0x400 "PDS - 1src"
  fill by 10x replication of `DAT_00049480`, all device-mem registration calls.
- INFERRED: descriptor words are PDS/USSE/CCB templates (firmware-side reading pending);
  [f18,f1c] config-word role.
- UNRESOLVED: firmware endpoint consumption per template; `FUN_0003bd80` semantics;
  "texture - SB" identity.

## Next actions (priority order)

(a) GLESv2: name the per-frame List-A stasher at ELF 0x39759–0x39769 span (Ghidra 0x49759,
inside the glGetActiveAttrib export gap) and correlate with DoKickTA List-A consumption;
(b) `FUN_0001cd50` kick-buffer factory kind map (7 kick buffers → TA/3D/VGP/PDS roles);
(c) `FUN_0003f1b0` finalize; (d) IMG-table entries [1],[3..11] body classification; (e)
kernel-side consumer of List A/B third words post-`PVRSRVLookupHandle` type-5 resolution (UNKNOWN — firmware consumption).
