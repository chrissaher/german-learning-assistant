# Language Learning Assistant — Architecture & Reusable Framework

## Overview

This is a **skill-based language learning platform** for generating structured, AI-driven practice materials at a consistent proficiency level. The system was originally built for German A1 (Momente A1 textbook) but is designed to be portable to any language pair and proficiency level.

### What This System Does

- **Generates content on demand** via Claude Code skills
- **Maintains consistent level** across all materials (A1, A2, B1, etc.)
- **Provides multiple practice modalities**: sentence drilling, grammar explanation + exercises, vocabulary extraction + exercises
- **Produces styled HTML** for easy review and sharing
- **Supports dark mode** automatically
- **Tracks learner context** via CLAUDE.md for personalized corrections and transfer notes

---

## Learner Profile Template

Before starting, define the learner's context in this section. This drives all content generation. Update as the learner progresses.

```yaml
Native Language:             [e.g., Spanish]
Other Strong Languages:      [e.g., English]
Instruction Language:        [e.g., English]
Current Level:               [e.g., A1, started 2026-03-04]
Target Level:                [e.g., A2 by 2026-06-01]
Goal:                        [e.g., Travel & tourism in German-speaking countries]
Textbook/Curriculum:         [e.g., Momente A1 (Hueber Verlag)]
Correction Style:            [strict | balanced | conversational]
Special Notes:               [learner-specific context, allergies to topics, etc.]
```

### Current Instance (German)

| Field | Value |
|-------|-------|
| **Native language** | Spanish (also fluent in English) |
| **Instruction language** | English (all lessons and tools use English) |
| **Current level** | A1 (started 2026-03-04, progressing daily) |
| **Goal** | Travel and tourism in German-speaking countries |
| **Textbook** | Momente A1 (Hueber Verlag) |
| **Correction style** | **Strict** — flag every error, including minor spelling, gender, case, and word order mistakes. Always show the correct form and briefly explain why. |

---

## Guidelines for All Skills

Apply these principles to every skill regardless of language pair or level.

1. **Level-appropriate content** — keep all vocabulary and grammar within the learner's current level unless explicitly asked to go beyond. Update the level tracker (below) as the learner progresses.

2. **Instruction language** — explanations and instructions use the learner's instruction language (e.g., English); target language only for practice content.

3. **Language transfer notes** — when relevant, point out similarities or false friends:
   - **Learner's native → target language**: e.g., Spanish gender vs. German gender
   - **Learner's other strong languages → target language**: e.g., English→German gender systems
   - Include both "helpful parallels" and "dangerous false friends"

4. **Textbook alignment** — when possible, align vocabulary and grammar topics with the learner's textbook chapter sequence.

5. **Correction style** — apply the learner's specified correction style (above) in all explanations and transfer notes.

6. **Realistic context** — sentences and examples should reflect the learner's stated goal (e.g., travel, tourism, daily life, professional) wherever natural.

---

## Project Structure

```
index.html                           ← master index (all lessons)
styles.css                           ← shared stylesheet (light + dark mode)
data/
  sentences/                         ← JSON source (sentence-builder output)
  sentences-html/                    ← rendered sentence pages
  grammar/                           ← JSON source (grammar-explainer output)
  grammar-html/                      ← rendered grammar pages
  vocabulary/                        ← JSON source (vocab-drill output)
  vocabulary-html/                   ← rendered vocabulary pages
    index.html                       ← vocabulary index (by lektion)
.claude/skills/                      ← skill definitions
  sentence-builder/SKILL.md
  sentence-renderer/SKILL.md
  grammar-explainer/SKILL.md
  vocab-drill/SKILL.md
resources/                           ← source material (textbook PDFs, glossaries, etc.)
CLAUDE.md                            ← this file (project context)
README.md                            ← user-facing documentation
```

---

## Skill Specifications

Four core skills generate all content. Each skill is defined in `.claude/skills/{name}/SKILL.md`. Adapt these specifications for your language pair.

### 1. Sentence Builder

**Purpose:** Generate N example sentences for a given word, grouped by difficulty, with translations and grammar notes.

**Invocation:** `sentence builder: [word]` or `sentence builder: [word] ([type])`

**Output:** JSON saved to `data/sentences/{word_slug}_{YYYY-MM-DD_HHMMSS}.json`

**Key Parameters (customize per level/language):**
- **Sentence count:** 10 total (3 simple, 3 medium, 4 complex)
  - *For other levels, adjust; for A1, keep at 10*
- **Grammar coverage (if word is verb):** at least 4 different persons (ich/tu, 3rd person, plural, formal); Präsens for most, Perfekt for complex
  - *Customize based on tense system of target language*
- **Grammar coverage (if word is noun):** singular + plural; nominative + accusative; definite + indefinite articles
  - *Customize based on case/gender system of target language*
- **Grammar coverage (if word is adjective):** predicate and attributive positions, with agreement shown
- **Vocabulary scope:** all A1-appropriate (or the learner's current level)
- **Context:** travel/tourism-relevant wherever natural, but allow other realistic contexts

**JSON Schema:**
```json
{
  "word": "<the word as user typed it>",
  "base_form": "<dictionary form>",
  "word_type": "<noun|verb|adjective|adverb|other>",
  "generated_at": "<ISO 8601 datetime>",
  "sentences": [
    {
      "id": 1,
      "german": "<target-language sentence>",
      "english": "<instruction-language translation>",
      "level": "<simple|medium|complex>",
      "grammar_note": "<1-line grammar note>"
    }
  ]
}
```

---

### 2. Sentence Renderer

**Purpose:** Convert unrendered sentence JSON files to styled HTML pages; skip already-rendered files.

**Invocation:** `render sentences`

**Output:** HTML saved to `data/sentences-html/{stem}.html` for each unrendered JSON

**Logic:**
1. List all `.json` files in `data/sentences/`
2. List all `.html` files in `data/sentences-html/`
3. Identify "unrendered" files (JSON stems not in HTML stems)
4. For each unrendered JSON, generate an HTML page grouping sentences by level
5. Regenerate `index.html` to include new files

**HTML Structure:**
- Header with word, word type, and generation date
- Navigation anchors (Simple | Medium | Complex)
- Sentence sections grouped by level
- Each sentence shows: ID + level badge + German + → translation + 📝 grammar note
- Footer with generation info
- Links to shared `styles.css` (no inline CSS)

---

### 3. Grammar Explainer

**Purpose:** Generate a full grammar lesson: explanation + 10+ examples + 12+ exercises (4 types) + solution key.

**Invocation:** `grammar: [topic]` or `grammar: [topic], focus: [specifics]`

**Output:**
- JSON → `data/grammar/{topic_slug}_{YYYY-MM-DD_HHMMSS}.json`
- HTML → `data/grammar-html/{topic_slug}_{YYYY-MM-DD_HHMMSS}.html`

**Explanation Sections (include all that apply):**
- **What is it?** — 1–2 sentence plain-language definition
- **When to use it** — rules and conditions
- **How to form it** — construction steps or tables
- **Common exceptions** — irregular forms at the learner's level
- **[Language 1] transfer note** — parallel or false friend (native → target)
- **[Language 2] transfer note** — parallel or false friend (other language → target)

Plus:
- **Transfer Notes** block (2–4 bullets covering both language pairs)
- **Key Takeaway** — one bold sentence with the single most critical point

**Examples:** 10+ total, grouped by level (3 simple / 3 medium / 4+ complex)
- Each example must directly illustrate the grammar topic
- One-line grammar note per example

**Exercises:** 12+ total, distributed across 4 types
1. **fill_in_blank** — sentence with `___` blanks; learner fills with correct form
2. **multiple_choice** — sentence with blank and 3 options (a/b/c); one correct answer
3. **translation** — English sentence; learner writes target-language sentence
4. **error_correction** — target-language sentence with one error; learner corrects it

Complexity progression: ~1/3 simple, ~1/3 medium, ~1/3 complex.

**JSON Schema:**
```json
{
  "topic": "<topic as user typed>",
  "focus": "<focus or null>",
  "topic_slug": "<lowercase, spaces→underscores, umlauts→ASCII>",
  "generated_at": "<ISO 8601>",
  "explanation": {
    "sections": [
      {"title": "<section>", "content": "<full text with markdown tables>"}
    ],
    "transfer_notes": ["<bullet>", "<bullet>"],
    "key_takeaway": "<sentence>"
  },
  "examples": [
    {
      "id": 1,
      "german": "<sentence>",
      "english": "<translation>",
      "level": "<simple|medium|complex>",
      "grammar_note": "<1-line>"
    }
  ],
  "exercises": [
    {
      "id": 1,
      "type": "<fill_in_blank|multiple_choice|translation|error_correction>",
      "level": "<simple|medium|complex>",
      "prompt": "<exercise text>",
      "options": ["a) ...", "b) ...", "c) ..."],  // only for multiple_choice
      "solution": "<correct answer>",
      "explanation": "<why this is correct>"
    }
  ]
}
```

---

### 4. Vocabulary Drill

**Purpose:** Extract vocabulary from textbook glossary (PDFs or other source) for a chapter; generate 40+ exercises covering all focus categories; provide a reference section.

**Invocation:** `vocab drill: lektion [N], [focus]`
- `lektion`: chapter number
- `focus`: `nouns` | `verbs` | `adjectives` | `others` | `all`

**Output:**
- JSON → `data/vocabulary/lektion{NN}_{focus}_{YYYY-MM-DD_HHMMSS}.json`
- HTML → `data/vocabulary-html/lektion{NN}_{focus}_{YYYY-MM-DD_HHMMSS}.html`
- Index → `data/vocabulary-html/index.html` (regenerated, grouped by lektion)

**PDF Source Format** (customize per textbook)

For this implementation:
- Lektionen 1–12 → `resources/glossar_a1_1.pdf`
- Lektionen 13–24 → `resources/glossar_a1_2.pdf`

**Word Type Classification** (adjust for target language)

| PDF Notation | Word Type | Example |
|--|--|--|
| Starts with article (der/die/das) | noun | der Name, -n |
| Infinitive form, no article | verb | heißen, arbeiten |
| No article, not verb, descriptive | adjective | groß, schön |
| Other single words | other | sehr (adverb), in (prep), und (conj) |
| Multi-word phrases | other (phrase) | Wie geht's?, Guten Morgen |

**Enrichment** (add to each word in JSON):

For **nouns:**
- `article`: der / die / das
- `gender`: masc / fem / neut
- `plural_marker`: from PDF notation (-n, -en, -, ..e, etc.)
- `plural_full`: complete plural form
- `pattern_hint`: one of a predefined set of gender rules (e.g., "-ung → always die")
- `pattern_reliability`: high / moderate / memorize

For **verbs:**
- `conjugation_praesens`: object with keys {ich, du, er/sie/es, wir, ihr, sie/Sie}
- `perfekt`: full Perfekt form (e.g., "hat gemacht", "ist gegangen")
- `auxiliary`: haben / sein
- `is_irregular`: boolean
- `verb_note`: optional note for separable verbs, modals, etc.

For **adjectives:**
- `antonym`: opposite adjective, or null
- `cognate_note`: optional note if cognate or false friend

For **others:**
- `subtype`: adverb / preposition / pronoun / conjunction / interjection / phrase
- `usage_note`: one sentence describing use in the learner's level

**Examples:** 8–12 medium + complex level only
- Each sentence must use at least one vocabulary word from the current chapter
- May use vocabulary from earlier chapters for supporting words
- Realistic context per the learner's goal

**Exercises:** 40+ total, distributed across 6 types:
1. **fill_in_blank** — sentence with `___`; learner fills correct form
2. **multiple_choice** — sentence with blank and 3 options
3. **translation** — English sentence; learner writes target sentence
4. **error_correction** — target sentence with one error; learner corrects
5. **plural_formation** (nouns only) — singular → write plural
6. **haben_vs_sein** (verbs only) — Perfekt sentence with blank for auxiliary

Complexity: ~30% simple, ~40% medium, ~30% complex. Every vocabulary word appears in at least one exercise.

**JSON Schema:**
```json
{
  "lektion": 5,
  "focus": "nouns",
  "file_slug": "lektion05_nouns",
  "generated_at": "<ISO 8601>",
  "vocabulary": [
    {
      "german": "der Bahnhof",
      "base_form": "Bahnhof",
      "article": "der",
      "gender": "masc",
      "plural_marker": "..e",
      "plural_full": "Bahnhöfe",
      "english": "train station",
      "word_type": "noun",
      "pattern_hint": "...",
      "pattern_reliability": "...",
      "lektion_source": 5
    }
  ],
  "examples": [
    {
      "id": 1,
      "german": "...",
      "english": "...",
      "level": "medium|complex",
      "note": "..."
    }
  ],
  "exercises": [
    {
      "id": 1,
      "type": "fill_in_blank|multiple_choice|translation|error_correction|plural_formation|haben_vs_sein",
      "level": "simple|medium|complex",
      "prompt": "...",
      "options": ["a) ...", "b) ...", "c) ..."],  // only for multiple_choice
      "solution": "...",
      "explanation": "..."
    }
  ]
}
```

---

## Design Patterns & Key Files

### Index Regeneration Pattern

Skills that save files should regenerate `index.html` and topic-specific indexes after saving. The pattern:

1. List all HTML files in output directory(ies)
2. Extract from filenames:
   - **stem**: everything before the datetime suffix
   - **label**: stem with datetime removed, underscores → spaces, capitalized intelligently
   - **date**: YYYY-MM-DD portion from filename
   - **href**: relative path from project root
3. Sort by date descending (newest first)
4. Write index following the master template structure

### Shared Stylesheet (styles.css)

A single `styles.css` provides:
- **Consistent visual identity** across all pages
- **Dark mode support** via system preference detection (media query `prefers-color-scheme`)
- **Reusable component classes**:
  - `.badge`, `.badge-simple`, `.badge-medium`, `.badge-complex`, `.badge-type`
  - `.file-list` — linked list of resources
  - `.sentence-card` — individual sentence display
  - `.example-item` — example grouping
  - `.exercise-item` — exercise display
  - `.solution-item` — solution display
  - `.gender-grid` — 3-column noun reference (DER | DIE | DAS)
  - `.verb-card` — verb reference with conjugation table
  - `.vocab-card` — vocabulary reference card
  - `.block-nav` — inline navigation (used in grammar/vocab pages)

Link from all HTML pages: `<link rel="stylesheet" href="../../styles.css" />`

No inline CSS; all styling in shared file enables global theme updates.

---

## Customization Guide for New Language Pairs

To adapt this system to a new language (e.g., French A1, Spanish B1), follow these steps:

### 1. Update Learner Profile (CLAUDE.md)

```yaml
Native Language:             [e.g., English, German]
Other Strong Languages:      [...]
Instruction Language:        [e.g., English]
Current Level:               [A1, A2, B1, etc.]
Goal:                        [tourism, business, academic, etc.]
Textbook/Curriculum:         [specific series, chapters, etc.]
Correction Style:            [strict | balanced | conversational]
```

### 2. Customize Sentence Builder

Edit `.claude/skills/sentence-builder/SKILL.md`:

**Language-specific adjustments:**
- Replace examples with target language
- Adjust grammar coverage based on target language:
  - If target has fewer tenses, remove Perfekt from complex
  - If target has gender/case system, adjust example coverage
  - If target has complex word order rules (e.g., German verb-second), emphasize in examples
- Replace transfer notes with parallels/false friends vs. learner's native + strong languages

**Example customization (French):**
- Verb conjugations: present indicative, Passé Composé (instead of Perfekt)
- Nouns: French has gender but no case (unlike German); emphasize article agreement
- Transfer notes: English→French (many cognates), Spanish→French (some False friends in pronunciation)

### 3. Customize Grammar Explainer

Edit `.claude/skills/grammar-explainer/SKILL.md`:

**Language-specific adjustments:**
- Update explanation sections to target language's grammar categories
- Adjust example complexity based on target level and language difficulty
- Customize transfer notes
- Adjust exercise types if needed (some languages may need special exercise types)

**Example customization (Mandarin A1):**
- "How to form it" sections explain tones, pinyin, character strokes instead of inflections
- Examples use simplified characters appropriate to A1
- Transfer notes compare tonal systems, character vs. alphabetic writing

### 4. Customize Vocabulary Drill

Edit `.claude/skills/vocab-drill/SKILL.md`:

**Language-specific adjustments:**
- Update word-type classification rules (PDF notation patterns)
- Adjust enrichment fields for word types:
  - If target language has gender: keep `article`, `gender` fields
  - If target has aspect system: add aspect fields instead of tense
  - If target has tone: add `tone` field for each word
- Update reference section layout if needed
- Customize exercise types (e.g., tone discrimination for tonal languages)
- Adjust plural formation for target language's pluralization rules (if any)

**Example customization (Japanese A1):**
- No articles, so remove `article` / `gender`
- Add `kanji`, `hiragana`, `romaji` fields
- Add `politeness_level` field (formal vs. casual)
- "Plural formation" replaced with "Verb conjugation" exercises
- Reference layout: organized by word type, not by gender

### 5. Prepare Source Materials

Organize textbook materials in `resources/`:
- **PDFs**: Glossaries, word lists, grammar reference sheets
  - Organize by chapter/lektion
  - Establish consistent naming: `glossar_{level}_{part}.pdf` or `textbook_{chapter}.pdf`
- **Other formats**: Spreadsheets, plain text, CSV
  - Adapt skill to read your format (Read tool, CSV parsing, etc.)

### 6. Update README.md

Update user-facing documentation:
```markdown
# [Language] Learning Assistant

[Brief description]

## Learner Profile
- **Native language:** [...]
- **Level:** [A1, A2, etc.]
- **Goal:** [...]
- **Textbook:** [...]

## What's inside
- [brief descriptions of outputs]

## Project Structure
[Update directory names if different]

## Skills
[Update skill invocations and descriptions]
```

---

## Important Design Considerations

### 1. Proficiency Level Consistency

All content **must** be at the learner's declared level. Avoid "scope creep" where advanced grammar or vocabulary leaks into A1 materials.

**How to maintain:**
- Define a **core vocabulary list** for the level (e.g., ~1500 words for A1)
- Reference this list when generating examples and exercises
- Use the CLAUDE.md learner profile as the source of truth
- Flag any content that exceeds the level scope

### 2. Language Transfer Notes

Transfer notes are **essential** for efficient learning, especially when:
- Native language and target language share origins (e.g., English → German, Spanish → Portuguese)
- There are dangerous false friends (e.g., English "gift" vs. German "Gift" = poison)
- Grammatical structures differ significantly (e.g., Spanish gender ≠ German gender)

**For every skill:**
- Include transfer notes after explanations and examples
- Structure as:
  - **Native → Target:** [helper or danger]
  - **Other language → Target:** [helper or danger]
- Use **bold** for the specific words or patterns being compared

### 3. Realistic Context

Sentences and examples should reflect the learner's stated goal. This dramatically improves motivation and retention.

**Examples:**
- Tourism learner: travel, hotel, food, directions
- Business learner: meetings, email, negotiation
- Academic learner: research, presentation, argumentation

Avoid abstract sentences ("The cat is blue.") unless they are grammatically instructive.

### 4. Strict vs. Conversational Correction

This system is opinionated about error correction. The current instance uses **Strict** (flag every error, explain why).

**When setting up a new instance:**
- Agree with the learner on the correction style upfront
- Document it in CLAUDE.md
- Apply it consistently across all skills (in grammar notes, explanations, transfer notes)

**Correction styles:**
- **Strict:** Every error flagged with explanation. Good for learners who want perfection.
- **Balanced:** Major errors flagged; minor ones only mentioned if relevant to current lesson.
- **Conversational:** Errors in context-building shown; focus on comprehension over form.

---

## Current Level Tracker

Update this table as the learner progresses. Update CLAUDE.md accordingly when changing levels (e.g., vocabulary set, grammar scope).

| Date | Level | Notes |
|------|-------|-------|
| 2026-03-04 | A1 | Starting point |

---

## Workflow Example (German)

To understand how all pieces fit together:

1. **Learner requests:** "sentence builder: fahren (verb)"
   - Skill generates 10 sentences covering different persons, tenses, contexts
   - Saves JSON with strict grammar notes and Spanish/English transfer notes
   - Regenerates index.html
   - Displays sentences in chat

2. **Learner requests:** "render sentences"
   - Skill scans for unrendered JSON files
   - Converts each to styled HTML
   - Regenerates index.html

3. **Learner requests:** "grammar: Nominativ Akkusativ Dativ"
   - Skill generates 3-section explanation covering when/how to use each case
   - Adds 12+ exercises (fill-in-blank, multiple choice, etc.) + solutions
   - Saves JSON + HTML
   - Regenerates index.html

4. **Learner requests:** "vocab drill: lektion 5, nouns"
   - Skill extracts all nouns from Lektion 5 from glossary PDF
   - Enriches each with gender, plural, pattern hints
   - Generates 40+ exercises
   - Saves JSON + HTML
   - Updates both vocabulary and root index.html

---

## Quality Assurance Checklists

Each skill includes an internal checklist (do not display to user). Before saving:

**Sentence Builder:**
- [ ] Exactly 10 sentences (3 simple / 3 medium / 4 complex)
- [ ] ≥4 different persons covered (for verbs)
- [ ] ≥1 Perfekt sentence in complex group
- [ ] All A1-appropriate vocabulary
- [ ] Grammar notes are clear and concise
- [ ] Transfer notes present (Spanish↔German, English↔German)

**Grammar Explainer:**
- [ ] ≥10 examples (3 simple / 3 medium / 4+ complex)
- [ ] ≥12 exercises across all 4 types
- [ ] Exercises distributed by complexity (~1/3 per level)
- [ ] Explanation sections cover all relevant topics
- [ ] Transfer notes present
- [ ] Key Takeaway displayed and memorable

**Vocabulary Drill:**
- [ ] All vocabulary for the chapter extracted
- [ ] Every word enriched with required fields
- [ ] ≥40 exercises generated
- [ ] Every vocabulary word appears ≥1 in exercises
- [ ] Examples use only medium/complex level
- [ ] Solutions section complete

---

## Future Extensions

Possible additions (not implemented yet):

- **Verb conjugation tables** — dedicated drill for all tenses/persons
- **Listening exercises** — audio pronunciation + comprehension
- **Speaking exercises** — record and compare to native speaker
- **Flashcard export** — Anki-compatible deck generation
- **Progress tracking** — quiz history, mastery levels, spaced repetition
- **Dialogue builder** — generate conversational exchanges on a topic
- **Level assessment** — auto-detect learner's actual level via quiz
- **Keyboard shortcuts** — for faster skill invocation

---

## Notes for Implementation

When porting this system to a new language:

1. **Define vocabulary scope** first (e.g., "1500-word core A1 set" or "Textbook chapters 1–5")
2. **Test one skill** (usually sentence-builder) with a few examples to validate transfer notes and level
3. **Build glossary lookup** if vocab-drill reads from PDFs (requires format understanding)
4. **Validate all transfer notes** with a native speaker if possible
5. **Iterate on example quality** — the first few skill runs are rough; refine as you go
6. **Gather learner feedback** on correction style, context appropriateness, exercise difficulty

---

## Files to Maintain

- **CLAUDE.md** — this file; update learner profile, level tracker, and any customizations
- **styles.css** — shared stylesheet; update colors/theme if needed, but try to avoid breaking layout changes
- **README.md** — user documentation; keep in sync with project state
- **.claude/skills/*/SKILL.md** — skill definitions; these are reference docs for Claude; update if you change behavior
- **.claude/settings.json** — CLI settings; usually no changes needed

---

## References

- **Momente A1 textbook** — [Hueber Verlag](https://www.hueber.de/)
- **German learning resources** — DW Learn German, Forvo, Duolingo (for comparison)
- **Language learning pedagogy** — "How Languages Are Learned" by H.D. Brown et al.

---

**Last updated:** 2026-03-16
**Current learner:** Spanish-native, A1 German, travel/tourism focus
**Status:** Active, skills fully implemented