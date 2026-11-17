# reMESA — Source Provenance, Reverse Engineering and Audit

> **Scope:** Public research provenance record; records inspected sources, implementation boundaries and evidence limits
> **Project:** [reMESA](https://github.com/reMESA-devgroup/reMESA)  
> **Primary target:** PowerVR SGX535, Intel Poulsbo, revision 121  
> **Reference device:** Dell Inspiron Mini 12 (Intel Atom Z520)  
> **Document date:** 2026-11-17

## 1. Purpose and scope

This document records the origins, use, restrictions and uncertainty associated with the historical source code, vendor binaries, research tools, AI assistance and experimental evidence consulted during reMESA development. It is intended to help maintainers and prospective reviewers trace technical claims to their supporting material.

This document is a provenance record, not a legal opinion or independent clean-room certification. Public availability does not itself grant reuse or redistribution rights. A successful hardware experiment does not establish copyright provenance.

The provenance of the SGX535 driver and the separate SGX544 research effort is recorded separately. See `projects/sgx544/docs/PROVENANCE.md` for the latter.

## 2. Research and implementation boundaries

The project aims to:

1. Identify historical material by repository, commit, path, version and hash.
2. Distinguish historical facts, inferred architecture, offline models and physical GPU observations.
3. Keep vendor implementations and restricted materials separate from new driver implementation.
4. Preserve original copyright and licensing notices.
5. Record LLM assistance rather than implying exclusively human authorship.
6. Avoid treating decompiled vendor code, model output or a public mirror as automatically reusable source.
7. Record questions for which the cited evidence does not establish an answer.

These are stated policies and documented research practices; they do not certify every prior session.

### Classification vocabulary

| Class | Meaning |
|---|---|
| `PUBLIC-OPEN` | Identified public source with a recorded open-source license; exact file and revision still matter |
| `PUBLIC-RESTRICTED` | Publicly obtainable material subject to additional restrictions |
| `MIRROR-UNVERIFIED` | Community-hosted copy not authenticated as an official vendor release |
| `VENDOR-ORACLE` | Original vendor software queried for comparison or interpretation |
| `HISTORICAL-BINARY` | Historical executable or library inspected as evidence |
| `QUARANTINED` | Material excluded from implementation use; rights/provenance are not asserted |
| `ORIGINAL-IMPLEMENTATION` | reMESA-authored implementation; provenance is described by the available records |
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

**Licensing qualification:** Official TI hosting and an open-license declaration on one file do not automatically establish the rights for every historical revision or neighboring file. The kernel DDK is distinct from proprietary user-mode components. Individual notices, including historical confidentiality markings, are source-specific. Historical source is recorded as research evidence, not presumed to have been transplanted into the new driver.

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

The mirror is a research source, not independently authenticated as an Intel release. Its files carry mixed source notices. Public mirroring does not establish unrestricted reuse.

### 3.5 TI user-mode DDK

- Official host: <https://git.ti.com/cgit/graphics/omap5-sgx-ddk-um-linux/>
- Recorded commit: `b6801bf89e00d69893c2957455bbeb3195c0ed52`
- Example binary: `targetfs/lib/libsrv_init.so.1.9.6.0`
- Recorded SHA-256: `a1a3572ee4ae6104505808d171d6b4cd023309959588f890b2192aea44fafdda`

This is an OMAP5/DRA7xx artifact, not an authenticated Poulsbo userspace stack. The project records metadata inspection; this record makes no redistribution or implementation-reuse claim.

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

The historical [Garnet/sgx540-reversing repository on Codeberg](https://codeberg.org/Garnet/sgx540-reversing) documents an `asm/preload_hack.c` utility for invoking the internal Series5 USSE disassembler in Imagination Technologies' official `GLSLESCompiler_Series5` executable through `LD_PRELOAD`.

The Codeberg repository history identifies commit [`03eadc4b54ab61e1816a9185bf20e543fdfb804a`](https://codeberg.org/Garnet/sgx540-reversing/commit/03eadc4b54ab61e1816a9185bf20e543fdfb804a), “Fix up the names for the x86 preload hack.” This technique predates reMESA and was not originally developed by reMESA or generated by its LLM-assisted workflow.

The reMESA developer reports that the research shim used by reMESA was not generated by AI. The surviving record does not establish a source-level derivation relationship between historical `preload_hack.c` and reMESA's `sdk35_decoder_preload.c`; they are therefore listed as distinct artifacts. The historical repository's notices and license remain attached to its files.

The shim and the decoder are distinct components. The shim invokes functionality within the original Imagination executable; it does not independently implement the vendor's USSE decoder.

The canonical repository is Codeberg at `https://codeberg.org/Garnet/sgx540-reversing`; the full commit above verifies the cited repository-history reference. Copyright notices and licensing terms for the shim are those recorded per file in that repository; no broader license conclusion is made here.

The supplied `SGX535-rev121-USSE-disassembler.zip` identifies the executable actually invoked as an **original Imagination Technologies Series5 GLSL ES compiler from PowerVR SDK 3.5**, Linux x86 32-bit:

- Included executable: `GLSLESCompiler_Series5-sdk35`
- Vendor SDK URL recorded in the package: <https://cdn.imgtec.com/sdk/OFFLINE/PowerVRSDKSetup-3.5.run-x86>
- Executable SHA-256 recorded in the package: `9349ed4d7643d81cb638b41eb41b69507cc2f9f15bae14c8f2116e79f70e7bd9`
- Acquisition route reported by developer: [Garnet/sgx540-reversing on Codeberg](https://codeberg.org/Garnet/sgx540-reversing), commit `03eadc4b54ab61e1816a9185bf20e543fdfb804a`.

The SDK URL is an **origin claim recorded by the packaged README**, not proof that the particular executable in the archive was independently downloaded from that URL. The developer's acquisition account and the package's SDK-origin statement are distinct provenance records.

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

### 5.4 Distribution and implementation scope

The supplied ZIP contains the vendor executable and bundled runtime libraries. The included glibc GPL/LGPL notices govern those components; they do not license the Imagination compiler. The inspected records do not state the compiler's distribution terms or independently authenticate the ZIP's acquisition route. They also do not establish a binary-level match to an official SDK release, the complete use history of oracle outputs as test vectors or LLM inputs, or source-to-implementation similarity. Those are evidence limits, not conclusions about infringement or permission.

Recorded classifications: `VENDOR-ORACLE`, `HISTORICAL-BINARY`, and a reMESA research shim using a technique documented in the historical Codeberg project. These labels describe roles and do not determine source derivation, license scope or redistribution rights.

See also `docs/phase9/USSE-VENDOR-ORACLE-20261027.md`

### 5.5 USSE-specific sources consulted, one by one

This source ledger separates the SGX535/rev121 vendor decoder used for bounded CPU interpretation from public research leads and from driver/OS documentation. A source's presence here does not mean its code or instruction constants were used in reMESA.

| Source | What was consulted | Recorded role and result |
|---|---|---|
| Imagination PowerVR SDK 3.5 Linux x86 compiler, `GLSLESCompiler_Series5-sdk35` | Original executable identified in `SGX535-rev121-USSE-disassembler.zip`; SHA-256 `9349ed4d7643d81cb638b41eb41b69507cc2f9f15bae14c8f2116e79f70e7bd9`; vendor archive URL recorded by the package: [PowerVRSDKSetup-3.5.run-x86](https://cdn.imgtec.com/sdk/OFFLINE/PowerVRSDKSetup-3.5.run-x86) | Queried as original vendor USSE decoder with descriptor `[4, 121, 0x40128f80, 0, 0x082d2800]`. The package's URL statement is an origin claim, not independent proof of the particular binary's download path. No vendor executable/source is incorporated into the driver or this repository. |
| `sdk35_decoder_preload.c`, `sdk35_decoder_preload.so`, `disassemble.py`, bundled i386 runtime, `hashes.json`, `SHA256SUMS`, `README-PT.txt`, `example_nop_end.bin` | Contents listed in the supplied decoder ZIP; shim behavior and package manifest described by `docs/phase9/USSE-VENDOR-ORACLE-20261027.md` | Research harness calls decoder VA `0x0829921b` inside the vendor executable, validates descriptor at `0x082d2a88`, input length, output canaries and termination. It runs CPU-side under QEMU i386 with a 15-second launcher timeout. These files are oracle tooling/artifacts, not a standalone decoder implementation. |
| Garnet `sgx540-reversing` (Codeberg) | [Repository](https://codeberg.org/Garnet/sgx540-reversing), commit [`03eadc4b54ab61e1816a9185bf20e543fdfb804a`](https://codeberg.org/Garnet/sgx540-reversing/commit/03eadc4b54ab61e1816a9185bf20e543fdfb804a), “Fix up the names for the x86 preload hack”; `asm/preload_hack.c` and project documentation | Historical Series5 decoder-invocation technique and attribution lead. Its stated target/example context is SGX540; the reMESA ZIP selects its own SGX535/rev121 descriptor. A source-level derivation between this shim and `sdk35_decoder_preload.c` is not established by the retained record. |
| SDK35 decoder internal analysis | `docs/phase9/USSE-DECODER-CATALOG-20261028.md`; retained REA/Ghidra snapshot referenced by that report; 171 reachable internal routines, 18,959 host x86 assembly lines | Describes the vendor decoder implementation, dispatcher, formatter and helper paths. The x86 assembly and Ghidra pseudocode describe host decoder code, not GPU instructions or original vendor source; they are analysis evidence and are not imported into production. |
| SDK35 bounded decoder outputs | `USSE_CPU_DECODED.tsv`, `RECORDS.json`, raw batches and `RESULT.json` under the catalog directory; initial 8,192 matrix records plus 33,170 mnemonic-guided probes | 41,362 checked CPU records: 39,446 decoded texts, 1,916 decoder errors and 169 distinct sampled mnemonic spellings. The configured mutations are not an enumeration of all 64-bit words. Synthetic words stayed on CPU; decoder output names byte patterns but does not establish hardware execution. |
| Original decoder operation dictionary | SDK35 table at internal VA `0x082ceec0`, documented in the decoder catalog | 370 records / 368 distinct names; flags are uninterpreted and operand counts feed formatting. IDs are internal dictionary indexes, not hardware opcode values. The catalog also records 58 error names. |
| Original decoder wrapper and sample records | Decoder VA `0x0829921b`; catalog examples `00000000 f8000140 → nop, return 0` and `00000000 f8040140 → nop.end, return 1` | Return value is false for decoder errors and otherwise reflects an internal flag bit; it is not a hardware-acceptance or execution result. Reports preserve it as uninterpreted except where the catalog explains its wrapper behavior. |
| Generated-code comparison against vendor decoder | `docs/phase9/USSE-VENDOR-ORACLE-20261027.md`; `audit_generated.c` calls `remesa_usse.c` and `remesa_usse_arithmetic.c` | 17 cases / 122 instructions were decoded, including arithmetic, parameter operations, MVP bodies, gears bodies, DOT3, reciprocal square root and selected control/event forms. This compares existing CPU encodings with vendor text; it does not qualify resource delivery, whole-program semantics or GPU behavior. |
| Texture/sample decoder follow-up | `docs/phase9/TEXTURE-SMP-ORACLE-20261028.md`, `docs/phase9/TEXTURE-STATE-ORACLE-20261028.md`, and `docs/phase9/USSE-SAMPLE-BODY-20261028.md` | Reports selected SMP/WDF fields and a bounded body comparison in the same SGX535/rev121 oracle context. The records explicitly leave sampler-state fetch footprint, register liveness, task resources, synchronization, retirement and physical sampling outside that decoder result. |
| TI/Imagination DDK SGX535 headers and kernel contracts | TI `omap5-sgx-ddk-linux`, commits `322bcda5f3076037e2e20ef9209f4f4d575a7d5f` and `cb46ba4d0c900f89f7ec0284f9803d476bfa98de`; especially `sgx535defs.h`, `sgxfeaturedefs.h`, `sgx_mkif_km.h`, `sgxconfig.h`, `sgxerrata.h`, `sgxreset.c`, `sgxinit.c`, `sgxinfo.h`, `sgx_bridge.h`, `sysconfig.c` | Establish core/revision, feature, reset, bridge, heap and USE/EDM integration context. These are kernel/DDK contracts, not a complete SGX535 USSE instruction-set specification. File identities and digests appear in the Phase 7 and Poulsbo tables below. |
| Linux GMA500 register definitions | `gregkh/linux` commit `d5eca7ebcf6f64c4aebf9684c365c130a3a069b3`, `drivers/gpu/drm/gma500/psb_reg.h`, plus driver initialization | Supplies USE base and PDS execution-base register declarations and initialization values. It documents register interface evidence, not PDS or USSE instruction encoding. |
| Historical proprietary Poulsbo binaries | `xpsb-glx` 0.18: `psb_dri.so` SHA-256 `74ca42991906741ee91be1bc088ae0b142fa71b6a85658c5dad0851ce50dd0d8`, `Xpsb.so` SHA-256 `da531587b1ec59fe433fa20ddd2e691b2f8b7fb35bb82525d4db99259c5e571f`; reports in `docs/phase7/psb-dri-re/` and `docs/phase7/xpsb-re/` | Static analysis records CPU-side historical compiler/command construction and generated USE/USSE buffers. The reports distinguish them from PDS buffers and do not claim a complete instruction specification or physical validation. Package restrictions and file-specific details are in `docs/phase7/*/source-provenance.md`. |
| Public patent US20090063824A1 | [Publication](https://patents.google.com/patent/US20090063824A1/en), [PDF](https://patentimages.storage.googleapis.com/85/e0/fa/15b48f6d6f3fc2/US20090063824A1.pdf), Figures 4–5 and paragraph 0037; PDF SHA-256 `dc6a81b3944c5a1fc0b01e5d3d16c41fa563ae6af0b904d9571333edeeebe0da` | Public generic multithreaded CPU example was inspected as HTML/PDF, including figure images. It is not identified as SGX/USSE/PDS/rev121; no fields were admitted to the encoder and no copying or patent-practice license is inferred. |
| Public patent EP2140347A1 | [Publication](https://patents.google.com/patent/EP2140347A1/en), Figures 4–5 and final-thread hazard discussion | Generic load/wait/hazard-counter examples were screened. SGX535 binding, applicable opcode values and operand fields are not supplied; no instruction fields were admitted. |
| Public patent US20040030864A1 | [Publication](https://patents.google.com/patent/US20040030864A1/en) | Screened for fixed-width instruction-template discussion in a general media processor. No SGX535 USSE/PDS identification or applicable opcode table; no fields or diagrams imported into implementation. |
| Public patent US20070101013A1 | [Publication](https://patents.google.com/patent/US20070101013A1/en), [PDF](https://patentimages.storage.googleapis.com/0e/29/64/5d9241714818a2/US20070101013A1.pdf) | Published HTML and public PDF consulted. Contemporaneous scalable multithreaded media/tile-rendering architecture; inspected body does not identify SGX535/rev121. Factual paraphrase only; no source, algorithm, diagram or instruction word imported. |
| EP1987422A1 | [Publication](https://patents.google.com/patent/EP1987422A1/en) | Search-result metadata only; patent body not opened in the recorded field-research pass. No instruction fields admitted. |
| US20080010640A1 | [Publication](https://patents.google.com/patent/US20080010640A1/ja) | Search excerpts only; body not opened. No instruction fields admitted. |
| US8743117B2 | [Publication](https://patents.google.com/patent/US8743117B2/en) | Search metadata/excerpt only; body not opened. No instruction fields admitted. |
| Public patent US20100241823A1 | [Publication](https://patents.google.com/patent/US20100241823A1/en) | Opened incidentally after a mistyped candidate identifier, as recorded in `usse-source-review-20261009.json`; no fields admitted and no implementation use. |
| Public patent US20220261248A1 / GB2556886A | Search excerpts only, recorded in `sgx5-field-research-20261009.json` | Variable-length instruction architecture lead; no Series5 target binding, source body not opened, no field admitted. |
| GNU PowerVR project and historical wiki | [LibrePlanet project page](https://libreplanet.org/wiki/Group:PowerVR_drivers), [mailing-list message](https://lists.goldelico.com/pipermail/openpvrsgx-devgroup/2021-November/000476.html), [Savannah](https://savannah.nongnu.org/projects/powervr/), [lkcl.net SGX page](http://lkcl.net/powervr/sgx/), archived [opcodes](https://web.archive.org/web/20130225101912/http://powervr.gnu.org.ve/doku.php?id=opcodes) and [instructionencoding](https://web.archive.org/web/20151004182955/http://powervr.gnu.org.ve/doku.php?id=instructionencoding) | Project/mailing-list metadata and search excerpts were reviewed; the instruction-table bodies were not fetched in the 2026-10-09 source review. Original methodology, licensing of instruction-format material and exact GPU revision are not established by that record; no numeric fields admitted. |
| Linux-sunxi SGX544 mirrors | [USSE_ISA](https://linux-sunxi.org/SGX544MP/USSE_ISA), [USP_OPCODE](https://linux-sunxi.org/SGX544MP/USP_OPCODE) | Search excerpts only in source review. Pages/tables were not opened there. SGX544MP label does not establish SGX535/rev121 compatibility; no numeric field admitted. |
| Project Cascadia | [rzxvx/Project-Cascadia](https://github.com/rzxvx/Project-Cascadia) | Repository metadata and README excerpts only; target identified in the record as Apple A5 SGX543MP2. Encoder/shader/PDS code was not opened; no field admitted. Top-level GPL metadata does not establish derivation or license of individual shader facts. |
| open-shacccg | [frangarcj/open-shacccg](https://github.com/frangarcj/open-shacccg) | Search excerpts surfaced; repository/code/payloads were not opened or fetched. The record identifies Vita SGX543+ USSE2 and compiler-oracle/GXP lineage; no bytes or fields were transcribed or used. |
| Reichel thesis bibliography | [University-hosted PDF](https://uol.de/f/2/dept/informatik/ag/svs/download/thesis/Reichel_Sebastian.pdf) | Bibliography excerpt only; PDF not opened. Used solely as a historical pointer to GNU opcode material, not as ISA or licensing evidence. |
| freemangordon Mesa PVR/TI branch | [mesa-pvr-ti](https://github.com/freemangordon/mesa/tree/mesa-pvr-ti), linked from the [openpvrsgx-devgroup message](https://lists.goldelico.com/pipermail/openpvrsgx-devgroup/2021-November/000476.html) | Mailing-list pointer only in the source review; repository/code not opened there. N900 user-space lead; independent shader implementation, target revision and component licensing are not established. |
| Lund thesis PDF | [Rapport_2008-08.pdf](https://fileadmin.cs.lth.se/cs/education/Examensarbete/Rapporter/2008/Rapport_2008-08.pdf) | Search excerpt only; PDF not opened or downloaded. Reported disassembler/manual provenance and SGX535 identity were not established. |
| Meta HTP instruction PDF | [meta_htp_insns.pdf](https://docs.alexrp.com/metag/meta_htp_insns.pdf) | Search excerpt only; Meta CPU ISA is outside SGX535 USSE/PDS scope. |
| Mesa modern PVR `pvr/csbgen` documentation | Search-result release metadata recorded in `sgx5-instruction-followup-20261009.json` | Source was not opened in that inquiry; modern PVR material is not evidence of Series5 compatibility. |
| PowerVR forum PDS programming thread | [PDS programming](https://forums.imgtec.com/t/pds-programming/333) | Search snippets surfaced in the instruction follow-up; separately, the historical public exchange was consulted for the exact document title and access context. It records a user statement and vendor response directing discussion to email; it does not establish public distribution or recover the cited PDF. |
| Vita graphics wiki | Search snippets surfaced in `sgx5-instruction-followup-20261009.json`; no page body opened | Cross-generation lead only; no instruction constants incorporated. |
| `Eurasia.3D Input Parameter Format.1.3.37a.SGX535 1.2.External.pdf` | Exact filename recorded in `docs/phase7/psb-dri-re/sgx535-pds-document-hunt.md` | The document itself was not recovered in that hunt. The forum post and withdrawn USITC exhibit index do not establish that this revision was publicly distributed. |
| USITC exhibit index | [Publication 4374, volume 2 PDF](https://www.usitc.gov/intellectual_property/documents/pub4374_vol_2_of_2.pdf) | Index lists an `Eurasia 3D Input Parameter Format` exhibit, without a revision match or public-release proof; the exhibit is marked withdrawn in the cited hunt. |
| Mesa 7.4.4, libdrm-poulsbo 2.3.0, xserver-xorg-video-psb 0.32.0 and PSB kernel 4.41.1 | Exact package hashes and notices in `docs/phase7/xpsb-re/source-provenance.md` | Public/historical software comparators for ABI, layout and platform behavior. They are not treated as an SGX535 ISA specification; file-by-file provenance remains in the source record. |

The full structured source screens and their access descriptions are [`usse-source-review-20261009.json`](docs/phase9/usse-source-review-20261009.json), [`sgx5-instruction-followup-20261009.json`](docs/phase9/sgx5-instruction-followup-20261009.json), and [`sgx5-field-research-20261009.json`](docs/phase9/sgx5-field-research-20261009.json). The local reports cited above provide the run-specific artifact paths, hashes, commands and evidence identifiers.

## 6. Restricted and excluded research materials

### Canonical `libgl1-mesa-dri-psb`

- Version: `0.24+repack+0038.1`
- Historical package context: Canonical `hardy-dell-mini`
- Package SHA-256: `14e870f4273e84759630499efeda4948116c29f3ba7d08e2b5f028a5da3ef8a2`
- Extracted ELF SHA-256: `427627909b0c528d154d1b247688f16536162a520d43d472e8a4f404c02e2a6e`

The research report records an `Intel Confidential / all rights reserved` notice. It says this package remained in disposable quarantine, was not executed, was not added to `references/`, and was not used for implementation details. Limited static occurrence/provenance checks were recorded.

**Classification:** `QUARANTINED`. A confidentiality marking alone does not prove unauthorized publication.

### PSP2 PowerVR repository — inspected, not used

The public `GrapheneCt/PVR_PSP2` repository was briefly checked as a possible PDS decoder locator. The inspected `sgxpdsdefs.h` carried restrictive Imagination copyright language. According to the project research record, no source from this repository was imported, used to decode selected SGX535 instructions, or treated as evidence for Poulsbo hardware behavior.

**Classification:** Excluded research lead; **not an implementation dependency**. It is included for transparency; the project record says no PSP2 source was used.

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

Evidence scope recorded in this document:

| Evidence | Supports | Does not automatically support |
|---|---|---|
| Historical source | Properties of the inspected revision | Identity of installed binaries |
| Vendor decoder text | Vendor interpretation of input words | Correct physical GPU execution |
| Offline model | Behavior of the tested software model | Real hardware correctness |
| Physical readback | Output of that particular experiment | General application compatibility |
| Unit-test pass | Tested software invariants | Copyright provenance or upstream eligibility |

Use `CONFIRMED`, `INFERRED`, `UNKNOWN`, `PASS`, and `FAIL` with explicit test scope. A one-draw result is not a claim of continuous rendering or OpenGL conformance. Historical reports may use `BLOCKED` to describe a specific experiment's recorded state.

## 8. AI-assisted development disclosure

reMESA has used extensive LLM assistance for source archaeology, searching and comparing code, documentation, implementation and debugging. Human maintainers remain accountable for submissions and claims.

This document does **not** assert that model training data can be reconstructed or that generated code is necessarily free of proprietary influence. In particular, the use of vendor-decoder output must be accounted for in any source-to-implementation audit.

The project records the following provenance attributes for significant components where source-session records survive:

- Current file and implementation commit.
- Source references and exact license scope.
- Known LLM tools and input context, where records survive.
- Whether vendor or restricted outputs were provided to a model.
- Human review and independent explanation of the implementation.
- Similarity review against restricted historical material.
- Offline tests and physical qualification evidence.
- Evidence attributes not established by surviving records.

Where a surviving record does not establish an attribute, this document marks it `UNKNOWN` rather than inferring it.

## 9. Primary project records

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

## 10. Final statement

reMESA maintains substantial records of historical sources, binary identities, hardware experiments and AI-assisted development. These are valuable foundations for independent review. They are **not** proof that all implementation code has been audited, that every historical artifact may be redistributed, or that no restricted material influenced AI-assisted changes.

The Imagination USSE decoder is accurately described as **original vendor functionality invoked through a research shim**. The developer reports an acquisition route through [Garnet/sgx540-reversing on Codeberg](https://codeberg.org/Garnet/sgx540-reversing); the included package separately identifies the original PowerVR SDK 3.5 executable. These are distinct provenance facts.

This record describes the inspected evidence and its limits; it does not certify that every implementation file has an independently verified clean-room history.

---

*Maintenance: update material claims with source identifiers, commits, hashes, experiment IDs or review records. This document does not alter third-party licenses.*

## 11. Implementation-level provenance and AI-assisted development audit

**Audit snapshot:** repository `main` at `66247b5927f312beb6f6abbd867fbfb539f9be7a`; working-tree audit performed against that revision. The map below follows available source, experiment, Git and session records. “Source origin” means evidence for the recorded CPU-side transformation; it does not imply that an opcode's hardware semantics have been independently specified.

### 11.1 USSE encoder field lineage

The principal implementation units are [`mesa/sgx535/remesa_usse.c`](mesa/sgx535/remesa_usse.c), [`mesa/sgx535/remesa_usse_arithmetic.c`](mesa/sgx535/remesa_usse_arithmetic.c), their headers and the corresponding `mesa/sgx535/tests/test_usse_*.c` tests. Numeric positions in these units are deliberately described as CPU writer fields where that is all the evidence establishes. The vendor decoder supplies sampled text for selected bytes; it does not independently prove every writer mask or physical meaning.

| Implemented family / functions | Numeric fields and construction evidence | Experiment, vendor-oracle and commit records | Scope of claim |
|---|---|---|---|
| Host semantic IR, validation and serialization: `remesa_usse_validate`, `remesa_usse_lower`, `remesa_usse_serialize_host`, `remesa_usse_deserialize_host`, `remesa_usse_identity` in `remesa_usse.c` | Host-only schema, operation tags, 24-byte host records and checksum. These are reMESA serialization choices, not SGX535 instruction encodings. | Introduced in [e2db5cd](https://github.com/reMESA-devgroup/reMESA/commit/e2db5cd180dcd6f3bebd16ee73773c4cf63474c3); semantic alias/constant follow-ups [6e159a4](https://github.com/reMESA-devgroup/reMESA/commit/6e159a459bd16c5cc260d150fa2f3b52b6bde6ff), [a85d062](https://github.com/reMESA-devgroup/reMESA/commit/a85d062828c40b0944f983ea30e9fc92e16c1083). | Host representation and deterministic validation only. |
| Ordinary CPU MOVE: `remesa_usse_cpu_move_encode/decode` | Two little-endian DWORDs. Writer evidence in retained code comments attributes source number to W0 bits 13:7, destination number to W0 bits 27:21, source class to W0 31:30, fixed W0 bit29; destination class W1 1:0; fixed major/group and controls in W1 bits 31:27, 23, 21, 18, 16, 15:12, 11. Decoder masks reject unmodelled bits. Evidence locator is the historical x86 writer cited by the implementation; the currently retained general oracle report does not list a direct MOVE text row. | Added in [0700034](https://github.com/reMESA-devgroup/reMESA/commit/070003454e7f938661170abcc57a551ae68c931c); later admitted gear parameter cases in [190724c](https://github.com/reMESA-devgroup/reMESA/commit/190724cabc3db375549909d5d0333eee182af334). Existing independent valid-frontend POSITION replay is recorded in `PDS-USSE-COMPLETION-20261021.md`; historical writer reference in code is `psb` ELF `0x250aeb`. | CPU-side writer correspondence. Register classes/numbers are not thereby physical SGX register banks. Exact origin relationship of the small reverse codec to each source helper is recorded in source comments, not as vendor instruction semantics. |
| Three-source CPU codec: `remesa_usse_cpu_three_source_encode/decode` | W0 source numbers at shifts 14, 7, 0; source1/source2 numeric classes at 30/28; controls 21 and 22–27. W1 fixed numeric pattern (major 31, subgroup 3, control bit21, fixed bit14), source0 class at 2, extensions at 19/17/16, low controls 1:0, controls 13:12 and 18. The exact masks and allowed ranges are in the two functions. | `docs/phase9/PDS-USSE-PRIORITY-20261020.md` attributes writer to original x86 `0x30a09`, allocator `0x2fabd`, `0x40a09` and four helpers; it records a fresh 827-byte producer audit, 14 numeric fields, two independent historical oracle records, 282 checks per native/UBSan/i386 variant and 23 fixed/unmodelled-bit rejections. Later complete-position replay is in `PDS-USSE-COMPLETION-20261021.md`. The current reachable blame attributes this implementation to [190724c](https://github.com/reMESA-devgroup/reMESA/commit/190724cabc3db375549909d5d0333eee182af334); the source lineage report and checkout do not expose a separate introduction commit for every subsequent line. | Establishes selected original CPU writer layout and byte production, not generic three-source semantics, physical banks, VS export or END. |
| Control-EE and event records: `remesa_usse_cpu_control_ee_encode`, `remesa_usse_psb_event_end_encode`, `remesa_usse_psb_event_transfer_encode` | EE: zero W0 and W1 numeric major31, bits 8/6 plus profile controls 11 and 18. Event-end: zero W0 and W1 major31/subgroup3 and selected bits 15/18. Transfer: 16 bytes, two CPU-constructed pairs; mode-derived `t=(mode<<6)|0x204` contributes through masks `0x0fc00000`, `0x00c00000` and `0x1f8`. Implementation comments identify original writer/finalizer locations `0x252d32..0x252d8a`, allocator `0x2fabd`, writer `0x30422`, finalizer `0x30fb7`, and arithmetic producer `0x391e0`. | The event records were committed in [0da1be9](https://github.com/reMESA-devgroup/reMESA/commit/0da1be991cb3e93d5f43ea2359e8f085a3c843ba). Event resource/auxiliary integration appears in [443e134](https://github.com/reMESA-devgroup/reMESA/commit/443e134a95b9bbc5596bb901d261d8e98f837c39), [c8c0f3c](https://github.com/reMESA-devgroup/reMESA/commit/c8c0f3c011bd10eca9c6d2798684be19305407ae) and [cd689f1](https://github.com/reMESA-devgroup/reMESA/commit/cd689f189e85fe76e2ed2428a26f35192b2888a0). | Values reproduce bounded legacy CPU producers and selected event chains. The event record names do not by themselves prove all hardware execution/retirement semantics. |
| Scalar ADD/MUL/MAX, DOT3 and RSQ: `remesa_usse_arithmetic_encode`, `remesa_usse_arithmetic_sources_encode`, `remesa_usse_lighting_body_encode`, `remesa_usse_cpu_arithmetic_encode` | ADD/MUL use normalized numeric source slots, class-6 constants 52/48, flags/repeat/destination fields; MAX uses a selected major/subgroup and two source helpers; DOT3 uses selected major2/sub0; RSQ uses major1/sub1 plus numeric modifier. Bit placements and masks are in `remesa_usse_arithmetic.c`; comments cite historical normalizer/writers/helpers including `0x249ccf`, `0x24b72d`, `0x24ad4a..58`, `0x24bb09/5d`, `0x24b1c7`, `0x2410df`, `0x241580` and related source helpers. | The SDK35 generated audit reports original-decoder spellings for selected ADD/MUL (`fmad...`), MAX (`fmax...`), DOT3 (`fdp...`) and RSQ (`frsq...`) samples, with exact source hashes and bytes in the offline audit artifacts cited by `USSE-VENDOR-ORACLE-20261027.md`. Initial selected codec implementation is attributed by current reachable blame to [190724c](https://github.com/reMESA-devgroup/reMESA/commit/190724cabc3db375549909d5d0333eee182af334); the final gears body integration is recorded by [190724c](https://github.com/reMESA-devgroup/reMESA/commit/190724cabc3db375549909d5d0333eee182af334). | Vendor text corroborates selected emitted encodings. It does not establish arithmetic precision, all legal encodings, execution, or source provenance of every helper-derived numeric field. |
| Parameter, first-term and component-control bodies: `remesa_usse_cpu_parameter_vertex_encode`, `remesa_usse_cpu_parameter_mad_vertex_encode`, `remesa_usse_cpu_parameter_first_term_vertex_encode`, `remesa_usse_component_control_encode` | Reuse the admitted arithmetic/MOVE/three-source writers, with constrained bases and selected control fields. The first-term record is built from the component-control writer; its numeric controls are grounded in observed frontend/linker arguments rather than mnemonic inference. | `USSE-VENDOR-ORACLE-20261027.md` reports selected parameter ADD/MUL/MAD and first-term `smlsi` decoder text. Full original frontend/setter/linker/builder POSITION trace and the 16-byte CPU output are recorded in `PDS-USSE-COMPLETION-20261021.md`; parameter/gears follow-ups in `docs/phase9/`. | Exact sampled CPU bytes and selected decoder spelling; parameter contents, launch path and physical interpretation remain separate evidence. |
| MVP/POSITION and lighting/gears bodies: `remesa_usse_cpu_mvp_position_body_encode`, `remesa_usse_cpu_gears_vertex_body_encode`, `remesa_usse_cpu_white_lambert_vertex_encode`, `remesa_usse_cpu_color_output_clamp_encode`, normalization and position/color builders | These are bounded typed constructors composed from the prior codecs plus literal numeric controls. Exact fixture/profile values are visible in each constructor and corresponding tests; they are not a general-purpose assembler. | `USSE-VENDOR-ORACLE-20261027.md` records selected MVP coefficient names and complete gears-body text; [0700034](https://github.com/reMESA-devgroup/reMESA/commit/070003454e7f938661170abcc57a551ae68c931c) records the `ALWAYS` candidate; clamp validation/integration commits [db35d4f](https://github.com/reMESA-devgroup/reMESA/commit/db35d4f06b26c021b3964370f02624ba7ee7de5f), [6c5d845](https://github.com/reMESA-devgroup/reMESA/commit/6c5d8451c8a3aea80468193d15ec5f315c462583), [b0cb162](https://github.com/reMESA-devgroup/reMESA/commit/b0cb162b174f5dc005803a95b6c65046158212bf). Related test names include `test_usse_mvp_position_body.c`, `test_usse_gears_vertex_body.c`, `test_usse_lighting_body.c`, `test_usse_normalize_vertex.c`, `test_usse_position_color.c`. | Fixed inputs / admitted IR profile only. `PDS-USSE-COMPLETION` explicitly keeps physical banks, DATA-to-register delivery, export and termination `UNKNOWN`. |
| SMP2D + WDF sample body: `remesa_usse_sample_2d_body_encode` in `remesa_usse_sample.c` | SMP word0: destination shift21, coordinate shift14, sampler shift7, fixed bank3 at30; word1 major28, channel mask at12, dimension at10, coordinate bank bit2 and DRC 1:0. WDF: major31, selector shift20, suboperation shift24, same DRC 1:0. Field ranges are encoded explicitly. | `TEXTURE-SMP-ORACLE-20261028.md` and `USSE-SAMPLE-BODY-20261028.md`; REA evidence IDs `ev_a33cca...` (SMP parser), `ev_2cd991...` (WDF parser), helpers `0x08289d26`/`0x0828a1cc`. Native/UBSan/i386 evidence in that report: 52,545 checks per variant; 2,916 tuples and oracle checks over 5,832 instructions. Commit [4d741a3](https://github.com/reMESA-devgroup/reMESA/commit/4d741a304bb707a819acd91bb5445868d1c8380e). | Oracle text validates sampled field-to-text generation in SGX535/rev121 context. It does not prove register allocation windows, sampler footprint or physical texture execution. |

**Opcode/mask completeness:** This table covers the encoder families identifiable in the current SGX535 USSE units and named source records, but does not claim each bit has a fully independent semantic proof. Where an exact source-side writer/helper citation or a per-case test receipt is absent from the committed tree, the source location remains the authority and the external experiment detail remains `UNKNOWN` from this checkout. The vendor decoder's shared mnemonic table is not used as a hardware opcode-number table.

### 11.2 Shim authorship, source comparison and licensing record

The Codeberg project was cloned read-only; the inspected HEAD is `c15884751ad870b3e06abd3d8b5a6ab00e2fb3cd`. Git history attributes `asm/preload_hack.c` to the project account Garnet; its file history includes commit `03eadc4b54ab61e1816a9185bf20e543fdfb804a` and earlier `3ab08a51698359af83c7622505ef3e69d554a3fe`. The file defines an SGX540 context with type 5, revision 101 and flags `0x16b800`; accepts one USSE file path, reads it, calculates `size / sizeof(uint32_t)` (despite its comment that file size is divisible by eight), and calls wrapper VA `0x082994cb`. The README says it is intended for `LD_PRELOAD` with Imagination's Series5 compiler. The checked repository tree has no top-level license file and `preload_hack.c` carries no SPDX or copyright header. The recorded commit author identity and absence of a license file do not establish copyright ownership or grant a reuse license.

The reMESA audit record describes a separate package source `sdk35_decoder_preload.c` that hooks `__libc_start_main`, verifies a five-word descriptor at `0x082d2a88`, selects SGX535/rev121 (`[4,121,0x40128f80,0,0x082d2800]`), and invokes inner decoder VA `0x0829921b` one eight-byte instruction at a time with length/canary/termination checks. This differs at the reported design level from Garnet's SGX540/rev101 fabricated context, wrapper call and file-size/4 loop. It is an intentional documented behavioral distinction, not a source diff: the packaged `sdk35_decoder_preload.c` was not present in the repository or the inspected local source tree during this audit, so byte-for-byte comparison, authorship metadata, independent creation date and exact derivation are `UNKNOWN`. The prior developer statement that the reMESA shim was not AI-generated is a statement, not a preserved authorship record. No license text for the Codeberg file was found; no permission to copy, modify or redistribute it is inferred.

### 11.3 PowerVR SDK 3.5 terms located

Imagination's current official [PowerVR SDK Software EULA](https://developer.imaginationtech.com/terms/powervr-sdk-end-user-licence-agreement/) presents the MIT license for SDK software/source, permits copying, modification and distribution subject to retaining the copyright and permission notice, and its FAQ says the PowerVR Framework and example source may be distributed under MIT. Imagination's [downloads page](https://developer.imaginationtech.com/downloads/) lists legacy SDK v3.5 and instructs users to read the applicable EULA before download. These official pages are current pages; the inspected page does not identify its effective date or state that its terms retroactively govern the 2014-era SDK 3.5 compiler binary.

A historical SDK EULA text was located in the `third_party/PowerVR_SDK/License.txt` file vendored by [SwiftShader](https://swiftshader.googlesource.com/SwiftShader/+/refs/heads/legacy-gles1/third_party/PowerVR_SDK/License.txt). That text describes a non-exclusive/non-assignable development license and conditional distribution of SDK components with an application, while restricting stand-alone distribution and stating restrictions on modification, disassembly, decompilation and reverse engineering; it also requires attribution. This is a third-party retained copy, not the official SDK 3.5 installer or an authenticated official v3.5 license bundle. Public third-party copies and current official web terms are not enough to identify which agreement accompanied the exact SDK 3.5 binary in the supplied ZIP.

The package URL recorded by the reMESA README points to the official SDK 3.5 installer, but the archive/binary's independent acquisition path and the exact click-through or bundled terms accepted for it are not preserved in this repository. The vendor decoder was used through a shim to inspect encodings, while the located historical EULA text contains restrictions that facially bear on reverse engineering. This audit makes no legal conclusion that use was permitted or prohibited. Exact v3.5 terms, acceptance record, and rights to redistribute the compiler remain `UNKNOWN`; no proprietary SDK executable is included in the Git repository.

### 11.4 AI-session input audit and privacy scope

Available development-session records, repository reports, Git commits and offline experiment receipts were inspected. The session index contains SGX535 development activity and an investigation of the SGX544 compiler in a separate project; that SGX544 investigation is not evidence that its compiler material was used by the SGX535 implementation. The session projections are not a complete export of every model/provider input, attached file, hidden context, compaction, or independently retained agent session.

Technical inputs demonstrably present in the retained project/research record include: the SGX535/Poulsbo source inventories and historical binary-analysis reports; the original-vendor SDK35 decoder's returned text for selected emitted bytes; the bounded 41,362-record CPU decode catalog; selected texture/SMP decoder observations; compiler-originated CPU writer traces/replays; test outputs and hardware evidence cited in the phase reports; and Mesa 25.2.0 API/build pins. These records show what the project consulted or compared. They do **not** identify, prompt by prompt, which source files, proprietary outputs, code fragments, screenshots or experiment artifacts were actually supplied to each LLM, nor which provider retained/processed them. Per-model input provenance therefore remains `UNKNOWN` except where an extant session item explicitly records it. No raw conversation, unrelated personal message or full tool log is reproduced in this section.

### 11.5 Driver implementation, source/evidence and validation map

| Implementation area | Files / entry points | Source and evidence trail | Commit/test records |
|---|---|---|---|
| USSE semantic and CPU encoders | `mesa/sgx535/remesa_usse.c`, `remesa_usse_arithmetic.c`, `remesa_usse_sample.c`, matching headers | Original Poulsbo x86 writer/normalizer traces as cited in source comments and Phase 9 reports; vendor oracle observations cited in 11.1. These sources support bounded CPU construction and selected decoder text. | Reachable implementation commits include e2db5cd, 0700034, 0da1be9, 4d741a3, 190724c and b0cb162. Named regression suites include `test_usse_arithmetic`, `test_usse_cpu_arithmetic`, `test_usse_parameter_first_term`, `test_usse_mvp_position_body`, `test_usse_gears_vertex_body`, `test_usse_lighting_body`, `test_usse_sample`, `test_usse_components`, `test_usse_export_contract`, `test_usse_position_color`, and `test_usse_pds`. Exact historic pass counts for each current file revision are in run receipts, not inferred from the test names. |
| NIR/Gallium shader front end and restricted profiles | `mesa/sgx535/remesa_nir_*`, `remesa_shader_*`, `remesa_screen.c`, `remesa_context.c`, `remesa_state.c` (exact active file set varies by commit) | Mesa 25.2.0 API baseline and independent semantic IR contracts; original compiler/replay evidence for selected input/output mappings; no claim that Mesa NIR alone specifies SGX instruction fields. | API pins in `mesa/sgx535/mesa-api-pin.json` and `mesa-loader-pin.json`; commits 5298633, 6e159a4, a85d062, 769bb87; tests named in the SGX535 test directory. A complete per-function source audit of all current NIR/Gallium callbacks is not present in this section; entries outside the named encoder families remain `UNKNOWN` at function-level granularity. |
| PDS construction, shader linkage and owned program resources | `mesa/sgx535/remesa_pds*`, shader-linkage and retained scene units | Original Poulsbo CPU producer evidence, selected structure/relocation traces, sealed fixtures and resource-closure checks; PDS and USSE remain distinct ISAs. | Examples: 443e134, 769bb87, 906bc1f, 50246df, db3133f, e991e31, 104f35d, f9cf994, ec43ead. Tests include `test_usse_pds`, `test_pds_cpu_trailer`, pipeline and structural-PDS suites. These are source/build/CPU contract records, not blanket physical PDS ISA proof. |
| Gallium resources, transport, submission, ownership and retirement | `mesa/sgx535/remesa_retained_*`, `remesa_linux_*`, screen/context/resource/state units; kernel endpoints under `kernel/remesa/gma500/` | Mesa 25.2.0 loader/API pins, Linux DRM/GMA500 source pins in sections 3/13/14, driver-owned resource invariants and hardware receipts under `docs/hardware-evidence/`. | Implementation lineage spans commits such as 8a6e35f, 0153e2a, 247b098, cd08dc8, 28957b3, 598c34b, 5ed1942 and current later integration. Receipts are operation-specific; they do not establish generic Mesa upstream readiness for every path. |
| Documentation, harnesses and tests | `docs/phase7/`, `docs/phase9/`, `mesa/sgx535/tests/`, `tools/psb-dri-re/`, `tools/remesa/` | Reports identify source snapshots, hashes, command lines, test inputs, failure history and limitations. The 72-source Poulsbo registry and 29-source Phase 7 index are expanded in sections 13–14. | Report commits 5035e09 and 3eeb870 document the vendor-oracle and texture observations. They record bounded results; the presence of a report or passing test is not itself a source license or a clean-room certification. |

This is a source-family map, not a complete symbol-by-symbol audit of the entire driver. Where a current C function lacks a retained source identifier, experiment ID, exact build receipt or matching implementation commit, its individual provenance is `UNKNOWN` rather than imputed from a neighboring function.

### 11.6 Human review and intervention evidence

The repository records human-authored task constraints and technical decisions in session projections and handoffs, and review corrections in committed reports. Examples with concrete artifacts include: keeping SGX535 and SGX544 research scopes separate; using the SGX535/rev121 descriptor rather than the external SGX540 context; correcting interpretation of the decoder wrapper return value in `USSE-DECODER-CATALOG-20261028.md`; preserving the distinction between CPU writer layout and hardware semantics in `PDS-USSE-PRIORITY-20261020.md` and `PDS-USSE-COMPLETION-20261021.md`; recording failed/intermediate test runs instead of counting them as final passes; and using source/commit pins for Mesa API integration.

Git metadata records a configured author identity for the listed commits; that metadata does not establish who independently reviewed each line or performed each command. The reports record decisions, corrections and validation outcomes, but the examined history does not supply a complete reviewer-signoff ledger, per-test operator identity, code-review approval for every encoder field, or complete human review notes for every AI-generated/AI-edited patch. Human-supervision evidence is therefore limited to the recorded decisions, authored commits, command receipts and corrections cited here; unrecorded review is `UNKNOWN`.

### 11.7 Upstream review implications

For Mesa review, the actionable source boundary is: reMESA's implementation must stand on its own interface/semantic tests and documented CPU-side producer evidence, while any proprietary oracle executable, shim bundle, generated decoder listing, pseudocode, unlicensed historical helper code or raw session material stays outside the upstream patch. The current evidence supports only the encoder families and profiles listed above. The exact SDK 3.5 agreement, shim source derivation/license, complete model-input lineage and per-function human sign-off remain unresolved in the records and should not be represented as cleared. This paragraph documents review-relevant evidence limits; it does not prescribe a release or make a legal determination.


## 12. Mesa upstream contribution source pins

The SGX535 Gallium work uses a pinned Mesa interface/build baseline. These pins identify the source APIs and build integration inspected; they are not an SGX implementation source. The recorded API baseline is Mesa 25.2.0.

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
