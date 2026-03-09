# German Learning Assistant

A personal A1-level German learning tool built with Claude Code. Generates sentence practice files and grammar lessons, rendered as styled HTML pages.

**Live site:** https://chrissaher.github.io/german-learning-assistant/

## What's inside

- **Sentence practice** — 10 example sentences per word, grouped by difficulty (simple / medium / complex), with English translations and grammar notes
- **Grammar lessons** — full explanations, 10+ examples, 12+ exercises across four types (fill-in-blank, multiple choice, translation, error correction), and a solution key
- **Dark mode** — automatically follows your OS preference via a single shared `styles.css`

## Learner profile

| Field | Value |
|---|---|
| Native language | Spanish (also fluent in English) |
| Instruction language | English |
| Level | A1 (started 2026-03-04) |
| Goal | Travel and tourism in German-speaking countries |
| Textbook | Momente A1 (Hueber Verlag) |

## Project structure

```
index.html                  ← master index of all lessons
styles.css                  ← shared stylesheet (light + dark mode)
data/
  sentences/                ← JSON source files (sentence-builder output)
  sentences-html/           ← rendered sentence HTML pages
  grammar/                  ← JSON source files (grammar-explainer output)
  grammar-html/             ← rendered grammar HTML pages
.claude/skills/             ← Claude Code skill definitions
```

## Skills (Claude Code)

| Invocation | What it does |
|---|---|
| `sentence builder: [word]` | Generates 10 A1 sentences and saves JSON |
| `render sentences` | Renders unrendered JSON files to HTML |
| `grammar: [topic]` | Generates a full grammar lesson (explanation + exercises) |
