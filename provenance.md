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

The developer reports obtaining the relevant decompiler/decoder research material through the **`sgx540-reversing` repository**. This is the reported **acquisition route**, not a claim that `sgx540-reversing` authored the underlying decoder. The exact repository URL, revision, distributed file and license terms still need to be added to the source ledger and independently checked.

The supplied `SGX535-rev121-USSE-disassembler.zip` identifies the executable actually invoked as an **original Imagination Technologies Series5 GLSL ES compiler from PowerVR SDK 3.5**, Linux x86 32-bit:

- Included executable: `GLSLESCompiler_Series5-sdk35`
- Vendor SDK URL recorded in the package: <https://cdn.imgtec.com/sdk/OFFLINE/PowerVRSDKSetup-3.5.run-x86>
- Executable SHA-256 recorded in the package: `9349ed4d7643d81cb638b41eb41b69507cc2f9f15bae14c8f2116e79f70e7bd9`
- Acquisition route reported by developer: `sgx540-reversing` (exact repository identity and revision **PENDING**).

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

**Classification:** `VENDOR-ORACLE`, `HISTORICAL-BINARY`, and project-authored research shim; exact acquisition and redistribution rights `PENDING`.

See also `docs/phase9/USSE-VENDOR-ORACLE-20261027.md` and the separately supplied `SGX535-rev121-USSE-disassembler.zip`.

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
