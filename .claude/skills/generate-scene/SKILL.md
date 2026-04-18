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

### Design the Everyday Practice sentence

Every scene gets a **second** practice card — the "Everyday Practice" — in addition to the scene-themed translation. Its purpose is to drag one core grammar pattern out of the Harry Potter narrative and into daily French, so the learner produces it in contexts they'll actually use.

Design it like this:

1. **Pick ONE focal pattern** from the scene's new grammar. Prefer the most transferable one — something they'd use talking about family, routines, food, feelings, weather. Good candidates: *faire + infinitif*, *venir de + infinitif*, *se laisser + infinitif*, *avoir l'air + adjectif*, reflexives, *il faut que + subjonctif*, *même si*, *si + conditional*. Bad candidates: heavily literary patterns like *plus-que-parfait* in isolation, or scene-specific vocab (*le seuil, trahir*).
2. **Write a daily-life English sentence** — 6–12 words, situationally concrete (parents, school, weekend, dishes, tired, hungry, late, music). Do NOT reuse Harry Potter vocabulary.
3. **Write the model French answer** — should be at or slightly below the lesson's grammatical complexity. The focal pattern must be the core of the sentence.
4. **Build a `CHECKS_2` array** with 3–5 checks: the focal pattern itself, the key vocab, and any tricky supporting elements (possessives, definite articles, time expressions).
5. **Write a success message** that explicitly notes the pattern transferring out of the story.

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
| Main practice English prompt | `.practice-english` div (first card) |
| `MODEL_ANSWER`, `ENGLISH`, `CHECKS` | `<script>` (main practice) |
| `scene:` value in the first `saveMistake` | `'Chapter N, Scene M'` |
| First success message in `overall-feedback` fallback | `<script>` |
| Everyday Practice card | Below the main practice card, before `<script>` |
| Everyday prompt (`Use <em>pattern</em> — translate into French`) | `.practice-prompt` of second card |
| Everyday English sentence (daily-life) | `.practice-english` of second card |
| `MODEL_ANSWER_2`, `ENGLISH_2`, `CHECKS_2` | `<script>` (everyday practice) |
| `scene:` value in second `saveMistake` | `'Chapter N, Scene M (Everyday)'` |
| Second success message | `<script>` |

The Everyday Practice card uses the same CSS as the main practice card plus a `.practice-badge-daily` class that tints the badge green-mint. If that class is missing from the scene file (scenes predating this feature), add it to the `<style>` block:

```css
.practice-badge-daily{color:#6ecf9a;background:rgba(110,207,154,.12);border-color:rgba(110,207,154,.35);}
.practice-prompt em{font-style:italic;color:rgba(255,255,255,.75);text-transform:none;letter-spacing:normal;font-family:'Playfair Display',serif;}
```

The Everyday card uses its own element IDs — `practiceInput2`, `submitBtn2`, `feedbackBlock2`, `loadingMsg2`, `feedbackContent2`, `yapBtn2`, `mapBtn2` — so it never collides with the main card. A standalone `checkAnswer2()` function (copy of `checkAnswer` with `_2` suffixes everywhere) handles it. Add a keydown listener on `practiceInput2` so Enter triggers `checkAnswer2()`.

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

The same rule applies to success-banner text inside `banners.perfect`, `banners.mostly`, and `banners['needs-work']`. If your banner text contains an apostrophe (*j'étais*, *avoir l'air*, *c'est*), the surrounding string must be double-quoted:

```js
// ✅ CORRECT
perfect: { cls:'perfect', icon:'✓', text:"Excellent — avoir l'air landed cleanly." }

// ❌ WRONG — single quote + apostrophe silently crashes checkAnswer2
perfect: { cls:'perfect', icon:'✓', text:'Excellent — avoir l\'air landed cleanly.' }
```

When in doubt, default all banner text to double quotes.



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
- [ ] All `explain` strings use double quotes (both `CHECKS` and `CHECKS_2`)
- [ ] `KEY_MISTAKES`, download filename, and scene tag all match the chapter number
- [ ] Main practice card: `MODEL_ANSWER`, `ENGLISH`, `CHECKS` updated for the scene-themed sentence
- [ ] Everyday practice card added: `MODEL_ANSWER_2`, `ENGLISH_2`, `CHECKS_2`, `checkAnswer2()`, keydown listener, `.practice-badge-daily` in CSS
- [ ] Second `saveMistake` call uses scene label `'Chapter N, Scene M (Everyday)'`
- [ ] Progress tracker got new vocab table + new grammar rules + footer date updated
- [ ] Grammar rules numbered sequentially from previous highest

Then tell the user: the new lesson filename, the new sentence numbers, the new grammar rule range, and the practice sentence. Keep it brief.

## When NOT to use this skill

- User wants to edit an *existing* scene — just edit it, don't treat it as generation.
- User wants a revision session (`revisions/french_revision_s[X]-[Y].html`) — different format, 4 cards, scene tags, teal/green background. Different template.
- User wants flashcards or a mistakes viewer — different files, different purpose.
