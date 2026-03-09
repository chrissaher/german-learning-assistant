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

Generate a styled HTML file that links to the shared stylesheet.

### Structure

```
<html>
  <head> — title, link to shared stylesheet </head>
  <body class="grammar-page">
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

### Stylesheet
Use `<link rel="stylesheet" href="../../styles.css" />` in `<head>` instead of inline CSS. Add `class="grammar-page"` to `<body>`. Use `<nav class="block-nav">` for the navigation. No `<style>` block needed.

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

## Step 8 — Regenerate index.html

After saving the grammar files, regenerate `index.html` at the project root so the new file appears in the browser index.

### How to rebuild index.html

1. Use Bash to list all HTML files in both output directories:
   ```bash
   ls data/grammar-html/*.html 2>/dev/null
   ls data/sentences-html/*.html 2>/dev/null
   ```

2. For each filename, extract:
   - **label**: strip the datetime suffix (`_YYYY-MM-DD_HHMMSS`) from the stem, replace remaining underscores with spaces; capitalize the first letter of the result. Use your own knowledge of the word to improve capitalization where appropriate (e.g., `die_kunst` → "die Kunst", `perfekt_past_tense` → "Perfekt Past Tense").
   - **date**: the `YYYY-MM-DD` portion embedded in the filename (second-to-last underscore segment, format `YYYY-MM-DD`).
   - **href**: the relative path from the project root, e.g. `data/grammar-html/{filename}`.

3. Sort each section by date descending (newest first).

4. Write `index.html` to the project root using the Write tool, following this structure exactly:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>German Learning Assistant — A1</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <div class="container">
    <header>
      <h1>German Learning Assistant</h1>
      <p class="meta">A1 Level · Momente A1 (Hueber) · Travel &amp; Tourism</p>
    </header>

    <section id="grammar">
      <h2><span class="section-badge badge-grammar">Grammar</span> Lessons</h2>
      <ul class="file-list">
        <!-- one <li> per grammar HTML file, newest first -->
        <li><a href="{href}"><span class="label">{label}</span><span class="date">{date}</span></a></li>
      </ul>
    </section>

    <section id="sentences">
      <h2><span class="section-badge badge-sentences">Sentences</span> Practice</h2>
      <ul class="file-list">
        <!-- one <li> per sentence HTML file, newest first -->
        <li><a href="{href}"><span class="label">{label}</span><span class="date">{date}</span></a></li>
      </ul>
    </section>

    <footer>German Learning Assistant · A1 · Generated {YYYY-MM-DD}</footer>
  </div>
</body>
</html>
```

Replace `{YYYY-MM-DD}` in the footer with today's date.
After saving, confirm: `"index.html regenerated — {N} grammar + {M} sentence files listed."`

---

## Step 9 — Display in Chat

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
- [ ] HTML links to `../../styles.css` — no inline `<style>` block; `<body class="grammar-page">` and `<nav class="block-nav">` present
- [ ] Both files saved before displaying output
- [ ] topic_slug uses only lowercase ASCII (no umlauts in filename)
- [ ] Key Takeaway displayed in chat
