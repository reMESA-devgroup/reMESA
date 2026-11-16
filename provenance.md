
# reMESA — Source Provenance and Clean-Room Audit

> **Status:** Public provenance record — independent audit pending  
> **Project:** [reMESA](https://github.com/reMESA-devgroup/reMESA)  
> **Target:** PowerVR SGX535 / Intel Poulsbo / SGX535 rev121  
> **Reference hardware:** Dell Inspiron Mini 12  
> **Last updated:** 2026-11-16

## 1. Purpose

This document records the origins, licensing considerations, reverse-engineering methods, AI assistance, and technical evidence associated with reMESA.

The purpose is to make the research and implementation history traceable, reproducible, and independently auditable.

**This document is not a legal certification or proof that a complete independent clean-room audit has been performed.**

Public availability does not automatically grant permission to copy, modify, or redistribute software. Hardware correctness and source-code provenance are separate questions.

## 2. Provenance principles

reMESA aims to follow these principles:

1. Record the origin of historical technical materials.
2. Preserve source identities, commits, versions, and hashes.
3. Distinguish observations from implementation code.
4. Keep restricted materials separate from implementation inputs.
5. Document uncertainty rather than silently treating hypotheses as facts.
6. Disclose AI-assisted development.
7. Require meaningful human accountability and review.
8. Respect third-party licenses and upstream contribution policies.
9. Avoid translating or regenerating restricted implementations as a substitute for permission.
10. Preserve reproducible evidence for technical claims.

These are project policies and audit objectives, not claims that every historical contribution has already passed an independent review.

## 3. Source classifications

| Classification | Meaning |
|---|---|
| `PUBLIC-OPEN` | Public source with an identified open-source license |
| `PUBLIC-RESTRICTED` | Public material with restrictions on use or redistribution |
| `HISTORICAL-BINARY` | Historical executable or library used as evidence |
| `MIRROR-UNVERIFIED` | Community mirror without authenticated vendor provenance |
| `QUARANTINED` | Material excluded from implementation pending review |
| `ORIGINAL-IMPLEMENTATION` | Code developed specifically for reMESA |
| `AI-ASSISTED` | Code or documentation created or modified with LLM assistance |
| `PHYSICAL-EVIDENCE` | Observations recorded on actual hardware |
| `OFFLINE-EVIDENCE` | Static analysis, models, or tests without target GPU execution |
| `UNKNOWN` | Origin, licensing, identity, or applicability not established |

A material may belong to multiple categories.

## 4. Linux DRM/GMA500

**Repository:** https://github.com/gregkh/linux

**Recorded commit:**

`d5eca7ebcf6f64c4aebf9684c365c130a3a069b3`

The Phase 7 source index records:

- `psb_drv.c`
- `psb_drv.h`
- `psb_device.c`
- `power.c`
- `psb_irq.c`
- `mmu.c`
- `accel_2d.c`
- `psb_reg.h`
- `gtt.c`

These files provide historical kernel integration and Poulsbo hardware context.

The project's retained source audit identifies correspondence with historical Git objects.

**Limitation:** Historical source correspondence does not prove the exact identity of the installed kernel module.

## 5. Texas Instruments PowerVR Kernel DDK

**Official repository:**

https://git.ti.com/cgit/graphics/omap5-sgx-ddk-linux/

**Historical Poulsbo-capable DDK commit:**

`cb46ba4d0c900f89f7ec0284f9803d476bfa98de`

**SGX535 header commit:**

`322bcda5f3076037e2e20ef9209f4f4d575a7d5f`

Documented files include:

| File | Purpose |
|---|---|
| `sgx535defs.h` | SGX535 hardware definitions |
| `sgxerrata.h` | Hardware errata |
| `sgxfeaturedefs.h` | Hardware features |
| `sgxreset.c` | Historical reset implementation |
| `sgxinit.c` | Historical initialization |
| `sgxinfo.h` | SGX information structures |
| `sgx_bridge.h` | Interface declarations |
| `bridged_sgx_bridge.c` | Historical bridge implementation |
| `sysconfig.c` | Poulsbo platform integration |
| Poulsbo D0 Makefile | SGX535 rev121 build configuration |

The historical source archaeology identifies an SGX535-specific header declaring MIT/GPLv2 licensing.

It also identifies a Poulsbo D0 build configuration containing:

- `SGXCORE := 535`
- `SGX_CORE_REV := 121`
- `PVR_SYSTEM := poulsbo`

### Licensing considerations

Texas Instruments publicly hosts the kernel DDK, and TI distribution metadata identifies open licensing for certain kernel DDK releases.

However:

- Each exact historical revision requires individual review.
- Copyright notices must be preserved.
- Licensing cannot be inferred from neighboring files.
- Kernel-mode licensing does not automatically apply to user-mode binaries.
- Public hosting does not establish unrestricted reuse rights.
- Historical confidentiality notices must be evaluated in context.

**Status:** Official repository provenance established; complete per-file licensing audit pending.

## 6. Historical Poulsbo kernel

**Repository:** https://github.com/gregkh/psb-kmp

**Commit:**

`98b5307e5158a9ac401b29128ddd1184ae06b4d7`

Recorded files:

- `psb_drv.c`
- `psb_mmu.c`
- `psb_xhw.c`

These provide historical context for memory management, driver architecture, and command submission.

Historical implementation details are not automatically applicable to the current hardware configuration.

## 7. Intel EMGD community mirror

**Repository:**

https://github.com/EMGD-Community/intel-binaries-linux

**Commit:**

`e6884ec2eaaf1afe88d5ff9dd44d70403525be5b`

Recorded materials:

- `drm/pvr/services4/srvkm/hwdefs/sgxerrata.h`
- `drm/pvr/services4/srvkm/hwdefs/sgx535defs.h`
- `drm/pvr/services4/srvkm/devices/sgx/sgxreset.c`
- `drm/emgd/state/power/plb/pwr_plb.c`
- `drm/include/emgd_drm.h`

These are publicly retained historical source snapshots.

The community mirror is not treated as an authenticated official Intel release.

**Status:** `MIRROR-UNVERIFIED`

The applicable copyright notices and redistribution rights require individual review.

## 8. TI user-mode DDK

**Repository:**

https://git.ti.com/cgit/graphics/omap5-sgx-ddk-um-linux/

**Commit:**

`b6801bf89e00d69893c2957455bbeb3195c0ed52`

Recorded materials:

- `README`
- `targetfs/lib/libsrv_init.so.1.9.6.0`

SHA-256 of the historical binary:

`a1a3572ee4ae6104505808d171d6b4cd023309959588f890b2192aea44fafdda`

This is an OMAP5/DRA7xx userspace artifact, not an authenticated Poulsbo userspace binary.

The historical record classifies its examination as metadata-only.

**Status:** Historical proprietary binary; separate licensing review required.

## 9. Historical Poulsbo binaries

### 9.1 psb_dri.so

**Package:** `xpsb-glx` 0.18

SHA-256:

`74ca42991906741ee91be1bc088ae0b142fa71b6a85658c5dad0851ce50dd0d8`

The package specification records redistribution restrictions, including no modification.

Documented investigation methods include:

- Static ELF inspection
- Symbol and offset inventories
- Ghidra analysis
- Targeted disassembly
- Historical ABI comparison
- CPU-side command analysis

The research records distinguish derived observations from implementation code.

### 9.2 Xpsb.so

SHA-256:

`da531587b1ec59fe433fa20ddd2e691b2f8b7fb35bb82525d4db99259c5e571f`

Documented investigation topics include:

- Initialization
- Shared structures
- Buffer-object operations
- Command-related fields
- Synchronization
- Historical communication interfaces

The original binary's build relationship with historical kernel packages has not been fully authenticated.

### Static-analysis boundary

Historical records describe disposable Ghidra projects and temporary decompilation outputs kept outside implementation code.

This describes the documented methodology; it is not an independent certification of every development session.

## 10. Additional historical packages

| Package | Version | Role |
|---|---|---|
| `libdrm-poulsbo` | 2.3.0 | Historical DRM ABI |
| `xserver-xorg-video-psb` | 0.32.0 | Historical Xorg implementation |
| PSB kernel source | 4.41.1 | Candidate kernel source |
| PSB firmware | 0.30-4 | Firmware package inventory |
| MesaLib | 7.4.4 | Public Mesa comparator |

Detailed package hashes and licensing information:

`docs/phase7/xpsb-re/source-provenance.md`

These sources are not assumed to form an authenticated, exactly matched historical driver stack.

## 11. Restricted and quarantined materials

### 11.1 Canonical libgl1-mesa-dri-psb

**Package:** `libgl1-mesa-dri-psb`

**Version:** `0.24+repack+0038.1`

**Historical source:** Canonical `hardy-dell-mini`

Package SHA-256:

`14e870f4273e84759630499efeda4948116c29f3ba7d08e2b5f028a5da3ef8a2`

Extracted ELF SHA-256:

`427627909b0c528d154d1b247688f16536162a520d43d472e8a4f404c02e2a6e`

The historical investigation records an original-content notice identifying Intel confidentiality and reserved rights.

According to the project report:

- The package was placed in a disposable quarantine directory.
- It was not added to the retained reference tree.
- It was not executed.
- It was not used for implementation details.
- Only limited static occurrence and provenance checks were performed.

**Status:** `QUARANTINED`

The confidentiality notice does not independently establish that the package was leaked or distributed without authorization.

### 11.2 PSP2 PowerVR sources

A public PSP2-related repository was examined as a possible research locator.

The investigated header contained restrictive Imagination copyright language.

The project records that it was not imported or used to decode the selected SGX535 instructions.

**Status:** Restricted research lead; excluded from implementation.

### 11.3 Confidential/NDA search result

The historical investigation records a result explicitly marked confidential/NDA.

According to the report, its contents were not opened.

**Status:** Excluded.

### Quarantine rules

Restricted or uncertain material must not be incorporated into the implementation without an appropriate rights and provenance review.

Quarantine must not be bypassed through:

- Copying
- Mechanical translation
- Superficial rewriting
- AI-generated reconstruction of restricted code
- Unattributed incorporation into patches

Historical contact with restricted material must not be concealed.

## 12. Reverse-engineering methodology

### Historical source archaeology

Used to investigate:

- Register definitions
- Hardware revisions
- Kernel interfaces
- Memory structures
- Command formats
- Initialization relationships
- Platform differences

### Static binary analysis

Used to investigate:

- ELF symbols
- Function addresses
- Data layouts
- Constants
- CPU-side command construction
- Historical ABI behavior

Decompiler output is not treated as authoritative hardware documentation.

### Independent software models

Used to evaluate:

- Packet layouts
- Relocation constraints
- Memory boundaries
- Resource ownership
- Instruction encodings
- Error handling
- Failure conditions

Offline success is not equivalent to physical GPU validation.

### Hardware experiments

Physical tests should identify:

- Hardware and GPU revision
- Kernel/module version
- Test input
- Expected output
- Actual output
- GPU submission and retirement
- Readback
- Memory guards
- Recovery behavior
- Relevant artifact hashes

Physical evidence is valid only within the tested scope.

## 13. Evidence terminology

| Term | Meaning |
|---|---|
| `CONFIRMED` | Established within the stated evidence scope |
| `INFERRED` | Supported by reasoning but not directly established |
| `UNKNOWN` | Not established |
| `BLOCKED` | Qualification or operation cannot proceed |
| `PASS` | Defined test passed |
| `FAIL` | Defined test failed |

A successful isolated rendering operation must not be represented as complete OpenGL conformance.

## 14. AI-assisted development

reMESA uses substantial AI assistance.

Documented uses include:

- Source archaeology
- Searching historical code
- Comparing implementations
- Organizing findings
- Drafting documentation
- Software development and debugging assistance

### Transparency

AI-generated and AI-modified code must not be represented as independently human-authored when that would misstate its provenance.

The project does not claim that LLM training data can be fully reconstructed or that generated code is guaranteed free from proprietary-code influence.

### Required audit

Before upstream submission, the following questions should be addressed:

1. Which files contain AI-generated or AI-modified implementation code?
2. Which tools and models were used, where records exist?
3. Which historical materials entered model context?
4. Did restricted or quarantined materials enter model context?
5. Which changes received substantive human review?
6. Can the implementation be independently explained?
7. Does any implementation substantially resemble restricted source?
8. Does the relevant upstream project permit the proposed contribution?

Missing records must be classified as `UNKNOWN`.

### Human accountability

Human review should cover:

- Correctness
- Licensing
- Memory safety
- Synchronization
- Error handling
- Maintainability
- Coding standards
- Security
- Hardware ownership
- Provenance

**A completed independent human audit is not claimed here.**

## 15. Source-to-implementation traceability

A future audit should associate each implementation component with its evidence and review status.

Suggested fields:

| Field | Description |
|---|---|
| Component | Driver subsystem |
| File | Implementation source |
| Commit | Implementation revision |
| Historical references | Relevant sources |
| License status | Reviewed / Pending / Blocked |
| AI involvement | Known / None documented / Unknown |
| Human reviewer | Reviewer identity if completed |
| Evidence | Tests and reports |
| Similarity audit | Review outcome |
| Final status | Approved / Needs work / Blocked |

Important components include:

- NIR lowering
- USSE encoding
- PDS construction
- Gallium frontend
- Texture sampling
- Raster state
- Memory management
- Kernel ioctls
- Submission and retirement
- Resource lifetime
- Presentation
- Error recovery

Passing tests does not automatically establish acceptable source provenance.

## 16. Test provenance

The project maintains extensive historical and current test records.

Tests should be classified as:

- Static analysis
- Unit tests
- Offline simulations
- Compiler validation
- Physical GPU experiments
- Recovery tests
- Application tests

Each important result should record:

- Test ID
- Date
- Hardware
- Software revision
- Inputs
- Expected result
- Observed result
- Status
- Artifact location
- Hash
- Reproduction instructions
- Limitations

The project's large local evidence archive is not automatically equivalent to a fully published, independently verified test corpus.

Test volume does not establish copyright provenance.

## 17. Upstream preparation

The following checklist is a proposed project audit gate, not a claim that every upstream project imposes identical requirements.

- [ ] Verify exact historical source licenses.
- [ ] Review all quarantined materials.
- [ ] Complete source-to-implementation traceability.
- [ ] Audit AI-generated implementation code.
- [ ] Perform substantive human code review.
- [ ] Review NIR/USSE compiler correctness.
- [ ] Review kernel safety and ownership.
- [ ] Verify build integration.
- [ ] Publish reproducible test summaries.
- [ ] Document unsupported functionality.
- [ ] Review applicable upstream AI policies.
- [ ] Resolve or disclose outstanding provenance issues.
- [ ] Seek maintainer guidance where necessary.

No item should be marked complete without evidence.

## 18. Known unresolved questions

1. Exact licensing of every consulted historical DDK revision.
2. Redistribution authorization for all mirrored materials.
3. Complete AI-generated code provenance.
4. Whether restricted material entered any LLM context.
5. Whether current implementation resembles restricted historical code.
6. Authentication of historical binary/kernel pairings.
7. Independent human audit completion.
8. Compatibility with applicable Mesa and Linux contribution policies.

These questions must remain open until supported by evidence.

## 19. Existing project evidence

Important audit entry points:

- [`docs/source-archaeology.md`](docs/source-archaeology.md)
- [`docs/phase7/source-index.json`](docs/phase7/source-index.json)
- [`docs/phase7/source-provenance.md`](docs/phase7/source-provenance.md)
- [`docs/phase7/xpsb-re/source-provenance.md`](docs/phase7/xpsb-re/source-provenance.md)
- [`docs/phase7/psb-dri-re/README.md`](docs/phase7/psb-dri-re/README.md)
- [`docs/phase7/psb-dri-re/cleanroom-backing-contract.md`](docs/phase7/psb-dri-re/cleanroom-backing-contract.md)
- [`docs/phase9/USSE-VENDOR-ORACLE-20261027.md`](docs/phase9/USSE-VENDOR-ORACLE-20261027.md)
- [`docs/phase9/PDS-CROSS-CORE-20261031.md`](docs/phase9/PDS-CROSS-CORE-20261031.md)
- [`docs/history/PROJECT-HISTORY.md`](docs/history/PROJECT-HISTORY.md)
- [`projects/sgx544/docs/PROVENANCE.md`](projects/sgx544/docs/PROVENANCE.md)

## 20. External references

### Texas Instruments

- Kernel DDK: https://git.ti.com/cgit/graphics/omap5-sgx-ddk-linux/
- User-mode DDK: https://git.ti.com/cgit/graphics/omap5-sgx-ddk-um-linux/

### Linux and historical Poulsbo

- https://github.com/gregkh/linux
- https://github.com/gregkh/psb-kmp

### Intel EMGD community

- https://github.com/EMGD-Community/intel-binaries-linux

### Mesa

- https://gitlab.freedesktop.org/mesa/mesa

### reMESA

- https://github.com/reMESA-devgroup/reMESA

## 21. Final statement

reMESA is an experimental, extensively AI-assisted reverse-engineering and graphics-driver development project.

Its historical research records include identifiable repositories, source revisions, binary hashes, technical observations, software models, and physical hardware experiments.

The project also documents encounters with proprietary, restricted, and potentially confidential material, including quarantine and exclusion decisions.

These records provide a foundation for provenance auditing.

However, they do not independently establish that every implementation change has acceptable provenance, that no restricted material influenced LLM-generated code, or that the driver satisfies the contribution requirements of Mesa or the Linux kernel.

**reMESA does not claim completed independent clean-room certification.**

The intended standard is transparent provenance, reproducible evidence, human accountability, and explicit disclosure of unresolved questions.

---

**Maintenance note:** Material changes to this document should identify supporting source revisions, evidence IDs, or review records.

**License note:** This document does not modify the licenses of third-party materials or existing project code.
