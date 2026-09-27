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
- [ ] **Style metadata quick-edit (name / center / zoom / bearing / pitch).** Maputnik has a
  small "style info" panel for the root style object's own fields. Ours never exposes
  `style.center`/`zoom`/`bearing`/`pitch` — useful mainly so a downloaded style opens centered on
  Utah (or wherever the user cares about) instead of whatever the template happened to load with.
  Small, self-contained addition.
- [ ] **Reconsider: visual expression editor (ƒx toggle) for `interpolate`/`match`/`step`.**
  This is Maputnik's single biggest UX advantage over us — a real form (stops as add/remove rows,
  a mini chart for `interpolate`) instead of a raw-JSON textarea for data-driven properties. It's
  explicitly called out as a non-goal in `STYLE_EDITOR_PLAN.md` §12 for v1, and it's real effort
  (a stop-editor UI plus per-expression-type parsing), so it's listed here rather than promoted
  above — but it's the one Maputnik feature actually worth revisiting that scoping decision for,
  since zoom-band road/label styling in these themes leans on `interpolate` a lot.
- **Not planned — data-source management (add/edit/remove vector/raster/GeoJSON/image/video
  sources).** Maputnik supports this because it edits arbitrary styles; we deliberately don't —
  our styles' sources are always UGRC's 3 fixed Esri `VectorTileServer` endpoints (or, for an
  upload, whatever the file already has), so a general source editor has no real use case here.

## Notes

- None of these touch `src/`, `config/`, or the Python build pipeline — scope stays inside
  `docs/`, same as the editor plan.
- Re-check against `STYLE_EDITOR_PLAN.md` §12 ("Known gaps / non-goals for v1") before expanding
  any of these — e.g. the diff-share idea above is deliberately *not* the full-style URL sharing
  that section rules out, and the expression-editor idea explicitly revisits a documented
  non-goal rather than silently overriding it.
