# French with Harry Potter

A self-guided French learning app that teaches through original sentences inspired by *Harry Potter et le Prince de Sang-Mêlé* (Book 6). Designed for the journey from A1 up to CEFR B2 / CLB7 — the level targeted by the TEF and TCF Canada exams.

**Everything is a plain HTML file.** Open one in a browser. No build step, no server, no install.

---

## What makes this different

- **Example-first, never drill-first.** You meet grammar through real sentences, not flashcard quizzes of isolated rules.
- **Story-shaped.** Each lesson stays inside a scene from the book. Vocabulary and grammar build around the narrative — which means they stick.
- **Original French, not reproductions.** Every sentence is written fresh, inspired by the events of the scene. The copyrighted text is never reproduced.
- **Spaced repetition baked in.** Progress trackers, revision sessions, flashcard decks, and a mistakes log all work together.
- **Polished audio.** Play buttons speak every sentence. Works out of the box with your browser's French voices, and upgrades to ElevenLabs (natural-sounding neural TTS) if you have an API key.

---

## Quick start

1. Clone or download this repo.
2. Open `chapter1/french_lesson_ch1_scene1.html` in your browser.
3. Read the French sentence at the top, hover the vocab table, press ▶ to hear it spoken.
4. Scroll to the Practice card at the bottom. Translate the English sentence into French and press **Check**. The app scores each grammatical element and tells you exactly what you got wrong and why.
5. Repeat for Scene 2, Scene 3, … After every 6 scenes, open the revision session (`revisions/french_revision_s1-6.html`) to consolidate.
6. When you want a vocabulary warm-up or refresher, open `flashcards/french_flashcards_ch1.html`.

---

## Directory layout

```
chapter1/     Lesson files for Chapter 1 (scenes 1–9)
chapter2/     Lesson files for Chapter 2 (scenes 1+)
revisions/    Revision sessions (one per 6 scenes)
flashcards/   Vocabulary flashcard decks, one per chapter
progress/     Running vocab & grammar trackers, one per chapter
mistakes/     Mistakes dashboard; your downloaded mistakes log also lives here
.claude/      Claude Code skills for generating new scenes/revisions/flashcards
```

## File guide

| Pattern | What it is |
|---|---|
| `chapter[N]/french_lesson_ch[N]_scene[M].html` | A lesson. Three French sentences, each with a pronunciation guide, vocabulary table, and grammar insight. One translation practice at the bottom with per-element feedback. |
| `revisions/french_revision_s[X]-[Y].html` | A revision session covering 6 consecutive scenes (counted globally across chapters). Four new sentences that recombine grammar and vocab you've already met, with scene tags showing where each pattern comes from. |
| `flashcards/french_flashcards_ch[N].html` | A full vocabulary flashcard deck for a chapter. Space to flip, 1/2 to mark still-learning / got-it, ← → to navigate, P to play the example sentence. |
| `progress/french_progress_ch[N].md` | A running catalogue of every word, grammar rule, and fixed expression introduced in that chapter. Updated after every scene. Useful as a reference and a table of contents. |
| `mistakes/french_mistakes_ch[N]_viewer.html` | A dashboard that reads your downloaded mistakes log and displays it. |
| `CLAUDE.md` | Project context for [Claude Code](https://claude.com/claude-code), so an AI assistant can help you extend the app. Not needed if you just want to study. |
| `.claude/skills/` | Custom Claude Code skills for generating new scenes, revisions, and flashcards. See "Extending" below. |

---

## Voice setup

### Option A — Browser voices (free, zero setup)

The play buttons use your browser's Web Speech API with French voices. On macOS that's typically Thomas or Amélie. On Windows, whatever French voice you have installed. Just press play — it works.

### Option B — ElevenLabs (much more natural)

Get a free [ElevenLabs](https://elevenlabs.io) API key. Open any lesson, paste the key into the yellow "🎙 ElevenLabs API Key" bar at the top, press Connect. The key is stored in your browser's `localStorage` and applied automatically in every lesson, revision, and flashcard deck from then on — you only enter it once.

All audio uses the `eleven_multilingual_v2` model — native-sounding French narration.

---

## Mistakes log

The Practice card records every grammatical element you get wrong. Press the **↓ Download mistakes log** button to export a `french_mistakes_ch[N].md` file — a markdown table of every mistake grouped by scene. This file is gold for spaced review: open it on a Sunday morning, read through, retry the sentences.

Your mistakes log lives in browser `localStorage`. It persists across sessions and across lessons within the same chapter.

---

## What's included

### Chapter 1 — *The Other Minister*
- All 9 scenes available as standalone lesson files (`chapter1/`)
- Flashcard deck covering all 9 scenes — **80+ cards**
- Revision session for scenes 1–6 in teal

### Chapter 2 — *Spinner's End* (in progress)
- Scenes 1 and 2 complete
- Grammar introduced: *venir de* + infinitif, plus-que-parfait, *faire* + infinitif (causative), *se laisser* + infinitif, *d'une voix + adjectif*, superlatives

### Progress trackers
Open `progress/french_progress_ch1.md` and `progress/french_progress_ch2.md` for a complete, human-readable catalogue of every word, grammar rule, and fixed expression covered.

---

## Where to start depending on your level

- **Absolute beginner (A1)** — start with `chapter1/french_lesson_ch1_scene1.html` and go in order. Open the `flashcards/` deck alongside for vocab reinforcement. Expect 15–30 min per scene.
- **A2–B1** — skim `progress/french_progress_ch1.md`. If most of it looks familiar, jump to the revision session and the Chapter 2 scenes. The practice exercises will surface any gaps.
- **Preparing for TEF/TCF Canada** — the later scenes flag formal-register alternatives (`cela` vs `ça`, `cependant` vs `mais`, `bien que + subjonctif`) with a **formal** tag. This is built in deliberately — the lessons escalate from story French to exam-register French over the course of the chapters.

---

## Extending the app

If you use [Claude Code](https://claude.com/claude-code), this repo ships with three custom skills that automate the next chapter of learning:

| Skill | What it does |
|---|---|
| `/generate-scene` | Generates the next lesson in the sequence — 3 sentences with vocab, grammar, practice, and mistake-checking, all wired up. |
| `/generate-revision` | Generates a revision session after every 6 scenes, consolidating grammar across the window. |
| `/update-flashcards` | Creates or updates the flashcard deck for a chapter based on the progress tracker. |

The skills read your progress tracker so they don't repeat patterns, and they follow a set of critical rules (about data-tts attributes and single-quote apostrophe traps) that prevent the HTML from silently breaking.

Without Claude Code, every HTML file is self-contained and can be copied and modified by hand — the CSS, voice bar, and answer checker are all inline.

---

## Keyboard shortcuts (flashcards)

| Key | Action |
|---|---|
| Space | Flip card |
| ← | Previous card |
| → | Next card |
| 1 | Mark "still learning" |
| 2 | Mark "got it" |
| P | Play example sentence |

---

## Credits & boundaries

- **French sentences:** all original, written to echo the narrative of *Harry Potter et le Prince de Sang-Mêlé* without reproducing the copyrighted text. This is a study companion, not a translation.
- **Voice synthesis:** optional — [ElevenLabs](https://elevenlabs.io) `eleven_multilingual_v2` model, or the browser Web Speech API as fallback.
- **Fonts:** [Google Fonts](https://fonts.google.com) — Playfair Display + DM Sans.
- **Built collaboratively with** [Claude Code](https://claude.com/claude-code).

This is a personal learning project. Not affiliated with J.K. Rowling, Warner Bros, Bloomsbury, or Pottermore.

---

## License

MIT for the code and generated original content. The lesson design, CSS, skills, and original French sentences are free to use and adapt.
