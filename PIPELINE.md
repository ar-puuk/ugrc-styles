# Pipeline: editor feature backlog

Candidate improvements to `docs/index.html` (the browser editor), scoped to *not* require a
redesign or new backend — same zero-build, `localStorage`-only, static-site constraints as
`STYLE_EDITOR_PLAN.md`. Pick one at a time and ask for it by name; each is meant to be
implementable independently of the others.

Status: `[ ]` not started · `[x]` done.

## High value, low effort

- [x] **Undo/redo (Ctrl+Z / Ctrl+Y).** Done — a bounded history stack (`recordHistory()`/`undo()`/
  `redo()`), gesture-coalesced so a dragged slider is one undo step, wired to header buttons.
- ~~**Eyedropper color sampling.**~~ Already exists — `docs/assets/colorpicker.js` (~line 148)
  already has a screen-wide eyedropper button, feature-detected on `window.EyeDropper`. Struck
  from the backlog; this was a research miss when the list was first drafted, not new work done.
- [x] **Keyboard shortcut sheet.** Done — `?`/header button opens a dialog listing Ctrl+Z/Y, `/`,
  Esc, `?`, with per-platform Ctrl/⌘ labeling; guarded so it doesn't hijack native text-field editing.

## Medium effort, distinctive

- [x] **Re-uploaded "Combined style" keeps its 3-service split; standalone per-service uploads get
  identified.** Done — `mergeStyles()` now stashes the `serviceOriginals` it already builds into
  `style.metadata["ugrc:serviceOriginals"]` (inert to any renderer, same precedent as the existing
  per-layer `ugrc:service`/`ugrc:originalId` tags), and `loadUploadedFile()` uses it (after
  `isValidServiceOriginals()`) instead of always falling back to a flat, unsplittable upload. Closes
  the gap where the "Combined style" download - meant to carry a session across devices/restarts,
  per UGRC's own ask to reuse this tool - would silently lose per-service grouping and the "3
  separate files" download on reload. Separately, `identifyPartialService()` matches the service
  name Esri embeds in every `VectorTileServer` URL to flag an upload that's just 1 of the 3 hosted
  services on its own (a raw file straight from UGRC, or one of this repo's own committed
  `UGRC_<Service>_<theme>.json` files) instead of silently loading it as if it were a complete
  basemap.
- [ ] **"Copy comparison image" button in compare mode.** Composite both map canvases
  (`canvas.toDataURL`) into one PNG when the swipe compare is active — turns the before/after
  screenshots currently taken by hand into a one-click export.
- [ ] **Shareable link for just the diff.** Encode only the *changed* properties (base
  theme/upload name + a compact JSON diff against pristine) into the URL hash, instead of the
  full merged style (ruled out in `STYLE_EDITOR_PLAN.md` §10 for being too large). Reconstructing
  on load means re-fetching the named base template, then replaying the diff.

## Polish

- [x] **Spec links on property labels.** Done — small "ⓘ" next to each property name (including
  boolean checkboxes) linking to `maplibre.org/maplibre-style-spec/layers/#<property-name>`.
- [ ] **First-visit callout for inspect + compare.** A dismissible tip (stored in `localStorage`,
  same pattern as `THEME_KEY`) pointing at the inspect tool and the "Compare against" picker on
  first load, so the two strongest features aren't left to be discovered by accident.

## From Maputnik (parity ideas)

Features Maputnik has that we don't, reviewed for whether they'd actually pay off given our
narrower scope (3 fixed Esri services, no arbitrary source editing, no accounts).

- [ ] **Debug view toggles: label collision boxes / tile boundaries / overdraw.** Maputnik exposes
  these as map-render debug overlays (`show-collision-boxes`, `show-tile-boundaries`,
  `show-overdraw-inspector` — MapLibre GL JS supports all three natively via
  `map.showCollisionBoxes` etc., no extra dependency). **Collision boxes are the standout pick
  for us specifically**: LiteBase/LiteLabels have dozens of zoom-band label layers, so "why isn't
  this label showing at this zoom" is a real, recurring question our current inspect tool
  (feature-under-cursor) doesn't answer — collision boxes would show it's being dropped for
  overlapping another label, directly. A small toggle group in the header or map corner, off by
  default.
- [x] **Inline schema validation, not just JSON-parse-on-blur.** Done — `validateRawPropertyValue()`/
  `validateRawFilterValue()` check parsed raw-JSON values against `PROPERTY_SPECS` (type/enum-
  membership only, never range min/max) plus a shape-only check for expressions/functions/filters,
  wired into both raw-JSON textareas' existing blur handlers. Unit-tested against the real
  `PROPERTY_SPECS` data (13 cases) plus a real-data browser pass (a genuine complex filter and a
  `text-font` array from the Shadow theme).
- [x] **Style metadata quick-edit (name / center / zoom / bearing / pitch).** Done — a "Style info"
  header button/dialog edits the style's own root fields (blank deletes the key), a "Use current
  map view" button captures the live camera, editing also jumps the live map for feedback, and both
  download paths (combined + the 3 split files) carry the fields through. Not wired into undo/redo
  or the "changed" badge — a deliberate style-level/layer-level scope line.
- ~~**Visual expression editor (ƒx toggle) for `interpolate`/`match`/`step`.**~~ Not planned —
  reversed from "reconsider" after checking both ends of the compatibility question (2026-09-26
  discussion). Pulled UGRC's *live* Esri source directly: 0 occurrences of `interpolate`/`match`/
  `step`/`case` across all 547 LiteBase+LiteLabels layers — ArcGIS Pro's vector-tile publisher
  never emits them, only plain literals, many duplicate zoom-banded layers, or (26 times, LiteLabels
  only) the legacy zoom-only `{stops:[...]}` function. Esri's own VectorTileLayer renderer doesn't
  support these expressions either, and Esri has stated no plans to - reports include `interpolate`
  silently not interpolating and a layer *disappearing entirely* once a `match` is added. Since the
  goal is one JSON that works across MapLibre/Mapbox/Esri, adding an expression editor would be the
  one feature capable of producing a file that renders fine in the first two and silently breaks in
  the third - a regression against that goal, not a neutral nice-to-have.
- [x] **Stops editor for the legacy zoom-only `{stops:[...]}` function.** Done —
  `isZoomOnlyStopsFunction()` detects a bare `{stops:[[zoom,value],...]}` (no `base`, no
  `property`) and routes it to `renderStopsRow()`'s add/remove-row editor instead of raw JSON;
  anything else still falls back to raw so nothing's silently dropped. Value input type follows
  `PROPERTY_SPECS`' own `kind`. Reuses `applyPropertyEdit()`, so undo/redo, the "changed" badge, and
  sibling bulk-apply all work for free. Verified against real LiteLabels `text-size` data (add/
  remove/re-sort-on-commit/undo) plus 8 unit-tested edge cases for the detector.
- **Not planned — data-source management (add/edit/remove vector/raster/GeoJSON/image/video
  sources).** Maputnik supports this because it edits arbitrary styles; we deliberately don't —
  our styles' sources are always UGRC's 3 fixed Esri `VectorTileServer` endpoints (or, for an
  upload, whatever the file already has), so a general source editor has no real use case here.

## Notes

- None of these touch `src/`, `config/`, or the Python build pipeline — scope stays inside
  `docs/`, same as the editor plan.
- Re-check against `STYLE_EDITOR_PLAN.md` §12 ("Known gaps / non-goals for v1") before expanding
  any of these — e.g. the diff-share idea above is deliberately *not* the full-style URL sharing
  that section rules out.
- Cross-platform compatibility (MapLibre + Mapbox + Esri) is a hard requirement, not just a nice-
  to-have — see the struck expression-editor item above for why that ruled it out specifically.
  Check any future data-driven-styling idea against what Esri's VectorTileLayer actually renders,
  not just what the spec technically allows.
