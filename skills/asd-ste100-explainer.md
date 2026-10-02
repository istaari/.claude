# ASD-STE100 Explainer Skill

This skill guides Claude in explaining complex topics using **ASD-STE100** (Simplified Technical English), a controlled language specification that dramatically improves clarity and readability.

## What is ASD-STE100?

ASD-STE100 is a controlled technical English standard originally developed for aerospace maintenance documentation. It enforces:
- **Clear vocabulary** — ~900 approved words with precise meanings
- **Simple sentence structure** — max 20 words per procedural sentence
- **Consistent grammar** — limited verb forms, banned constructions
- **Logical organization** — one topic per paragraph, active voice only

The result: explanations that are easier to parse, translate, and understand.

---

## Quick Reference: Key ASD-STE100 Rules

### Vocabulary
- **Use only approved words** — see Dictionary section below
- **One meaning per word** — "CLOSE" always means "to move together; to stop flow"
- **No synonyms** — use the same word consistently for the same concept
- **No abbreviations** unless in the Part 2 dictionary (e.g., "max", "qty")

### Sentence Structure
- **Procedural sentences** — max 20 words, one action per sentence
  - Format: `[Subject] [verb (imperative)] [object] [details].`
  - Example: `Close the valve.`
  
- **Descriptive sentences** — max 25 words, describe state or property
  - Example: `The pump supplies fuel to the engine when the switch is on.`
  
- **Safety instructions** — WARNING or CAUTION prefix, separate paragraph
  - Example: `WARNING: Do not touch hot parts.`

### Grammar Constraints
- **Approved verb forms only** (Section 3):
  - ✓ Imperative (command): "Close the valve."
  - ✓ Simple present: "The valve closes."
  - ✓ Simple past: "The valve closed."
  - ✗ Progressive ("-ing"): "The valve is closing." — NOT APPROVED
  - ✗ Perfect tenses: "The valve has closed." — NOT APPROVED
  
- **No ambiguous pronouns** — avoid "it", "they"; repeat noun or use technical name
- **Active voice only** — "The operator closes the valve" not "The valve is closed"
- **One instruction per sentence**
- **Vertical lists for complex text** — one item per bullet

### Word and Sentence Limits
- **Procedural sentence**: max 20 words
- **Descriptive sentence**: max 25 words  
- **Noun cluster** (e.g., "hydraulic pump motor assembly"): max 3 words
- **Instructions per sentence**: max 1 simultaneous action

---

## Approved Verb Forms (Section 3)

| Form | Example | Status |
|------|---------|--------|
| Imperative (command) | `Close the valve.` | ✓ Approved |
| Simple present | `The valve closes.` | ✓ Approved |
| Simple past | `The valve closed.` | ✓ Approved |
| Simple future | `The valve will close.` | ✓ Approved |
| Infinitive | `Turn the knob to close it.` | ✓ Approved |
| Progressive ("-ing") | `The valve is closing.` | ✗ NOT approved |
| Perfect tenses | `The valve has closed.` | ✗ NOT approved |
| Passive, in procedures | `The panel is removed.` | ✗ NOT approved |

---

## Common Approved Words by Category

### Procedure Verbs
- CLOSE, OPEN, TURN, PUSH, PULL, MOVE, MAKE, PUT, START, STOP, CHECK, TEST

### Technical Nouns
- PUMP, VALVE, SWITCH, PANEL, ENGINE, FUEL, PRESSURE, FLOW, SYSTEM

### Descriptive Words
- HOT, COLD, FULL, EMPTY, CLEAN, DIRTY, CORRECT, WRONG, SAFE, DANGEROUS

### Connecting Words
- AND, OR, IF, WHEN, BEFORE, AFTER, UNTIL, TO, FROM

**Note:** Always consult Part 2: Dictionary for the complete approved word list and preferred alternatives.

---

## Using This Skill with Claude

### Format 1: Explain a concept (Basic)
```
I want you to explain [topic/concept] using Simplified Technical English (ASD-STE100). 

Follow these rules:
- Use only simple, approved vocabulary
- Keep sentences short (max 20 words for procedures, 25 for descriptions)
- Use imperative or simple present/past tense only
- One idea per sentence
- Use active voice exclusively
- Avoid complex noun phrases
```

### Format 2: Explain with compliance strictness (Advanced)
```
Explain [topic] at 80% ASD-STE100 compliance:
- Use clear, simple words (but allow some common technical terms)
- Procedural sentences under 20 words
- Active voice, simple tense
- One action per sentence
- If needed, soften strict vocabulary constraints but keep structure strict
```

### Format 3: Rewrite to STE100 (Revision)
```
Rewrite the following text to meet ASD-STE100 standards:

[existing text]

Rules:
- Replace complex words with simple ones
- Break long sentences into short ones (max 20 words)
- Remove passive voice and progressive tenses
- Use only approved verb forms
- One topic per paragraph
```

---

## Example: Before & After

**BEFORE (Hard to parse):**
> The hydraulic system's sophisticated pressure-regulating mechanism, which is responsible for maintaining optimal operational parameters, has been comprehensively engineered to accommodate diverse environmental conditions whilst simultaneously ensuring that potentially hazardous fluctuations are mitigated through the implementation of redundant safety protocols.

**AFTER (ASD-STE100 ~ 80%):**
> The hydraulic system has a pressure regulator. It keeps the pressure correct for all work. If the pressure becomes too high, a safety valve opens. This stops damage to the system.

---

## Benefits

1. **Clarity** — Shorter sentences, simpler words = easier to understand
2. **Consistency** — Approved vocabulary prevents vague synonyms
3. **Translatability** — Constrained grammar translates well to other languages
4. **Scannability** — One idea per sentence = faster to parse
5. **Accessibility** — Works better for readers with language barriers, dyslexia, or cognitive load

---

## When NOT to Use Full ASD-STE100

- High-level strategic overviews (too constrained)
- Creative or persuasive writing (needs flexibility)
- Informal discussions (unnecessary rigor)
- When explaining advanced theory (may oversimplify)

**Good uses:**
- API documentation
- System architecture explanations
- Troubleshooting guides
- Educational walkthroughs
- Safety-critical instructions

---

## References

- **Specification**: ASD-STE100, Simplified Technical English, AECMA
- **Source**: asd-ste100.org
- **Maintenance**: STEM: Simplified Technical English Maintenance Group

---

## Prompting Tips

1. **Start with a basic request**: "Explain X in ASD-STE100" — Claude will adapt naturally.
2. **Allow flexibility**: "Explain X at 80% ASD-STE100" if strict rules feel too limiting.
3. **Use for clarity checks**: Ask Claude to rewrite your own explanations to STE100 and see what becomes clearer.
4. **Combine with visuals**: ASD-STE100 + diagrams = powerful pairing for explaining complex systems.
5. **Iterative refinement**: Ask Claude to "make it simpler" or "shorter" — it understands the direction.
