# SGX544-reMESA — Round 17 (2026-11-08, English): SGXKickTA KM_arg composer copies ALL THREE WORDS of every record (record 0 included, accumulator w2 verbatim) — H1/H3/H4 REFUTED; PVRSRVLookupHandle body fully decoded (mcount-preserved regparm ABI; bucket walk by handle class/byte; err arms 0x95/0x96/0x97); H2 (count semantics exclude the tag record) is the only viable escape — round-18 = GLESv2 kick driver FUN_00044f20

> NEW incremental file (never overwrites prior files). Prior rounds: `SGX544-*-ROUND11..16-EN-2026-11-0{7,8}.md` in this folder.
> Source of record: `/home/gama/sgx544-re/docs/INVESTIGACAO-LIBGLSLCOMPILER-SYSTEM-P5200.md` (round-17 addendum) + handoff "Update 2026-11-08 (round 17)" (#13).
> Blobs cited: `pvrsgx.ko` SHA-256 `aa0320a5ba477d5e4d394f74f1f16e71fc1a91921f170f768ff267422923072d`; kernel dis `artifacts/dis/pvrsgx_ko.dis` (54,046 lines, SHA-256 `fd6d3bea71ca1cdd…`); libsrv_um blob SHA-256 `ccafe90d…` + dis `artifacts/dis/libsrv_um.dis`; GLESv2 blob SHA-256 `2d8c70a0…` + dis `artifacts/glesv2/libGLESv2.dis` SHA-256 `6bd69020…`.

## 1. SGXKickTA KM_arg composer — CONFIRMED (libsrv_um, asm windows 0x14a70..0x14b60)

`SGXKickTA` (ELF 0x13b40..0x155e5) composes its kick argument block **directly in place on its own stack**, and the layout offsets line up exactly with the kernel-side `KM_arg` mirror (both prior rounds' offsets reproduce on this side):

| stack address (SGXKickTA frame) | content | kernel-side identity (BW/KM reads) |
| --- | --- | --- |
| `esp+0xb0` | countA ← `*(kick+0xe8)` (load 0x14a87, store 0x14a99) | `KM_arg+0x30` ⇒ **KM_arg = esp+0x80** |
| `esp+0xb4` | countB ← `*(kick+0xec)` (load 0x14a72 via 0x48(%esp), store 0x14aa0) | `KM_arg+0x34` |
| `esp+0xb8/0xbc/0xc0` | List A buffer, record base esp+0xb8 + 12i (w2 aliases at esp+0xc0 = base+8) | `KM_arg+0x38+12i` (words 0/1/2); record-0 w2 = `KM_arg+0x40` ✓ (BW reads S+0x48 with KM_arg = S+8) |
| `esp+0x238/0x23c/0x240` | List B buffer, record base esp+0x238 + 12i | `KM_arg+0x1b8+12i` ✓ |

- **Count staging — CONFIRMED**: `movl 0xe8(%ebx),%edx` @0x14a87 → `movl %edx,0xb0(%esp)` @0x14a99 (countA); `movl 0xec(%ebx),%edx` @0x14a72 (held via `0x48(%esp)` @0x14a7b) → `movl %edi,0xb4(%esp)` @0x14aa0 (countB). **NEW fact: user-side countB ≤ 4 guard** `cmpl $0x4,%edx; ja 0x1544d` @0x14a78/0x14a7f — mirrors the kernel-side bound (SGXDoKickBW @0x233df, round 16). No analogous inline guard seen for countA in this window (kernel enforces ≤ 0x20 @0x23360).
- **List A composer loop (0x14ad0..0x14af6) — CONFIRMED, triple copy**: `esi = 0x184(%ebx,%ecx)`, `edi = 0x188(%ebx,%ecx)` (kick-relative record base +12i, EBX = kick struct), `(%edx,%ecx) = esi` (EDX = esp+0xb8), `esi = 0x18c(%ebx,%ecx)`, `0x4(%edx,%ecx) = edi`, `(%eax,%ecx) = esi` (EAX = esp+0xc0) — **copies words 0, 1 AND 2 for every record**, stride `0xc(%ecx)` @0x14aef, bound = `countA*12` (`leal (%edx,%edx,2); shll $0x2` @0x14ab0/0x14ac0 held at `0x40(%esp)`). Loop entered **unconditionally whenever countA ≠ 0** (`testl %edx,%edx; je 0x14afc` @0x14a97/0x14aae) — there is no index skip, no rewriting of word 2: **record 0 is included with its w2 verbatim.**
- **List B composer loop (0x14b28..0x14b4e) — CONFIRMED, same pattern**: `0x304/0x308/0x30c(%ebx,%eax)` → `(%edx,%eax)`/`0x4(%edx,%eax)`/`(%ecx,%eax)` with EDX = esp+0x238, ECX = esp+0x240, bound = countB*12 @0x14b20; skipped only when countB == 0 (@0x14b02).
- Side staging: `movl %eax,0xa8(%esp)` @0x14aa7 (setne flag from `testb $0x5,%al` @0x14a85 — a kick-flags bit test feeding a later field; exact consumer not yet traced — pended).

**Verdict on the round-16 hypotheses (this closes the composer half of the tension)**:
- **H1 (SGXKickTA composer excludes record 0 or re-wires w2) — REFUTED.** The composer copies w2 of record 0 (and every record) verbatim into the KM_arg block.
- **H3 (accumulator value coincidentally a valid handle) — REFUTED as structural**: see §2 — the lookup is a bucket-walk by (h>>8, h&0xff) into real kernel allocation nodes seeded by the bridge; a small accumulated int cannot name an allocation node (round-15: real handles come from the AllocDeviceMem bridge out-struct `*(devmem+0x20)`).
- **H2 (count semantics exclude the tag record) — REMAINS the only viable escape** at the kernel boundary: when countA == 0 the List A copy is skipped entirely (@0x14aa7 `0xa8(%esp)` flag path aside, `je 0x14afc` @0x14aae), the kernel List-A loop runs 0 iterations (loop test confirmed below), and record 0's w2 is never type-5-looked-up. Mechanism-dependent variant pending round 18: what `*(kick+0xe8)` (countA) actually holds at kick time, given GLESv2 seeds ≥ 7 CBUF records at init.
- Observation (flagged, INFERRED-hypothesis to verify in round 18): when countA == 0 the consumer slots esp+0xb8.. (⇒ KM_arg+0x38..) are left as **stack residue** — they are only valid records if the driver pre-staged them; if GLESv2 stages a nonzero countA without re-staging records, the kernel would look up residue. The safe design implies countA==0 must pair with no List A at all.

## 2. PVRSRVLookupHandle (ELF 0x13c80, 398 B) — body FULLY decoded — CONFIRMED (supersedes the round-16 regparm-artifact framing)

Instrumentation note: the entry call at 0x13c8f is **`mcount`** (reloc 00013c90 → `mcount`), and the Linux kernel mcount trampoline **preserves EAX/EDX** — which dissolves the round-16 "regparm artifact" hedge: the caller's EAX/EDX survive to EBX/EDI (`movl %eax,%ebx` @0x13c97, `movl %edx,%edi` @0x13c99). Real argument contract (regparm-3 + stack):

| carrier | content (verified at both ends) |
| --- | --- |
| EAX → EBX | **handle-table base** (kernel side passes `movl 0x10(%esi),%eax` = `*(psProc+0x10)` @0x23398); table meta fields used: `0x1c(%ebx)` (range/count @0x13ca3), `0x8(%ebx)` (@0x13ca9), tables indexed `(%eax,%ebx,4)` @0x13cc5 |
| EDX → EDI | **out-ptr** where the RESOLVED kernel object pointer is written (success store `movl %eax,(%edi)` @0x13e00) |
| ECX | **handle value** (kernel passes `movl (%edi),%ecx` = record w2 @0x233a2); internally used as `type = *(table+0x1c)` bound check ECX−1 (0x13ca6/0x13cac) AND as bucket id: class = `h>>8` (0x13cb4/0x13cb6, stride-12 table select 0x13cb9), slot = `(h&0xff)` (0x13cbc) × **0x30 B per row** (0x13cbf/0x13cc2) |
| `0x8(%ebp)` → ESI | first stack arg = **requested type/role word** (kernel pads `movl $0x5,(%esp)` @0x2339b in the List-A loop; $2/$6 at the head/in-path sites); compared against **node[0]** `cmpl %eax,%esi` @0x13cd6 (node[0] loaded @0x13cc8) |

- Bucket walk — CONFIRMED: node base = `table[(h>>8)] + (h&0xff)*0x30`; `node[0]` = type/handle-class word; `node[1]` (`0x4(%edx)`, @0x13df8) = **the resolved kernel object/data pointer** delivered through the out-ptr.
- **Error arms — CONFIRMED** (each: stack-arg debug set, `PVRSRVDebugPrintf` with `.rodata` level-4/.str1.1 strings, unwind EBX): **0x95** @0x13d84 (string `0x496c`, EBX ← $0x95) — range/existence origin; **0x96** @0x13d1c (string `0x49a8`) — **node == NULL** (`testl %eax,%eax; je 0x13d18` @0x13cc8/0x13ccc → "handle not allocated/pending"); **0x97** @0x13ce2 (string `0x49e0`) — **requested type ≠ node[0]** "not this channel/type". All arms converge on the common return tail @0x13d45 (second print with string `0x4ab8`, @0x13d49) returning **EBX** in EAX (`movl %ebx,%eax` @0x13d6d).
- **Success path — CONFIRMED**: `movl 0x4(%edx),%eax` @0x13df8, `xorl %ebx,%ebx` @0x13dfb (err 0), `movl %eax,(%edi)` @0x13e00 (**`*out = node[1]` — the resolved object pointer is written in place**), return 0.
- **NULL-type side path — CONFIRMED** (0x13c9d `testl %esi,%esi; je 0x13db8`): type word NULL → debug-print + `warn_slowpath_null` arm (@0x13dbb..0x13dea, reloc `warn_slowpath_null` @0x13dea) that **re-enters the flow at 0x13ca3** (i.e., continues, not an error return); degenerate/incomplete-request path.
- Caller-side post-call store (BW List-A loop) — CONFIRMED as instruction, semantics **INFERRED**: `movl %eax,(%ecx)` @0x233ae with ECX = saved loop-cursor EDI (saved @0x23373, stride `addl $0xc,%edi` @0x23386), executed BEFORE the `testl %eax,%eax` @0x233ac — i.e. the call's return/status word is also written into the record-slot family; whether the visible final record w2 keeps the internal resolved store (via preserved EDX out-ptr) or the caller's store of the return value is the last refinement gap (**INFERRED/UNRESOLVED-level detail**; does not change verdicts below). Success continues the loop (`je 0x23380` @0x233b0); failure arms the abort path (@0x233b2).
- **Kernel List A loop bounds — CONFIRMED (corrects the round-16 word-check table)**: loop test **@0x23380/0x23389** `movl -0x18(%ebp),%edx; addl $0x1,%ebx; addl $0xc,%edi; cmpl %ebx,0x38(%edx); jbe 0x233d8` — i counts 0..countA−1, stride +0xc per record, EDX = list/struct base. The body @0x2338e..0x233b2 (builder `leal 0x48(%ecx,%eax,4)` = &record[i].w2, table base, type 5, handle, call) runs **record 0 first** — record 0 is ALWAYS in the resolved set whenever countA ≥ 1.

## 3. Consequence chain (kernel boundary) — CONFIRMED as a whole

1. GLESv2 per-draw repack overwrites record-0 w2 with the small accumulator (round 16, stores @0x39759/0x39763/0x39769 — `ctx+0x2e4/0x2e8/0x2ec` = kick+0x184/0x188/0x18c).
2. SGXKickTA copies that accumulator **verbatim** into `KM_arg+0x40` (composer §1).
3. SGXDoKickBW type-5-looks-up record-0 w2 (first loop iteration, counts from 0) with the real contrast: the accumulator is not a kernel allocation handle ⇒ bucket walk fails (`err 0x96` node-NULL arm is the expected miss mode for a small int that passes the ≤ max range check; `0x95` if beyond max) ⇒ **the kick ABORTS at the kernel boundary** (`test eax; jne` → bail @0x233b2).
4. Therefore, IF record-0 w2 = accumulator at kick time, a hardware kick cannot pass. The remaining escape candidates: **H2** — the driver-side count staging (`*(kick+0xe8)`) or per-draw record staging is such that the accumulator-bearing record never enters the resolved window (countA == 0 skips List A entirely; or the tag record is not counted in countA); to be CLOSED ONLY from the GLESv2 side, round 18: decode the kick driver **FUN_00044f20 (DoKickTA, GLESv2 ELF 0x34f20)** — what sits at kick+0xe8 and whether record-0 w2 at kick time is the handle seeded by GLES2CreateGC (round-15: `*(devmem+0x20)`) or the accumulator, including which of the two flows reaches the kick struct per context.
5. **H4 (a late GLESv2 store converts w2 to a real handle) — REFUTED**: fresh grep evidence — **zero** sites touching `0x188(`/`0x18c(` in the GLESv2 span 0x22000..0x23460 (`artifacts/glesv2/libGLESv2.dis`, SHA `6bd69020…`), i.e. no conversion consumer exists in the init/kick-composing region.

## 4. Classification

- **CONFIRMED**: composer layout/in-place mapping counts+records (both lists, triple copy, unconditional for count>0); user countB ≤ 4 guard; KW List-A loop bounds (0x23380/0x23389, record 0 first); PVRSRVLookupHandle full ABI + bucket walk + err arms 0x95/0x96/0x97 + success `*out = node[1]` @0x13e00 + mcount-preserving-EAX/EDX identity (reloc 00013c90); H1/H3/H4 refutation evidence.
- **INFERRED**: exact final w2 value after the BW caller-side store @0x233ae (interplay with @0x13e00); the residue hypothesis for countA==0; the `0xa8(%esp)` setne flag consumer; "type 0 = any/degenerate" role of the stack word.
- **UNRESOLVED**: countA actual value at kick time (GLESv2 kick driver FUN_00044f20 — round 18 assignment); why record-0 w2 = accumulator can ever pass (H2 variant); SGXKickTA bridge handoff tail (0x14b60..0x155e5) which moves the stack KM_arg into the ioctl; consumer of the resolved third words inside SGXScheduleCCBCommandKM (ELF 0x2bd40).

## 5. Next action (round 18)

Open GLESv2 for FUN_00044f20 (DoKickTA, ELF 0x34f20): (a) countA staging into kick+0xe8; (b) per-draw record staging into kick+0x184+12i (who writes what for w0/w1/w2 — handle seed vs accumulator); (c) whether the accumulator store flow (@0x39759/0x39769 repack) is on the same code path as the kick driver or a sibling usage of the same object; then kernel side: PVRSRVLookupHandle success-path final-word refine only if still needed, and SGXScheduleCCBCommandKM (0x2bd40) for the resolved-word consumption.
