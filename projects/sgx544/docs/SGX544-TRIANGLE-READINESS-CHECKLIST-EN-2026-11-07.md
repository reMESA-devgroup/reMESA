# SGX544-reMESA — Triangle readiness checklist (2026-11-07, English; static-evidence state)

> Scope: smallest technically valid path to ONE hardware-accelerated triangle on SGX544,
> expressed as the 7 checklist stages of the session goal. No physical execution has been
> performed or claimed. Every item cites the current evidence class (CONFIRMED / INFERRED /
> UNRESOLVED) against the source repo (`/home/gama/sgx544-re/docs/…`, rounds 1–13).
> SGX544 command formats are used ONLY from SGX544 evidence; nothing assumes SGX535 interchangeability.

## 1. GPU init + memory mappings — ⛔ UNRESOLVED at the kernel/UM-install layer (evidenced endpoints exist)

- CONFIRMED (user-mode): service open chain `dlopen gralloc → hal_init → OpenPVRServices →
  SRV_SGXServicesInit`; device context `SGXCreateTransferContext(hDev, 0x20000, devmemctx,
  out=HMI+0x1e0)` (rounds 8–9); render context `SGXCreateRenderContext(queue+4, queue+0x3f8,
  &queue+0x3ec, &queue+0x3f0)` + `SGXSetContextPriority` (round 8).
- CONFIRMED (kernel, prior rounds): pvrsgx.ko ioctls incl. `0xc01c6758` (SGXKickTA bridge in/out 0x2a0/8);
  `SGXDoKickBW 0x33280 → SGXDoKickKM 0x3d220`.
- UNRESOLVED: full memory-map install (gap fill, PAT/PF layout, CCB alloc via `FUN_0003bd80
  (param_1, 0x10000, 0x40, 0x400, …)` semantics — round 11), device-mem class table
  (classes 0x4/0xb/0xd/0xf/0x4f etc. observed; full mapping UNRESOLVED).

## 2. Vertex + fragment shader preparation — partly CONFIRMED

- CONFIRMED: shader objects / hash table (`opengles2/shader.c` unlink `FUN_0004c050`), shader
  binary memcpy sources `*(*(ctx+0x128)+0x10)` / `*(*(ctx+0x1c4)+0x10)` inside the per-draw
  packer `FUN_000481d0` (round 12); GLSL→USSE offline compiler interface = separate research axis
  (report main body, `libglslcompiler.so`/USC).
- UNRESOLVED: exact upload of compiled PDS+USSE segments into REGION-type device memory
  ánh per-triangle (which object field carries the program and where the firmware fetches it).

## 3. Vertex buffer + render-target allocation — partly CONFIRMED

- CONFIRMED: 0x40-byte object factory `FUN_0001cd50` with three PVRSRVAllocDeviceMem align
  arms (0x20/0x40/0x100) + `size>>1` store at obj+0x24 (round 12); transfer-buffer allocator
  `FUN_0002a340` (0x130-byte descriptor, name-tagged) (round 13); Z-buffer slots
  `surface[0x4e]/[0x4f]` (round 10); Fence ID 4-B devmem.
- UNRESOLVED: render-target (color) allocation wiring — surface fields 0x90..0x97 identity
  (round 10 wait-loop fields); obj+0x3c writer arm of the factory.

## 4. TA + 3D command construction — ✅ CONFIRMED end-to-end (static)

- Kicks: `FUN_00044f20` DoKickTA (ELF 0x34f20) is the SOLE GLESv2→SGXKickTA producer; two callers
  (per-frame draw flow; release/flush flags 0xc via FUN_000318e0) (round 12).
- Kick object: `kick = ctx+0x160`; init `kick+0xe8=7, kick+0xec=2` (0x22ffa/0x23000);
  per-kick counts ±; List-A record triple `kick+0x184/0x188/0x18c` = per-draw packed attribute
  descriptor {(dword_count<<24)|0x1000/0x1100, 3-bit-per-attr TYPE TAGS, precision mask} produced
  by `FUN_000481d0` (round 12); List-B staged in-place kick+0x300..0x30c (≤3 bound) (round 10).
- hTQ: producer-consumer chain closed rounds 8–9; consumed at kick time into `ctx+0x494`.
- libsrv_um: `SGXKickTA` export ELF 0x13b40 (0x1aa5 B) builds bridge payload + KM_arg (Nov-5 map);
  `misc6w` back-channel decoded (round 13).
- Kernel: count A → CCB+4, records A → `CCB+0x348+8i`; count B → CCB+8, records B → `CCB+0x448+8i`;
  third words resolved in-kernel by `PVRSRVLookupHandle` type 5 (Nov/round-10 evidence).

## 5. Kernel submission — ✅ CONFIRMED (ioctl level)

- `SGXKickTA` → ioctl `0xc01c6758` (in 0x2a0/out 8) on the param_1 connection (Nov-5).
- UNRESOLVED beyond static: firmware-side scheduling nuance (PDS/USSE loader entry points
  per kick — statically INFERRABLE only from the template trove).

## 6. Completion + retirement — partly CONFIRMED

- CONFIRMED: WaitForRender loop inside `FUN_00044f20` (`"WaitForRender: Timeout"`,
  `PVRSRVClientEvent(0, hDev, 0)` on timeout; force-flip workaround); kick-return flags;
  `misc6w[0]` back-channel (round 13); counter decrements on success AND failure (round 10).
- UNRESOLVED: full sync/retire object lifecycle (sync area `CCB+0x48..` from `KM_arg+0x1f4/+0x1f8`
  mapped; retirement queue/day semantics beyond the wait-loop UNRESOLVED).

## 7. Framebuffer readback + pixel validation — UNRESOLVED (planned, not evidenced)

- Static prerequisites identified: surface flip fields 0x4e/0x4f/0x90..0x97; gralloc TQ path
  (SGXQueueTransfer) closed for the copy-back direction (round 9).
- Pixel validation requires physical execution — explicitly out of scope until separately authorized.

## Verdict

Stages 4–5 (command construction + submission) are CLOSED on static evidence. The leading
blockers for a physical triangle are: (a) shader/program upload object identity (§2);
(b) render-target wiring fields (§3); (c) init/CCB gap semantics `FUN_0003bd80` + device-mem
class map (§1); (d) firmware consumption endpoints (static limit — requires either the
template trove cross-ref completed kernel-side or a hardware-authorized experiment).
NEXT ACTION (round-14): `FUN_00026200` + kick-tracking object identity; GLESv2 factory
PIC resolution; then checklist re-run.
