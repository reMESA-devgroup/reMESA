# SGX535 beta checkpoint — November 13, 2026

**0.0.9-beta.0 is a development target, not a released, usable accelerated
Desktop/OpenGL driver.** Normal application admission remains closed. This is
a documentation-only checkpoint: the latest implementation and private test
bundles are still in the local development tree, not included in this update.

## Target for the first beta

- A responsive accelerated X11 desktop at **1280×800**, with a measured **50 FPS
  target**. This is a target, not a current benchmark result.
- Minecraft **Beta 1.7.3** using the SGX535. Opening the game and GPU-rendered
  menus are the first milestone; entering a world is deferred.
- CPU fills and basic 2D operations; GPU framebuffer copies where qualified.
  SGX535 rendering for the supported 3D path, with no software fallback counted
  as GPU acceleration.
- Safe installation, activation and rollback. Continue with `gma500`; this
  milestone does not migrate to `powervr_sgx` or change the default driver.

## Confirmed results and their scope

| Evidence | Result | What it does not prove |
| --- | --- | --- |
| Real SGX535, restricted sampled scene | Three draws with changed vertices and two intervening CPU COLOR clears passed, with retirement/readback/guards checked. Recovery to the original driver also passed. | General OpenGL, normal applications or sustained desktop performance. |
| Real Mini 12 Xorg, 1280×800 | One comparison produced 120 **CPU-generated** frames, 60 per publisher, with six complete server RGB checks. SHM publication median was about 10.89 ms; CPU fill plus publication median was 17.50 ms, with P95 23.48 ms. | GPU rendering, scanout/vsync latency or stable 50 FPS. The X11 client made no GPU submission. |
| Read-only queries on the current Mini 12 module | Dumb buffers supported; PRIME, framebuffer modifiers and syncobj reported zero. | A working DRI2 application or actual buffer-sharing/coherence validation. |
| Local NIR/Gallium/provider models | A 120-frame callback loop passed with 119 changed-vertex updates, 120 CPU COLOR clears, one session OPEN/CLOSE and no new GEM allocation after preparing the graph. | Hardware execution: native/GEM replies were simulated. |
| Local integration and transport tests | Persistent context/factory lifetime, standard frontbuffer callbacks, bounded asynchronous SHM publication and public legacy GEM display storage passed their focused tests. Display transport: 18 scenarios in native, UBSan and i486 variants. | A physically qualified current driver or complete application compatibility. |

The SHM comparison measures whole publisher paths, including persistent GC,
sealed snapshots and IPC changes; it is not a single-variable microbenchmark.
The historical [16.1345 FPS GLXGears result](PHYSICAL-GLX-STATUS-20261027.md)
retains its original restricted conditions and is a separate measurement.

The [machine-readable checkpoint](BETA-STATUS-20261113.json) records receipt
identifiers and SHA-256 digests. Raw captures, latest development binaries and
full private qualification bundles are not distributed by this documentation
commit. These summaries do not make the new results reproducible from a clone.

## What still blocks normal applications

1. **DRI2 buffer exchange.** The registered DRM frontend needs image
   import/export and `flush_resource` integration. The new legacy public-GEM
   transport is groundwork; it is not wired into those Gallium resource hooks.
   PRIME/dma-buf support cannot be assumed on the tested module. Public display
   buffers must remain separate from sealed private SGX command/source storage.
2. **Normal native admission.** The current sampled execution endpoint still
   requires `CAP_SYS_ADMIN` and qualification controls. Its restricted 640×640,
   one-triangle profile is not a general multi-context driver contract.
3. **Application-required behavior.** Transactional shader/state/texture changes,
   required clear/preservation semantics and actual GLX/EGL programs need work
   and hardware evidence. OpenGL 2.1 completeness and Minecraft compatibility
   have not been established.
4. **Delivery qualification.** The isolated installer framework does not yet
   provide a qualified system-driver activation. End-to-end installation,
   upgrade and rollback, sustained interactive desktop testing, and the
   kernel/distribution/device matrix remain pending.

The 1600×768 test remains an **offscreen renderer target on the Mini 12**;
checking a storage/view layout at that size is not a rendering test. There is no
Vaio P hardware qualification. Kernels 5.10–7.2, Arch and other distributions
remain validation targets, not a compatibility claim.

The work continues to preserve MMU/GEM ownership, PDS/ISP/PARAM isolation,
fences, retirement, coherence, quarantine and the original recovery paths.
No new qualification gate was opened to turn these limited results into a
normal-driver support claim.
