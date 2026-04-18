# CLAUDE.md — French Learning Project
*Context file for Claude Code and new conversations*

---

## What this project is

An interactive French learning app built around *Harry Potter et le Prince de Sang-Mêlé* (Book 6). The learner is at A1 level, targeting CLB7 (≈ CEFR B2) for TEF/TCF Canada. Each lesson uses original French sentences inspired by scenes from the book — never reproducing the copyrighted text directly. Learning philosophy: example-first, grammar emerges from real sentences, no drilling.

---

## Where we are

- **Chapter 1** — 9 scenes complete (Sentences 1–27)
- **Chapter 2** — Scene 1 complete (Sentences 28–30), Scene 2 is next
- **Chapter 2 is:** *La Ruelle Sans Issue* (Spinner's End) — Narcissa and Bellatrix visiting Snape

---

## File naming conventions

| File | Contents |
|---|---|
| `french_lesson_ch[N]_scene[N].html` | Lesson card — use template |
| `french_revision_s[X]-[Y].html` | Revision session after every 6 scenes — X and Y are global scene numbers (counted across all chapters) |
| `french_progress_ch[N].md` | Vocab & grammar tracker — one per chapter |
| `french_mistakes_ch[N]_viewer.html` | Mistakes dashboard — one per chapter |
| `french_flashcards_ch[N].html` | Flashcard app — one per chapter |
| `prompt_history.md` | Feature changelog + learning plan |
| `lesson_template.html` | Reusable lesson template — read this first |
| `CLAUDE.md` | This file |

---

## How to generate a new lesson

1. **Read `lesson_template.html`** first — it contains all boilerplate CSS, voice bar, TTS, mistakes storage, and answer checker
2. Copy it and fill in the `%%PLACEHOLDER%%` variables (see table below)
3. Use a Python script to do the substitution — do NOT regenerate the boilerplate from scratch

### Placeholder table

| Placeholder | Replace with |
|---|---|
| `%%CHAPTER%%` | e.g. `Chapter 2` |
| `%%SCENE%%` | e.g. `Scene 2` |
| `%%SCENE_DESC%%` | Short italic scene-setting sentence |
| `%%N1%%` / `%%N2%%` / `%%N3%%` | Sentence numbers (sequential) |
| `%%FRENCH_n%%` | French sentence |
| `%%ENGLISH_n%%` | English translation in quotes |
| `%%PRONUN_n%%` | Pronunciation guide (natural, not phonetic symbols) |
| `%%TTS_n%%` | Text for TTS — put in `data-tts` attribute, NOT inline onclick |
| `%%TABLE_ROWS_n%%` | HTML `<tr>` rows for vocab table |
| `%%INSIGHT_n%%` | Grammar insight with `<strong>`, `<span class="word-highlight">`, `<span class="formal-tag">formal</span>` |
| `%%PRACTICE_ENGLISH%%` | English sentence to translate |
| `%%MODEL_ANSWER%%` | Correct French answer |
| `%%PRACTICE_ENGLISH_PLAIN%%` | Same, no HTML |
| `%%CHECKS_ARRAY%%` | JS check objects (see rules below) |
| `%%SUCCESS_MSG%%` | Message on perfect score |

---

## ⚠️ Critical rules for CHECKS_ARRAY

**Always use double quotes for `explain` strings — never single quotes.**

English prose contains apostrophes (don't, it's, we're, you'd) which break single-quoted JS strings and **silently crash the entire script** — the voice bar stops working, play buttons do nothing, checker does nothing.

```js
// ✅ CORRECT
{ explain: "Don't confuse <em>venir de</em> with <em>revenir</em>." }

// ❌ WRONG — the apostrophe in "Don't" closes the string early
{ explain: 'Don\'t confuse <em>venir de</em> with <em>revenir</em>.' }
```

**Also:** TTS text for play buttons must use `data-tts` HTML attributes, not inline `onclick` strings. Apostrophes in French (d'éteindre, l'homme, c'est) break inline onclick strings.

```html
<!-- ✅ CORRECT -->
<button class="play-btn" id="btn30" data-tts="Il venait d'éteindre la lumière." onclick="speak(this.id)">

<!-- ❌ WRONG — apostrophe in d'éteindre closes the JS string -->
<button class="play-btn" id="btn30" onclick="speak('btn30','Il venait d'éteindre la lumière.')">
```

The `speak()` function in the template already handles `data-tts` — it reads `btn.dataset.tts` when no text argument is passed.

---

## Voice / TTS setup

- Uses **ElevenLabs** API (`eleven_multilingual_v2` model)
- API key stored in browser `localStorage` under key `elevenlabs_api_key`
- Selected voice ID stored under `elevenlabs_voice_id`
- Falls back to Web Speech API if no ElevenLabs key present
- Voice bar is `display:flex` in CSS (always visible) — JS updates the name label

---

## Mistakes storage

- **Chapter 1** mistakes: `localStorage` key `french_ch1_mistakes`, download as `french_mistakes_ch1.md`
- **Chapter 2** mistakes: `localStorage` key `french_ch2_mistakes`, download as `french_mistakes_ch2.md`
- New chapter = new storage key
- `french_mistakes_ch[N]_viewer.html` reads from the relevant key and shows a dashboard

---

## Progress tracker rules

- One `french_progress_ch[N].md` per chapter
- Update after every scene — append new vocab, grammar rules, useful phrases
- Grammar rules are numbered sequentially across all chapters (Ch1 ends at rule 47, Ch2 starts at 48)
- Add a TEF/TCF register section flagging formal alternatives

---

## Learning plan summary

**Goal:** CLB7 (CEFR B2) on TEF/TCF Canada
**Starting level:** A1 | **Daily study:** 3–4 hrs

| Stage | Duration |
|---|---|
| A1 → A2 | 5–6 weeks |
| A2 → B1 | 6–8 weeks |
| B1 → B2 | 8–10 weeks |

**Formal register integration (built into lessons):**
- Now → Month 2: pure story method
- Month 2–3: slip in formal equivalents alongside casual (ça → cela, mais → cependant)
- Month 3–4: swap some practice for mini formal writing prompts
- Month 4–5: TEF/TCF-style lesson cards (lettre formelle, argument structure)
- Month 5–6: mock exam conditions

Flag formal vocabulary in lessons with `<span class="formal-tag">formal</span>`.

---

## Revision sessions

Generate a revision session (`french_revision_s[X]-[Y].html`) after every 6 scenes. **Scene numbers X and Y are global** — counted sequentially across all chapters (Ch1 Scene 1 = 1 … Ch1 Scene 9 = 9, Ch2 Scene 1 = 10, etc.). Revision windows may therefore cross chapter boundaries. Each revision has:
- 4 new original sentences combining grammar/vocab from the specified scenes
- Scene tags on each card showing which scenes the patterns come from (use the same `Chapter N · Scene M` or lesson-local scene label on the chip — global numbers only appear in the filename)
- Teal/dark green background to visually distinguish from lessons
- Same practice card format with full checker

Global scene index so far:
- Scenes 1–9 → Chapter 1 Scenes 1–9
- Scene 10 → Chapter 2 Scene 1
- Scene 11 → Chapter 2 Scene 2

---

## Learner profile

- Background: architecture + computer science, software engineer
- Learning style: example-first, dislikes drilling the same thing repeatedly in one session
- Spaced repetition across sessions preferred
- Living in the US for 6+ years — English is strong
- Studied German for 2 years in college — some European language intuition
- Not currently working — can do 3–4 hours/day

---

## Sentence numbering

Sentences are numbered sequentially across all scenes and chapters:
- Ch1 Scenes 1–9: Sentences 1–27
- Ch2 Scene 1: Sentences 28–30
- Ch2 Scene 2: starts at Sentence 31

---

## Grammar rule numbering

Rules are numbered sequentially across all chapters:
- Ch1 covered rules 1–43
- Ch2 Scene 1 added rules 44–47 (venir de, plus-que-parfait, mal + past participle, perception verbs)
- Ch2 Scene 2 starts at rule 48
