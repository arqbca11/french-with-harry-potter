---
name: generate-revision
description: Generate a 4-sentence revision session consolidating grammar and vocabulary from a 6-scene window of French lessons. Use when the user asks to "generate a revision", "create a revision session", or "make revision for scenes X–Y". Also triggers automatically after every 6 completed scenes.
---

# Generate a French revision session

Revision sessions appear after every **6 scenes**. They consolidate grammar and vocabulary already learned, never introduce new material, and use a visually distinct teal color scheme.

## Global scene numbering

Scene numbers in revision filenames are **global** — counted sequentially across all chapters, not reset per chapter. A revision window can therefore span a chapter boundary.

Current mapping:

| Global scene | Chapter · Scene | Lesson file |
|---|---|---|
| 1 | Ch1 · Scene 1 | `chapter1/french_lesson_ch1_scene1.html` |
| 2 | Ch1 · Scene 2 | `chapter1/french_lesson_ch1_scene2.html` |
| … | … | … |
| 9 | Ch1 · Scene 9 | `chapter1/french_lesson_ch1_scene9.html` |
| 10 | Ch2 · Scene 1 | `chapter2/french_lesson_ch2_scene1.html` |
| 11 | Ch2 · Scene 2 | `chapter2/french_lesson_ch2_scene2.html` |

Always list the existing lesson files in sort-version order and assign global indices from 1:

```
ls chapter*/french_lesson_ch*_scene*.html | sort -V
```

## Step 1 — Determine the scene window

1. **Find the total scene count.** Count the lesson files above.
2. **Find the next unrevised window.** Revisions cover `s1-6`, `s7-12`, `s13-18`, and so on — windows of 6, never overlapping, always starting at a multiple-of-6 boundary + 1.
3. **Check the window is full.** If the user has only 11 scenes, the next window (`s7-12`) is not yet eligible — tell them they need N more scenes and stop. Do not generate early.
4. **Filename:** `revisions/french_revision_s[X]-[Y].html` — e.g. `revisions/french_revision_s7-12.html`. **No chapter prefix in the filename.**
5. **Resolve global → (chapter, scene)** for each scene in the window, so you know which lesson files to open.

If the user explicitly names a window (e.g. "generate revision for scenes 7–12"), trust that and still verify the files exist.

## Step 2 — Read the source material

Before writing anything, read:

1. **The latest lesson HTML file** — this is the canonical runtime template (ElevenLabs + `data-tts`, correct storage key, modern `speak()`). Use it as the structural base. The existing `revisions/french_revision_s1-6.html` uses a pre-ElevenLabs pattern and is **not** the template — do not copy it.
2. **All 6 scene HTML files in the window** — extract which grammar patterns and vocabulary were introduced in each. You will recombine these; never introduce new material.
3. **Every progress tracker touched by the window** (`progress/french_progress_ch1.md`, `progress/french_progress_ch2.md`, …) — to confirm grammar rule coverage across the window.

## Step 3 — Design the 4 revision sentences

Each sentence must:

- Be **original French**, combining grammar and vocabulary from **multiple scenes in the window**. Aim for 2–4 scene sources per sentence.
- Be more ambitious than a single lesson sentence — they're consolidation.
- Use **only** patterns the learner has already met. Never sneak in new grammar.
- Include the formal/literary register where appropriate — revisions are a good place to reinforce TEF/TCF alternatives (*cela*, *cependant*, *bien que + subjonctif*, etc.).

For each sentence prepare:

- French, English, pronunciation guide
- Vocab table (same 4-column format as lessons)
- Insight block highlighting how the patterns stack
- **Scene tags** — 3–4 chips showing which scenes each pattern comes from. Use per-chapter scene labels on the chips (not global numbers), e.g. `Ch1 S2 — reflexives`, `Ch2 S1 — venir de`. Global numbers belong in the filename only — chips should point the learner back to the lesson they know.

Then design a **practice sentence** combining elements from all 4 revision cards.

## Step 4 — Write the revision HTML

Copy the latest lesson file as the base and adapt:

### Structural differences from a lesson

| Lesson | Revision |
|---|---|
| 3 lesson cards | **4** revision cards |
| No scene tags on cards | Each card has `.scene-tags` + multiple `.scene-tag` chips above the French sentence |
| Blue/navy color scheme | **Teal** color scheme |
| Subtitle: `Chapter N · Scene M` | Subtitle: `Revision — Scenes X–Y` (no chapter prefix) |
| Practice badge: `✦ Practice` | Practice badge: `✦ Revision Practice` |
| `overall-feedback` copy | Custom text acknowledging cross-scene recall |

### Revision palette — different shade of green per revision

Every revision gets its own shade of green, indexed by **revision number**, not by scene window. Revision #1 is `s1-6`, Revision #2 is `s7-12`, Revision #3 is `s13-18`, and so on. Compute the revision number as `ceil(Y / 6)` where Y is the last scene in the window.

Pick the row matching the revision number. When Revision #6 arrives, extend the palette with a sixth distinct green.

```css
/* Revision #1 (s1-6) — teal (baseline) */
body { background: #1a3a3a; background-image: radial-gradient(ellipse at top left, #1e5050 0%, #1a3a3a 55%, #122a2a 100%); }
/* Panel color: #122a2a */

/* Revision #2 (s7-12) — forest */
body { background: #1e3a2c; background-image: radial-gradient(ellipse at top left, #2a5540 0%, #1e3a2c 55%, #122a1e 100%); }
/* Panel color: #122a1e */

/* Revision #3 (s13-18) — sage */
body { background: #2c3a2a; background-image: radial-gradient(ellipse at top left, #405538 0%, #2c3a2a 55%, #1e2a1a 100%); }
/* Panel color: #1e2a1a */

/* Revision #4 (s19-24) — deep emerald */
body { background: #123a2c; background-image: radial-gradient(ellipse at top left, #1e5540 0%, #123a2c 55%, #0a281e 100%); }
/* Panel color: #0a281e */

/* Revision #5 (s25-30) — moss */
body { background: #2a4030; background-image: radial-gradient(ellipse at top left, #3c583e 0%, #2a4030 55%, #1a2c1e 100%); }
/* Panel color: #1a2c1e */
```

Also shift `.pronunciation-bar`, `.practice-card`, and the voice picker backgrounds from `var(--dark)` (navy) to the **Panel color** shown for the chosen revision — the darkest stop of that revision's body gradient. Gold accents (`var(--gold)`) stay unchanged; they provide continuity across revisions.

Add the relevant semantic color variable to `:root` if helpful — e.g. `--teal: #1a4a4a;` for Rev #1, `--forest: #1e3a2c;` for Rev #2. Not required but keeps the stylesheet self-documenting.

### Scene tag styling

```css
.scene-tags { display: flex; gap: 6px; flex-wrap: wrap; margin-bottom: 8px; }
.scene-tag { font-size: 10px; letter-spacing: 1px; text-transform: uppercase; background: rgba(201,168,76,0.12); border: 1px solid rgba(201,168,76,0.3); color: var(--gold); border-radius: 4px; padding: 2px 7px; }
```

### Card header with tags

```html
<div class="card-header">
  <div>
    <div class="scene-tags">
      <span class="scene-tag">Ch1 S2 — reflexives</span>
      <span class="scene-tag">Ch1 S4 — avoir peur</span>
      <span class="scene-tag">Ch1 S4 — ce qui</span>
    </div>
    <div class="french-sentence">…</div>
    <div class="english-sentence">…</div>
  </div>
  <button class="play-btn" id="btnR1" data-tts="…" onclick="speak(this.id)">…</button>
</div>
```

Use button IDs `btnR1`, `btnR2`, `btnR3`, `btnR4` to avoid colliding with lesson sentence numbering.

### Mistakes storage — cross-chapter handling

Revisions save mistakes into the chapter-level log for the **chapter that contains the LAST scene in the window**. Rationale: by the time the learner runs a revision, they've progressed into the later chapter, so that's where the revision mistake naturally lives.

- `s1-6` → all in Ch1 → `KEY_MISTAKES = 'french_ch1_mistakes'`, download filename `'french_mistakes_ch1.md'`
- `s7-12` → last scene (12) is Ch2 S3 → `KEY_MISTAKES = 'french_ch2_mistakes'`, download filename `'french_mistakes_ch2.md'`

In the `saveMistake` call, use a window label that makes the cross-chapter case legible:

```js
scene: 'Revision — Scenes X–Y'
```

No chapter prefix in the scene string. The mistakes viewer and downloaded markdown will show a clean `## Revision — Scenes 7–12` heading regardless of which chapter log the entries live in.

## Step 5 — CRITICAL RULES (same as lessons)

These apply to revisions exactly as they do to lessons. A single violation silently kills the page.

- **`data-tts` attribute** on every play button — NEVER inline onclick with text. French apostrophes break inline strings.
- **Double quotes** on every `explain` string in `CHECKS`. English prose apostrophes break single-quoted strings.
- **`KEY_MISTAKES`** and download filename match the chapter of the *last* scene in the window.
- **`scene:` in `saveMistake`** is `'Revision — Scenes X–Y'` (no chapter prefix).

See the lesson-generation skill for the full reasoning if these rules need a refresher.

## Step 6 — Final checklist

- [ ] Filename: `revisions/french_revision_s[X]-[Y].html` — no `ch[N]` segment in the filename itself
- [ ] X and Y are global scene numbers (count across chapters, window of exactly 6)
- [ ] Page subtitle: `Revision — Scenes X–Y` (no chapter prefix)
- [ ] 4 revision cards, each with scene-tag chips using per-chapter labels (`Ch1 S2`, `Ch2 S1`)
- [ ] Teal color scheme (body, pronunciation bar, practice card, voice picker)
- [ ] All play buttons use `data-tts`
- [ ] Practice badge reads `✦ Revision Practice`
- [ ] `scene:` in `saveMistake`: `'Revision — Scenes X–Y'`
- [ ] `KEY_MISTAKES` and download filename match the chapter of the LAST scene in the window
- [ ] `CHECKS` explain strings are all double-quoted
- [ ] No new grammar or vocabulary introduced — only recombination

Tell the user: filename, global scene window covered (with the chapter·scene mapping so they know what's in it), which grammar threads got consolidated, and the practice sentence. Keep it brief.

## When NOT to use this skill

- The next 6-scene window isn't full yet — tell the user how many scenes remain, don't generate early.
- The user wants a regular lesson — use `generate-scene` instead.
- The user wants flashcards — use `update-flashcards` instead.
