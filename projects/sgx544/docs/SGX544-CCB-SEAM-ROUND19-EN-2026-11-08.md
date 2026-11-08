# SGX544-reMESA — Round 19 (2026-11-08, English): kernel→CCB seam CLOSED — SGXScheduleCCBCommand worker body (32-B CCB slots, command table, kick count, MISR) + PVRSyncPatchCCBKickSyncInfos full binding (FD array at KM_arg+0x1f8, descriptor rows at slot+0x48, third word NEVER reaches the persistent slot or the CCB stream) — UM SI hash shows ui32CallerID convention (−1 bridged UM / −3 MISR)

> NEW incremental file (never overwrites prior files). Prior rounds: `SGX544-*-ROUND11..18-EN-2026-11-0{7,8}.md` in this folder.
> Source of record: `/home/gama/sgx544-re/docs/INVESTIGACAO-LIBGLSLCOMPILER-SYSTEM-P5200.md` (round-19 addendum) + handoff "Update 2026-11-08 (round 19)" (#15).
> Blob: `pvrsgx.ko` SHA-256 `aa0320a5ba477d5e4d394f74f1f16e71fc1a91921f170f768ff267422923072d`; dis `artifacts/dis/pvrsgx_ko.dis` (54,046 lines, SHA-256 `fd6d3bea…`). REA evidence this round: wrapper pseudo `ev_dd151b23…`; worker pseudo `ev_52a09f6d…` (Ghidra 0x3bae0); wrapper callees `ev_5c742349…`; sync patch pseudo `ev_5c9825b6…`; Expander pseudo `ev_9c784372…`; session closed with snapshot `artifacts/rea/pvrsgx-ghidra-snapshot-r19`.

## 1. The wrapper chain and the worker's address — CONFIRMED (relocs + dis)

- Worker call inside `SGXScheduleCCBCommandKM` (ELF 0x2bd40): `calll` @**0x2be7a**, reloc `0002be7a → **SGXScheduleCCBCommand**`, addenda **0x2bac0** ⇒ **worker = ELF 0x2bac0 = Ghidra 0x3bae0**. (Round-19 planning note that guessed 0x3bae0 = PVRSRVSetDevicePowerStateKM is **SUPERSEDED**: 0x3bae0 IS the worker; PVRSRVSetDevicePowerStateKM = Ghidra 0x26b20 (ELF 0x16ae0, reloc 0002bd7c), PVRSRVPowerLock = Ghidra 0x26a10 (ELF 0x169f0, reloc 0002bd63).)
- Wrapper prologue: `mcount` @0x2bd50; EAX/EDX/ECX preserved across `PVRSRVPowerLock` pattern confirmed (`movl %eax,%ebx`, `movl %edx,-0x10(%ebp)`, `movl %ecx,%edi`).
- `SGXScheduleProcessQueuesKM` (ELF **0x2be60**) calls the wrapper @0x2bf51 with **stack arg 0 = 0xfffffffd** — WHICH IS EXACTLY −3. So the two callerID values binding the wrapper: **−1 (0xffffffff) = bridged UM kick** (SGXDoKickKM @0x2d4d1) and **−3 (0xfffffffd) = kernel MISR** (process-queues path). The wrapper's `param_1 == -3` arm (`cmpl $-0x3,%esi` @0x2be18) therefore fires from the MISR path: `*(EAX+0x8c) = 1; OSScheduleMISR(); return 0` (reloc 0002be34 `OSScheduleMISR`).
- MISR payload: the 8-dword local array at `-0x24(%ebp)` zeroed (0x2bef6..0x2bf01 loop), `EDX = 7` (command **type 7**), `ECX = &local array`, `EAX = dev node` — so MISR schedules a **type-7 command** with an all-zero payload; the type-7 path in the worker skips the cache-flush class ({1,2}∪(0 & callerID≠0) — 7 is neither) and still bumps rd/events/kick-count. Semantics of type 7 beyond scheduling (device "idle/lockup polling"?) — INFERRED, non-blocking.

## 2. SGXScheduleCCBCommand worker body — CONFIRMED (Ghidra 0x3bae0 pseudo `ev_52a09f6d…` + asm ELF 0x2bac0..0x2bd3f)

Argument carriers (regparm-3 + stack, mcount-preserving):
| carrier | content |
| --- | --- |
| EAX (→ −0x2c… saved via 0xb8(%eax)) | **device node ptr** (per-queue "psDevNode"-like); `dev_data = *(EAX+0xb8)` |
| ECX (−0x14(%ebp)) | **8-dword command payload base** = the KM_arg words 0..7 from SGXDoKickKM |
| EDX (−0x1c(%ebp)) | **command type** (0 = kick from DoKickKM; 7 = MISR path) |
| stack[0] | **ui32CallerID**: −1 (bridged UM) / −3 (MISR); used ONLY in the cache-flush condition |
| stack[1..3] | param_2..4 passthrough (not consumed in this body) |

Body (ELF asm anchors):
1. `dev_data = *(EAX+0xb8)` (@0x2bad0); `event = *(dev_data+0xdfc)` (@0x2bad9, saved −0x20); `CCB = *(dev_data+0x48)` (@0x2bae2, EBX); `dev_data` saved −0x18.
2. **Timeout loop**: `OSClockus` @0x2baeb; free-space test @0x2bb1d..0x2bb32: `rd = *(*(CCB+0xc))`, `lim = *(*(CCB+0x10))`, `(rd + 1) & 0xff != lim` → free; else sleep (@0x2bb3a, 0.5 s budget `cmpl $0x7a11f` = 499,999 µs with wrap-aware arithmetic) → **"Wait for CCB space timed out"** (string 0xc4f0) + **return 9** (@0x2bce4).
3. **Slot address**: `slot = *(CCB+0x8) + (rd << 5)` (@0x2bb60..0x2bb65) ⇒ **CCB entry = 32 bytes (8 dwords), ring indexed by rd, wrap mask 0xff** ⇒ ring ≤ 256×32 B in this CCB view. Non-NULL check else timeout err 9.
4. **Payload copy**: `payload[1] = *(dev_data+0xa0); *(dev_data+0xa0) = 0` (@0x2bb75/0x2bb84 — a one-shot stashed dword consumed into dword 1, then cleared) then full 8-dword copy `slot[0..7] = ECX[0..7]` (@0x2bb91..0x2bbbc, unrolled)". ⇒ **the CCB kick command is EXACTLY the first 8 dwords of KM_arg** (words beyond 0x1c — including countA/countB at +0x30/+0x34 and the List A/B records — are NOT in the CCB command).
5. **Type validate** (`cmpl $0xa,type; jg` @0x2bb71/0x2bbbf — AFTER the copy): ≥ 0xa ⇒ "SGXScheduleCCBCommand: Unknown command type: %d" (0xc524) + **return 0x69**.
6. **Data-cache op condition** (@0x2bbc5..0x2bd3a): `type ∈ {1,2}` always, or `type == 0 && callerID != 0` (`movl 0x14(%ebp),%edx; test; jne` @0x2bbd4 — bridged UM = 0xffffffff qualifies; MISR type-7 doesn't); operation keyed on `*(gpsSysData+0xdc)`: 2 → OSFlushCPUCacheKM (@0x2bcf1), 1 → OSCleanCPUCacheKM (@0x2bd00); reset `*(gpsSysData+0xdc) = 0` (@0x2bcad). gpsSysData==NULL → syscommon.h:0xdc assert (`warn_slowpath_null`) and continue.
7. **Command word + counters** (@0x2bbdf..0x2bc0c): `slot[0] = *(dev_data + 0x60 + type*4)` — **per-type command code table at dev_data+0x60**; `rd_store = (rd + 1) & 0xff` written back @0x2bbf3/0x2bbf9; and the **mirror counter** `*(*(dev_data+0x58)) = +1 & 0xff` (@0x2bbfb..0x2bc0c — dev-side copy kept in sync).
8. **Event + doorbell** (@0x2bc0e..0x2bc2d): `*( *(dev_data+0xdfc) + 0x24 ) |= 2`; `lock`-prefixed barrier pairs around **`OSWriteHWReg(*(dev_data+0x14), 0x8ac8, 1)`** (@0x2bc27 — register 0x8ac8 = kick doorbell, value 1, dev-reg base from dev_data+0x14).
9. **Kick count**: 64-bit global **`ui64KickCount`** (symtab name; abs32 relocs 0002bc33/0002bc39 at the LSW/MSW) incremented `LSW+1; MSW+carry` (@0x2bc31/0x2bc38). return 0.

## 3. SGXDoKickKM (ELF 0x2d200..0x2d5e1) — kickoff/staging full re-read — CONFIRMED (supersedes/completes round-16 row)

- **Sanity gate** @0x2d216..0x2d22d: `A = *(KM_arg+0x20)`; `*(A+0xc) > 0x467` AND `*(KM_arg+0x1f0) <= *(A+0xc) − 0x468` — i.e. the per-kick staging **offset at KM_arg+0x1f0 must fit with 0x468 bytes to spare** inside the block anchored at `*( *(KM_arg+0x20) )`. **0x468 = exactly the full staging extent** (see §4): layout fits [0x348, 0x448) = List A (32×8 B) + [0x448, 0x468) = List B (4×8 B) ✓✓. Failure → debug print (0xce08) + return 3.
- `slot = *( *(KM_arg+0x20) ) + *(KM_arg+0x1f0)` (@0x2d220/0x2d260/0x2d266) — a **per-kick staging block inside a persistent buffer**; this is where r16's "CCB+0x348/0x34c+8i" writes actually land (base = slot, not a CCB struct).
- **Sync-object stage** @0x2d260..0x2d2dd (three KM_arg objects): +0x27c → slot+0x34 = **(slot+0x38) = *(obj+4)**, slot+0x34 = **(obj) / *(obj)** (deref of the obj's field-0), refcount `*( *(KM_arg+0x278) → obj+0x0? — *( *(KM_arg+0x278) ) obj … += 1` on the object field-0; +0x280 → slot+0x10/0x14/0x0c + refcount `*(*(obj)+8) += 1`; +0x284 → slot+0x20/0x28/0x24 + same bump. These are the RESOLVED sync objects (from r16/r17 handle resolution). Any NULL → skipped branch; abort path releases the corresponding refcounts again (@0x2d36a..0x2d3a2, incl. the kernel-owner object at **KM_arg+0x27c using field-0 refcount**, +0x280/+0x284 using field+8 refcount).
- **List A stage** @0x2d2eb..0x2d314: count = `*(KM_arg+0x30)` → `slot+4`; loop base `KM_arg+0x38+12i`: **`slot+0x348+8i = record.w0; slot+0x348+8i+4 = record.w1`** — ONLY words 0/1; **record.w2 is never copied to slot** (already consumed by the r17 in-place type-5 resolution inside SGXDoKickBW).
- **List B stage** @0x2d318..0x2d34e: count `*(KM_arg+0x34)` → `slot+8`; pairs from `KM_arg+0x1b8+12i` → `slot+0x448+8i` (:w0) / `slot+0x44c+8i` (:w1).
- **Sync fence patch call** @0x2d352..0x2d361: `EBX = KM_arg; ECX = KM_arg+0x1f4; EDX = slot+0x48; EAX = KM_arg+0x1f8; call PVRSyncPatchCCBKickSyncInfos` (r19 earlier evidence ev_5c9825b6… binding re-verified in this turn's asm window 0x2d350..0x2d368).
- **Post-patch** @0x2d3d8..0x2d4ae: `slot+0x44 = *(KM_arg+0x1f4)` (patched/expanded fence count = now synced count); **descriptor-row loop** keyed on `*(KM_arg+0x24)` count / `*(KM_arg+0x2c)` table / capacity check `needed = count*0x18 + 0x20 ≤ *(*(KM_arg+0x28)+0xc)` (breaks to assert 0xce5c @line 0x140 but CONTINUES); per-row into the device-object block `*( *(KM_arg+0x28) )` (pitch 0x18, fields at +0x8..+0x1c): **`row.0x8 = *(obj+8)`, `row.0xc = *(obj)`, `row.0x10 = *( *(obj) )`** (value copied before the +1 write... refcount-era detail), `row.0x14 = *(obj+4)`, `row.0x18 = *( *(obj) + 0x10 )`, `row.0x1c = *(obj+0xc)`, **`row.0x20 = ui64KickCount.LSW`, `row.0x24 = ui64KickCount.MSW`** (@0x2d42a/0x2d430, reloc 0002d42c `ui64KickCount`) — each descriptor row is stamped with the **global kick serial number**; zero-object rows zeroed (@0x2d482..0x2d4a5).
- **Kick flag + Schedule** @0x2d4b4..0x2d4e0: `slot+0 |= 1`; `SGXScheduleCCBCommandKM(EAX = dev node (saved @0x2d213), EDX = 0 (type kick), ECX = KM_arg (payload), stack {0xffffffff, 0, 0, *(KM_arg+0x1ec)})` — reloc `0002d4e0 → SGXScheduleCCBCommandKM` ✓.
- **Return handling**: == 0x18 → **release loop** (stash items `*(KM_arg+0x1f8+4i)` doubly-decremented (`subl $1, 8(%esi)` + `subl $1, 8(%ecxInstall)`), descriptor entries `*(KM_arg+0x2c)[i]` doubly-decremented, objects 0x27c/0x280/0x284 released) and return 0x18 (device was powered-down/deferred); == 0 → return 0; else debug print (0xcf08) → release path → return error.

## 4. Slot (per-kick staging block) layout — CONFIRMED/INFERRED mix

| interval | content | status |
| --- | --- | --- |
| +0x00 | kick flag word (`| 1` @0x2d4b4) | CONFIRMED |
| +0x04 / +0x08 | List A count / List B count | CONFIRMED (round 16 + 19) |
| +0x0c, +0x10, +0x14 | sync object A fields (from KM_arg+0x280) | CONFIRMED |
| +0x1c, +0x20, +0x24, +0x28 | sync object C fields (from KM_arg+0x284) | CONFIRMED |
| +0x34, +0x38 | sync object B fields (from KM_arg+0x27c) | CONFIRMED |
| +0x44 | expanded fence count (post-patch `*(KM_arg+0x1f4)`) | CONFIRMED |
| +0x48+24*i (i<count) | fence descriptor rows, 6 dwords each (written by PVRSyncPatchCCBKickSyncInfos; each row's +0 dword written at `−4(%edx)` — the refcount-read slot) | CONFIRMED (patch asm 0x94b6..0x9500) |
| +0x348+8*i | List A: (record.w0, record.w1) pairs — countA ≤ 32 ⇒ extent 0x100 | CONFIRMED |
| +0x448+8*i | List B: (record.w0, record.w1) pairs — countB ≤ 4 ⇒ extent 0x20 | CONFIRMED |
| total extent | 0x468 (= the sanity-gate constant) | CONFIRMED (structural fit) |
| +0x20/+0x24 of the descriptor rows also nomenclature at the `*(KM_arg+0x28)` block | kickcount stamps — separate device-sync-obj rows | CONFIRMED (different container from the patch rows) |

## 5. PVRSyncPatchCCBKickSyncInfos (ELF 0x9400, 279 B) — full asm decode — CONFIRMED

Entry: mcount @0x940c; carriers saved: EAX → −0x11c (= KM_arg+0x1f8), EDX → −0x114 (= slot+0x48), ECX → −0x118 (= KM_arg+0x1f4).
1. Zero-fill a 32-dword local array (edi = −0x8c, `rep stosl` ×0x20 = −0x8c..−0x0c) @0x9417..0x9430.
2. Single call @0x9464 → **ELF 0x7d20** (static, unnamed in symtab): the expander/collect helper. Binding at that call: **regparm-3 (EAX = *(KM_arg+0x1f4) = input sync count; EDX = KM_arg+0x1f8 = the FD array; ECX = 0x20 = capacity 32)** + stack {zeroed array (Expander's `0x8(%ebp)`), count-out ptr (−0x110 → Expander's `0xc(%ebp)`), dedup-unique array ptr (−0x10c → Expander's `0x10(%ebp)`/EBX)}. Default error EBX = 0x9a.
3. On success (ret != 0): rows loop @0x9498..0x9500: `rowptr = slot+0x48+4`, for i < *count-out:
   - `row[0] = *(item+8)`, `row[+8] = *(item+4)`, `row[+0x10] = *(item+0xc)`,
   - `row[+4] = *( *(item) )` (deref of the item's object field-0),
   - `row[+0xc] = *( *(item) + 0x10 )`,
   - `row[−4] = *( *(item) + 8 )` — the refcount READ slot (`−4(%edx)` = the row's own dword 0 because rowptr advances by 0x18 per row after an initial +0x4),
   - refcount `*( *(item) + 8 ) += 1` @0x94f0,
   - **stash: `*(KM_arg+0x1f8 + 4i) = item`** (overwrites the consumed FD int with the item pointer for later release),
   - count → `*(KM_arg+0x1f4)` @0x9510.
   Return 0; on expander failure return 0x9a.
4. Layout detail: the rows written occupy `[slot+0x48+24i … +24i+0x18)` in the code layout (write pointer starts at slot+0x48+4, increments 0x18/row, and the first dword of each row is written at ptr−4) — 6 dwords/row, 24 B pitch, matching the r19 evidence rows statement exactly.

## 6. ELF 0x7d20 — the fence expander/collect helper — CONFIRMED (static, unnamed; decompiled round-19 as `FUN_00017d40`/"ExpandAndDeDuplicateFenceSyncs" `ev_9c784372…`)

- regparm-3 + 3 stack args: **EAX = input count; EDX = FD list base; ECX = capacity 32**; stack: array-A out (the zeroed 32-dword block; `A[i] = fence obj` per fd), count2 out (`0xc(%ebp)`), dedup-unique array (`0x10(%ebp)`→EBX).
- Walk: for i < count: `fd = fdlist[i]` (@0x7d6b `movl (%edx),%eax`); **skip negative fds** (`js`) — matching the owned pseudo's "entries ≥ 0" clause; **`sync_fence_fdget(fd)`** (reloc `00007d82` ✓) @0x7d81; NULL → error path @0x7fc9.
- `fence->list_head` at **+0x28** (list, `esi = fence+0x28`; empty ⇒ skip = local fence with no inbound timeline? / non-PVR hybrid).
- Member check `*(list_entry − 0x20) + 0x4 == 0x160` (@0x7da0/0x7dbb — **the fence object probing for `0x160`** — the PVR fence class marker used by the earlier evidence's `PTR_s_pvr_sync_00040dc0` comparison) — 0x160 ⇒ walk the fence's wait-list directly (0x7db8..0x7dcc); ≠ 0x160 ⇒ **foreign-sync path** @0x7e58: `kmalloc_caches`-based `kmem_cache_alloc_trace` (reloc `00007e75` ✓, first alloc size 0x14, second alloc via 0x7ea1 → 0xc bytes), a PVRSRVAllocSyncInfoKM-class call (@0x7ead, consts 0x808/0x80c globals), tagging `$0x7c60` (rodata — likely "pvr_sync" fence-op/stable string), and **`sync_fence_wait_async(fence, ...)`**-class call @0x7f21.
- Dedup loop @0x7dd0..0x7e11 — compares (array item +4) fields against the currently-walked list member to prevent duplicates; appends unique items to the EBX (array) with count increments at count2 (@0x7e4e / @0x7f4a). Early exit when the dedup count reaches the capacity (−0x24) check @0x7e40.

## 7. Third-word consumption — FINAL VERDICT — CONFIRMED now at all three ends

1. **In**: SGXDoKickBW in-place type-5 resolution (r17) — the handle-value third word is replaced by the **resolved kernel object pointer** in the KM_arg records.
2. **Through**: SGXDoKickKM staging — copies record w0/w1 to slot+0x348/+0x448 pairs; **record.w2 is NOT copied to slot** (r16 evidence refined: the never-copied word is the resolved object pointer, only read via the sync-object field chains at +0x27c/+0x280/+0x284 and the fence descriptor chain — those appear in the kick struct as a **kernel object pointer family**, not as CCB data).
3. **Out**: SGXScheduleCCBCommand worker — the CCB command is EXACTLY 8 dwords from KM_arg+0x00..0x1c; nothing from the slots/records structure enters the CCB stream given it was never in the payload words.

⇒ The kernel-resolved/patched back-end and the CCB 8-dword command are two INDEPENDENT planes: (a) the CCB plane is the UM-composed 8-word command (whose words 2..6 are the UM composer's payload, not yet UM-side decoded — next round's composer/bridge-tail window); (b) the slot plane carries device-addr/tag pairs and fence descriptor rows for the kernel/device visibility to consume (contents propagated via the block the UM handed through KM_arg+0x20/0x1f0 maps). No third-word data reaches the CCB — refuting any persistent suspicion that the third words are injected into the CCB command contents.

## 8. Classification

- **CONFIRMED**: worker ELF 0x2bac0 body end-to-end (all 9 mechanics above with asm anchors); the worker call/binding; MISR -3 (0xfffffffd) path with type-7 zero-payload scheduling; SGXDoKickKM's sanity gates (0x468), slot computation, sync halos 0x27c/0x280/0x284 staging, List A/B pair staging, sync patch call binding, post-patch hashable stamps (ui64KickCount into the `(*(KM_arg+0x28))` block rows) and 0x18→release path; PVRSyncPatchCCBKickSyncInfos full asm (incl. stash overwrite semantics and one refcount bump per item); Elf 0x7d20 expander walk incl. `sync_fence_fdget` reloc + `kmem_cache_alloc_trace` + 0x160-member check; the "third word never enters the CCB stream" verdict at three independent ends; wrapper's −1/−3 callerID convention (−3 = MISR, bridged UM −1).
- **INFERRED**: the exact UM-side meaning of the CCB payload words 2..7 (composer output — not decoded this round); the semantic name of the type-7 MISR command; the coordination between the two 24-B row regions (`slot+0x48` patch rows and the `*(KM_arg+0x28)` kickcount-stamped rows) — separate containers CONFIRMED but no shared view shown; the `*(dev_data+0xa0)` one-shot dword semantics; ring limit being also wrap-managed by the consumer side.
- **UNRESOLVED**: SGXKickTA bridge tail (libsrv_um 0x14b60..0x155e5) and its KM_arg→ioctl handoff (payload words 2..7 origin); GLESv2 writers of `0x4a4(` (per-kick record source object cuent identity, the `param_2[6]` List-B source object); countB post-cap lifecycle pseudo pass; **CCB→PDS/USSE/firmware segment** (the big remaining triangle-readiness gap).

## 9. Next action (round 20)

(a) libsrv_um `SGXKickTA` bridge tail 0x14b60..0x155e5 — decode the KM_arg→ioctl handoff, capture exactly which payload words (KM_arg+0x00..0x1c) and pointer-vs-device-addr fields the UM fills (bind the 8-word CCB command to its composer); (b) GLESv2 sweep for writers of `0x4a4(` to name the per-kick List-A record source object; (c) optional: countB post-cap pseudo pass; (d) open the CCB→PDS/USSE question with the doorbell reg 0x8ac8 trail (OSWriteHWReg constant, dev_data+0x14 mapping) and the dev_data+0x60 command table as entry points into the device-visible side.
