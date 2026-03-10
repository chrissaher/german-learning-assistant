---
name: vocab-drill
description: >
  Use this skill when the user writes "vocab drill: lektion [N], [focus]".
  Extracts vocabulary from the Momente A1 glossary PDFs for the specified Lektion,
  filtered by focus category, then produces a vocabulary reference section, usage
  examples, and a large exercise set (40+ drills). Saves JSON to data/vocabulary/
  and styled HTML to data/vocabulary-html/. Regenerates data/vocabulary-html/index.html.
---

## Allowed Tools
* Read: to read the glossary PDFs and the SKILL.md
* Write: to save JSON and HTML files
* Bash: to create directories and get the current datetime, and to list existing files for index regeneration

---

## Step 1 — Parse Input

Input format: `vocab drill: lektion [N], [focus]`

Extract:
- **lektion** (required): integer 1–24
- **focus** (required): one of `nouns`, `verbs`, `adjectives`, `others`, `all`
- **lektion_padded**: zero-padded to 2 digits (e.g. 5 → "05")
- **file_slug**: `lektion{lektion_padded}_{focus}` (e.g. `lektion05_nouns`)

---

## Step 2 — Read the Glossary PDF

- Lektionen **1–12** → `resources/glossar_a1_1.pdf`
- Lektionen **13–24** → `resources/glossar_a1_2.pdf`

Read the full PDF. Locate the section labeled "Lektion {N}" (or "Lektion {NN}" with leading zero). Extract **every vocabulary entry** listed under that Lektion, across all numbered sub-sections within it. Include entries from any "Magazin" or "Fokus Beruf" mini-sections if they appear within the Lektion boundary.

### How to identify word types from PDF notation

| Notation pattern | Word type | Example |
|-----------------|-----------|---------|
| Starts with `der`, `die`, or `das` | **noun** | `der Name, -n` |
| Infinitive form, no article, ends in `-en` / `-eln` / `-ern` | **verb** | `heißen`, `arbeiten` |
| No article, not a verb infinitive, clearly descriptive | **adjective** | `groß`, `schön`, `kalt` |
| Single-word function words (nicht a noun/verb/adjective): adverbs, prepositions, pronouns, conjunctions, interjections | **other** | `sehr`, `heute`, `und`, `in` |
| Multi-word phrase or fixed expression | **other** (subtype: phrase) | `Wie geht's?`, `Guten Morgen` |

For **others**, further classify with a subtype:
- **adverb**: describes how/when/where (sehr, heute, hier, immer, nie)
- **preposition**: in, auf, an, mit, bei, nach, von, zu, aus, über, unter, vor, hinter, zwischen, neben, gegenüber
- **pronoun**: ich, du, er, sie, es, wir, ihr, sie, Sie, mein, dein, …
- **conjunction**: und, oder, aber, denn, weil, dass, wenn, …
- **interjection**: Hallo, Tschüss, Bitte, Danke, Entschuldigung, …
- **phrase**: multi-word fixed expressions

### Plural marker notation from the PDF

| PDF notation | Meaning | How to form plural_full |
|-------------|---------|------------------------|
| `-n` | add -n | Straße → Straßen |
| `-en` | add -en | Name → Namen |
| `-e` | add -e | Zug → Züge (if umlaut noted separately) |
| `..e` | add umlaut + e | Zug → Züge |
| `-` (dash only) | no change | Zimmer → Zimmer |
| `..` | umlaut only (no ending) | Mutter → Mütter |
| `..er` | add umlaut + er | Kind → Kinder (if umlaut) |
| `-er` | add -er | Kind → Kinder |
| `-s` | add -s | Hotel → Hotels |
| `(Sg.)` | singular-only noun | — |
| `(Pl.)` | plural-only noun | use as-is |

---

## Step 3 — Filter and Enrich Vocabulary

Apply filter based on focus:
- `nouns` → keep only nouns
- `verbs` → keep only verbs
- `adjectives` → keep only adjectives
- `others` → keep only other-typed entries (all subtypes)
- `all` → keep all word types

For each word, **enrich** using your German grammatical knowledge:

### Nouns — enrichment
- `article`: der / die / das
- `gender`: masc / fem / neut
- `plural_marker`: from PDF notation (e.g. "-n", "..e", "-")
- `plural_full`: full plural form (e.g. "Namen", "Züge", "Zimmer")
- `pattern_hint`: one of these strings based on the ending/type:
  - `-ung ending → always die (feminine)` — for -ung words
  - `-heit / -keit ending → always die (feminine)` — for -heit/-keit words
  - `-tion / -sion ending → always die (feminine)` — for -tion words
  - `-chen / -lein ending → always das (neuter)` — for diminutives
  - `-um ending (Latin) → usually das (neuter)` — for -um words
  - `-ment ending → usually das (neuter)` — for -ment words
  - `-er ending (agent noun) → usually der (masculine)` — for -er person nouns
  - `Infinitive as noun → always das (neuter)` — for das Essen, das Frühstück
  - `Borrowed word → often das (neuter)` — for modern loanwords
  - `Days / months / seasons → always der (masculine)` — for calendar words
  - `Must memorize — no reliable pattern` — for all others
- `pattern_reliability`: `high` / `moderate` / `memorize`

### Verbs — enrichment
- `conjugation_praesens`: object with keys ich, du, `er/sie/es`, wir, ihr, `sie/Sie`
  - Include any stem vowel changes (e.g. fahren → du fährst, er fährt)
  - Include any separable prefix behavior in a note
- `perfekt`: full Perfekt form as a string (e.g. "hat gemacht", "ist gegangen")
- `auxiliary`: "haben" or "sein"
- `is_irregular`: true if stem changes in du/er forms or has irregular Partizip II
- `verb_note`: optional brief note for separable verbs, modals, or strong verbs

### Adjectives — enrichment
- `antonym`: opposite adjective if A1-appropriate (e.g. groß ↔ klein), or null
- `cognate_note`: optional note if word is a Spanish/English cognate or false friend

### Others — enrichment
- `subtype`: adverb / preposition / pronoun / conjunction / interjection / phrase
- `usage_note`: one sentence describing how/when to use it in A1 context

---

## Step 4 — Generate Examples (medium + complex only)

Generate **8–12 usage examples**. For `all` focus, generate 4–5 per category.

Rules:
- **Medium level only** (7–10 words, Präsens or common phrases) and **complex level** (8–14 words, may include Perfekt or modal verb)
- Each sentence must use at least one word from the current Lektion's focus vocabulary
- You **may and should** use vocabulary from **Lektionen 1 through N−1** for supporting words (verbs, adjectives, nouns) in the same sentence
- Sentences should be realistic: travel, daily life, tourism, meeting people, ordering food, finding places — themes aligned with A1/Momente A1
- Avoid simple sentences (subject-verb-object with nothing else)
- Each example gets a `note` explaining which focus words appear and any grammar point worth noting

Format per example:
```json
{
  "id": 1,
  "german": "...",
  "english": "...",
  "level": "medium|complex",
  "note": "..."
}
```

---

## Step 5 — Generate Exercises (minimum 40)

Every focus word must appear in at least one exercise. Target 40–50 exercises total.

### Exercise type distribution

Distribute across these types (~10 each for a single-focus run; adjust proportionally for `all`):

#### 1. fill_in_blank
- A sentence with one or two `___` blanks
- Learner fills in the correct German form
- For nouns: article + noun, or just article, or just noun (vary the challenge)
- For verbs: correct conjugated form
- For adjectives: correct adjective from the word list
- For others: correct word, preposition, or expression
- Include an English cue in parentheses where helpful

#### 2. multiple_choice
- A sentence with one blank and exactly 3 options (a / b / c)
- Only one is correct; distractors should be plausible errors
- For nouns: wrong article or wrong noun; for verbs: wrong conjugation or wrong auxiliary

#### 3. translation
- English sentence → learner writes full German sentence
- Sentence must require using at least one focus word correctly
- Use only A1-level vocabulary in the English prompt

#### 4. error_correction
- German sentence with exactly one error related to the focus
- Learner identifies and corrects the error
- Error types: wrong article, wrong conjugation, wrong auxiliary (haben/sein), wrong adjective form, wrong word order for separable verb

#### 5. plural_formation (nouns only — add ~5 of these)
- Give singular form: `der Zug` → learner writes plural
- Vary: give article + singular, ask for full plural (with article: die Züge)

#### 6. haben_vs_sein (verbs only — add ~5 of these)
- Give a Perfekt sentence with blank for auxiliary
- Learner fills in `hat` or `ist`
- Example: `Er ___ nach Berlin gefahren.`

### Complexity distribution
- ~30% simple (direct form drills, short sentences)
- ~40% medium (adds context, article + noun together, full short sentence)
- ~30% complex (multi-clause, Perfekt, or combining multiple focus words)

### Each exercise must include:
- `id`: sequential starting at 1
- `type`: fill_in_blank | multiple_choice | translation | error_correction | plural_formation | haben_vs_sein
- `level`: simple | medium | complex
- `prompt`: exercise text shown to learner
- `options`: array of 3 strings — only for `multiple_choice` (omit key for all other types)
- `solution`: correct answer
- `explanation`: 1–2 sentences explaining why

---

## Step 6 — Build the JSON Object

```json
{
  "lektion": 5,
  "focus": "nouns",
  "file_slug": "lektion05_nouns",
  "generated_at": "2026-03-10T14:30:22",
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
      "pattern_hint": "Must memorize — no reliable pattern",
      "pattern_reliability": "memorize",
      "lektion_source": 5
    },
    {
      "german": "fahren",
      "base_form": "fahren",
      "english": "to drive / to travel",
      "word_type": "verb",
      "conjugation_praesens": {
        "ich": "fahre",
        "du": "fährst",
        "er/sie/es": "fährt",
        "wir": "fahren",
        "ihr": "fahrt",
        "sie/Sie": "fahren"
      },
      "perfekt": "ist gefahren",
      "auxiliary": "sein",
      "is_irregular": true,
      "verb_note": "Stem vowel change a→ä in du/er forms",
      "lektion_source": 5
    },
    {
      "german": "schön",
      "base_form": "schön",
      "english": "beautiful / nice",
      "word_type": "adjective",
      "antonym": "hässlich",
      "cognate_note": null,
      "lektion_source": 5
    },
    {
      "german": "sehr",
      "base_form": "sehr",
      "english": "very",
      "word_type": "other",
      "subtype": "adverb",
      "usage_note": "Intensifier placed before adjectives or adverbs: sehr gut, sehr schön.",
      "lektion_source": 5
    }
  ],
  "examples": [
    {
      "id": 1,
      "german": "...",
      "english": "...",
      "level": "medium",
      "note": "..."
    }
  ],
  "exercises": [
    {
      "id": 1,
      "type": "fill_in_blank",
      "level": "simple",
      "prompt": "___ Bahnhof ist groß.",
      "solution": "Der",
      "explanation": "der Bahnhof — masculine. Must memorize; no reliable ending pattern."
    }
  ]
}
```

Notes:
- `generated_at`: use actual current datetime from Bash (`date +"%Y-%m-%dT%H:%M:%S"`)
- `options` key: include only for `multiple_choice`; omit entirely for all other types
- For `all` focus, `vocabulary` array contains all word types mixed together; the `word_type` field distinguishes them

---

## Step 7 — Build the HTML File

Use `<link rel="stylesheet" href="../../styles.css" />` in `<head>`. No inline `<style>` block.
Add `class="grammar-page"` to `<body>`. Use `<nav class="block-nav">`.

### Header
```html
<header>
  <h1>Vocabulary Drill: Lektion {N} — {Focus capitalized}</h1>
  <p class="focus">Focus: {focus description}</p>
  <p class="meta">Generated {YYYY-MM-DD} · Level: A1 · Momente A1</p>
</header>
```

### Nav
```html
<nav class="block-nav">
  <a href="#reference">📚 Vocabulary Reference</a>
  <a href="#examples">✏️ Examples</a>
  <a href="#exercises">🏋️ Exercises ({count})</a>
  <a href="#solutions">✅ Solutions</a>
</nav>
```
For `all` focus, add category sub-anchors in the nav: `#nouns`, `#verbs`, `#adjectives`, `#others`.

---

### Reference Section — NOUNS

Use the `.gender-grid` 3-column layout (same as der_die_das page):

```html
<section id="reference">
  <h2>📚 Vocabulary Reference</h2>
  <div class="gender-grid">

    <div class="gender-box masc">
      <h4>🔵 DER (masculine)</h4>
      <ul>
        <li>
          der Bahnhof, -höfe
          <span class="noun-en">train station</span>
          <span class="pattern-tag ptag-m">memorize</span>
        </li>
        <!-- more nouns -->
      </ul>
    </div>

    <div class="gender-box fem">
      <h4>🔴 DIE (feminine)</h4>
      <ul>
        <li>
          die Wohnung, -en
          <span class="noun-en">flat</span>
          <span class="pattern-tag ptag-f">-ung → die</span>
        </li>
      </ul>
    </div>

    <div class="gender-box neut">
      <h4>🟢 DAS (neuter)</h4>
      <ul>
        <li>
          das Hotel, -s
          <span class="noun-en">hotel</span>
          <span class="pattern-tag ptag-n">borrowed word</span>
        </li>
      </ul>
    </div>

  </div>
</section>
```

The pattern tag text should be short: "memorize", "-ung → die", "-chen → das", "borrowed", "calendar → der", etc.

---

### Reference Section — VERBS

One `.verb-card` per verb:

```html
<section id="reference">
  <h2>📚 Vocabulary Reference</h2>

  <div class="verb-card">
    <h4>fahren <span class="noun-en">to drive / to travel</span></h4>
    <table class="conj-table">
      <tr><td>ich</td><td>fahre</td></tr>
      <tr><td>du</td><td>fährst</td></tr>
      <tr><td>er / sie / es</td><td>fährt</td></tr>
      <tr><td>wir</td><td>fahren</td></tr>
      <tr><td>ihr</td><td>fahrt</td></tr>
      <tr><td>sie / Sie</td><td>fahren</td></tr>
      <tr class="perfekt-row"><td colspan="2">Perfekt: ist gefahren (sein)</td></tr>
    </table>
    <p class="example-note">Stem vowel change a → ä in du/er forms</p>
  </div>

  <!-- more verb cards -->
</section>
```

---

### Reference Section — ADJECTIVES

```html
<section id="reference">
  <h2>📚 Vocabulary Reference</h2>

  <div class="vocab-card">
    <span class="vc-german">schön</span>
    <span class="vc-english">beautiful / nice</span>
    <span class="vc-antonym">↔ hässlich</span>
  </div>

  <!-- more adjective cards -->
</section>
```

---

### Reference Section — OTHERS

```html
<section id="reference">
  <h2>📚 Vocabulary Reference</h2>

  <div class="vocab-card">
    <span class="vc-german">sehr</span>
    <span class="badge badge-adverb">adverb</span>
    <span class="vc-english">very</span>
    <span class="vc-note">Intensifier before adjectives: sehr gut.</span>
  </div>

  <!-- more cards -->
</section>
```

Badge class per subtype: `.badge-adverb`, `.badge-preposition`, `.badge-pronoun`, `.badge-conjunction`, `.badge-interjection`, `.badge-phrase`.

---

### Reference Section — ALL

For `all` focus, render all four subsections inside one `<section id="reference">`, each with its own `<h3>` heading and the appropriate layout above:
- `<h3 id="nouns">🔵 Nouns</h3>` → gender-grid
- `<h3 id="verbs">🟡 Verbs</h3>` → verb cards
- `<h3 id="adjectives">🟠 Adjectives</h3>` → vocab cards
- `<h3 id="others">⚪ Others</h3>` → vocab cards with subtype badges

---

### Examples Section

Use the same `.example-item` styling as grammar pages. Show only medium and complex examples:

```html
<section id="examples">
  <h2>✏️ Usage Examples</h2>

  <div class="example-item medium">
    <div class="example-german">1. Die Wohnung hat drei Zimmer.</div>
    <div class="example-english">→ The flat has three rooms.</div>
    <div class="example-note">Uses: die Wohnung (Lektion focus); das Zimmer (earlier lektion)</div>
  </div>

  <!-- more examples -->
</section>
```

---

### Exercises Section

Group by type with colored `<h3>` headers (same color scheme as grammar-explainer):

```html
<section id="exercises">
  <h2>🏋️ Exercises — {N} drills</h2>
  <p>Cover all exercises before checking solutions.</p>

  <h3 style="margin-bottom:12px; color:#1565c0;">Fill in the Blank</h3>
  <div class="exercise-item">
    <div class="exercise-header">
      <span class="exercise-num">1.</span>
      <span class="badge badge-fill_in_blank">Fill in blank</span>
      <span class="badge badge-simple">Simple</span>
    </div>
    <div class="exercise-prompt">___ Bahnhof ist sehr groß.</div>
  </div>

  <h3 style="margin: 24px 0 12px; color:#6a1b9a;">Multiple Choice</h3>
  <!-- multiple choice exercises -->

  <h3 style="margin: 24px 0 12px; color:#00695c;">Translation</h3>
  <!-- translation exercises -->

  <h3 style="margin: 24px 0 12px; color:#4e342e;">Error Correction</h3>
  <!-- error correction exercises -->

  <!-- For nouns: -->
  <h3 style="margin: 24px 0 12px; color:#37474f;">Plural Formation</h3>
  <!-- plural exercises -->

  <!-- For verbs: -->
  <h3 style="margin: 24px 0 12px; color:#1b5e20;">Haben vs. Sein (Perfekt)</h3>
  <!-- haben/sein exercises -->

</section>
```

Badge class for plural_formation: use `.badge-type` (dark grey). For haben_vs_sein: use `.badge-type`.

---

### Solutions Section

```html
<section id="solutions" class="solutions-section">
  <h2>✅ Solutions</h2>

  <div class="solution-item">
    <div class="solution-answer">1. Der Bahnhof</div>
    <div class="solution-explanation">der Bahnhof — masculine. Must memorize; no reliable ending pattern.</div>
  </div>

  <!-- one solution-item per exercise, in order -->
</section>
```

---

### Footer

```html
<footer>
  Generated by German Learning Assistant &nbsp;·&nbsp; Level: A1 &nbsp;·&nbsp; {YYYY-MM-DD}
</footer>
```

---

## Step 8 — Save Files

Get current datetime:
```bash
date +"%Y-%m-%dT%H:%M:%S"
date +"%Y-%m-%d_%H%M%S"
```

Ensure directories exist:
```bash
mkdir -p data/vocabulary data/vocabulary-html
```

Save:
- JSON → `data/vocabulary/{file_slug}_{YYYY-MM-DD_HHMMSS}.json`
- HTML → `data/vocabulary-html/{file_slug}_{YYYY-MM-DD_HHMMSS}.html`

Confirm to user:
> "Saved to `data/vocabulary/{json_filename}` and `data/vocabulary-html/{html_filename}`"

---

## Step 9 — Regenerate data/vocabulary-html/index.html

After saving, regenerate the vocabulary index:

1. List all HTML files in `data/vocabulary-html/` (exclude `index.html` itself)
2. Parse each filename: `lektion{NN}_{focus}_{YYYY-MM-DD_HHMMSS}.html`
   - Extract `lektion_num`, `focus`, `date` (YYYY-MM-DD)
   - Label: "Lektion {N} — {Focus capitalized}" (e.g. "Lektion 5 — Nouns")
3. Group by Lektion number ascending; within each group, sort by date descending (newest first)
4. Write `data/vocabulary-html/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Vocabulary Drills — A1</title>
  <link rel="stylesheet" href="../../styles.css" />
</head>
<body>
<div class="container">

  <header>
    <h1>Vocabulary Drills — Momente A1</h1>
    <p class="meta">Organized by Lektion · All drill sessions listed below</p>
  </header>

  <nav>
    <a href="../../index.html">← Main Index</a>
  </nav>

  <!-- One block per Lektion -->
  <section>
    <h2>Lektion 1</h2>
    <ul class="file-list">
      <li><a href="{filename}">
        <span class="label">Lektion 1 — Nouns <span class="badge badge-simple">nouns</span></span>
        <span class="date">2026-03-10</span>
      </a></li>
      <!-- more entries for this lektion -->
    </ul>
  </section>

  <!-- more lektion sections -->

  <footer>German Learning Assistant · A1 · {today}</footer>
</div>
</body>
</html>
```

Use focus-colored badges in the label:
- nouns → `badge-grammar` (purple)
- verbs → `badge-sentences` (teal)
- adjectives → `badge-medium` (orange)
- others → `badge-type` (dark grey)
- all → `badge-complex` (red)

---

## Step 10 — Also Update Root index.html

After saving vocab files, also check if `index.html` at project root exists. If yes, add a new **Vocabulary** section listing the new file. If that section already exists, append to it. The structure should follow the same `.file-list` pattern as the other sections in that file.

---

## Quality Checklist (internal — do not display)

Before saving, verify:
- [ ] All vocabulary for the Lektion extracted (not just a subset)
- [ ] Every focus word enriched with required fields
- [ ] Examples use only medium/complex level
- [ ] At least 40 exercises generated
- [ ] Every focus word appears at least once in the exercises
- [ ] HTML links to `../../styles.css` — no inline `<style>` block
- [ ] `<body class="grammar-page">` and `<nav class="block-nav">` present
- [ ] Solutions section is complete (one per exercise, in order)
- [ ] JSON `options` key present only for `multiple_choice` exercises
- [ ] Both files saved before reporting to user
- [ ] `data/vocabulary-html/index.html` regenerated

---

## Display in Chat (brief)

After saving, show in chat:
1. **Header**: `## Vocabulary Drill: Lektion {N} — {Focus}`
2. **Vocabulary count**: "Extracted {N} {focus} from Lektion {N}"
3. **Examples**: Show all examples in the format:
   ```
   **[ID]. [German]**
   → [English]
   📝 [note]
   ```
4. **Exercise summary**: "Generated {N} exercises across {types} types"
5. **Do not show exercises or solutions in chat** — they are in the HTML file
6. File paths saved