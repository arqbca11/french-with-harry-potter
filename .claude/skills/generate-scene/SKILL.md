---
name: generate-scene
description: Generate the next French lesson scene (HTML lesson card + progress-tracker update) for the Harry Potter French learning project. Use when the user asks to "generate Chapter N Scene M", "make the next scene", "create the next lesson", or "do scene X".
---

# Generate the next French lesson scene

You are generating a 3-sentence lesson scene for the Harry Potter French learning app, plus updating the chapter's progress tracker. The project lives in the user's current working directory.

## Step 1 — Figure out what's next

Before writing anything, establish:

1. **Current chapter and next scene number.** Read `CLAUDE.md` (section "Where we are" and "Sentence numbering"). If the user specified a scene explicitly ("Chapter 2 Scene 3"), trust that.
2. **Next sentence numbers (3 sequential).** Sentences are numbered sequentially across all chapters. Read the latest existing lesson file and take its highest sentence number + 1, + 2, + 3.
3. **Next grammar rule number.** Read `progress/french_progress_ch[N].md` and take the highest rule number + 1. Rules are sequential across all chapters.
4. **What's already covered.** Skim the chapter's progress tracker so you don't reintroduce vocab or grammar the user already has. Build on — don't repeat.

## Step 2 — Read the canonical template

The **latest existing scene HTML file** is the canonical template, not `lesson_template.html`. The latest file has all critical runtime fixes already baked in (the `data-tts` pattern, the voice-bar visibility, the `french_ch[N]_mistakes` storage key, etc.). Read it end-to-end before writing the new scene.

To find it: `ls chapter*/french_lesson_ch*_scene*.html | sort -V | tail -1`. New scenes within an existing chapter go into the same `chapter[N]/` directory; new chapters start a fresh `chapter[N]/` directory at the repo root.

## Step 3 — Design the scene content

Pick 3 sentences that:

- Are **original French**, inspired by the scene from the book — never reproduce copyrighted text.
- Introduce **2–4 new grammar points** total across the three sentences, to be numbered sequentially from the next available rule number.
- Use vocab at a level appropriate to the learner's current stage (A1 → A2 early, building to B2). Mix known patterns with new ones.
- Reinforce patterns from recent scenes where natural.

Then design a **practice sentence** in English that combines elements from all three lesson sentences. The model French answer should exercise the new grammar points.

Flag formal/literary register with `<span class="formal-tag">formal</span>` when applicable (per the TEF/TCF register integration plan in CLAUDE.md).

## Step 4 — Write the lesson HTML

Copy the structure of the latest scene file and change **only** these parts:

| What to change | Where |
|---|---|
| `<title>` | `<head>` |
| Page subtitle (`Chapter N · Scene M`) | `.page-subtitle` div |
| Scene description (italic, 1–2 sentences) | `.scene-desc` div |
| 3 sentence blocks | `<!-- SENTENCE N -->` cards |
| Sentence numbers, French, English, pronunciation, vocab table, insight | Inside each card |
| Practice English prompt | `.practice-english` div |
| `MODEL_ANSWER` constant | `<script>` |
| `ENGLISH` constant | `<script>` |
| `CHECKS` array | `<script>` |
| `scene:` value in the `saveMistake` call | `'Chapter N, Scene M'` |
| Success message in `overall-feedback` fallback | `<script>` |

Do **not** touch: the CSS block (except the body background — see below), the voice-bar / key-entry / picker HTML, the `speak()` function, `fetchVoices`, `renderVoicePicker`, `loadMistakes`, `saveMistake`, `downloadMistakes`, or `norm()`.

### Chapter palette

Each chapter gets its own shade of blue on the body background. When you generate a scene inside an existing chapter, copy the body background line from the most recent scene in that chapter — it's already using the chapter's palette, so no lookup needed.

You only consult the palette table below when **starting the first scene of a new chapter**. Pick the next unused row.

```css
/* Ch1 — navy (baseline) */
body{background:#1e3a5f;background-image:radial-gradient(ellipse at top left,#2a5080 0%,#1e3a5f 55%,#162d4a 100%);}

/* Ch2 — steel teal-blue */
body{background:#1a4458;background-image:radial-gradient(ellipse at top left,#2a6580 0%,#1a4458 55%,#10283a 100%);}

/* Ch3 — indigo */
body{background:#2a3f6a;background-image:radial-gradient(ellipse at top left,#3a5590 0%,#2a3f6a 55%,#1e2f52 100%);}

/* Ch4 — deep midnight */
body{background:#16314f;background-image:radial-gradient(ellipse at top left,#22497a 0%,#16314f 55%,#0e2238 100%);}

/* Ch5 — royal */
body{background:#2c466c;background-image:radial-gradient(ellipse at top left,#3e6590 0%,#2c466c 55%,#1e304e 100%);}
```

Only the body background changes per chapter. Leave `--dark` (navy), the voice bar, pronunciation bar, and practice card colors untouched — the in-card UI stays consistent across chapters so the learning surface feels familiar. When Ch6 arrives, extend the palette with a sixth distinct blue.

## Step 5 — CRITICAL RULES (the ones that silently break everything)

These are non-negotiable. A single violation breaks the entire lesson page silently (voice bar stops, play buttons stop, checker stops).

### Rule A — Use `data-tts` on play buttons, never inline onclick text

```html
<!-- ✅ CORRECT -->
<button class="play-btn" id="btn31" data-tts="Il venait d'éteindre la lumière." onclick="speak(this.id)">

<!-- ❌ WRONG — the apostrophe in d'éteindre closes the JS string -->
<button class="play-btn" id="btn31" onclick="speak('btn31','Il venait d'éteindre la lumière.')">
```

French sentences contain apostrophes (`d'une`, `l'homme`, `qu'elle`, `c'est`) that break inline onclick strings. The `speak()` function already reads `btn.dataset.tts` when no text argument is passed.

### Rule B — Always use DOUBLE quotes for `explain` strings in CHECKS

```js
// ✅ CORRECT
{ explain: "Don't confuse <em>venir de</em> with <em>revenir</em>." }

// ❌ WRONG — apostrophe in "Don't" closes the string early, script crashes
{ explain: 'Don\'t confuse <em>venir de</em> with <em>revenir</em>.' }
```

English prose (*don't*, *it's*, *we're*, *you'd*, *isn't*) destroys single-quoted JS strings. Use double quotes and put inner English quotes in single quotes: `"'In a ___ voice' = <em>d'une voix + adjectif</em>."`

### Rule C — Storage and file naming

- `KEY_MISTAKES = 'french_ch[N]_mistakes'` — new chapter = new key
- Download filename: `'french_mistakes_ch[N].md'`
- `scene:` in `saveMistake({scene: 'Chapter N, Scene M', …})` — match exactly

## Step 6 — Update the progress tracker

Append to `progress/french_progress_ch[N].md`:

1. A new `### Scene M` subheading under **📚 Vocabulary** with a vocab table (French / English / Note).
2. Any new fixed-expression sets (body language, register alternatives, manner phrases) — group them in labeled mini-tables if there are 3+ of a kind.
3. New **grammar rules** under **🧱 Grammar**, numbered sequentially. Each rule: a short heading, one-sentence explanation, 2–4 example sentences with the target pattern bolded.
4. Update the footer: `*Last updated: Chapter N, Scene M*`
5. If the learner is past Month 2 of the plan, add or extend a **TEF/TCF register notes** section with casual ↔ formal equivalents drawn from the scene.

## Step 7 — Final checklist before handing back

Verify you did all of these:

- [ ] Three lesson cards, sentence numbers continue from last scene
- [ ] Each play button uses `data-tts="…"` (no inline onclick text)
- [ ] All `explain` strings use double quotes
- [ ] `KEY_MISTAKES`, download filename, and scene tag all match the chapter number
- [ ] `MODEL_ANSWER`, `ENGLISH`, `CHECKS` updated for the new practice sentence
- [ ] Progress tracker got new vocab table + new grammar rules + footer date updated
- [ ] Grammar rules numbered sequentially from previous highest

Then tell the user: the new lesson filename, the new sentence numbers, the new grammar rule range, and the practice sentence. Keep it brief.

## When NOT to use this skill

- User wants to edit an *existing* scene — just edit it, don't treat it as generation.
- User wants a revision session (`revisions/french_revision_s[X]-[Y].html`) — different format, 4 cards, scene tags, teal/green background. Different template.
- User wants flashcards or a mistakes viewer — different files, different purpose.
