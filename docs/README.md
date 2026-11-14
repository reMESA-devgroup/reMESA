# Documentation

Start with the [project overview](PROJECT-OVERVIEW.md), then choose a reproduction
guide. The detailed phase records remain available for reverse engineers.

[Current milestone status](CURRENT-STATUS.json) is also available as JSON.

For the latest SGX535 development checkpoint, read the [November 13 beta status](phase9/BETA-STATUS-20261113.md) and its [scoped evidence summary](phase9/BETA-STATUS-20261113.json). It distinguishes physical GPU results, CPU-only X11 publication and simulated integration tests; it is not a driver release.

## Reproduce a result

- [Triangle](reproduction/TRIANGLE.md): the first nonzero SGX readback.
- [Square](reproduction/SQUARE.md): two indexed foreground triangles in one draw.
- [Physical display](reproduction/DISPLAY.md): CPU/Xorg publication of preserved GPU pixels.
- [Reproduction limits and terminology](reproduction/README.md).

## Understand the implementation

- [Current experimental pipeline](PROJECT-OVERVIEW.md).
- [Memory and address spaces](mmu-bif.md), [Poulsbo integration](poulsbo-architecture.md), and [historical ABI](poulsbo-historical-abi.md).
- [Phase 7 reconstruction](phase7/psb-dri-re/README.md).
- [Phase 8 results and active work](phase8/README.md).
- [3D roadmap](phase8/FIRST-REAL-3D-ROADMAP.md) and [frame reuse design](phase8/REUSABLE-RENDER-FRAMES.md).

## History and evidence

- [Project history](history/PROJECT-HISTORY.md) and [archive policy](history/README.md).
- [First triangle full-hash manifest](phase8/FIRE3-TRIANGLE-REPRODUCTION.json).
- [Square full-hash manifest](phase8/MULTI-TRIANGLE-REPRODUCTION.json).
- [First visible triangle manifest](phase8/VISIBLE-TRIANGLE-REPRODUCTION.json).
- [Square display result](phase8/square-display-result-20261006.json).
- [Privacy audit](PRIVACY-AUDIT-20261006.md) and [cleanup inventory](history/REPOSITORY-CLEANUP-20261006.md).

Older documents describe the state at their recorded date. In particular, a
historical blocked gate does not undo the later successful experiments. Local
`references/` links require the separately obtained checkout identified by the
linked research record; they are not files included in this repository.
