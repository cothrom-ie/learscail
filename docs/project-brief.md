# Léarscáil — Project Brief

**Status:** Proposed specification (v0.1)  
**Date:** 1 November 2026  
**Repository:** `cothrom-ie/learscail`  
**Licence:** GNU GPL v3.0 (application; source datasets retain their own licences)

## Purpose

Léarscáil is an open-source Python command-line application that converts publicly available geographical data into practical, layered, three-dimensional CAD reference drawings. Its initial audience is festival and outdoor-event production professionals, but its output should be useful in other site-planning workflows.

A user supplies a latitude, longitude and radius. Léarscáil discovers and retrieves supported terrain and mapped-feature datasets, processes them into a common coordinate system, and exports an editable DXF with a source and quality report. The basic workflow should require no GIS knowledge or proprietary GIS software.

## Intended outcome

A production manager or technical director should be able to produce an initial ground plan containing terrain, established roads, paths, building footprints, waterways, mapped boundaries and vegetation where data exists. The drawing is a **planning reference**, not a substitute for a current measured site survey or engineering verification.

## Agreed requirements

### Area and orientation

- Input centre: WGS 84 latitude and longitude in decimal degrees.
- Input `--radius`: half the side length of a **square** output area, in metres. A radius of 500 generates a 1,000 × 1,000 m square, centred on the specified point.
- Optional `--rotation` in decimal degrees, default 0. Positive degrees rotate the output **counterclockwise** about the centre.
- Rotation defines the square's orientation and the visual orientation of its content in the exported drawing. Acquire enough source data to cover the full rotated footprint, then clip to that footprint. Do not rotate one layer without all others.
- Preserve the source/projected CRS, the centre, output coordinate transformation and rotation in metadata so users can recover the original geographical position. Exact local-origin and georeferencing conventions will be validated with CAD import tests.

### Terrain

- Export **elevation-aware 3D contour polylines** and/or a **triangulated 3D mesh**, selected by the runtime `--terrain` flag: `contours`, `mesh`, `both`, or `none`.
- Default to `both`, subject to real-world performance and file-size validation.
- Place contours and mesh on separate, switchable CAD layers.
- Provide a configurable contour interval in metres. Do not imply that finer spacing increases the accuracy of the underlying survey.
- Maintain valid elevation values and identify their source vertical datum, if known. Do not silently merge incompatible vertical datums.

### Built and natural features

- Include mapped **2D building footprints**, not inferred building heights or structural models.
- Include established roads, tracks and paths, waterways, mapped boundaries, walls/fences and woodland/vegetation where public data is available.
- Separate significant feature classes into named, individually controllable CAD layers.
- Do not invent missing objects or imply that absence from an open dataset means absence on the ground.
- 2D footprints are plan geometry; do not present them as surveyed foundations or structural information.

### Output

- Primary format: **DXF**, containing editable CAD entities rather than a flattened raster.
- Keep correct metric dimensions and consistent layer naming. Export valid 3D geometry when requested.
- Provide data-source and processing metadata alongside the DXF, including coverage, acquisition time, survey date where available, nominal resolution, stated accuracy, horizontal CRS, vertical datum where available, licences/attribution and warnings.
- Validate DXF in Vectorworks as the initial reference application; avoid application-specific dependencies where possible.

### CLI

Illustrative interface (design target, not yet implemented):

```bash
learscail generate \
  --lat 54.5973 --lon -5.9301 --radius 500 \
  --rotation 35 --terrain both --contours 0.5 \
  --features buildings,roads,paths,water \
  --output festival_site.dxf
```

Required: `--lat`, `--lon`, `--radius`. Optional: `--rotation` (0), `--terrain` (both), `--contours` (1.0 m), `--features` (all supported), `--output` (generated path). Validate coordinate range, finite angles, positive dimensions and contour intervals. Exact flag spelling and defaults may be refined during the CLI milestone.

## Geographic scope and data acquisition

The architecture should be international; first provider integrations prioritise the Republic of Ireland, Northern Ireland and Great Britain. Providers must be pluggable and should distinguish source coverage, licensing, spatial resolution, positional/vertical accuracy and data currency. Potential providers include publicly accessible national terrain products and OpenStreetMap for mapped features. Publicly accessible is **not synonymous with unrestricted reuse**: honour per-source terms, attribution, APIs and distribution limits.

## Technical principles

- Python CLI with an independently testable pipeline: selection → discovery → acquisition → normalisation → terrain/features processing → DXF export → provenance.
- Prefer established open-source geographic libraries; specific packages are implementation choices, not requirements.
- Reliable geography and editable geometry take priority over visual styling.
- Gracefully degrade if a category is absent, but **fail clearly** when a requested terrain output cannot be generated reliably.
- Avoid fabricated precision or unlabelled interpolation.
- Provide understandable defaults and useful error messages.
- Keep reproducible test fixtures and validate transformations, geometry bounds, units, layers and elevations.

## Initial deliverable and acceptance

An initial usable release must:

1. Accept centre coordinates and a radius and resolve a correctly oriented square.
2. Retrieve a supported terrain dataset automatically for at least one documented test region.
3. Produce a DXF with selectable 3D contours, mesh, or both.
4. Add available building footprints and established roads on separate layers.
5. Preserve correct dimensions and alignment between terrain and features when rotation is applied.
6. Import successfully into Vectorworks and permit meaningful site-model/3D inspection.
7. Generate a source/limitations report without suggesting survey-grade certainty.

## Out of scope for initial release

Web interface, automated festival layout design, structural or ground-bearing assessment, detailed accessibility certification, inferred 3D buildings, survey certification, and guaranteed worldwide high-resolution coverage.

## Open design questions to resolve by testing

- Whether exports retain full projected coordinates or use a local CAD origin by default, and how that interacts with visual rotation and Vectorworks internal-origin behaviour.
- Exact DXF entities for mesh and contour interoperability.
- Performance limits, dataset selection/fallback rules and acceptable incomplete-data behaviour.
- Source-specific vertical datum transformations and licence-handling policy.
