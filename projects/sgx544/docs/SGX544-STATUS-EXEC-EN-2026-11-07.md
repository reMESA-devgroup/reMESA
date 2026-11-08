# SGX544 GPU submission chain — Executive status (2026-11-07, English)

**Scope / method** — Static reverse engineering only (REA/Ghidra 12.x on ELF32 i386 blobs; Ghidra VA = ELF VA + 0x10000, verified per-blob). No proprietary binary executed, no hardware touched, no physical triangle demonstrated. Source of record: `/home/gama/sgx544-re/docs/INVESTIGACAO-LIBGLSLCOMPILER-SYSTEM-P5200.md` (rounds 4-10 addenda, this folder `SGX544-ADDENDA-EN-…`), hashes verified per artifact.

## CONFIRMED
1. **Submission chain GLESv2 -> libsrv_um -> pvrsgx.ko -> CCB is traced end-to-end (static)**:
   - GLESv2 kick init in `GLES2CreateGC` (`FUN_000325d0`): 7 kick-buffer objects via `FUN_0001cd50(hDev, kind 0..6, queue+0xd8/0xd4, queue+0xdc, queue+0x74, size-prm)`, handles at ctx+0x1b20+4i; per-object List-A record pair staged at **kick+0x184 = *(obj+0x3c)+8**, **kick+0x18c = *(obj+0x3c)+0x20**; then **kick+0xe8 = 7, kick+0xec = 2** (`0x22ffa/0x23000`) and `FUN_0003f1b0(ctx)` finalize (evidence `ev_2d68c2a9…`/`ev_cc2f379f…`).
   - GLESv2 imports exactly ONE SGX function: `SGXKickTA` (dynsym UND; single call site ELF `0x3591b`; rel.dyn call+1 = `0x3591c`).
   - `FUN_00044f20` (ELF `0x34f20`) = **DoKickTA + WaitForRender**: signature (ctx, surface, flags); increments `kick+0xe8/0xec` pre-kick, decrements both success AND failure; List-B records staged in place at kick+0x300/0x304/0x308/0x30c (bound <= 3); **hTQ consumed at kick time**: `%ecx = *(queue+0x3f4)` -> `ctx+0x494`, gated by kick-flag bits 0xa00000/0x1400000 (asm `0x3588d-0x358b9`); `ctx+0x4a8` (= KM_arg w3 source via `*(kick->0x338+0x10)`) decremented per kick (asm `0x35946`); timeout -> `PVRSRVClientEvent(0, hDev, 0)` (evidence `ev_ddefa8d4…`).
   - The single submission call: **`SGXKickTA(hDev=ctx[0x56]=queue+4, kick=ctx+0x160, local_c8[6] @ESP+0x54, 0, 0)`** — Ghidra signature `int SGXKickTA(undefined4 *param_1, int *param_2, undefined4 *param_3)` (evidence `ev_64a9a2c6…`; trailing 2 args unused in pseudocode).
   - **libsrv_um `SGXKickTA` = ELF `0x13b40`, size `0x1aa5`** (`llvm-objdump -T`): the 2026-11-05 KM_arg/buf2/bridge dump (payload+8 = KM_arg, buf2+0x18 = 0x0C000000, pair list at buf2+0x51c, ioctl cmd `0xc01c6758` in/out 0x2a0/8) falls INSIDE this export — alias "SGXKickSubmit" superseded.
   - Kernel (pvrsgx.ko, already documented rounds earlier): `SGXDoKickBW` 0x33280 -> `SGXDoKickKM` 0x3d220 (KM_arg = input+8): count A -> CCB+4, records A `KM_arg+0x38+12i` -> `CCB+0x348+8i`; count B -> CCB+8, records B `KM_arg+0x1b8+12i` -> `CCB+0x448+8i`; **third word of each record resolved in place by `SGXDoKickBW` via `PVRSRVLookupHandle` type 5**; sync area `CCB+0x48..` from `KM_arg+0x1f4/+0x1f8`; scheduler `0x3bd60`/`0x3bae0` copy `KM_arg+0x00..0x1f` to the 0x20-byte HW slot (w0/w1 overwritten by `*(dev+0xa0)`/device field).
2. **hTQ chain closed end-to-end**: gralloc.clovertrail (.so ctor `hal_init` -> `OpenPVRServices` -> `SGXCreateTransferContext(hDev 0x125d8, 0x20000, devmemctx, out=HMI+0x1e0=ELF 0x126a0)`); IMGegl `SRV_SGXServicesInit` -> `hw_get_module("gralloc")` -> `(*(module+0x110))(module, &queue[0xfd])` = **`gralloc_module_gettransfercontext`** (gralloc ELF `0xa0c0`: guard `FUN_0001b980` ctor flag, mutex HMI+0x114, `*out = *(HMI+0x1e0)`); failure clears queue[0xfd] and skips `FUN_000183e0` (PDS fragment buffer list — success-gated). Teardown `hal_exit` -> `SGXDestroyTransferContext(0x126a0, 0)`.
3. **obj338/vistest**: normal-path producer `SGXCreateRenderContext(queue+4, queue+0x3f8(cfg), &queue+0x3ec, &queue+0x3f0)`; recovery producer `eglAwakenProcessIMG` (0xcee0) same offsets; helper `FUN_00028ef0` = `SGXSetContextPriority(queue+4, ver, *(queue+0x3ec), *(queue+0x3f4))`.
4. **Layout/template writer** `FUN_00044060` (ELF `0x34060`) fills `*(ctx+0x1ae8)`: {+0x174:0x23c, +0x178:0xa2, +0x17c:0x400, +0x184:0x414, +0x188:0xf0000, +0x18c:0x41c, **+0x190:0x1e3ce508**, +0x194:0x420, **+0x198:0x1e3ce508**, +0x19c:0x424, +0x1a0:0xe9(+0x1cc..:0x4bc,0x300,0x4b8,**0x3f800000**,0xa5c,0xa60,0xa64)} — small-offset layout descriptors, semantics pending cross-ref (evidence `ev_6e6b0ee2…`).

## INFERED (working hypotheses)
- `kick+0xe8/0xec` are per-kick occupancy tallies (increment/decrement around each SGXKickTA), not record counts; the <=0x20/<=4 asserts cap them.
- `FUN_00044060` block = TA/3D packet-layout template; `0x1e3ce508` = packet state magic.
- The 7 kick-buffer objects map to TA/3D/VGP/PDS roles by kind (needs `FUN_0001cd50` semantics).

## UNRESOLVED (next targets, priority order)
1. **Firmware-consumed addresses**: extract the CCB/PDS/USSE template trove `DAT_000498c0` (libsrv_um, ELF .rodata 0x398c0; sole code ref `addl $0x398c0, %ebp` at ELF `0x1b329`) and cross-ref with `0x1e3ce508`.
2. Consumer of the List A/B third words post-`PVRSRVLookupHandle` resolution (kernel passes nothing of it to CCB; firmware consumption unknown).
3. Per-frame List-A staging producer at GLESv2 ELF `0x39759-0x39769` span (glGetActiveAttrib gap) — function unnamed.
4. `local_c8` (SGXKickTA arg2) 6-word field map; surface fields 0x4e/0x4f/0x90..0x97 identity (Z + flip).
5. `FUN_0001cd50` kind semantics; `FUN_0003f1b0` finalize; GLESv2 IMG-table entries [1],[3..11] (Ghidra funcs collected at handoff round-10).

## TRIANGLE READINESS (static evidence only)
- Submission plumbing and driver surface: closed end-to-end on paper (init -> context -> kick -> CCB -> kernel).
- Missing for a physical triangle: (a) firmware-side command template knowledge (`DAT_000498c0` trove + PDS/USSE templ from GLESv2-americans), which owns actual TA_PDS@/USE_SETUP/kick register words; (b) render-target/PB allocation wiring (surface struct fields above); (c) execution + framebuffer validation on the physical device (explicitly out of scope until separately authorized).

## NEXT ACTION
Open libsrv_um in REA/Ghidra (already the active target as of this note): decompile the trove consumer containing ELF `0x1b329` (Ghidra `0x2b329`), extract `DAT_000498c0` byte layout, correlate with `FUN_00044060` constants and the Nov-5 buf2 pair-list deltas (0xc0..0xd4 family).
