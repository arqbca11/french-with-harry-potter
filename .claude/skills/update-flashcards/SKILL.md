---
name: update-flashcards
description: Create or update the vocabulary flashcard deck for a chapter. Use when the user asks to "update flashcards", "add flashcards for scene N", "create flashcards for chapter N", or "refresh the flashcard deck". Also suitable when a new scene or two has been added and the flashcards should catch up.
---

# Update the chapter vocabulary flashcards

Each chapter has one flashcard deck at `flashcards/french_flashcards_ch[N].html`. Cards are vocabulary items pulled from the chapter's lessons — one per useful word, phrase, or grammar chunk. This skill either creates the file for a new chapter or appends cards for newly added scenes.

## Step 1 — Decide: create or update

```
ls flashcards/french_flashcards_ch[N].html
```

- **File does not exist → create mode** (Step 2A).
- **File exists → update mode** (Step 2B).

## Step 2A — Create mode (new chapter)

1. **Copy the existing Ch1 deck as the structural template.** `flashcards/french_flashcards_ch1.html` has the full CSS, flip mechanics, ElevenLabs integration, keyboard shortcuts, session summary, and filter dropdown. Replicate everything except the content.
2. **Change the title** and `.page-subtitle` to the new chapter.
3. **Empty the `CARDS` array** — you'll populate it in Step 3.
4. **Update the filter dropdown** to match available scene groupings (see Step 4).

## Step 2B — Update mode (existing file)

1. **Read the existing `CARDS` array** to see what's already covered. Do not duplicate — a card for the same `fr` term already in the deck should be skipped, even if the user ran `update-flashcards` twice.
2. **Determine which scenes are new.** Compare existing scene tags (the `s:` fields in the CARDS array) against the scenes present in the chapter (`ls chapter[N]/french_lesson_ch[N]_scene*.html`).
3. **Append new cards** to the end of the `CARDS` array for the uncovered scenes.
4. **Update the filter dropdown** if new scene groupings are needed (Step 4).

## Step 3 — Populate cards from the lessons

For **each scene** being covered, pull cards from two sources:

### Source A — The scene's vocab table (required)

Open `chapter[N]/french_lesson_ch[N]_scene[M].html` and extract every row from the three `<table>` blocks. Each row becomes a card candidate. Filter out duplicates across the three tables of the same scene.

### Source B — The chapter's progress tracker (required)

Open `progress/french_progress_ch[N].md` and read the `### Scene M` vocab table and any grammar patterns worth turning into cards (e.g. `venir de + infinitif` deserves a card with an example; an entire multi-paragraph grammar rule does not).

### Each card has 6 fields

```js
{
  s:  'Scene M',                          // scene tag — individual per scene (not grouped) unless scene has <3 items
  fr: 'venir de + infinitive',            // the French term or pattern — what the learner sees on the front
  en: 'to have just done something',      // concise English gloss — on the back
  note: 'Present for "just now"; imperfect for "had just"',   // one-line usage note
  ex: 'Il venait de partir quand je suis arrivé.',            // example sentence — prefer one from a lesson; fall back to a natural original if needed
  exEn: 'He had just left when I arrived.'                    // example translation
}
```

**Example selection:** whenever possible, `ex` should be one of the three sentences from the scene the card comes from, or a direct echo of it. This reinforces the lesson's context. If the lesson sentence is too long or illustrates multiple patterns, write a shorter original sentence that uses only this card's target.

**Quality bar for what becomes a card:**
- ✅ Single vocabulary word (*le seuil, trahir, mal éclairé*)
- ✅ Fixed expression (*les bras croisés, d'une voix calme, la moindre*)
- ✅ Grammar pattern used like a chunk (*faire + infinitif, se laisser + infinitif, ne … plus*)
- ✅ Collocation worth memorising (*avoir l'air + adjectif, responsable de*)
- ❌ Entire rules with multi-sentence explanations — those stay in the progress tracker
- ❌ Inflected forms the learner will derive automatically (skip *tombée* if *tomber* is already a card)

## Step 4 — Update the filter dropdown

The filter groups scenes into manageable sets. Default grouping: every 3 scenes per group, matching the flashcard browsing experience of Ch1 (`Scenes 1–3 / 4–6 / 7–9`). For Ch2 with sparser vocabulary per scene, you can group larger windows or keep per-scene options — use judgement based on total card count.

The `m` mapping object in `applyFilter()` needs to match the dropdown `<option value="…">` keys and use the same scene-tag strings as the CARDS array:

```js
const m = {
  s1: ['Scene 1', 'Scene 2', 'Scene 3'],
  s2: ['Scene 4', 'Scene 5', 'Scene 6'],
};
```

Also update the `<option>` labels in the `<select class="fsel">` block to match.

## Step 5 — String escaping in the CARDS array

Each card field is a **single-quoted JS string literal** in the `CARDS` array. French apostrophes must be escaped with a backslash:

```js
// ✅ CORRECT
{s:'Scene 7', fr:'s\'appeler', en:'to be called', ex:'Il s\'appelle Severus.', exEn:'His name is Severus.'}

// ❌ WRONG — the apostrophe in s'appeler closes the string
{s:'Scene 7', fr:'s'appeler', en:'to be called', ex:'Il s'appelle Severus.', exEn:'His name is Severus.'}
```

This is different from the lesson-generation rules because flashcard cards are pure data strings — they are displayed via `.textContent` assignment, not parsed as HTML or passed through onclick attributes. The single-quote + escape pattern is safe here. The *reason* it's safe: the CARDS array is not in an HTML attribute and not executed as onclick — so there's only one layer of quoting to worry about. Still, **always escape French apostrophes with `\'`** inside card strings.

Double-quoted card strings are also acceptable if you prefer — consistency with the existing deck wins.

## Step 6 — Per-card background color

Every flashcard shows a different pastel color on its front face. The color is chosen by a deterministic hash of the `fr` string, so the same card always appears in the same color — no flickering on re-render, stable across shuffles and sessions.

The deck's `<script>` block already contains (or should contain — verify on update) these two pieces:

```js
const CARD_COLORS = ['#faf7f2','#fdeee3','#e8efdd','#e0ebf3','#ece2f0','#faf3d8','#f6e3e8','#dff0e7','#fce4d6','#dde4f2','#ebe5d8','#e9dde9'];
function cardColor(fr) {
  let h = 0;
  for (let i = 0; i < fr.length; i++) h = ((h << 5) - h + fr.charCodeAt(i)) | 0;
  return CARD_COLORS[Math.abs(h) % CARD_COLORS.length];
}
```

And `renderCard()` contains this line right after the `cfrench` textContent assignment:

```js
document.querySelector('.cfront').style.backgroundColor = cardColor(c.fr);
```

On **create mode**, include all three pieces in the new deck.

On **update mode**, verify they exist. If the existing file predates this feature, add them — the palette and hash function belong near the other top-of-script constants; the `cardColor()` call belongs immediately after `document.getElementById('cfrench').textContent = c.fr;` in `renderCard()`.

The back face stays dark navy across all cards — only the front varies. The ink-colored French text must remain readable, so keep any palette additions in the pastel / light-tint range. If you extend `CARD_COLORS` later, choose colors with luminance > 85% so `var(--ink)` text stays legible.

## Step 7 — TTS and voice setup

The flashcard file already uses the ElevenLabs + Web Speech fallback pattern. It reads the example sentence from `document.getElementById('efrText').textContent` at playback time, so there's no inline onclick TTS text to worry about — the apostrophe trap doesn't apply here.

Do not modify the voice-setup code unless asked. Leave `K_API`, `K_VOICE`, `playAudio()`, `stopAudio()`, and the keyboard shortcuts alone.

## Step 8 — Final checklist

- [ ] `CARDS` array populated with new cards (create mode) or appended with new cards (update mode)
- [ ] No duplicate `fr` fields across cards
- [ ] Every French apostrophe inside a card string is escaped (`s\'appeler`, `quelqu\'un`, `c\'est`)
- [ ] Every card has all 6 fields: `s, fr, en, note, ex, exEn`
- [ ] `ex` uses or closely echoes a sentence from the scene when possible
- [ ] Filter dropdown options and the `m` mapping in `applyFilter()` both reflect available scenes
- [ ] Total card count shown in `.prow` progress text and `.nnum` counter updates automatically — don't hard-code
- [ ] Title and subtitle match the chapter
- [ ] `CARD_COLORS` array and `cardColor()` function are present in the `<script>` block
- [ ] `renderCard()` calls `document.querySelector('.cfront').style.backgroundColor = cardColor(c.fr);` after setting the French text

Tell the user: filename, how many cards were added (or total in a new deck), the scene range covered, and which filter groups now exist.

## When NOT to use this skill

- The user wants to generate a lesson — use `generate-scene`.
- The user wants a revision session — use `generate-revision`.
- The user wants to view or review mistakes — different file (`mistakes/french_mistakes_ch[N]_viewer.html`).
