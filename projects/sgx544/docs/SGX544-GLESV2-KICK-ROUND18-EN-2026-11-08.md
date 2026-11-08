# SGX544-reMESA — Round 18 (2026-11-08, English): record-0 w2 tension DISSOLVED — the "per-draw repack" superseded twice (callers are glLinkProgram/glProgramBinaryOES; target is the PROGRAM object, not ctx records); DoKickTA mapped (kick = ctx+0x160; countA = ctx+0x248 init 7; per-kick List-A append = real handle; List-B append = real handle); the kernel chain passes with no H-substitution needed

> NEW incremental file (never overwrites prior files). Prior rounds: `SGX544-*-ROUND11..17-EN-2026-11-0{7,8}.md` in this folder.
> Source of record: `/home/gama/sgx544-re/docs/INVESTIGACAO-LIBGLSLCOMPILER-SYSTEM-P5200.md` (round-18 addendum) + handoff "Update 2026-11-08 (round 18)" (#14).
> Blobs cited: GLESv2 blob `libGLESv2_POWERVR_SGX544_115.so.1.12.2701748` SHA-256 `2d8c70a0e4432f2b9dddc85fe8ae38a0f2421621fa60064ca7e57aa73ef8f91c`; dis `artifacts/glesv2/libGLESv2.dis` SHA-256 `6bd69020efa2e9a8f275ed7d8563d935dc2f691df444fda497239a3b428a466c`; kernel + libsrv_um facts as in rounds 16/17 (unchanged).
> REA evidence (session closed with snapshot `artifacts/rea/libGLESv2-snapshot-r18`, 132,800 B, 8 evidence records): DoKickTA pseudo `ev_e28d360b6933cb165457a933a163d668687225a40fe9b73ee1a3157561d26c82`; wrapper pseudo `ev_433f694e7cac07b1895d8b182ddb337e199a6db9364aa389c745e4c3f06cbdc4`; repack pseudo `ev_e45d8c4512db1ddcf97574926d242c3cddc9d199cf3c66bcd4dc1d8cddd32c5e`; repack callers list `ev_fe4d593e53fd03c3ccbfaa0e6b350c20f2f62110d4334c03d62ed5c8f71fe1d6`; GLES2CreateGC pseudo `ev_3890c77ecb5231f79a81c9c72c83002053fc7110a8fea065853b8f5eac2e4a52`; glLinkProgram pseudo (partial) `ev_75cc1ec0…`; glProgramBinaryOES pseudo (partial) `ev_149f1e6a…`.

## 1. Naming/addressing fixes (supersede prior notes) — CONFIRMED

- **GLESv2 addressing convention: Ghidra VA = ELF address + 0x10000** (e.g. `FUN_00044f20` = ELF 0x34f20; `FUN_000481d0` = ELF 0x381d0; `FUN_000325d0` = ELF 0x225d0=...GLES2CreateGC head window). Reconstructed this round by the dis/decompile cross-check (dis labels show "glScissor+0x…" names because all these unnamed functions sit below the last exported symbol).
- The session-level name **FUN_00044f20 = DoKickTA (Ghidra 0x44f20, ELF 0x34f20)** — CONFIRMED by owned assert strings inside it: `"DoKickTA: KEGLResizeRenderSurface() failed"`, `"DoKickTA: SetupPixelEventProgram() failed"`, `"DoKickTA: Couldn't allocate memory for Z buffer"`, `"DoKickTA: SGXKickTA() failed with error %d"`, plus `"FlushRenderSurface: ScheduleTA did not work properly…"` in the 0x34e20 wrapper (FUN_00034e20 = ELF 0x24e20).
- **Kick struct identity closed: `kick = ctx + 0x160`** — DoKickTA ends with `SGXKickTA(param_1[0x56], param_1 + 0x58, local_c8, 0, 0)` — arg1 = `*(ctx+0x158)` (module/service), **arg2 = ctx+0x160 = the kick struct** (matches every prior mirror identity: kick+0x184 = ctx+0x2e4 ✓, kick+0xe8 = ctx+0x248 = `param_1[0x92]` ✓, kick+0xec = ctx+0x24c = `param_1[0x93]` ✓, List-B family ctx+0x464 +12i = kick+0x304+12i ✓).

## 2. DoKickTA decode (Ghidra 0x44f20, ELF 0x34f20..~0x3613f) — CONFIRMED structure

Signature `DoKickTA(ctx = param_1 (uint*), param_2 = state (int*), flags param_3)`. Flow (pseudo anchors, asm cross-checks in `artifacts/glesv2/libGLESv2.dis`):

1. **Early duplicate-surface flush**: if `param_2[0x7f] != 0 && (flags & 9 || param_2[0x81])` it recursively kicks `param_1[0x6ba]`'s state first (`FUN_00044f20(param_1, param_1[0x6ba], 0)`), with assert-mutex `PVRSRVLockMutex(*(state+0x21c))` when `param_1[0x6ba] != param_2`.
2. **Surface/PPE setup path** (when `*param_2 != 0`): KEGLResizeRenderSurface / SetupPixelEventProgram (`FUN_0006a480`) / Z-buffer allocation via `PVRSRVAllocDeviceMem(ctx[0x57]+4, *(ctx[0x57]+0xd8), 0xb, size, 2, param_2[:9])` — the Z devmem out-struct lands at `param_2+3` (state+0x0c); page-level constants `0x480/0x484/0x488/0x48c/0x490` written into `param_2[0x69..0x72]`; Z `ui32DevAddr` chain kept in `param_2[0x6c..0x72]` (both allocated and static Z paths).
3. **Flag assembly** (uVar13/uVar15): bits 0x1/0x2/0x4/0x8/0x10/0x20/0x80 tested; result field `param_1[0x59] = flagword | 0x30000`, plus `0x20/0x28/0x110/0x2400/0x1400000/0xa00000/0x30000` variants; **hTQ-style marker**: `param_1[0x125] = *(ctx[0x57]+0x3f4)` when the 0xa00-class bits set (TQ sync id — the round-11 chain slot, same `*(queue+0x3f4)`? identity UNRESOLVED at GLES level), and a second identical read for `param_2[0x8e]`.
4. **List-B record append — CONFIRMED** (guard: `if (3 < param_1[0x93]) assert(sgxif.c 0x95c)` — countB rebuild capped at 4, kernel bound ≤ 4 ✓):
   - `param_1[param_1[0x93]*3 + 0x11b] = *(param_2[6] + 0x20)` — **List-B record[countB].w2 = kernel mem handle** (devmem out-struct pattern, +0x20 = `__ptr[8]` = handle per round-15) — asm `movl 0x1b3c(%ebx)…` block 0x355ef..0x3562b staging 0x464/0x468/0x46c/0x470/0x474/0x478 = kick+0x304/0x308/0x30c + 12i family ✓;
   - `param_1[… + 0x119] = *(param_2[6] + 8)` — **w0 = device address** (devmem+0x08 = `__ptr[2]` ✓ round-15); `param_1[… + 0x11a] = param_2[7]` — **w1 = state tag** (`param_2[7]` incremented when `flags & 9`);
   - `param_1[0x93] = param_1[0x93] + 1`; rolled back (`-1`) only on SGXKickTA failure.
5. **List-A record refresh (CBUF tags) — CONFIRMED** (asm 0x35568..0x3558a): for i = 0..6 walk `param_1[0x6c8 + i]` (= ctx+0x1b20+4i, the seven GLES2CreateGC CBUF pointers): **`record[i].w1 = *(cbuf_i + 0x1c)`** stored `0x2e8(%ebx,%ecx,4)` with ecx = i*3 = kick+0x188+12i ✓; count N = number of non-null CBUFs (held in uVar15/ebp).
6. **List-A record append — CONFIRMED** (asm 0x3558c..0x355e9):
   - `countA = *(ctx+0x248)` (0x3558c); guard `cmpl $0x1f, countA; ja` (0x35596/0x35599 — append index must be ≤ 0x1f pre-increment, keeps the post-state ≤ 0x20 = kernel bound; the > side bails to the assert path 0x35bf7);
   - `edx = ctx + 3*countA` (0x355a5/0x355a8) = &record[countA]; `obj = *(ctx+0x4a4)` (0x3559f/0x355bb — the kick stream's devmem out-struct pointer, devmem-0x34-B shape per round-15);
   - **`record[countA].w2 = *(obj + 0x20)`** (0x355b2/0x355b5 — kernel mem handle, `__ptr[8]` ✓);
   - **`record[countA].w0 = *(obj + 8)`** (0x355bb/0x355c8 — device address, `__ptr[2]` ✓);
   - **`record[countA].w1 = *(ctx+0x4a8)`; `*(ctx+0x4a8) += 1`** (0x355ce..0x355dd — per-kick tag counter ✓ round-15 "word1 = per-draw tag", now CONCRETE: monotonic counter at ctx+0x4a8);
   - `countA = countA + 1` (0x355e9).
7. **Kick**: `param_1[0x58/0x59]` (flag word) + SGXKickTA(`param_1[0x56]`, `param_1+0x58` = kick, local_c8 errout, 0, 0); on failure (err ≠ 0): roll back countA/countB/local counters, restore `param_1[0x6cf/0x6d0]` from the previous state, `return 1` with debug print.
8. **Post-kick cleanup — CONFIRMED at pseudo level, asm anchor partial** (asm `movl %edx,0x248(%ebx)` @0x35b92; fail-path decrements `subl $0x1,0x248(%ebx)` @0x35953 / `0x4a8` @0x35946): records `[uVar15 .. old countA − 1]` are zeroed (w2 and w0 slots) and **countA RESET to uVar15 (the refreshed-CBUF count N)** — i.e., the appended per-kick record and any drift above N are consumed by the kernel and the list returns to the long-lived CBUF-seed base. `local_c8[0]` (err/flag out) feeds `param_1[0x128]` when conditions match; `param_2[0x81] = 0`.

## 3. GLES2CreateGC seed count — CONFIRMED (Ghidra FUN_000325d0; ELF 0x22ea4..0x2301b)

- Seven CBUF types (switch on uVar3 = 0..6, jump-table `0x90dbc(,%edi,4)` per asm): common body `FUN_0001cd50(ctx[0x56], uVar3, size_src, *(dev+0xdc), *(dev+0x74), uVar9)` = the CBUF allocator (ELF 0xcd50 ✓ round-16's 0x22f05 call); `CBUF[i] = ctx[0x6c8+i]`-stored (0x1b20+4i ✓).
- For each kind: `dev = *(cbuf + 0x3c)` = the 0x34-B AllocDeviceMem out-struct; **record[i].w2 = `dev[8]`** (= `*(devmem+0x20)` = KERNEL MEM HANDLE ✓ round-15); **record[i].w0 = `dev[2]`** (= `*(devmem+0x08)` = DEVICE ADDRESS ✓ round-15); `*(cbuf+0x10) = dev[0]` (CPU mapping slot); record cursor `local_40 += 3` (12-B stride ✓); **`uVar3++` per kind; at uVar3 == 7 → LAB_00032ffa**.
- Tail: **`countA = ctx+0x248 = 7`** (asm `movl %edi,%edx` @0x22ff8 → `0x248(%ebx)` @0x22ffa, edi = post-loop 7), **`ctx+0x24c = 2`** (=`param_1[0x93]`, countB/other-list marker 2), then `FUN_0003f1b0(ctx)` (ELF 0x2f1b0) and success.
- ⇒ **The GLES2CreateGC-seeded records 0..6 carry REAL kernel handles in w2 at all times** (only w1 is refreshed per kick by DoKickTA; w0/w2 untouched after seed until the post-kick clear, which starts at record N ≥ seed-count, see §2.8 — exact interplay of the zero-range boundary INFERRED, does not affect record 0).

## 4. The "repack" FUN_000481d0 — DOUBLE SUPERSEDE (kills the tension)

- **Callers (REA, `ev_fe4d593e…`): only two — `glProgramBinaryOES`-body (Ghidra 0x4cc80 ≈ FUN within glProgramBinaryOES, assert tag "eurasiacon/opengles2/shader.c" 0x16da) and `glLinkProgram` (FUN_0004d090).** NOT per-draw, NOT kick-path. The round-16 label "per-draw repack" is **REFUTED** (was inferred from FUN_000481d0's content alone).
- **Base-object identity: the stores use the *second regparm arg* (pseudo `in_EDX`, saved in the callee at `0x4c(%esp)`), NOT the context.** Evidence:
  - glProgramBinaryOES pre-call asm (0x3cf5a/0x3cf6c): `movl %esi,%edx; movl %ebx,%eax; calll 0x381d0` with `&(esi+0x2c) = 1` set right before (`movl $0x1, 0x2c(%esi)` @0x3cf5e) — matching the glProgramBinaryOES pseudo `*(iVar6 + 0x2c) = 1` → **EDX = iVar6 = the program object**; EAX = ctx (the lock idiom `PVRSRVLockMutex(*(ctx+0x2af0)+0x10c)` in both the caller pseudo and FUN_000481d0's own head ✓).
  - FUN_000481d0 head asm: `movl %eax, 0x64(%esp)` (arg1 save), `movl %edx, 0x4c(%esp)` (arg2 save); stores at 0x39759/0x39763/0x39769 target `0x2e4/0x2e8/0x2ec(%edx)` with `%edx = 0x4c(%esp)` = **arg2**.
  - ⇒ **the triple store writes program-object +0x2e4/0x2e8/0x2ec — NOT ctx+0x2e4/0x2e8/0x2ec, NOT the kick List-A record-0.** Round-16's "record-0 w2 = ctx+0x2ec overwritten per draw" chain is **REFUTED in both links (who runs it, where it writes)**.
- The "accumulator" story survives recast (no kernel tension): the value `local_548` built in FUN_000481d0 is an **attribute-class bitmask** (`local_548 |= 1 << bit` per attribute, pseudo line ~1073) stored to program-obj+0x2ec; w0-equivalent `(cost<<24)|0x1000/0x1100` and w1-equivalent type-tag pack stored to program-obj+0x2e4/0x2e8. Semantics of the program-object's +0x2e4 family = link-result metadata (UNRESOLVED detail, non-blocking).

## 5. Verdict — the round-16/17 hypothesis family

- **H1 (SGXKickTA composer excludes/re-wires record-0 w2): REFUTED (r17) — and now IRRELEVANT**: nothing needed excluding.
- **H2 (count semantics exclude the tag record): COLLAPSED** — countA is 7+ per kick with real handles at records 0..7; the kernel type-5 walk resolves record 0 legitimately.
- **H3/H4: REFUTED (r17) — and the last support ("accumulator may land in w2") is gone**: the accumulator never lived in a kick record; it lives in the program object.
- **Round-16's GLESv2 reading superseded**: item "(6) GLESv2 record-0 verification … w2 = 0x34(%esp) accumulator … kernel tension" must be reinterpreted as *program-object metadata* (see §4). Everything GLESv2 in that addendum about record/triple offsets at FUN_000481d0 remains valid **as facts of the program-object layout**, not of the ctx.
- **Kernel half unchanged and now CONSISTENT**: SGXDoKickBW resolves all countA records type-5 in place; every record it sees (0..countA−1 = 0..7 at a typical kick) carries a bridge-seeded kernel handle or the per-kick appended handle (§2.6). The FAIL-mode analysis of r17 stands as a guard (what would happen IF a non-handle were present), no longer a live path.

## 6. Classification

- **CONFIRMED**: kick = ctx+0x160; countA/countB fields (ctx+0x248/0x24c, init 7/2); CBUF seed records (w0 = dev-addr = `*(devmem+0x08)`, w2 = handle = `*(devmem+0x20)`, stride 12 B, N = 7); per-kick List-A append (three-word, real handle from `*(ctx+0x4a4)`-object, tag counter ctx+0x4a8, countA guard ≤ 0x1f/pre-state) and List-B append (same devmem pattern from `param_2[6]`, countB cap 4); per-kick CBUF w1-tag refresh; repack callers (glLinkProgram/glProgramBinaryOES only) and repack base = program object (regparm EDX); DoKickTA completion flow + SGXKickTA call args; ELF+0x10000 convention.
- **INFERRED**: post-kick zero-range boundary wording (uVar15 vs countA interplay at record N); countB lifecycle after the 4-cap (the assert is a hard care/hit every kick ≥ 5 — needs one more pass with the pseudo variable states, non-blocking for the chain); semantics of program-obj+0x2e4 family metadata; `param_1[0x124/0x125]` hTQ-slot identity at GLES level.
- **UNRESOLVED**: producer of `ctx+0x4a4` (the per-kick devmem object — presumably the render-target stream's staging; next consumer step ties it to the List-A record ranges); the SGXKickTA bridge-tail (0x14b60..0x155e5) ioctl handoff; `SGXScheduleCCBCommandKM` (ELF 0x2bd40) + `PVRSyncPatchCCBKickSyncInfos` (ELF 0x9400) third-word/sync consumption — the next round's main items; CCB→PDS/USSE/firmware segment.

## 7. Next action (round 19)

(a) Back to pvrsgx.ko: decode `SGXScheduleCCBCommandKM` (ELF 0x2bd40) and `PVRSyncPatchCCBKickSyncInfos` (ELF 0x9400) — how the lists land in the CCB and what the sync patch consumes → closes the kernel→CCB seam; (b) libsrv_um SGXKickTA tail 0x14b60..0x155e5 (KM_arg → ioctl) to bind the user-side object to the bridge; (c) producer of ctx+0x4a4 in GLESv2 (one grep pass over writers of `0x4a4(`), to name the per-kick record source object; (d) countB lifecycle confirm (pseudo post-kick segment only).
