# German Learning Assistant — Project Context

## Learner Profile

- **Native language:** Spanish (also fluent in English)
- **Instruction language:** English (all lessons and tools use English)
- **Current level:** A1 (started 2026-03-04, progressing daily)
- **Goal:** Travel and tourism in German-speaking countries
- **Textbook:** Momente A1 (Hueber Verlag)

## Correction Style

**Strict** — Flag every error, including minor spelling, gender, case, and word order mistakes. Always show the correct form and briefly explain why.

## Planned Skills / Tools

Skills will be created incrementally. Ideas in scope:

- **Sentence builder** — generate and practice forming German sentences
- **Verb conjugation** — drill verb conjugation across tenses and persons
- **Vocabulary drills** — flashcard-style vocabulary practice
- **Grammar explainer** — clear explanations with examples at A1 level

## Guidelines for All Skills

1. **Level-appropriate content** — keep vocabulary and grammar within A1 scope unless the user explicitly asks to go beyond. As the user progresses, update the level note below.
2. **English interface** — explanations and instructions in English; German only for the practice content itself.
3. **Spanish transfer notes** — when relevant, point out similarities or false friends between Spanish and German (e.g., gender systems differ, false cognates).
4. **English transfer notes** — when relevant, point out similarities or false friends between English and German (e.g., gender systems differ, false cognates).
5. **Momente A1 alignment** — when possible, align vocabulary and grammar topics with the Momente A1 chapter sequence.
6. **Strict corrections** — see above.

## Available Skills

| Skill               | Invocation                                                          | Description                                                                                                                            |
|---------------------|---------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| sentence-builder    | `sentence builder: [word]` or `sentence builder: [word] ([type])`  | Generates 10 A1-level example sentences for a given German word and saves them as JSON                                                |
| sentence-renderer   | `render sentences`                                                  | Renders all unrendered sentence JSON files to styled HTML; skips already-rendered files; saves to `data/sentences-html/`              |
| grammar-explainer   | `grammar: [topic]` or `grammar: [topic], focus: [specifics]`       | Generates a detailed grammar explanation with examples, exercises (4 types), and solutions; saves JSON + HTML output                  |

**Invocation examples:**
- `sentence builder: Haus`
- `sentence builder: essen (verb)`
- `sentence builder: groß (adjective)`
- `render sentences`
- `grammar: Dativ`
- `grammar: Perfekt, focus: haben vs. sein`
- `grammar: lokale Präpositionen`

Skill definitions: `.claude/skills/sentence-builder/SKILL.md`, `.claude/skills/sentence-renderer/SKILL.md`, `.claude/skills/grammar-explainer/SKILL.md`
Generated sentence files saved to: `data/sentences/` (JSON) and `data/sentences-html/` (HTML)
Generated grammar files saved to: `data/grammar/` (JSON) and `data/grammar-html/` (HTML)

---

## Current Level Tracker

| Date       | Level | Notes                        |
|------------|-------|------------------------------|
| 2026-03-04 | A1    | Starting point                |

Update this table as the learner progresses.