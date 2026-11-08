# SGX544-reMESA — Round 14 (2026-11-07, English): kick-buffer factory = CBUF_CreateBuffer; List-A seeding chain closed

> NEW incremental file. Prior rounds: the 2026-11-07 files above in this folder.
> Source of record: `/home/gama/sgx544-re/docs/INVESTIGACAO-LIBGLSLCOMPILER-SYSTEM-P5200.md`
> (round-14 addendum) + handoff round-14 block.

## 1. Factory identity + PIC resolution (CONFIRMED)

`FUN_0001cd50` (GLESv2) = **CBUF_CreateBuffer wrapper**, source file
`eurasiacon/common/buffers.c`. PIC base pinned: EBX = `_GLOBAL_OFFSET_TABLE_` =
`.got.plt` VA **0xb6ff4** (readelf: `.got` 0xb6ff0 / `.got.plt` 0xb6ff4 size 0x5c;
note .got/.got.plt file-offset delta +0x1000 while .text/.rodata are VA==offset).

- String cross-checks (all land in .rodata 0x8c800..0xa3dc1):
  `-0x2a7d7(%ebx)` → 0x8c81d `"eurasiacon/common/buffers.c"`;
  `-0x29760/-0x2970c(%ebx)` → 0x8d894/0x8d8e8 `"CBUF_CreateBuffer: Requested buffer type"`;
  `-0x2968c(%ebx)` → 0x8d968 `"CBUF_CreateBuffer: DeviceMem alloc of 0x…"`.
- Jump table at `EBX − 0x29604` = ELF .rodata **0x8d9f0** (signed EBX-relative entries):

| kind | target (ELF VA) | arm |
| --- | --- | --- |
| 0 | 0xcf50 | align **0x100** arm |
| 1, 2 | 0xcde8 | align **0x40** arm |
| 3, 4, 5, 6 | 0xcf10 | align **0x20** arm |
| 7, 8 | 0xcf20 / 0xcf68 | error arms (matches assert kind ≤ 6) |

## 2. CBUF object layout (0x40 B; CONFIRMED stores)

Two `PVRSRVAllocDeviceMem` calls per CBUF:

1. **Main buffer** (size 0x40-aligned/0x20-aligned/0x100-rounded per kind): object
   `+0x00` = devmem result struct ptr, `+0x04` = `*(devmem)` CPU VA, `+0x08` = size,
   `+0x0c`/`+0x28` = `*(devmem+8)` device address, `+0x14..0x20/+0x2c/+0x30` = 0,
   `+0x24` = `size >> 1`, `+0x34/+0x38` = call params.
2. **`obj+0x3c` writer**: second `PVRSRVAllocDeviceMem` (class **0xf**, size **4**,
   align **4**, out = `obj+0x3c`), then `PVRSRVMemSet(*(obj+0x3c), 0, 4)` — a 4-byte
   zeroed device slot per CBUF.

## 3. GLES2CreateGC List-A seeding loop (CONFIRMED; 0x22ecd..0x22f5c)

Per CBUF i: handle → `0x1b20(%ebx,%edi,4)`; `%edx = *(CBUF+0x3c)`;
- **`kick+0x184+12i = *(devmem+8)`** (GPU address of the 4-B slot; `0x22f2f`);
- **`kick+0x18c+12i = *(devmem+0x20)`** (devmem field; `0x22f26`);
- `CBUF+0x10 = *(devmem)` (CPU VA; `0x22f37`);
- kick cursor += **0xc** per CBUF (`0x22f4e`) — List-A = 12-byte records;
- special kinds: 5 → 0x23434, 6 → 0x2342b; switch table `jmpl *0x90dbc(,%edi,4)`
  (absolute, .rodata) with per-kind size selects from `ctx+0x1b98 andl $-0x20` /
  `ctx+0x1b94` and heap por `0xd4(%eax)`.

## 4. Firmware-consumed addresses (List-A) — chain now evidence-backed end-to-end

init (7 CBUFs seed records) → per-draw repack of record 0 by `FUN_000481d0` (round 12) →
libsrv_um SGXKickTA mirrors records into KM_arg `+0x38+12i` / `+0x1b8+12i` →
kernel SGXDoKickKM copies to `CCB+0x348+8i` / `CCB+0x448+8i`, resolving the third word via
`PVRSRVLookupHandle` (type 5) → **CCB holds GPU addresses of CBUF device memory that the
PDS/USSE firmware reads**. The per-draw refresh covers record 0 only; records 1..6 keep the
CBUF-seeded values (refresh path for >1 draws UNRESOLVED).

## Classification / next

- CONFIRMED: §1–§3 facts (dis + strings + readelf; blob `2d8c70a0…`).
- INFERRED: CBUF kind ↔ GPU-role naming (align classes only known); `*(devmem+0x20)` = handle.
- UNRESOLVED: special-kind arms (0x23430..0x23460 + absolute table 0x90dbc entries —
  statically readable, next step); writers of ctx+0x1b88..0x1b9c size fields; List-B
  unchanged (kick+0x300..0x30c, ≤3, third words same resolution path).
- NEXT ACTION: table 0x90dbc dword dump + arms; ctx+0x1b8x writer sweep; kernel-side
  recheck of the third-word copy at `CCB+0x348+8i` (pvrsgx.ko).
