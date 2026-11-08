# SGX544-reMESA — Round 16 (2026-11-08, English): kernel-side two-list third-word verification CLOSED — SGXDoKickBW is the resolver (supersedes round-10/15 attribution); SGXDoKickKM copies only words 0/1; pvrsgx.ko addressing convention Ghidra = ELF + 0x10020; GLESv2 record-0 word2 = per-draw accumulator (kernel-validation tension sharpened, not closed)

> NEW incremental file (never overwrites prior files). Prior rounds: `SGX544-*-ROUND11..15-EN-2026-11-07.md` in this folder.
> Source of record: `/home/gama/sgx544-re/docs/INVESTIGACAO-LIBGLSLCOMPILER-SYSTEM-P5200.md` (round-16 addendum) + handoff "Update 2026-11-08 (round 16)".
> Blobs cited: `pvrsgx.ko` SHA-256 `aa0320a5ba477d5e4d394f74f1f16e71fc1a91921f170f768ff267422923072d`; kernel dis `artifacts/dis/pvrsgx_ko.dis` (54,046 lines, SHA-256 `fd6d3bea71ca1cdd…`); GLESv2 blob SHA-256 `2d8c70a0e4432f2b9dddc85fe8ae38a0f2421621fa60064ca7e57aa73ef8f91c`; GLESv2 dis `artifacts/glesv2/libGLESv2.dis` SHA-256 `6bd69020efa2e9a8f275ed7d8563d935dc2f691df444fda497239a3b428a466c`.

## 1. pvrsgx.ko addressing convention — CONFIRMED (supersedes the prior "+0x10000" note)

Ghidra image VA = ELF symbol/file address + **0x10020** for this module (Ghidra loads base 0x10000 with a +0x20 first-section skew). Checks made while switching targets:

| check | result |
| --- | --- |
| REA procedure query at Ghidra `0x3d200` | resolves to `SGXWaitClocks` (evidence `ev_e513e2bf…`) ⇒ SGXDoKickKM is NOT at 0x3d200 |
| ELF symtab `SGXDoKickKM` @ 0x2d200 (size 993) ⇒ Ghidra 0x3d220 | decompile at 0x3d220 = `int SGXDoKickKM(void)` (evidence `ev_c14e3ce6d…` = `ev_c14e3ce64d25f1a5bfee8187189ab93e7f5444e5926dff7b9416146077b5aac1`) |
| "SGXDoKickBW" Ghidra 0x33280 ⇒ ELF 0x23260 | unnamed static function (mcount window [0x2326f..0x23598)); owned .rodata string `%s: SGXDoKickBW: Invalid pasDstSyncHandles pointer` (evidence `ev_08c75ad535f817b22520f440a266511ae9db7108d4202ea46da1cf40fa8dd716`) |

All Nov-6 kernel-round addresses (0x33280 / 0x3d220 etc.) were therefore Ghidra-space all along; ELF conversions below use ELF offsets directly.

## 2. PVRSRVLookupHandle census + attribution — CONFIRMED (readelf relocations + mcount-call windows)

- **Census**: 136 × `R_386_PC32` sites against `PVRSRVLookupHandle` (ELF 0x13c80); **zero** relocations against `PVRSRVLookupHandleAnyType`; exactly one absolute `R_386_32` at ELF 0x26157 (function-pointer store inside `SYSPVRFillCallback`, ELF 0x260c0 + 0x201).
- **Attribution method**: `R_386_PC32` relocation offset = call site + 1; relocations against `mcount` (761 total) mark function starts (mcount-prologue idiom), so every lookup call site is attributed by the enclosing mcount window. Symtab sizes proved unreliable at cluster boundaries (e.g., `DummyBW` 0x1e280 + 0x66 does not contain 0x1e2f3/0x1eeec/…; `BridgedDispatchKM` 0x22850 + 0x658 does not contain 0x23086/0x242c9/0x2498f).
- Attribution result (relocation addr = call+1):

| sites (ELF) | enclosing function (ELF base + size, or mcount window) |
| --- | --- |
| 0x53a7 | `PVRSRVRelease` (0x5370 + 0x308) |
| 0x5641, 0x5804 | `PVRSRV_BridgeDispatchKM` (0x55f0 + 0x974) |
| 0x18a3a, 0x18bdc | `PVRSRVGetMiscInfoKM` (0x18720 + 0x1460) |
| 0x26157 (R_386_32) | `SYSPVRFillCallback` (0x260c0 + 0x201) |
| 0x2c79d / 0x2ca8d / 0x2ce7d | `SGXRegisterHWRenderContextKM` (0x2c6c0+0x612) / `SGXRegisterHWTransferContextKM` (0x2c9b0+0x612) / `SGXRegisterHW2DContextKM` (0x2cda0+0x612) |
| dense cluster 0x1e2f3..0x24ee1 | unnamed local (static) functions; key mcount windows: [0x2326f..0x23598) = SGXDoKickBW wrapper; [0x23859..0x24229) = 19-lookup-site helper (loops over `*(ebx+0xd70/0xd74/0xd78/0xd7c)` and `0xe0c(%ebx,%i,4)` type-5 checks — ownership/context validation INFERRED) |
| — | **No PVRSRVLookupHandle site falls inside SGXDoKickKM (0x2d200..0x2d5e1)** ← decisive negative |

- No relocation hits against 0x23850 (grep on `00023850` empty) — direct caller of the 19-site helper not resolvable via relocations; caller attribution pending (documented search failure).

## 3. SGXDoKickKM decode — CONFIRMED (ELF 0x2d200..0x2d5e1, 993 B, evidence `ev_c14e3ce6…`)

- Validity: `*(ccb+0xc) > 0x467` AND `*(KM_arg+0x1f0) <= *(ccb+0xc) − 0x468`, else string `"SGXDoKickKM: Invalid CCB offset"` (exit 3). Means: the CCB carries a 0x468-byte per-kick control region; KM_arg+0x1f0 = offset of the new kick block.
- `slot = *(ccb+0) + *(KM_arg+0x1f0)` — per-kick slot base inside the CCB command stream.
- **List A copy loop (0x2d2f0..0x2d314)**: records at `KM_arg+0x38+12i` (strided 0xc), count = `*(KM_arg+0x30)` → `slot+4`; **copies words 0 and 1 ONLY** into `CCB+0x348+8i` / `CCB+0x34c+8i` (8-byte stride in the CCB stream). **The third word (record+8) is NEVER copied.**
- **List B loop (0x2d32b…)**: records at `KM_arg+0x1b8+12i`, count = `*(KM_arg+0x34)` → `slot+8`; words 0/1 → `CCB+0x448+8i` / `CCB+0x44c+8i`. Third word never copied.
- Sync-object staging: `KM_arg+0x27c/0x280/0x284` → slot+0x34 / +0x10 / +0x0c / +0x14 / +0x28 (+0x24) with counter increments; `PVRSyncPatchCCBKickSyncInfos(slot+0x48, KM_arg+0x1f4, KM_arg+0x1f8)` called at 0x2d361 (reloc → ELF 0x9400).
- Kick-present flag: `slot+0 |= 1` at 0x2d4b4; global `ui64KickCount` updated (absolute R_386_32 pair at 0x2d42c/0x2d432).
- Scheduling: `SGXScheduleCCBCommandKM` (ELF 0x2bd40) call at 0x2d4dc/0x2d4e0 with args {EAX = original Arg1, EDX = 0, ECX = KM_arg, stack = 0xffffffff, 0, 0, `*(KM_arg+0x1ec)`}; failure paths use `PVRSRVDebugPrintf` / `warn_slowpath_null` (0x2d5bd..0x2d5d8).

## 4. SGXDoKickBW decode — CONFIRMED (ELF 0x23260..0x23554; Ghidra 0x33280; evidence `ev_08c75ad5…`)

Struct: BW receives EDX = S with **KM_arg = S + 8** (cross-checks: S+0x284 = KM_arg+0x27c sync field ✓; S+0x38 = KM_arg+0x30 count A ✓; S+0x48 = KM_arg+0x40 = record-0 third word ✓).

- Validation chain: local lookups (type 2 @0x23292, type 5 @0x232c8 into IPCC-context-adjacent fields), then **type-6 in-place lookups on S+0x284/0x288/0x28c (= KM_arg+0x27c/0x280/0x284)** @0x232ed/0x23312/0x2333e. (REA pseudo renders `PVRSRVLookupHandle(2)/(5)/(6)` with `*in_ECX =` stores — a regparm-argument artifact; the type constants are carried by the stack slot of the actual calls — split INFERRED for the exact out-param semantics.)
- Bounds (failure ⇒ `*(err)=3`): **count A ≤ 0x20** at 0x23360 (`cmpl $0x20,%eax; ja`, count loaded from S+0x38 @0x2335d) and **count B ≤ 4** at 0x233df (count loaded from S+0x3c @0x233dc); the truncated REA pseudo shows an additional `< 0x21` conjunction on S+0x1fc (= KM_arg+0x1f4, the sync-patch count) — conjunction framing INFERRED.
- **List A loop (0x2338e..0x233b3): for i < countA — `PVRSRVLookupHandle` with EAX = `*(psProc+0x10)` = handle table base, EDX = `S+0x48+12i` (= &record[i].word2), ECX = `*` that address (the third-word value), stack type = 5 — resolves/validates record+8 IN PLACE** (per-iteration pointer built as `0x48 + 4*(3i)` @0x23391–0x23394; stride +0xc per iteration).
- **List B loop (0x23406..0x23431): third words at `KM_arg+0x1c0+12i` (= record[0].w2 of List B) resolved in place, type 5.**
- `pasDstSyncHandles`: count `*(KM_arg+0x24)`, user pointer `*(KM_arg+0x28)`; after `OSAccessOK`/`OSAllocMem_Impl`/`OSCopyFromUser(count*4)`, the kernel copy replaces the KM_arg+0x2c flag (store at 0x234af: BW+0x34 = KM_arg+0x2c), and **per-element type-6 in-place resolution @0x234c0..0x234e2**; final single type-5 in-place lookup on KM_arg+0x28 (0x23518..0x2352a). Failure string `%s: SGXDoKickBW: Invalid pasDstSyncHandles pointer`.
- **`call SGXDoKickKM` at 0x2353d (reloc 0x2353e) — the ONLY relocation against SGXDoKickKM in the whole module ⇒ sole caller CONFIRMED.**

## 5. Supersedences (explicit)

1. **Round-10/15 attribution corrected**: the third-word resolution happens in **SGXDoKickBW** (before the kick), NOT inside SGXDoKickKM. Round-15 List-A table row "word 2 → resolved in place via PVRSRVLookupHandle type 5 [by SGXDoKickKM] → CCB+0x348+8i" must now read: word 2 is resolved **in the kernel-side KM_arg copy by SGXDoKickBW**, and the CCB stream copy (SGXDoKickKM) transfers **only words 0/1**.
2. Consequence for round-15's "firmware consumes GPU addresses through List-A/B third words": **CONFIRMED only for word 0** (device address streamed to CCB+0x348/0x448+8i). The resolved third word is a kernel-internal object pointer (handle-table resolution) that is **not** streamed into the CCB command area; whether the firmware consumes it through the slot/sync structures is **UNRESOLVED** (candidate consumer: `SGXScheduleCCBCommandKM` body — pending).

## 6. GLESv2 record-0 verification (asm, `artifacts/glesv2/libGLESv2.dis` windows 0x39400..0x397b0) — writes CONFIRMED; kernel-name tension sharpened

- Triple store confirm (0x39759/0x39763/0x39769; base `%edx = 0x4c(%esp)` = ctx, record 0 = kick+0x184/0x188/0x18c = ctx+0x2e4/0x2e8/0x2ec):
  - `[ctx+0x2e4] = (esi<<24) | 0x1000/0x1100` (0x39750/0x39753; the 0x1100 variant chosen when flag bit 0x2000 set, 0x39745/0x39748);
  - `[ctx+0x2e8] = ebp` — bit-packed per-attribute type tags (`orl 3<<cl / 5<<cl / 7<<cl` sites 0x396d3–0x396ee);
  - **`[ctx+0x2ec] = 0x34(%esp)`** (0x3975f load, 0x39769 store).
- `0x34(%esp)` flow CONFIRMED (ELF-assembly truth): initialized `movl $0x4, 0x34(%esp)` @0x39535; per-attribute `addl %esi, 0x34(%esp)` @0x395c4 (per-type cost 3/4, plus the attribute-size table entries `0x2c(%ebp,%i,4)`); used as a shift amount via `movzbl 0x34(%esp),%ecx` @0x3958d for the 64-bit bit-pack pair `0x28(%esp)/0x2c(%esp)`. Other flows reuse the slot (init 0/1/ edi; cursor `+0x18` @0x38e67; counter loop 0x3936d/0x3937b; `orl` @0x39438; `$4` @0x39535) — stack-slot shared. On the store path the content is an **accumulated small value, NOT a resource handle** (this closes the round-12 "pseudocode vs asm" question about `local_548`: word C of record 0 = accumulator; round-12's "precision-qualifier<3 mask" label stays INFERRED).
- **Kernel tension (sharpened, not closed)**: the kernel resolves EVERY List-A third word (i < countA) via PVRSRVLookupHandle type 5 and aborts the kick on failure (failure-early pattern of the head validations); yet GLESv2 per-draw overwrites record 0's w2 (ctx+0x2ec = kick+0x18c) with that accumulator — the same slot GLES2CreateGC seeded with `*(devmem+0x20)` (kernel handle, round-15 CONFIRMED). How the assembled KM_arg passes the type-5 check for record 0 is **UNRESOLVED**. Hypotheses: (H1) the `SGXKickTA` KM_arg composer excludes record 0 or re-wires its w2 (candidate — check the Nov-5 SGXKickTA staging dump); (H2) count A semantics exclude the tag record from the mirrored list; (H3) the accumulator value is legislatively a handle in limited configurations (weak).
- Supporting new facts (CONFIRMED): **ctx+0x18c (= kick+0x2c) is a realloc-managed attribute-source array**, not a handle slot: `PVRSRVUnlockMutex(*(x+0x10c))` 0x382b9 (reloc), `realloc(old kick+0x2c, 4 * *(*(ctx+0x128)+0x10))` 0x382ee (reloc 0x382ef = `realloc`) → result stored to kick+0x2c (0x382ff), `memcpy(new, *(*(ctx+0x128))+0x10, size)` 0x38319; cleanup arms `free(old)` then `kick+0x2c = 0` (0x3902c..0x3903e / 0x3913d..0x3915c; relocs = `free` @0x39036/0x3914e/0x3916a). This disambiguates every fixed `0x18c(%reg)` hit inside FUN_000481d0 from the 12-byte record slots (`kick+0x184+12i` family).
- Candidate span for the backlog (glCopyTexSubImage2D symbol gap, ~0x48170..0x48290): a struct builder storing `+0x184(%ebp)` (`%edx = 0x68(%esp)`), `+0x188 = 0`, `+0x18c = *(arg field + 0x1c)` plus constants (+0x4=9, +0xf0=1, +0x108=3, +0x16c=1, +0x170..0x17c, +0x180, +0x198=1, +0x1a4, +0x1a8, +0x1d0). Possibly a second command-object builder (identity UNRESOLVED; queue with the List-B producer sweep).

## 7. Classification / next

- **CONFIRMED**: addressing convention (+0x10020); lookup census + mcount attribution; SGXDoKickKM body incl. words-0/1-only copy, CCB offsets, sync patch, schedule args; SGXDoKickBW validation/bounds/in-place type-5/6 resolution chains + sole caller; GLESv2 record-0 triple stores and the accumulator value flow (0x34(%esp)); kick+0x2c realloc array identity; free-cleanup arms.
- **INFERRED**: helper-cluster 0x23850 semantics (ownership/context checks); out-param write semantics of PVRSRVLookupHandle (regparm EDX store); the `< 0x21` conjunction framing on KM_arg+0x1f4; H1/H2 record-0 exclusion hypotheses.
- **UNRESOLVED**: consumer of the resolved third words beyond validation (SGXScheduleCCBCommandKM body, ELF 0x2bd40); the KM_arg composer inside `SGXKickTA` (libsrv_um 0x13b40) — which kick offsets land at KM_arg+0x38+12i words — the key to close the record-0 tension; caller of the 0x23850 helper (relocation search returned empty); identity of the 0x48xxx builder span.
- **NEXT ACTION**: (1) reopen libsrv_um (snapshot `artifacts/rea/libsrv_um-ghidra-snapshot-r15`) and decode the SGXKickTA KM_arg composer call sites kick+0x184/0x188/0x18c (asm 0x13c68..0x14b4a windows) → closes H1/H2; (2) decode `SGXScheduleCCBCommandKM` (ELF 0x2bd40) and `PVRSyncPatchCCBKickSyncInfos` (ELF 0x9400) in pvrsgx.ko for third-word/sync consumption detail; (3) triangle-readiness checklist §4 refresh (word-2 supersede).

Session note: pvrsgx.ko REA analysis closed this round with snapshot `artifacts/rea/pvrsgx-ghidra-snapshot-r16` (analysis metadata only; original blob preserved, SHA unchanged above).
