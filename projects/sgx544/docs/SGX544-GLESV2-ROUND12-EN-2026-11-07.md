# SGX544-reMESA — Round 12 (2026-11-07, English): GLESv2 List-A packer named; two-list handle/kick-evidence refresh

> NEW incremental file (round-12 findings only; prior rounds in the 2026-11-07 files above).
> Source of record: `/home/gama/sgx544-re/docs/INVESTIGACAO-LIBGLSLCOMPILER-SYSTEM-P5200.md`
> (round-12 addendum) + handoff round-12 block. No binaries or disassemblies copied here.

## 1. The List-A record triple is now a NAMED producer chain (CONFIRMED)

Round 10 established: SGXKickTA (libsrv_um) fills KM_arg list-A records from GLESv2
`kick+0x184/0x188/0x18c`, and kernel SGXDoKickKM copies them to `CCB+0x348+8i` after
`PVRSRVLookupHandle` (type 5) resolves the *third word* in place. Round 12 names the
GLESv2-side producer of the triple:

- **`FUN_000481d0` (Ghidra VA; ELF `0x381d0`; 38 KB internal function in the `glScissor`..`glGetActiveAttrib` symbol gap, "eurasiacon/opengles2/shader.c" strings)== per-draw vertex-attribute packer**.
- Writes (exact pseudo anchor):
  - `*(ctx+0x2e4) = dword_count << 0x18 | 0x1000` (or `| 0x1100` when state dword bit 0x2000 set; also mirrored to `ctx+0x2d0`), where `dword_count = 4 + Σ attr_cost` with attr_cost {type 2 → 2, type 3 → 3, type 4 → 4} dwords per enabled attribute;
  - `*(ctx+0x2e8) = per-attribute 3-bit TYPE TAGS` (slot `3*index`: type 2→1, type 3→3 or 5, type 4→7);
  - `*(ctx+0x2ec) = precision-qualifier mask` (attributes with qualifier < 3; `> 3` → `"Invalid precision qualifier"` error).
  - These are exactly byte offsets shocking `kick+0x184/0x188/0x18c` (kick = ctx+0x160).
- Supporting behavior: contiguous-index allocator over bitmap `local_4a4[16]`
  (`"Not enough contiguous indices for attribute %s."`), asserts shader.c 0x21/0xa92,
  attribute-location remap `*(ctx+0xd0+iVar14*4) = iVar13` from `*(ctx+0xf8+iVar13*4)`,
  dirty flag `*(ctx+0x2e0) = 1`, KEGLSetBlob/KEGLGetBlob(0x41), shader-binary memcpy from
  `*(*(ctx+0x128)+0x10)` and `*(*(ctx+0x1c4)+0x10)`, final `*(in_EAX+8) |= 0x1000; return 1`.
- Ordering vs GLES2CreateGC init: init stages one record set from the 7 buffer objects
  once; the per-draw packer OVERWRITES the same slots afterwards — so at kick time the
  resident triple is the per-draw packed descriptor.

## 2. Correction to round 10 (CONFIRMED)

`FUN_0003f1b0` (ELF `0x2f1b0`) is the **GL extensions-string builder**, called at user-mode-ESC tail
(scans 0x29 entries, mask `0x1fffffbd`; first tag "GL_OES_rgb8_rgba8"; allocs base
store `ctx+0x1afc`; returns 1 on calloc success) — it is NOT a "finalize kick state"
routine. Kick init ends with `kick+0xe8 = 7` / `kick+0xec = 2` (ELF `0x22ffa`/`0x23000`).

## 3. Kick-buffer factory `FUN_0001cd50` — partial decode (CONFIRMED facts; kind map UNRESOLVED)

- Signature `(param_1, kind)`; assert `kind > 6` → PVRSRVDebugAssertFail (line 0x362);
  allocates the 0x40-byte object.
- Indirect dispatch confirmed in asm at ELF `0xcdd8`:
  `movl -0x29604(%ebx,%ebp,4),%eax; addl %ebx,%eax; jmpl *%eax`
  (PIC jump table at `%ebx − 0x29604 + 4*kind`; EBX-relative entries unresolved statically).
- Arms: PVRSRVAllocDeviceMem arm with three align variants — 0x20 (`0xcf10`), 0x40 (`0xcde8`),
  0x100 (`0xcf50`, `edi+=0xff; edi&=-0x100`); object stores `+0/+4 = *(devmem)`, `+8 = size`,
  `+0xc/+0x28 = *(devmem+8)`, `+0x24 = size >> 1`, `+0x14..0x20 = 0`, `+0x34/+0x38 = params`;
  common ret arm `0xced3`; error/log arms `0xcf20/0xcf68/0xcf70`.
- GLES2CreateGC consumes `*(obj+0x3c)` (+8 → kick+0x184, +0x20 → kick+0x18c); the
  obj+0x3c writer arm is undecoded (sub-pending).
- INFERRED: the align variants map to buffer classes (PDS program / TA-heap / param buffer).

## 4. SGXKickTA arg3 (`local_c8`, "misc6w") is an OUTPUT buffer (CONFIRMED)

Pre-call asm window ELF `0x358c0..0x3591b` contains no stores to `0x54(%esp)`;
the 6-dword stack buffer is passed **uninitialized** and read after the call
(`local_c8[0] != 0` post-kick check) ⇒ libsrv_um SGXKickTA fills it. Exact field map
requires the libsrv_um side of param_3 (next-round target). Pre-call kick staging
witnessed in the same window: `kick+0x4 |= 0x30000`, `kick+0x0 = *(ctx+0x1b44)`,
`kick+0x24 = seta(cmpl $0x1, 0xb0(%esi))`.

## 5. IMG-table entries [1],[3..11] — bodies classified (all 10, CONFIRMED class labels)

| Entry fn | ELF | Class |
| --- | --- | --- |
| FUN_00033590 | 0x23590 | GL extension-name → internal fn-ptr resolver (strcmp chain: glMapBufferOES / glUnmapBufferOES / glGetBufferPointervOES …, 12.4 KB) |
| FUN_000324f0 | 0x224f0 | GLES2DestroyGC wrapper (deinit + free) |
| FUN_00031bb0 | 0x21bb0 | 5.1 KB op (staging of local_1e4; detail UNRESOLVED) |
| FUN_00031aa0 | 0x21aa0 | default-object unbind sweep (current-context teardown) |
| FUN_00031980 | 0x21980 | bind/unbind helper (default object ctx+0x950) |
| FUN_00034370 | 0x24370 | object-state setter/validator (enum window 0x30b1..0x30b9; FUN_00040a80 lookup; obj+0x84/0x88/0x6c/0x28; FUN_0005cb70 upload) |
| FUN_000318e0 | 0x218e0 | **release/flush path: direct DoKickTA call `FUN_00044f20(ctx, *(ctx+0x950), 0xc)`** + unbind sweep |
| FUN_00031a50 | 0x21a50 | thin binder (delegates to FUN_00031980) |
| FUN_00032560 | 0x22560 | share-group linear scan vs `*(*(ctx+0x2af8)+0x14)` |
| FUN_000317e0 | 0x217e0 | debug vsprintf formatter into ctx+0x70c (gated `*(ctx)&0x800`) |

Net: the ONLY GLESv2→SGXKickTA producer remains `FUN_00044f20` (DoKickTA), with **two
confirmed callers** (per-frame draw flow; release path flags = 0xc).

## Classification / next

- CONFIRMED: §1–§5 facts (hashes + evidence ids in the source report).
- INFERRED: dword_count/type-tag semantics on the TA side; align variants ↔ buffer classes.
- UNRESOLVED: SGXKickTA-side param_3 field map (opens misc6w); obj+0x3c writer arm;
  FUN_00031bb0 identity; kernel-side consumer of two-list third words AFTER PVRSRVLookupHandle
  resolution (firmware endpoint — the CCB copy is seen; the PDS/USSE read-out is not dynamically observable statically).
- NEXT ACTION: REA → libsrv_um: (a) SGXKickTA param_3 store map; (b) `FUN_0002a340`
  "PDS - 1src" builder body (generates the 0x400 block used by SGXCreateTransferContext);
  (c) kick+0x30 template object cross-ref (GLESv2 side might be different name).
