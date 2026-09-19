---
description: Explain any concept clearly and effectively, with a focus on software engineering concepts and skill sets.
argument-hint: <concept> [--shallow]
---

# /explain-concept — Concept Explainer

---

## Step 1 — Help / No Arguments

If `$ARGUMENTS` is empty or `--help`, print the following and stop:

```
/explain-concept — Concept Explainer
──────────────────────────────────────────────────────────────────
Explain any concept clearly, with a focus on software engineering.

USAGE
  /explain-concept <concept> [--shallow]

FLAGS
  --shallow    Quick mode: Concept + Mental Model + Why It Matters + Examples only

EXAMPLES
  /explain-concept idempotency
  /explain-concept "consistent hashing" --shallow
  /explain-concept SOLID open-closed principle
──────────────────────────────────────────────────────────────────
```

---

## Step 2 — Parse Arguments

From `$ARGUMENTS`:
- If `--shallow` is present: set `SHALLOW = true`, remove it from args
- Otherwise: `SHALLOW = false`
- `TOPIC` = the remaining text
- If `TOPIC` is empty after stripping: print the help text from Step 1 and stop.

---

## Step 3 — Explain

Use `###` headers for every section. **Every section must include at least one visual element** — blockquote analogy, table, ASCII diagram, or code block. No section may be plain prose only.

---

### Concept

Open with the analogy as a `>` blockquote. Then one sentence of precise technical definition — no more.

---

### Mental Model

A `>` blockquote capturing the core intuition in one sentence. Then the guarantees/limitations table.

| Guarantees | Does NOT handle |
|---|---|
| ... | ... |

---

### Why It Matters

Before/after ASCII diagram (4–6 lines). One sentence naming the failure mode the "before" side causes.

---

### How It Works

ASCII flow or sequence diagram showing the key steps. One sentence on the critical invariant maintained.

*Skip this section in `--shallow` mode.*

---

### Examples & Use Cases

| Use Case | Why this concept fits |
|---|---|
| [real-world scenario] | [the specific property that makes it the right tool] |

3–4 rows. Span different domains (distributed systems, data structures, application design, infrastructure). Prefer examples a senior engineer would recognise from production.

---

### When NOT to Use It

The conditions under which this concept is the wrong tool — where its costs outweigh its benefits, or where a simpler alternative is better.

```
Avoid when:
  - [condition 1]
  - [condition 2]
  Prefer [simpler alternative] instead.
```

*Skip this section in `--shallow` mode.*

---

### Connections

ASCII sketch in a plain fenced block showing how this concept sits relative to 2–3 related concepts. One-line label per connection.

```
[Related A] ──── "relationship" ───→ [TOPIC] ←─── "relationship" ──── [Related B]
                                         │
                                   "relationship"
                                         ↓
                                    [Related C]
```

*Skip this section in `--shallow` mode.*
