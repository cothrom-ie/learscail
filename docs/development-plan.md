# Léarscáil — Development Plan

**Status:** Proposed implementation plan (v0.1)  
**Date:** 1 November 2026  
**Related:** [Project Brief](project-brief.md)

## Working method

Build **small, independently testable vertical slices**. A milestone is complete only when its output works in a real CAD import, not merely when an API call or file export succeeds. Use GitHub issues for discrete implementation tasks and pull requests for review. Record consequential decisions in `docs/decisions/` as short ADRs.

The existing repository already uses Python packaging, pytest, Ruff, Pyright and MkDocs. Retain the existing tooling, but review and correct inherited template settings (including outdated MkDocs description/repository URL and inconsistent Python version classifier) as part of foundation work.

## Milestone 0 — Foundation and CLI contract

**Goal:** Installable command with validated location/extent/orientation input.

- Establish package entry point and `generate` subcommand.
- Accept `--lat`, `--lon`, `--radius`, `--rotation`, `--terrain`, `--contours`, `--features` and `--output`.
- Validate arguments and implement clear CLI diagnostics.
- Implement square footprint in a local metric projected frame, centred on input location; support rotation.
- Define internal geometry/CRS data structures; write unit tests for corner coordinates, dimensions, rotated bounds and coordinate round trips.
- Clean repository metadata and document basic development installation.

**Acceptance:** `learscail generate --lat … --lon … --radius … --rotation …` validates, reports the intended square's corners/CRS and exits predictably (no data download yet). Rotating does not change square side length.

## Milestone 1 — First 3D contour DXF

**Goal:** Produce useful contours from a single, known, openly reusable DTM.

- Choose a well-documented test site and obtain source DTM coverage.
- Establish raster acquisition/caching for that source and crop/resample appropriately.
- Transform to an appropriate metric CRS and record units/elevation datum.
- Generate elevation-aware 3D contour polylines at a configurable interval.
- Clip geometry to the oriented output square and apply the agreed drawing transform consistently.
- Export a layered DXF using an open-source writer.
- Verify in Vectorworks; test coordinate/height samples against the source raster.

**Acceptance:** First real site DXF opens, contour heights are usable in 3D, scale is correct, rotation and extents are correct, and source limitations are recorded.

## Milestone 2 — Buildings, roads and other mapped features

**Goal:** Generate a useful layered reference site plan.

- Retrieve OSM mapped features within source bounds; avoid excess requests and respect API usage policy.
- Initially support building footprints and established roads; expand to tracks, paths, waterways, landcover and physical boundaries.
- Use closed 2D polylines for building footprints, without inferred Z/heights.
- Transform, clip and orient features consistently with contours.
- Define stable DXF layer names; allow selected feature categories at runtime.
- Add fixtures for missing, partial and malformed feature data.

**Acceptance:** Buildings and roads align with terrain and remain separate editable layers in DXF. Missing features are reported, not invented.

## Milestone 3 — Mesh terrain and output selection

**Goal:** Support contours, mesh, both or neither.

- Evaluate TIN generation from terrain rasters and DXF `3DFACE` or equivalent supported entities.
- Retain valid surface elevations, ensure triangle winding/edge behaviour is predictable and clip at drawing boundary.
- Route `--terrain contours|mesh|both|none` to independent DXF layers.
- Set pragmatic limits for large radii and high-density rasters; add decimation/quality controls only as needed.
- Validate opening, visibility and 3D interpretation in Vectorworks; test against known elevations.

**Acceptance:** All terrain modes generate as selected, and meshes can be inspected as a 3D surface without conflicting with contour layers.

## Milestone 4 — Provider discovery and coverage

**Goal:** Work across Ireland, Northern Ireland and Great Britain with best-available supported data.

- Define a provider contract: geographical coverage, access, licence, resolution, vertical datum and fetch methods.
- Add initial regional providers, including a reliable fallback where appropriate.
- Score eligible datasets transparently; do not confuse grid spacing with published accuracy.
- Handle tile mosaics, mixed coverage, failure/retry, local cache and reasonable data-download limits.
- Avoid silently mixing vertical datums or replacing missing data with supposedly precise output.

**Acceptance:** Same CLI works at reference sites in all three regions, transparently reporting chosen sources and gaps.

## Milestone 5 — Source and drawing provenance

**Goal:** Make drawings professionally interpretable.

- Write structured `metadata.json` and optionally a concise plain-text summary.
- Include input centre, side length, rotation sign/convention, effective clipping area, drawing origin/transform, horizontal CRS, vertical datum, contour interval and terrain mode.
- Record actual datasets, provider, source dates, retrieval time, source licences/attribution, resolution and any stated accuracy.
- Warn where source age, data coverage or inconsistent datums compromise applicability.
- Test metadata against known datasets and exported CAD geometry.

**Acceptance:** A user can understand where the geometry originated and reverse the drawing-coordinate transform.

## Milestone 6 — Release and reproducibility

**Goal:** Useful, documented initial public release.

- Build automated offline tests using licensed/sample fixtures.
- Test flat rural, sloping rural, urban and low-coverage sites.
- Establish CI for formatting/type checks/tests and package installation.
- Document CLI flags, data-source behaviour, licensing and limitations.
- Validate representative output in Vectorworks; document interoperability findings and size/performance limits.
- Tag the first version when acceptance requirements are met.

**Acceptance:** Another user can install the package, generate a sample drawing and understand its limitations without developer assistance.

## Candidate implementation stack

| Concern | Starting choice | Notes |
| --- | --- | --- |
| CLI | Typer | Can be changed if standard library CLI is sufficient |
| CRS and transformations | PyProj | Coordinate round-trip tests required |
| Raster/terrain | GDAL and/or Rasterio | Use source DTM directly; preserve provenance |
| Vector processing | Shapely, GeoPandas | Spatial clipping and feature normalization |
| OSM | Overpass / OSMnx | Respect rate limits and attribution |
| DXF | ezdxf | Explicit geometry and layer control |
| Tests | pytest | Existing project tool |
| Documentation | MkDocs | Existing project tool |

Defer adoption of individual dependencies until the related vertical slice needs them.

## Initial issue backlog (suggested)

1. **CLI:** configure entry point and validated arguments.
2. **Geometry:** implement centred square and rotation in metric coordinates.
3. **CRS:** document projected CRS and local-origin strategy with tests.
4. **Terrain:** obtain initial DTM fixture and a repeatable fetch path.
5. **Contours:** generate true 3D contour geometry.
6. **DXF:** export contours with units and layer conventions.
7. **CAD validation:** import and examine contour DXF in Vectorworks.
8. **OSM:** retrieve buildings and established roads.
9. **Features:** convert polygons/lines to 2D DXF layers.
10. **Terrain mesh:** produce configurable 3D mesh and terrain modes.
11. **Providers:** modular source selection for Ireland/NI/GB.
12. **Provenance:** create source and processing metadata.

These are proposed backlog items, **not yet created GitHub issues**.

## Reference-site test matrix

- Flat urban area: building/road geometry and alignment.
- Rural festival-like field: paths, boundaries and sparse mapped features.
- Sloped terrain: elevation accuracy, contour interval and mesh quality.
- Limited-coverage site: warnings and graceful degradation.
- Rotated output: positive/negative angle, dimensions, source-bounds completeness and exact alignment.

## Risk register

| Risk | Mitigation |
| --- | --- |
| Inconsistent CRS or datums | Centralise transforms, test round trips, report vertical references |
| Apparent precision exceeds data quality | Expose source resolution/accuracy and warn on fine contours |
| Incomplete OpenStreetMap mapping | Report coverage/absence and permit independent source providers |
| Large mesh DXFs | Test file sizes and introduce controlled detail limits |
| CAD origin/rotation interoperability | Test early in Vectorworks and document transform |
| Open-data licences and API limits | Maintain source-aware attribution and usage policies |
