---
name: grammar-explainer
description: >
  Use this skill when the user writes "grammar: [topic]" or "grammar: [topic], focus: [specifics]".
  Generates a detailed A1-level grammar explanation with examples grouped by difficulty, followed
  by a set of exercises (fill-in-the-blank, multiple choice, translation, error correction) and
  their solutions. Saves a JSON metadata file to data/grammar/ and a styled HTML file to data/grammar-html/.
---

## Allowed Tools
* Write: to save JSON and HTML files
* Bash: to create directories and get the current datetime

---

## Step 1 — Parse Input

Input format:
- `grammar: [topic]`
- `grammar: [topic], focus: [specifics]`

Extract:
- **topic** (required): the grammar topic (e.g. "Dativ", "lokale Präpositionen", "Perfekt")
- **focus** (optional): specific aspect to emphasize (e.g. "prepositions with movement", "haben vs. sein")
- **topic_slug**: lowercase, spaces replaced with underscores, umlauts replaced (ä→ae, ö→oe, ü→ue, ß→ss)
  - Example: "Dativ" → "dativ", "lokale Präpositionen" → "lokale_praepositionen"

---

## Step 2 — Build the Grammar Explanation

Produce a full English explanation of the topic, appropriate for an A1 learner. The explanation must contain the following **sections** (include all that are relevant; skip sections that genuinely do not apply):

| Section | Content |
|---------|---------|
| **What is it?** | 1–2 sentence plain-language definition |
| **When to use it** | The rules and conditions that trigger this grammar form |
| **How to form it** | Tables or step-by-step construction rules |
| **Common exceptions** | The most important irregular forms or edge cases at A1 level |
| **Spanish transfer note** | Helpful parallel or false friend vs. Spanish |
| **English transfer note** | Helpful parallel or false friend vs. English |

If **focus** was given, devote extra depth to that specific aspect.

End the explanation with:
- **Transfer Notes** block: 2–4 bullets covering Spanish→German and English→German parallels or traps
- **Key Takeaway**: one bold sentence summarizing the single most critical point

---

## Step 3 — Generate Examples

Produce exactly **10 examples** (you may include more if the topic genuinely requires it), split:

| Group   | Count | Description |
|---------|-------|-------------|
| Simple  | 3     | Short (≤ 6 words), Präsens, one grammar feature at a time |
| Medium  | 3     | 7–10 words, Präsens; may add an adverb, time phrase, or prepositional phrase |
| Complex | 4+    | 8–14 words; at least 1 Perfekt sentence; may use a modal verb or coordinating conjunction |

Rules:
- All vocabulary must be A1-appropriate
- Sentences should be realistic and travel/tourism-relevant where natural
- Each example must directly illustrate the grammar topic being explained
- Each example gets a one-line `grammar_note` tying it back to the explanation

---

## Step 4 — Generate Exercises

Generate at least **12 exercises**, distributed across all four types. Apply the same complexity progression (roughly 1/3 simple, 1/3 medium, 1/3 complex). Generate more exercises if the topic has many sub-cases worth practicing.

### Exercise Types

#### fill_in_blank
- Present a sentence with one or more blanks (`___`)
- The answer fills in the correct German word/form
- Include an English cue in parentheses if helpful: `Ich gehe ___ (to the) Bahnhof.`

#### multiple_choice
- Present a sentence with a blank and exactly 3 options (a / b / c)
- Only one answer is correct
- Distractors should be plausible errors a learner might make

#### translation
- Give an English sentence
- The learner must produce the full German sentence
- Use only A1 vocabulary

#### error_correction
- Present a German sentence that contains exactly one grammatical error relevant to the topic
- The learner must identify and correct the error
- Keep all other parts of the sentence correct

### Complexity Guidelines for Exercises

| Level   | Fill-in-blank | Multiple choice | Translation | Error correction |
|---------|--------------|-----------------|-------------|-----------------|
| Simple  | 1–2 blanks, basic form | straightforward distractors | short sentence | obvious error |
| Medium  | 1–2 blanks, add case/article | similar-form distractors | includes prep phrase or adverb | subtle error |
| Complex | 2 blanks or full phrase | tricky form distractors | full clause or Perfekt | compound error context |

Each exercise must have:
- `id`: sequential starting at 1
- `type`: fill_in_blank | multiple_choice | translation | error_correction
- `level`: simple | medium | complex
- `prompt`: the exercise text shown to the learner
- `options`: array of 3 strings (only for multiple_choice; omit for other types)
- `solution`: the correct answer (for error_correction, give the corrected sentence)
- `explanation`: 1–2 sentences explaining why the answer is correct

---

## Step 5 — Build the JSON Object

Schema:

```json
{
  "topic": "<topic as user typed it>",
  "focus": "<focus as user typed it, or null>",
  "topic_slug": "<slug>",
  "generated_at": "<ISO 8601 datetime>",
  "explanation": {
    "sections": [
      {
        "title": "<section title>",
        "content": "<full text — may include markdown tables>"
      }
    ],
    "transfer_notes": ["<bullet 1>", "<bullet 2>"],
    "key_takeaway": "<one sentence>"
  },
  "examples": [
    {
      "id": 1,
      "german": "<German sentence>",
      "english": "<English translation>",
      "level": "<simple|medium|complex>",
      "grammar_note": "<1-line note>"
    }
  ],
  "exercises": [
    {
      "id": 1,
      "type": "<fill_in_blank|multiple_choice|translation|error_correction>",
      "level": "<simple|medium|complex>",
      "prompt": "<exercise text>",
      "options": ["a) ...", "b) ...", "c) ..."],
      "solution": "<correct answer>",
      "explanation": "<why this is correct>"
    }
  ]
}
```

- `generated_at`: use today's date and current time in ISO 8601
- `options`: include only for multiple_choice; omit the key entirely for other types

---

## Step 6 — Build the HTML File

Generate a self-contained, styled HTML file (inline CSS only — no external dependencies).

### Structure

```
<html>
  <head> — title, inline CSS </head>
  <body>
    <header>         — topic title, focus (if any), generated date
    <nav>            — anchor links: Explanation | Examples | Exercises | Solutions
    <section id="explanation">  — all explanation sections rendered as <h3> + <p>/<table>
    <section id="examples">     — examples grouped by level with color-coded badges
    <section id="exercises">    — all exercises, numbered, type badge, level badge; solutions hidden
    <section id="solutions">    — solution key: id, correct answer, explanation
    <footer>         — "Generated by German Learning Assistant · A1"
  </body>
</html>
```

### CSS Guidelines
- Clean sans-serif font (system-ui or Arial)
- Level color coding: simple = #2e7d32 (green), medium = #f57c00 (orange), complex = #c62828 (red)
- Type color coding: fill_in_blank = #1565c0 (blue), multiple_choice = #6a1b9a (purple), translation = #00695c (teal), error_correction = #4e342e (brown)
- Badges: small colored pill labels for level and type on each exercise
- Exercise prompts in a light gray box (`background: #f5f5f5`)
- Solutions section uses a distinct background (`background: #fffde7`) to visually separate it
- Responsive max-width: 800px centered

---

## Step 7 — Save Files

### Determine filenames
- Pattern: `{topic_slug}_{YYYY-MM-DD_HHMMSS}`
- JSON: `data/grammar/{topic_slug}_{YYYY-MM-DD_HHMMSS}.json`
- HTML: `data/grammar-html/{topic_slug}_{YYYY-MM-DD_HHMMSS}.html`
- Use the same datetime as `generated_at`

### Create directories and save
Use Bash to ensure directories exist:
```
mkdir -p data/grammar data/grammar-html
```

Then use the Write tool to save both files.

After saving, confirm to the user:
> "Saved to `data/grammar/{json_filename}` and `data/grammar-html/{html_filename}`"

---

## Step 8 — Display in Chat

Show the full content in chat using this structure:

1. Header: `## Grammar: [topic]` (and `**Focus:** [focus]` if given)
2. All explanation sections (rendered markdown)
3. `### Examples`  grouped by level — same format as sentence-builder:
   ```
   **[ID]. [German sentence]**
   → [English translation]
   📝 [grammar note]
   ```
4. `### Exercises` — show all exercises with type/level labels; do NOT show solutions inline
5. `### Solutions` — show all answers with explanations, clearly separated
6. Transfer Notes block
7. Key Takeaway block

---

## Quality Checklist (internal — do not display)

Before saving and displaying, verify:
- [ ] At least 10 examples (3 simple / 3 medium / 4+ complex)
- [ ] At least 12 exercises spread across all 4 types
- [ ] Exercises follow complexity distribution (~1/3 per level)
- [ ] JSON matches schema (options key omitted for non-multiple_choice)
- [ ] HTML is self-contained with inline CSS only
- [ ] Both files saved before displaying output
- [ ] topic_slug uses only lowercase ASCII (no umlauts in filename)
- [ ] Key Takeaway displayed in chat
