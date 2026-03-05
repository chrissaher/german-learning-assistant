---
name: sentence-builder
description: Use this skill when the user writes "sentence builder: [word]" or "sentence builder: [word] ([type])". Generates 10 A1-level German example sentences for a given word, grouped by difficulty (simple/medium/complex), with English translations and grammar notes, then saves them as a JSON file to data/sentences/.
---
## Allowed tools
* Write: to write a file into a folder
* Bash: to verify if a certain folder has been created

## Step 1 — Parse Input

Extract:
- **word** (required): the German word to practice
- **word_type** (optional): noun / verb / adjective / adverb / other
  If not given, infer it from the word itself.
- **base_form**: dictionary form (infinitive for verbs, nominative singular for nouns, base form for adjectives)

---

## Step 2 — Generate 10 Sentences

Produce exactly **10 sentences** at A1 level, split into three groups:

| Group    | Count | Description |
|----------|-------|-------------|
| Simple   | 3     | Short (≤ 6 words), Präsens, basic structure (Subject + Verb + Object/Predicate) |
| Medium   | 3     | 7–10 words, Präsens; may include an adverb, time phrase, or prepositional phrase |
| Complex  | 4     | 8–14 words; at least 1 sentence in Perfekt; may have a modal verb or coordinating conjunction (und, aber, oder) |

### Conjugation & Declension Rules

**For verbs:**
- Cover at least 4 different persons across the 10 sentences: ich, du, er/sie/es, wir, Sie
- Use Präsens for most sentences; include 1–2 Perfekt sentences in the "complex" group

**For nouns:**
- Use both singular and plural forms across the sentences
- Use at least nominative and accusative cases
- Use definite and indefinite articles correctly (der/die/das, ein/eine)

**For adjectives:**
- Use predicate position (Das Haus ist groß.) and attributive position (ein großes Haus) across sentences
- Show agreement (gender/case) in attributive sentences

### Content Rules
- All vocabulary must be A1-appropriate
- Sentences should be realistic, travel/tourism-relevant where natural
- Avoid abstract or literary language

---

## Step 3 — Build the JSON Object

Construct a JSON object using this schema exactly:

```json
{
  "word": "<the word as the user typed it>",
  "base_form": "<dictionary/base form>",
  "word_type": "<noun|verb|adjective|adverb|other>",
  "generated_at": "<ISO 8601 datetime, e.g. 2026-03-04T14:30:22>",
  "sentences": [
    {
      "id": 1,
      "german": "<German sentence>",
      "english": "<English translation>",
      "level": "<simple|medium|complex>",
      "grammar_note": "<1-line grammar note>"
    }
  ]
}
```

- `generated_at`: use today's date and current time in ISO 8601 format
- `id`: sequential 1–10
- `level`: must match the group ("simple", "medium", or "complex")

---

## Step 4 — Save the JSON File

Determine the filename:
- Pattern: `data/sentences/{base_form_lowercase}_{YYYY-MM-DD_HHMMSS}.json`
- Example: `data/sentences/essen_2026-03-04_143022.json`
- Use the same datetime you put in `generated_at`
- Lowercase the base form; replace spaces with underscores if the base form is multiple words

Before writing, use the **Bash** tool to ensure the directory exists:
```
mkdir -p data/sentences
```

Then use the **Write** tool to save the file to `data/sentences/` (path relative to the project root).

After saving, confirm to the user:
> "Saved to `data/sentences/{filename}`"

---

## Step 5 — Display the Sentences

Show the sentences grouped by level. Use this exact format for each sentence:

```
**[ID]. [German sentence]**
→ [English translation]
📝 [1-line grammar note]
```

Display order:
1. A header: `## Sentence Builder: [word] ([word_type])`
2. `### Simple (3)`
3. The 3 simple sentences
4. `### Medium (3)`
5. The 3 medium sentences
6. `### Complex (4)`
7. The 4 complex sentences

After the sentences, add a short **Transfer Notes** block (2–4 bullets) flagging:
- Any Spanish → German false friends or helpful parallels
- Any English → German false friends or helpful parallels
- Key gender/case points for this word

Then add a **Key Takeaway** block — one bold sentence summarizing the single most important thing to remember about this word (structure, usage trap, or helpful analogy). Format:

> **Key takeaway for [word]:** [one sentence]

---

## Quality Checklist (internal — do not display)

Before displaying, verify:
- [ ] Exactly 10 sentences (3 simple, 3 medium, 4 complex)
- [ ] At least 4 different verb persons covered (if word is a verb, or if verbs appear in sentences)
- [ ] At least 1 Perfekt sentence in complex group
- [ ] Nouns use both singular/plural and nom/acc if word is a noun
- [ ] All sentences are A1-appropriate
- [ ] JSON matches schema exactly
- [ ] Filename follows the naming convention
- [ ] Key Takeaway displayed in chat