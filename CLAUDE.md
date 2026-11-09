# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

"Text Curtain": a single-page canvas experiment where vertical text hangs on ropes (or fabric) that you can push, pluck and grab. Plain HTML/CSS/JS, no build step, no dependencies, no tests, no linter. Run it by opening `index.html` in a browser (Geist fonts load from Google Fonts, so it needs network for the intended look).

## Files

- `index.html`: canvas `#c` plus the settings panel shell (`#panel`, `#panelBody` is filled by JS).
- `styles.css`: panel chrome only.
- `script.js` (~1300 lines): everything else. It is one global-scope script, with no modules.

## Architecture (script.js, top to bottom)

- **`DEFAULTS` / `FIELDS` / `S`**: `DEFAULTS` holds every setting. `FIELDS` is the declarative panel layout (sections, scrub ranges, selects, notes). `modes: [...]` on a field or section limits it to Rope or Fabric. `effect` says what a change triggers: `"rebuild"` (`build()`, re-creates the grid) or `"chars"` (`assignChars()`, re-lays out text only). `S` is the live copy, merged from `localStorage["curtainSettings"]`; panel UI state (width, collapsed sections) is in `localStorage["curtainPanelUI"]`.
- **Adding a setting** means touching three places: add the default in `DEFAULTS`, add a `FIELDS` entry, and read `S.key` in the simulation. The panel, per-field reset, persistence and undo come for free. Bump `_v` and delete stale keys in the load block if an old saved value would break the new default.
- **Grid / text layout**: `build()` makes a `cols x rows` grid of particles `pts`. Each column is a rope (`ropes[c]`, top particle pinned). Text is laid out column by column, top to bottom, centred (`assignChars()`); the text is vertical, so a "line" is a column. `cellX` is line spacing and `cellY` is letter spacing, which can read backwards.
- **Physics**: `step()` dispatches on `S.mode` to `stepRope()` (independent verlet ropes, a solid "finger" cursor, grab and carry, per-line variation from the deterministic `rnd`) or `stepFabric()` (neighbour links in `cons`, a cursor that shoves the cloth). `MAX_SPEED` and `LINK_DAMP` tame whip-like flicks. `loop()` calls `step()` then `draw()` every frame.
- **Sound** (rope mode only): Web Audio. Each line that holds letters is a string with a pitch from `lineFreq(k, n)` spread across the chosen scale. `sndPluck` fires on finger contact, and `pluckBuffer` pre-renders the Karplus-Strong-style buffers, warmed in idle time by `sndWarm`. Audio has to be unlocked by a user gesture (`sndUnlock`).
- **Per-letter formatting**: `S.marks` is a string with one hex digit per character of `S.text` (bit 1 bold, 2 italic, 4 underline, 8 strike). `syncMarks` keeps it aligned when the text is edited. The text is 1:1 with the editor (newline becomes a space), so a selection index is also a letter index. Text is capped at 2400 letters.
- **Panel**: `makeScrub` (drag, click-to-type, arrow keys, Shift for fine), `makeDropdown`, `makeEditor` (rich textarea), `makeFormatBar`, `makeColors`, and `makeField` dispatches on `f.type`. Each field registers in `controls[key]` with a `set(value)`, used by `syncControls()`. Everything funnels through `applyEffect(key, effect)`, which saves and calls `record()` for undo/redo (JSON snapshots of `S`, with changes to the same key within 600 ms merged). `H` toggles the panel, and Ctrl+Z / Ctrl+Shift+Z or Ctrl+Y does undo/redo.

## Known issue

The last event listener in `script.js` (the `.minibar` scrollbar reveal near the end of the file) passes unquoted identifiers (`scroll`, `minibar`, `scrolling`) where strings are meant, so it does not work as intended.

## Notes

- Windows 11 / PowerShell environment. The parent folder (`..\Explorations`) holds one folder per experiment; this repo is just `dangling-text`.
- Keep it a plain-JS, no-build experiment, and keep the defaults at the top of `script.js`.
