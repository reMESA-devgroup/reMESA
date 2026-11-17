# reMESA — Source Provenance, Reverse Engineering and Audit

> **Status:** Public research provenance record; independent implementation and licensing audit pending  
> **Project:** [reMESA](https://github.com/reMESA-devgroup/reMESA)  
> **Primary target:** PowerVR SGX535, Intel Poulsbo, revision 121  
> **Reference device:** Dell Inspiron Mini 12 (Intel Atom Z520)  
> **Document date:** 2026-11-16

## 1. Purpose and scope

This document records the origins, use, restrictions and uncertainty associated with the historical source code, vendor binaries, research tools, AI assistance and experimental evidence consulted during reMESA development. It is intended to help maintainers and prospective reviewers trace technical claims to their supporting material.

**This is not an independent clean-room certification, legal opinion, or proof that every implementation file has been audited.** Public availability does not itself grant reuse or redistribution rights. A successful hardware experiment does not establish copyright provenance.

The provenance of the SGX535 driver and the separate SGX544 research effort must not be conflated. See `projects/sgx544/docs/PROVENANCE.md` for the latter.

## 2. Research and implementation boundaries

The project aims to:

1. Identify historical material by repository, commit, path, version and hash.
2. Distinguish historical facts, inferred architecture, offline models and physical GPU observations.
3. Keep vendor implementations and restricted materials separate from new driver implementation.
4. Preserve original copyright and licensing notices.
5. Record LLM assistance rather than implying exclusively human authorship.
6. Avoid treating decompiled vendor code, model output or a public mirror as automatically reusable source.
7. Expose unresolved questions for independent audit instead of asserting that they are closed.

These are stated policies and documented research practices, **not a guarantee that every prior session followed them**.

### Classification vocabulary

| Class | Meaning |
|---|---|
| `PUBLIC-OPEN` | Identified public source with a recorded open-source license; exact file and revision still matter |
| `PUBLIC-RESTRICTED` | Publicly obtainable material subject to additional restrictions |
| `MIRROR-UNVERIFIED` | Community-hosted copy not authenticated as an official vendor release |
| `VENDOR-ORACLE` | Original vendor software queried for comparison or interpretation |
| `HISTORICAL-BINARY` | Historical executable or library inspected as evidence |
| `QUARANTINED` | Material excluded from implementation use pending rights/provenance review |
| `ORIGINAL-IMPLEMENTATION` | reMESA-authored implementation; authorship and inputs still require audit |
| `AI-ASSISTED` | Work generated, edited, researched or organized with LLM assistance |
| `PHYSICAL-EVIDENCE` | Observation obtained on the actual target GPU |
| `OFFLINE-EVIDENCE` | Static inspection, software modeling, or tests without physical GPU execution |
| `UNKNOWN` | Claim not established by the available evidence |

## 3. Historical source inventory

### 3.1 Linux DRM/GMA500

- Repository: <https://github.com/gregkh/linux>
- Recorded historical commit: `d5eca7ebcf6f64c4aebf9684c365c130a3a069b3`
- Recorded Phase 7 files: `psb_drv.c`, `psb_drv.h`, `psb_device.c`, `power.c`, `psb_irq.c`, `mmu.c`, `accel_2d.c`, `psb_reg.h`, `gtt.c`.
- Role: Linux kernel architecture, GMA500/DRM interfaces, memory management and hardware integration reference.
- Qualification: retained source-to-Git-object correspondence does not prove that the Dell's installed module was built from precisely those sources.

See `docs/kernel-5.10.240-antix-audit.md`, `docs/phase7/source-index.json` and `docs/phase7/source-provenance.md`.

### 3.2 Texas Instruments / Imagination PowerVR kernel DDK

- Official source host: <https://git.ti.com/cgit/graphics/omap5-sgx-ddk-linux/>
- Recorded Poulsbo-capable DDK commit: `cb46ba4d0c900f89f7ec0284f9803d476bfa98de`
- Recorded SGX535-header commit: `322bcda5f3076037e2e20ef9209f4f4d575a7d5f`
- Examined source names include `sgx535defs.h`, `sgxerrata.h`, `sgxfeaturedefs.h`, `sgxreset.c`, `sgxinit.c`, `sgxinfo.h`, `sgx_bridge.h`, `bridged_sgx_bridge.c`, `sysconfig.c`, and a Poulsbo D0 build configuration.

The historical `sgx535defs.h` identifies SGX535 and declares MIT/GPLv2 licensing. The recorded Poulsbo build configuration specifies `SGXCORE := 535`, `SGX_CORE_REV := 121`, and `PVR_SYSTEM := poulsbo`.

**Licensing qualification:** Official TI hosting and an open-license declaration on one file do not automatically establish the rights for every historical revision or neighboring file. The kernel DDK must also be distinguished from proprietary user-mode components. Individual notices, including any historical confidentiality markings, require exact-revision review. Historical source is recorded as research evidence, not presumed to have been transplanted into the new driver.

See `docs/source-archaeology.md` and `docs/poulsbo-data/sources.json`.

### 3.3 Historical Poulsbo kernel

- Repository: <https://github.com/gregkh/psb-kmp>
- Commit: `98b5307e5158a9ac401b29128ddd1184ae06b4d7`
- Recorded files: `psb_drv.c`, `psb_mmu.c`, `psb_xhw.c`.
- Role: historical memory-management, command and kernel-interface evidence; per-file license and platform scope apply.

### 3.4 Intel EMGD community mirror

- Repository: <https://github.com/EMGD-Community/intel-binaries-linux>
- Recorded commit: `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b`
- Recorded examples: `sgxerrata.h`, `sgx535defs.h`, `sgxreset.c`, `pwr_plb.c`, `emgd_drm.h`.
- Classification: `MIRROR-UNVERIFIED`.

The mirror is a research source, not independently authenticated as an Intel release. Mixed source notices must be checked individually. Public mirroring does not establish unrestricted reuse.

### 3.5 TI user-mode DDK

- Official host: <https://git.ti.com/cgit/graphics/omap5-sgx-ddk-um-linux/>
- Recorded commit: `b6801bf89e00d69893c2957455bbeb3195c0ed52`
- Example binary: `targetfs/lib/libsrv_init.so.1.9.6.0`
- Recorded SHA-256: `a1a3572ee4ae6104505808d171d6b4cd023309959588f890b2192aea44fafdda`

This is an OMAP5/DRA7xx artifact, not an authenticated Poulsbo userspace stack. The project records metadata inspection; redistribution and implementation reuse require separate review.

## 4. Historical proprietary Poulsbo binaries

### `psb_dri.so`

- Historical package: `xpsb-glx` 0.18 (`home:lkundrak:poulsbo`)
- SHA-256: `74ca42991906741ee91be1bc088ae0b142fa71b6a85658c5dad0851ce50dd0d8`
- Recorded package restriction: redistributable, no modification permitted.

### `Xpsb.so`

- Same historical package family.
- SHA-256: `da531587b1ec59fe433fa20ddd2e691b2f8b7fb35bb82525d4db99259c5e571f`

These binaries were used as historical static-analysis evidence for interfaces, data layouts and CPU-side behavior. The research records address-level observations and disposable analysis projects; those records are **not** a substitute for a code-level review demonstrating that proprietary implementation code was not incorporated into reMESA.

A package version or matching structure does not authenticate a historical kernel/userspace build pairing.

See `docs/phase7/psb-dri-re/README.md`, `docs/phase7/source-provenance.md`, and `docs/phase7/xpsb-re/source-provenance.md`.

### Other historical comparators

| Material | Version | Recorded role |
|---|---|---|
| `libdrm-poulsbo` | 2.3.0 | Historical userspace/kernel ABI |
| `xserver-xorg-video-psb` | 0.32.0 | Historical Xorg driver behavior |
| PSB kernel source | 4.41.1 | Candidate historical kernel interface |
| PSB firmware | 0.30-4 | Package inventory; not proof of SGX render firmware dependence |
| MesaLib | 7.4.4 | Public Mesa ABI/layout comparator |

Exact archive hashes, provenance and restrictions are recorded in `docs/phase7/xpsb-re/source-provenance.md`.

## 5. Original Imagination USSE decoder accessed through a research shim

**This is a vendor-backed decoder, not an independently implemented open-source USSE disassembler.**

### 5.1 Acquisition and origin: separate claims

The historical "sgx540-reversing" project documents an "asm/preload_hack.c" utility for invoking the internal Series5 USSE disassembler in Imagination Technologies' official "GLSLESCompiler_Series5" executable through "LD_PRELOAD".

The supplied repository history identifies Garnet and commit "03eadc4b54" ("Fix up the names for the x86 preload hack"). This technique predates reMESA and was not originally developed by reMESA or generated by its LLM-assisted workflow.

The reMESA developer reports that the research shim used by reMESA was not generated by AI. However, the relationship between the historical "preload_hack.c" and reMESA's "sdk35_decoder_preload.c" has not yet been established through a source-level comparison. Any reused or adapted code must retain appropriate attribution and comply with the applicable license.

The shim and the decoder are distinct components. The shim invokes functionality within the original Imagination executable; it does not independently implement the vendor's USSE decoder.

The canonical repository URL, complete historical revision, relevant copyright notices, and licensing terms remain subject to verification.

The supplied `SGX535-rev121-USSE-disassembler.zip` identifies the executable actually invoked as an **original Imagination Technologies Series5 GLSL ES compiler from PowerVR SDK 3.5**, Linux x86 32-bit:

- Included executable: `GLSLESCompiler_Series5-sdk35`
- Vendor SDK URL recorded in the package: <https://cdn.imgtec.com/sdk/OFFLINE/PowerVRSDKSetup-3.5.run-x86>
- Executable SHA-256 recorded in the package: `9349ed4d7643d81cb638b41eb41b69507cc2f9f15bae14c8f2116e79f70e7bd9`
- Acquisition route reported by developer: `garnet/sgx540-reversing` 

The SDK URL is an **origin claim recorded by the packaged README**, not proof that the particular executable in the archive was independently downloaded from that URL during this audit. The developer's acquisition account and the package's SDK-origin statement can both be true, but their relationship has not yet been independently reconstructed.

### 5.2 What the shim does

The supplied archive includes:

- `sdk35_decoder_preload.c` — C shim source.
- `sdk35_decoder_preload.so` — compiled preload library.
- `disassemble.py` — Python launcher.
- `GLSLESCompiler_Series5-sdk35` — original vendor executable.
- `runtime/` — bundled i386 glibc runtime components.
- `hashes.json` and `SHA256SUMS` — integrity records.
- `README-PT.txt` — package documentation.
- `example_nop_end.bin` — small example input.

The C shim intercepts `__libc_start_main` through `LD_PRELOAD` and substitutes a research entry point. It checks a hard-coded SDK descriptor at `0x082d2a88` and calls an internal decoder function at `0x0829921b` **inside the original vendor executable**. These are binary-specific internal addresses, not supported public SDK interfaces or GPU addresses.

The tool accepts little-endian USSE instruction words, eight bytes per instruction, with a documented limit of 1–512 instructions per call. The shim checks output canaries and termination; the launcher verifies package hashes and imposes an execution timeout. The supplied README describes use with an i386 runtime or QEMU on ARM Linux.

The selected descriptor identifies **SGX535 revision 121**, not a fabricated SGX540 context. The textual result is vendor-decoder output; it is not proof that the instruction will execute correctly on physical SGX535 hardware.

### 5.3 What the shim does *not* establish

- It does **not** make the Imagination decoder open source.
- It does **not** constitute an independent USSE decoder implementation.
- It does **not** demonstrate that the vendor executable may be redistributed with reMESA.
- It does **not** independently prove physical execution or OpenGL correctness.
- It does **not** by itself establish that the reMESA USSE encoder/compiler is free of restricted code.
- It does **not** establish that the original vendor decoder implementation was copied into the driver; that requires a separate implementation audit.

The provided C shim contains an invocation of a function inside the vendor binary rather than an extracted implementation of that function. This is a narrow, inspectable statement about the supplied shim, **not** a conclusion about every file in the driver.

### 5.4 Redistribution and clean-room audit questions

The supplied ZIP contains the vendor executable and bundled runtime libraries. The included glibc GPL/LGPL notices govern those components; they do not license the Imagination compiler. The compiler's distribution terms and the rights applicable to the acquisition route require review before public redistribution.

The implementation audit should determine:

1. The exact `sgx540-reversing` repository URL, commit, and relevant artifact.
2. Whether that repository distributed the original binary, an extracted decoder, a script, or only instructions for obtaining it.
3. Whether the binary in the ZIP matches the official SDK release or a repackaged copy.
4. Which vendor outputs were used as reference vectors and whether they entered LLM prompts or generated code.
5. Whether any vendor decoder tables, functions or reconstructed code entered the reMESA implementation.
6. Which vendor redistribution terms apply to the ZIP and to any published release assets.

VENDOR-ORACLE, HISTORICAL-BINARY, and a reMESA research shim using a historically documented LD_PRELOAD technique; exact source derivation, licensing, and redistribution rights remains to garnet.

See also `docs/phase9/USSE-VENDOR-ORACLE-20261027.md`

## 6. Restricted and excluded research materials

### Canonical `libgl1-mesa-dri-psb`

- Version: `0.24+repack+0038.1`
- Historical package context: Canonical `hardy-dell-mini`
- Package SHA-256: `14e870f4273e84759630499efeda4948116c29f3ba7d08e2b5f028a5da3ef8a2`
- Extracted ELF SHA-256: `427627909b0c528d154d1b247688f16536162a520d43d472e8a4f404c02e2a6e`

The research report records an `Intel Confidential / all rights reserved` notice. It says this package remained in disposable quarantine, was not executed, was not added to `references/`, and was not used for implementation details. Limited static occurrence/provenance checks were recorded.

**Status:** `QUARANTINED`. A confidentiality marking alone does not prove unauthorized publication.

### PSP2 PowerVR repository — inspected, not used

The public `GrapheneCt/PVR_PSP2` repository was briefly checked as a possible PDS decoder locator. The inspected `sgxpdsdefs.h` carried restrictive Imagination copyright language. According to the project research record, no source from this repository was imported, used to decode selected SGX535 instructions, or treated as evidence for Poulsbo hardware behavior.

**Status:** Unsuccessful research lead; **not an implementation dependency**. This is included for transparency, not to imply that reMESA used PSP2 code.

### Other excluded leads

The historical report also records a search result explicitly marked Confidential/NDA that was excluded unopened, and a `freemangordon/mesa` historical branch inspected as a research lead without importing its source.

The presence of a research lead in a provenance record does not mean its code was used.

## 7. Reverse-engineering and evidence methodology

Research methods documented across the project include:

- Historical source archaeology with exact commits and file hashes.
- Static ELF inspection and targeted disassembly.
- Vendor-oracle queries, including the USSE decoder described above.
- Independently authored byte-layout and semantic-constraint models.
- NIR/USSE compiler tests and negative-case validation.
- Controlled physical GPU submissions and readback comparisons.
- Kernel ownership, retirement, memory-guard and recovery experiments.

Evidence must be scoped correctly:

| Evidence | Supports | Does not automatically support |
|---|---|---|
| Historical source | Properties of the inspected revision | Identity of installed binaries |
| Vendor decoder text | Vendor interpretation of input words | Correct physical GPU execution |
| Offline model | Behavior of the tested software model | Real hardware correctness |
| Physical readback | Output of that particular experiment | General application compatibility |
| Unit-test pass | Tested software invariants | Copyright provenance or upstream eligibility |

Use `CONFIRMED`, `INFERRED`, `UNKNOWN`, `BLOCKED`, `PASS`, and `FAIL` with explicit test scope. A one-draw result is not a claim of continuous rendering or OpenGL conformance.

## 8. AI-assisted development disclosure

reMESA has used extensive LLM assistance for source archaeology, searching and comparing code, documentation, implementation and debugging. Human maintainers remain accountable for submissions and claims.

This document does **not** assert that model training data can be reconstructed or that generated code is necessarily free of proprietary influence. In particular, the use of vendor-decoder output must be accounted for in any source-to-implementation audit.

For each significant component, record:

- Current file and implementation commit.
- Source references and exact license scope.
- Known LLM tools and input context, where records survive.
- Whether vendor or restricted outputs were provided to a model.
- Human review and independent explanation of the implementation.
- Similarity review against restricted historical material.
- Offline tests and physical qualification evidence.
- Remaining uncertainties.

Missing records should be marked `UNKNOWN`, not reconstructed from assumptions.

## 9. Upstream readiness and unresolved work

Before proposing substantial Mesa or Linux kernel integration, the project should:

- [ ] Verify exact licensing of consulted historical DDK revisions.
- [ ] Authenticate or explicitly qualify mirrored source provenance.
- [ ] Resolve the exact `sgx540-reversing` acquisition URL, commit and artifact identity.
- [ ] Check the Imagination SDK compiler's redistribution terms.
- [ ] Keep the vendor-oracle binary separate from any open-source driver release unless distribution is authorized.
- [ ] Audit how vendor-decoder output informed the USSE implementation and LLM inputs.
- [ ] Review source-to-implementation lineage and potential restricted-code similarity.
- [ ] Complete substantive human review of kernel and Mesa code.
- [ ] Provide reproducible build, correctness, ownership and recovery evidence.
- [ ] Document remaining normal DRI/GLX, presentation and application limitations accurately.
- [ ] Check applicable Mesa and kernel contribution policies, including AI-related rules.
- [ ] Obtain maintainer feedback without representing incomplete checks as completed certification.

## 10. Primary project records

- [`docs/source-archaeology.md`](docs/source-archaeology.md)
- [`docs/poulsbo-data/sources.json`](docs/poulsbo-data/sources.json)
- [`docs/phase7/source-index.json`](docs/phase7/source-index.json)
- [`docs/phase7/source-provenance.md`](docs/phase7/source-provenance.md)
- [`docs/phase7/xpsb-re/source-provenance.md`](docs/phase7/xpsb-re/source-provenance.md)
- [`docs/phase7/psb-dri-re/README.md`](docs/phase7/psb-dri-re/README.md)
- [`docs/phase7/psb-dri-re/cleanroom-backing-contract.md`](docs/phase7/psb-dri-re/cleanroom-backing-contract.md)
- [`docs/phase9/USSE-VENDOR-ORACLE-20261027.md`](docs/phase9/USSE-VENDOR-ORACLE-20261027.md)
- [`docs/phase9/PDS-CROSS-CORE-20261031.md`](docs/phase9/PDS-CROSS-CORE-20261031.md)
- [`projects/sgx544/docs/PROVENANCE.md`](projects/sgx544/docs/PROVENANCE.md)

## 11. Final statement

reMESA maintains substantial records of historical sources, binary identities, hardware experiments and AI-assisted development. These are valuable foundations for independent review. They are **not** proof that all implementation code has been audited, that every historical artifact may be redistributed, or that no restricted material influenced AI-assisted changes.

The Imagination USSE decoder is accurately described as **original vendor functionality invoked through a research shim**. The developer reports an acquisition route through `sgx540-reversing`; the included package separately identifies the original PowerVR SDK 3.5 executable. Neither fact should be erased or conflated.

**No completed independent clean-room certification is claimed.** Outstanding licensing, provenance, implementation and upstream-policy questions remain open until supported by reviewable evidence.

---

*Maintenance: update material claims with source identifiers, commits, hashes, experiment IDs or review records. This document does not alter third-party licenses.*


## 12. Mesa upstream contribution source pins

The SGX535 Gallium work is developed against a pinned Mesa interface/build baseline. These pins identify the source APIs and build integration inspected; they are not an SGX implementation source and must be revalidated against the Mesa branch targeted by any future merge request.

- Official Mesa source archive: [Mesa 25.2.0](https://archive.mesa3d.org/mesa-25.2.0.tar.xz), SHA-256 `7c726b21c074d14d31d253d638b741422f3c0a497ce7f1b4aaaa917d10bd8d4f`.
- Gallium API headers: `src/gallium/include/pipe/p_screen.h` (SHA-256 `84a9cdf9f66ce7d95a402e4de993bff3a687700b70622584f02dce5b1612b271`), `p_context.h` (`1cc1145fb8856657edecbc775f1a9a1de31f0620b175391dc9a025bf68d7dc67`), `p_state.h` (`066e9ee6248c56233ac2e9bb1d3c04aae135fc1132ca6a3f8503a98992939f15`) and `p_defines.h` (`3f28e565d2f5b392cfab59bf73ec4161d52bdbf1e83507a0817002b89c68fbd8`).
- Format generation inputs: Mesa `src/util/format/u_format.yaml` (`0aabe9d19cd05ce9bed963023b5790080a468a9316ef69bf162f27366a642a67`) and `u_format_table.py` (`2978c931030b391bb315947d143e89a0461458a77248b24483e99c35cc56bd3b`); generated format header SHA-256 `1fadb92f35a3819da0981530e8a8ec4f7d0c295e894f1f8bb75bd2adf97d1d65`.
- Build-only dependency: [PyYAML 6.0.2](https://github.com/yaml/pyyaml/archive/refs/tags/6.0.2.tar.gz), SHA-256 `9377c381ac3fccad8df73d96b5139ef8b1a2c57a0d913e95ab0a2275d66b5caa`; recorded role is generating format enums.
- Loader and build integration: `pipe_loader_drm.c`, target-helper DRM headers, DRI/DRIL target files and Mesa Meson definitions are individually pinned in [`mesa/sgx535/mesa-loader-pin.json`](mesa/sgx535/mesa-loader-pin.json). The full interface ledger is [`mesa/sgx535/mesa-api-pin.json`](mesa/sgx535/mesa-api-pin.json).
- These interface files carry their own upstream notices. No source from a third-party SGX implementation was used as the implementation reference for the native Gallium adapter; historical reverse-engineering sources and the disclosed vendor oracle are catalogued elsewhere in this document.

## 13. Source-by-source Phase 7 index

The table expands all 29 entries in [`docs/phase7/source-index.json`](docs/phase7/source-index.json). SHA-256 values identify the exact source files recorded there. The JSON remains authoritative for local snapshot path, line range, platform, license, evidence ID and provenance qualification.

| Source ID | Repository | Commit | Original file | SHA-256 |
|---|---|---|---|---|
| `target:psb_drv.c` | `https://github.com/gregkh/linux` | `d5eca7ebcf6f64c4aebf9684c365c130a3a069b3` | `drivers/gpu/drm/gma500/psb_drv.c` | `db2699f9301563dfe7cd8c87e7087d00a1c7af432a41161a6e2ec21b07d3d576` |
| `target:psb_drv.h` | `https://github.com/gregkh/linux` | `d5eca7ebcf6f64c4aebf9684c365c130a3a069b3` | `drivers/gpu/drm/gma500/psb_drv.h` | `6dce38176eccc8d7037783ec6c41d101c3c614068a3b7b11f40b06e1d1ca6261` |
| `target:psb_device.c` | `https://github.com/gregkh/linux` | `d5eca7ebcf6f64c4aebf9684c365c130a3a069b3` | `drivers/gpu/drm/gma500/psb_device.c` | `c02f8c825350ed89675d7618e158d503fbb92169a7f2f8aba4255b353de9a493` |
| `target:power.c` | `https://github.com/gregkh/linux` | `d5eca7ebcf6f64c4aebf9684c365c130a3a069b3` | `drivers/gpu/drm/gma500/power.c` | `94f01101e484ef73f111278a225c74c8528f99a5ffee79b97f368ee5e5de6a9d` |
| `target:psb_irq.c` | `https://github.com/gregkh/linux` | `d5eca7ebcf6f64c4aebf9684c365c130a3a069b3` | `drivers/gpu/drm/gma500/psb_irq.c` | `9dfbb8dca48e68a1f9eef266e85c4b55807e2689d598d5f24995f9aa9938bd97` |
| `target:mmu.c` | `https://github.com/gregkh/linux` | `d5eca7ebcf6f64c4aebf9684c365c130a3a069b3` | `drivers/gpu/drm/gma500/mmu.c` | `d9a7e839d24c3a6e5a6be77c8583a8073ea4e20fb64eb1fc0228048db4e8fd24` |
| `target:accel_2d.c` | `https://github.com/gregkh/linux` | `d5eca7ebcf6f64c4aebf9684c365c130a3a069b3` | `drivers/gpu/drm/gma500/accel_2d.c` | `090002c14bf5e0f0777382f735807faa6dfa0dde98a1a57aff8f65aee20a2f1c` |
| `catalog:H535` | `omap5-sgx-ddk-linux` | `322bcda5f3076037e2e20ef9209f4f4d575a7d5f` | `services4/srvkm/hwdefs/sgx535defs.h` | `cb2a3e9119d65b421a16744137b1114b0ebd6982cb27549b4fce1d8405bd41c3` |
| `catalog:PSB_psb_drv_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_drv.c` | `8aae96d114be1349e22f156a4f00f2bf6335f0c08be19d3d6e72382e1f9801fe` |
| `catalog:PBUILD` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/eurasiacon/build/linux2/pc_i686_poulsbo_d0_linux/Makefile` | `cc9814e9e9cb575ad28223ebc5239f3799d7a92460d2eac4e2e36a88fc08a7a8` |
| `catalog:ERRATA` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/hwdefs/sgxerrata.h` | `b6b7a7e53708f9b9516d3b53a317c27bea9aa9b2302c853b1d1d944ac3554e4d` |
| `external:errata` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/srvkm/hwdefs/sgxerrata.h` | `d88b66f34fd17fcc3303f04e14996098623e4931b81f7ec5a0c16f87c4b1d3a7` |
| `catalog:RESET` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/devices/sgx/sgxreset.c` | `9eef409b4acfd96e096ce0cf372bdbeb0ba57fd2b8842e64853f9e18c71460d3` |
| `catalog:FEATURE` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/hwdefs/sgxfeaturedefs.h` | `0f3e2d7170c81a882b371023159b5a6112232431c43c2113b031b4bf7b9ba7e4` |
| `catalog:PSYSC` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/system/poulsbo/sysconfig.c` | `725faa537e192c1e680a7646a9dcb7588691b8b5fe3796668b8e810ba071d5e1` |
| `catalog:EMGD_drm_emgd_state_power_plb_pwr_plb_c` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/emgd/state/power/plb/pwr_plb.c` | `81bb7c947cc06b3ceae033eed4ceeea0dc27ef0fec53392fb73669345f71cfc5` |
| `catalog:EMGD_drm_pvr_services4_srvkm_hwdefs_sgx535defs_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/srvkm/hwdefs/sgx535defs.h` | `634fbb74645d373e0c60ad5e01194fa96e91bb6b26d60dbc35a71a2eda49d12c` |
| `target:psb_reg.h` | `https://github.com/gregkh/linux` | `d5eca7ebcf6f64c4aebf9684c365c130a3a069b3` | `drivers/gpu/drm/gma500/psb_reg.h` | `a340d4fb7bb9df2403eb196639d930b8c56463ddc7a3754d152e6ed7ab1daef7` |
| `catalog:PSB_psb_mmu_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_mmu.c` | `401954c022c8c7a76e6b30ae86b141e8a673f0b8587b7786f8c09bf7e4d05f5b` |
| `catalog:EMGD_drm_pvr_services4_srvkm_devices_sgx_sgxreset_c` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/srvkm/devices/sgx/sgxreset.c` | `f4cc92f191c0124ea35c44e05a7b0a64eba299d3391c07cca9b5cc9986fa2ff7` |
| `target:gtt.c` | `https://github.com/gregkh/linux` | `d5eca7ebcf6f64c4aebf9684c365c130a3a069b3` | `drivers/gpu/drm/gma500/gtt.c` | `388c5b251cfa17545031c1841a4ef6afcf1bc7385a424e34221948959e15818d` |
| `catalog:INFO` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/include/sgxinfo.h` | `b16e76e79dc24b3e3356815514103cd8c2d305e5bfcd74abd6192d637e0f468f` |
| `catalog:SBRIDGE` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/include/sgx_bridge.h` | `33448ba5d1c3d833e29198fadd4b7b0c1d181dcefdee1201d1da27c7a00b7612` |
| `catalog:INIT` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/devices/sgx/sgxinit.c` | `da5facd0ac7db01c20dab240d1fe07515da42580c86f6afb5821ef04afb3e2a5` |
| `git:bridge` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/bridged/sgx/bridged_sgx_bridge.c` | `07dd4296bc64cbb39d374958ced63eca860aef589852d01222cb16993dedf494` |
| `um:lib` | `omap5-sgx-ddk-um-linux` | `b6801bf89e00d69893c2957455bbeb3195c0ed52` | `targetfs/lib/libsrv_init.so.1.9.6.0` | `a1a3572ee4ae6104505808d171d6b4cd023309959588f890b2192aea44fafdda` |
| `um:readme` | `omap5-sgx-ddk-um-linux` | `b6801bf89e00d69893c2957455bbeb3195c0ed52` | `README` | `4c8aebfd572bb01f040f0ee1a9a63247f75c669136866aa1383b084df01a8135` |
| `catalog:PSB_psb_xhw_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_xhw.c` | `73783fad6b072b655d50386621d09d7fe6f4cdd896cb991ed0258a2494330fc1` |
| `catalog:EMGD_drm_include_emgd_drm_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/include/emgd_drm.h` | `8291ea2191be5fc0691b07967b4bf3620c78bb5d577b06d75520c7e528d5a726` |

## 14. Source-by-source Poulsbo archaeology catalog

The table expands all 72 records in [`docs/poulsbo-data/sources.json`](docs/poulsbo-data/sources.json). This is an archaeology/platform catalog, so inclusion does not mean every record informed Mesa code. `—` means the source record did not provide that field. Consult the JSON for blob IDs, retained snapshot paths, line counts, notices, confidence and platform qualifications.

| Source ID | Repository | Commit | Original file | SHA-256 |
|---|---|---|---|---|
| `H535` | `omap5-sgx-ddk-linux` | `322bcda5f3076037e2e20ef9209f4f4d575a7d5f` | `services4/srvkm/hwdefs/sgx535defs.h` | `cb2a3e9119d65b421a16744137b1114b0ebd6982cb27549b4fce1d8405bd41c3` |
| `PBUILD` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/eurasiacon/build/linux2/pc_i686_poulsbo_d0_linux/Makefile` | `cc9814e9e9cb575ad28223ebc5239f3799d7a92460d2eac4e2e36a88fc08a7a8` |
| `PSYS` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/system/poulsbo/sysconfig.h` | `2fbf534d641f3c512cf7a93284cb5a75e7f26250f02b13e1767965d444920340` |
| `PSYSC` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/system/poulsbo/sysconfig.c` | `725faa537e192c1e680a7646a9dcb7588691b8b5fe3796668b8e810ba071d5e1` |
| `PINFO` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/system/poulsbo/sysinfo.h` | `8826a5839d48bd203ce264e9cca3726069392fc1950e79c6334211d95f0c880a` |
| `FEATURE` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/hwdefs/sgxfeaturedefs.h` | `0f3e2d7170c81a882b371023159b5a6112232431c43c2113b031b4bf7b9ba7e4` |
| `ERRATA` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/hwdefs/sgxerrata.h` | `b6b7a7e53708f9b9516d3b53a317c27bea9aa9b2302c853b1d1d944ac3554e4d` |
| `MMU` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/hwdefs/sgxmmu.h` | `5a2e8509359631907ac8a4917d50c299d687bb7b7235f8c50c29f131b88b8940` |
| `RESET` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/devices/sgx/sgxreset.c` | `9eef409b4acfd96e096ce0cf372bdbeb0ba57fd2b8842e64853f9e18c71460d3` |
| `VER14` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/include4/pvrversion.h` | `5730251c117e8f22b7b0cc50a6884fb07c5ea42219cf4e629358f3efbfd58141` |
| `LREG` | `linux` | `9b87fdc9af2fbfcdb5c24a64139685ef80f6573f` | `drivers/gpu/drm/gma500/psb_reg.h` | `124e8e80602a25d0bbe7a1019e95f6d780abcad5798b7d11f26496b7f3ead3d3` |
| `LDRVH` | `linux` | `9b87fdc9af2fbfcdb5c24a64139685ef80f6573f` | `drivers/gpu/drm/gma500/psb_drv.h` | `17b81c7c8e33c9c2f278d926864589fcdf8f70bffa55a13b37c4d4bd5f4ce3b2` |
| `LDRVC` | `linux` | `9b87fdc9af2fbfcdb5c24a64139685ef80f6573f` | `drivers/gpu/drm/gma500/psb_drv.c` | `3a470f23080e5aed93cc3882b5258de284143403b7b80d82eb7863bef75165d5` |
| `LMMU` | `linux` | `9b87fdc9af2fbfcdb5c24a64139685ef80f6573f` | `drivers/gpu/drm/gma500/mmu.c` | `7212b3cdc7a92887c6c65f7b424b9b83feb87618737a22760976e962f3c27751` |
| `LIRQ` | `linux` | `9b87fdc9af2fbfcdb5c24a64139685ef80f6573f` | `drivers/gpu/drm/gma500/psb_irq.c` | `8daf6b0f887ea1c376efd117df202f3c9107ffcbabe358af5698f3da8c9b999f` |
| `DRMS` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/include/env/linux/pvr_drm_shared.h` | `2e43dddbdcb621231b1c49eb4e9942e3daa9b6830c9db6302200cbbc24bfca25` |
| `DRMC` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/env/linux/pvr_drm.c` | `315909a93c4a6e54c06d7bd9dbc7a5d2f875e7d9018ec889f6a258c5431d27b4` |
| `BRIDGE` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/include/pvr_bridge.h` | `c48cf6e4c5000ccf4edaf530c83905db10881cf288281045fa1efde209d2d902` |
| `SBRIDGE` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/include/sgx_bridge.h` | `33448ba5d1c3d833e29198fadd4b7b0c1d181dcefdee1201d1da27c7a00b7612` |
| `DCS` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/3rdparty/dc_poulsbo/3rdparty_dc_drm_shared.h` | `36a23b6bed784c74576a3822303b16aee85d2480453f525fd79bb1f1de3b7a0d` |
| `DCL` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/3rdparty/dc_poulsbo/poulsbo_linux.c` | `b1cfb521e90d6b541f003b3a23eb1c6c497f65c6097430c005fbb543fd2e6ad1` |
| `DCR` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/3rdparty/dc_poulsbo/poulsbo_regs.h` | `f4f0f3315895274ba853a3ba7231465bb4f298ebc5973d6222e1c84dd1fcc612` |
| `INIT` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/devices/sgx/sgxinit.c` | `da5facd0ac7db01c20dab240d1fe07515da42580c86f6afb5821ef04afb3e2a5` |
| `MKIF` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/include/sgx_mkif_km.h` | `181d7f21bf0ec78022b43b988c9ea49714d4843b79d3d3ed6a944344807ed5db` |
| `INFO` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/include/sgxinfo.h` | `b16e76e79dc24b3e3356815514103cd8c2d305e5bfcd74abd6192d637e0f468f` |
| `UTIL` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/srvkm/devices/sgx/sgxutils.c` | `95a3afa3bce7d85cc2bb832fc2baf25b67af6fff04eb473c286f07c0f96ba1ff` |
| `SYSUTIL` | `omap5-sgx-ddk-linux` | `cb46ba4d0c900f89f7ec0284f9803d476bfa98de` | `eurasia_km/services4/system/poulsbo/sysutils.c` | `e3d49837aa8682cb99876dcef5ccd44ba1b59736b36e0a73bbce6f58c1ec1b26` |
| `GTT` | `linux` | `9b87fdc9af2fbfcdb5c24a64139685ef80f6573f` | `drivers/gpu/drm/gma500/gtt.c` | `6ee3c406c02f9d34328f566743899272e596d37924edad1da7e204fe225a8166` |
| `GEM` | `linux` | `9b87fdc9af2fbfcdb5c24a64139685ef80f6573f` | `drivers/gpu/drm/gma500/gem.c` | `84e4b67b55f696d81a9eb08d8320715d8945008df789245c13ced993f8edd94e` |
| `POWER` | `linux` | `9b87fdc9af2fbfcdb5c24a64139685ef80f6573f` | `drivers/gpu/drm/gma500/power.c` | `ed871429e827f8098f387effa3a772aa80aa64206b89fb8e5a6cd7c7cfa262fe` |
| `DEVICE` | `linux` | `9b87fdc9af2fbfcdb5c24a64139685ef80f6573f` | `drivers/gpu/drm/gma500/psb_device.c` | `e744a0902d458748c1570ee7d048151030f173f215ab0df5053353ad38978ca2` |
| `PSB_README_drm` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `README.drm` | `666c5228eea8392ac9a6cfb1a285d7d3a7b94e45f706ec81bddb9ff59e57a27e` |
| `PSB_psb_drm_h` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_drm.h` | `85f30c2f01eb29884778de7959f4f122353f0cbb4ed5cec00db9fb862707dc21` |
| `PSB_psb_drv_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_drv.c` | `8aae96d114be1349e22f156a4f00f2bf6335f0c08be19d3d6e72382e1f9801fe` |
| `PSB_psb_drv_h` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_drv.h` | `08740e64688d04efc6a6b6c61fa1a3c7ef62af4bfb9823576ded590b5867ec62` |
| `PSB_psb_fence_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_fence.c` | `edb8303c144d50e32e746ad7540fb5e81fd9b4969741d487a4ff7c7d78792a3f` |
| `PSB_psb_irq_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_irq.c` | `1240c2ff093450fa578e6759ecedaf71f20df9f9a0897285be2db893cf19f1b3` |
| `PSB_psb_mmu_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_mmu.c` | `401954c022c8c7a76e6b30ae86b141e8a673f0b8587b7786f8c09bf7e4d05f5b` |
| `PSB_psb_reg_h` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_reg.h` | `512563057c48f651176928ccd33666a8a1145f19eb28c60296382776be1c4344` |
| `PSB_psb_reset_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_reset.c` | `fb66f4de7259d56c26b0b295ea53f434e6c061d43a71c37c6ac38a2c098f367f` |
| `PSB_psb_scene_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_scene.c` | `07309f6a189eb0ede64aaa98e7fbd858f44314256e2262226f48ba33d9b7528e` |
| `PSB_psb_xhw_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_xhw.c` | `73783fad6b072b655d50386621d09d7fe6f4cdd896cb991ed0258a2494330fc1` |
| `EMGD_License_txt` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `License.txt` | `e007ff8e2a484b18d7e4bd9770596023baf0394c47faffaa4fbff09590e7f5d1` |
| `EMGD_README_md` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `README.md` | `c299641217fcc6095d0cf86c96e03aaf372ce6bed5c836e47c6a118f1eccb589` |
| `EMGD_drm_include_emgd_drm_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/include/emgd_drm.h` | `8291ea2191be5fc0691b07967b4bf3620c78bb5d577b06d75520c7e528d5a726` |
| `EMGD_drm_pvr_include4_pvrversion_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/include4/pvrversion.h` | `9158cbfed5032132ef1b547257ab25d214122d8648936614f28c6031bedcfc12` |
| `EMGD_drm_pvr_services4_srvkm_hwdefs_sgx535defs_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/srvkm/hwdefs/sgx535defs.h` | `634fbb74645d373e0c60ad5e01194fa96e91bb6b26d60dbc35a71a2eda49d12c` |
| `EMGD_drm_pvr_services4_system_common_sysconfig_c` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/system/common/sysconfig.c` | `4ad525c14516d749a5e16c4fa32b008c1e057a627a02fc0019e1d6ec8ad14abd` |
| `EMGD_drm_pvr_services4_system_common_sysutils_c` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/system/common/sysutils.c` | `e0c2d5c3a98c0af5d2dde3846c24ca47b7ca7bd290cd0d2ec94dae77b0e24468` |
| `EMGD_drm_pvr_services4_system_include_sys_pvr_drm_shared_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/system/include/sys_pvr_drm_shared.h` | `5444be8b446141c713201809a0b20ce9b5bc4049133b82c0e29a88499efb556d` |
| `EMGD_drm_pvr_services4_system_include_sysconfig_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/system/include/sysconfig.h` | `a746e2593309805fb463cd22170a1839f32350d311b81c592f52cc49b9cd6c88` |
| `EMGD_drm_pvr_services4_system_plb_sysconfig_c` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/system/plb/sysconfig.c` | `fdc9bed037553914136a37b6c5577ed5b56196527119efdc69ee3c6af310510b` |
| `EMGD_drm_pvr_services4_system_tnc_sysconfig_c` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/system/tnc/sysconfig.c` | `9c641af1f4f2f8e4602021b40103c5a30529d99ac77e7c6593c28b1da68af003` |
| `EMGD_readme_txt` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `readme.txt` | `b8c4728e7cfc0098d8e27cee2ddec51387bb9f1d716b1f37a078aad02dca3886` |
| `PSB_psb_sgx_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_sgx.c` | `f17805b4a8959c064e222255baa5c885366c1970630b655dea04047daec79d02` |
| `PSB_psb_schedule_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_schedule.c` | `1a139a24cfbd6593e7a6c71315bd63d5ea649fda712555dc34ae30650f24eeca` |
| `PSB_psb_setup_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_setup.c` | `2e47db2e3a21958ec4bdf068dd88f9ea80c3470f508f8d31fe1a9de766f2142a` |
| `PSB_psb_buffer_c` | `https://github.com/gregkh/psb-kmp` | `98b5307e5158a9ac401b29128ddd1184ae06b4d7` | `psb_buffer.c` | `f2e6833fc82de488c6c41e99a5da2c5b00ec0fea36b9defeb4057e70f464c147` |
| `EMGD_drm_emgd_core_init_plb_init_plb_c` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/emgd/core/init/plb/init_plb.c` | `e7b5148b41f26caae601e2ef2ec08d6e9b43a9a18ef50b4caecb53425bec9aa5` |
| `EMGD_drm_emgd_state_power_plb_pwr_plb_c` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/emgd/state/power/plb/pwr_plb.c` | `81bb7c947cc06b3ceae033eed4ceeea0dc27ef0fec53392fb73669345f71cfc5` |
| `EMGD_drm_emgd_include_plb_sgx_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/emgd/include/plb/sgx.h` | `de8196c8c4da6a749656c544826b157b27e24717ffa815125ab64cbb5bbf291c` |
| `EMGD_drm_pvr_services4_system_plb_sysplb_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/system/plb/sysplb.h` | `f8ba56898922cde19a657e6d5fbcfda48b366d535fba61d5143f8abacbc1da9c` |
| `EMGD_drm_pvr_services4_system_common_sysconfig_c` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/system/common/sysconfig.c` | `4ad525c14516d749a5e16c4fa32b008c1e057a627a02fc0019e1d6ec8ad14abd` |
| `EMGD_drm_pvr_services4_system_include_sysconfig_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/system/include/sysconfig.h` | `a746e2593309805fb463cd22170a1839f32350d311b81c592f52cc49b9cd6c88` |
| `EMGD_drm_pvr_services4_include_env_linux_pvr_drm_shared_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/include/env/linux/pvr_drm_shared.h` | `7bd1b1f19ed6d72ea09f7908b1fac999e83554ab79a37396cddb33b4f39b66c9` |
| `EMGD_drm_pvr_services4_srvkm_hwdefs_sgxmmu_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/srvkm/hwdefs/sgxmmu.h` | `3211b42afb84a307fc62226838e4c3e7ef0bd9d61ba10e7bf770b1604792da61` |
| `EMGD_drm_include_emgd_shared_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/include/emgd_shared.h` | `8a18f45c4b499556ec5a28b1e3061a8177fc8ad7f22825145075a862ffb095fc` |
| `EMGD_drm_pvr_services4_srvkm_devices_sgx_sgxreset_c` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/srvkm/devices/sgx/sgxreset.c` | `f4cc92f191c0124ea35c44e05a7b0a64eba299d3391c07cca9b5cc9986fa2ff7` |
| `EMGD_drm_pvr_services4_srvkm_hwdefs_sgxfeaturedefs_h` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/srvkm/hwdefs/sgxfeaturedefs.h` | `4274aa574c87f99b996fc0d526557d588029031e4d1eb7af20f137d0a7701a8e` |
| `EMGD_drm_pvr_services4_srvkm_devices_sgx_mmu_c` | `https://github.com/EMGD-Community/intel-binaries-linux` | `e6884ec2eaaf1afe88d5ff9dd44d70403525be5b` | `drm/pvr/services4/srvkm/devices/sgx/mmu.c` | `f97597fec1af125a65ac7d0f4584f36558f3d93135d0571e74086b8f2586242d` |
| `CLOCK` | `linux` | `9b87fdc9af2fbfcdb5c24a64139685ef80f6573f` | `drivers/gpu/drm/gma500/gma_device.c` | `d970a14700c7c5ccddd2d87b3f5c4787da1bb164b1853252b61ba297a60939c8` |
| `OMAP` | `omap5-sgx-ddk-linux` | `430673f78b79eccdf308a6bbfb524209b485d2cc` | `eurasia_km/services4/system/omap4/sysutils_linux.c` | `0c6eb598c060774c6846aca7b8f50c462ea929c067d7dec65211599624a1240b` |
