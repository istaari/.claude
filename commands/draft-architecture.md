---
description: Generate a structured architecture plan document for any product or feature
argument-hint: '"<product-title-or-description>" [--prd <path-or-url>] [path/to/reference-resources/]'
---

# /draft-architecture — Architecture Planner

**Goal:** Generate a complete, structured architecture document for any product or feature — covering an architecture diagram, numbered component sections, and an end-to-end interaction flow.

---

## Step 1 — Help / No Arguments

If `$ARGUMENTS` is empty or `--help`, print the following and stop:

```
/draft-architecture — Architecture Planner
──────────────────────────────────────────────────────────────────
Generate a complete architecture plan document for any product or feature.

USAGE
  /draft-architecture "<product-input>" [--prd <path-or-url>] [resources-folder]

ARGUMENTS
  product-input      Required. A product title, short description, or feature
                     name/description. Wrap in quotes if it contains spaces.
                     Examples:
                       "Order Management System"
                       "Real-time fraud detection pipeline for card transactions"
                       "AI-powered customer support triage"

  --prd <path-or-url>  Optional. Path to a local PRD file or a URL pointing to
                     a product requirements document. Accepts .md, .txt, or any
                     plain-text format. Used as the primary requirements
                     source — takes precedence over resources-folder context.

  resources-folder   Optional. Path to a folder containing reference materials:
                     papers, design docs, RFCs, existing architecture files, code.
                     All readable files (up to 20) are used as context.

OUTPUT
  Full architecture document printed to stdout.
  Optionally saved to planning/<kebab-product-name>-architecture.md.

EXAMPLES
  /draft-architecture "Order Management System"
  /draft-architecture "Order Management System" --prd docs/prd.md
  /draft-architecture "Fraud Detection Pipeline" --prd https://notion.so/prd docs/research/
──────────────────────────────────────────────────────────────────
```

---

## Step 2 — Parse Arguments

From `$ARGUMENTS`, extract:

- `PRODUCT_INPUT` — the first argument; required.
  - If the argument starts with a `"`: extract the content between the first pair of double quotes.
  - Otherwise: use all text before the first flag (`--`) or path-like token as `PRODUCT_INPUT`.
  - `PRODUCT_INPUT` is the primary generation context — it may be a short name, a sentence-length description, or a feature specification.

- `PRD_PATH` — value of `--prd <value>` if present; otherwise empty. May be a local file path or a URL.

- `RESOURCES_FOLDER` — any remaining text that is not a flag or its value (may be empty).

If `PRODUCT_INPUT` is empty or missing, print:

```
Error: product-input is required.
Usage: /draft-architecture "<product-title-or-description>" [--prd <path-or-url>] [resources-folder]
```

Then stop.

Derive `PRODUCT_NAME` from `PRODUCT_INPUT`:
- If 1–5 words: use as-is.
- If longer: extract a 2–5 word name capturing the core subject (e.g., "Real-time fraud detection pipeline for card transactions" → "Fraud Detection Pipeline").

---

## Step 3 — Load Reference Resources

Only run this step if `RESOURCES_FOLDER` is set.

1. List all files in `RESOURCES_FOLDER` (recurse one level deeper if fewer than 3 files are found at the top level).
   - Skip: `*.png`, `*.jpg`, `*.gif`, `*.svg`, `node_modules/`, `.git/`, `target/`, `dist/`, `build/`
   - Cap at 20 files. If more exist, prefer `.md` and `.txt` first, then others by ascending file size.

2. Read each file. For files > 400 lines, read the first 300 lines and note truncation.

3. Write a 1–2 sentence summary of each file's content and store collectively as `RESOURCE_CONTEXT`.

4. If the folder is empty or unreadable:
   ```
   Warning: No readable files found in <folder>. Proceeding without reference resources.
   ```
   Set `RESOURCE_CONTEXT = ""` and continue.

---

## Step 4 — Load PRD Document

Only run this step if `PRD_PATH` is set.

**If `PRD_PATH` is a URL** (starts with `http://` or `https://`):
- Fetch the page content using the web fetch tool.
- Extract the main body text; discard navigation, headers, and boilerplate.
- If the fetch fails: print `Warning: Could not fetch PRD from <url>. Proceeding without it.` and set `PRD_CONTEXT = ""`.

**If `PRD_PATH` is a local path:**
- Read the file. For files > 600 lines, read the first 500 lines and note truncation.
- If the file does not exist: print `Warning: PRD file not found at <path>. Proceeding without it.` and set `PRD_CONTEXT = ""`.

Store the loaded content as `PRD_CONTEXT`.

`PRD_CONTEXT` is the highest-priority context source — it defines what the product must do. When it conflicts with `RESOURCE_CONTEXT` or `PRODUCT_INPUT`, prefer `PRD_CONTEXT`.

---

## Step 5 — Derive Architecture Skeleton

Before generating any output, reason through the system implied by `PRODUCT_INPUT`, `PRD_CONTEXT`, and `RESOURCE_CONTEXT`. Build a brief internal skeleton (not shown in final output):

1. **Identify 3–6 logical layers** for this product. Common patterns:
   - Data & Perception (sources, feeds, classifiers, enrichers)
   - Planning / Reasoning (LLM, rules engine, scheduler, router)
   - Execution (processors, handlers, actioners, publishers)
   - Persistence (databases, queues, caches, object stores)
   - Background / Offline (batch jobs, retraining, policy evolution, feedback loops)
   Name and adapt layers to fit the specific product domain.

2. **Identify 5–15 components** across those layers — one per distinct responsibility. Each component owns one clear output.

3. **Identify primary data flows** between components — note the data type each edge carries.

Anchor technology choices, component names, and data types in `PRD_CONTEXT` first, then `RESOURCE_CONTEXT`, wherever possible.

---

## Step 6 — Generate the Document

Produce the complete architecture document following the structure below.

---

### Title and Overview

Open with an H1 title: `# <PRODUCT_NAME>: <one-line tagline>` where the tagline captures the system's primary value in under 10 words.

Follow with a blockquote (using `>`): 2–3 sentences covering the system's purpose, its key design backbone, and the primary problem it solves.

---

### Architecture Section

Open with `## Architecture`.

Produce a Mermaid `flowchart TD` diagram with the following rules:

**Subgraphs:** one per logical layer. Label format: `subgraph LAYER_ID["🔷 Layer Name"]`. Choose a distinct emoji per layer. Suggested emojis: 🛰️ for data/perception, 🧠 for planning/reasoning, 📈 for execution, 💾 for persistence, 🌙 for background/offline processing.

**Node label format:**
- Standard component: `ID["Label<br/><i>technology note</i><br/><i>what it produces</i>"]`
- Store or database: `ID[("Label<br/><i>technology note</i><br/><i>key fields stored</i>")]`
- Decision gate: `ID{{"Label<br/>gate condition"}}`
- External API or service: `ID[/"Label"/]`

Every node's subtitle (`<i>…</i>`) must describe what the component *does and produces* — not constraints or what it doesn't do. Be specific.

**Edges:** directed only (`-->`). Label each edge with the data type it carries. Use quoted labels when the label contains spaces or special characters.

**Color coding:** define one `classDef` per layer, giving each class a name that matches its actual chosen layer (not the example names below). Draw fill/stroke colors from this palette — one entry per layer, in order:

```
fill:#1e3a5f,stroke:#4a9eff,color:#fff;
fill:#2d1b4e,stroke:#9b59b6,color:#fff;
fill:#1a3a2a,stroke:#27ae60,color:#fff;
fill:#3a2a1a,stroke:#e67e22,color:#fff;
fill:#2a1a2a,stroke:#c0392b,color:#fff;
fill:#1a2a3a,stroke:#3498db,color:#fff;
```

For example, with layers named `perception`, `planning`, and `execution`:

```
classDef perception fill:#1e3a5f,stroke:#4a9eff,color:#fff;
classDef planning   fill:#2d1b4e,stroke:#9b59b6,color:#fff;
classDef execution  fill:#1a3a2a,stroke:#27ae60,color:#fff;
```

Close the diagram with `class NODE_ID layer_name;` assignments for every node, using the layer-matched class names.

---

### Components Section

Open with `## Components`.

Number components sequentially starting from 1. Order them by data-flow dependency through the product's actual chosen layers — earliest input first, then each layer that consumes the previous one's output, ending with persistence and any offline/background layer. (For a `perception → planning → execution → persistence → background` product this reads "data enters → data is reasoned about → data is acted on → data is persisted → offline loop"; adapt the ordering to whatever layers this product actually has.)

For each component, produce the following structure in exact order:

**H3 heading:** `### N. Component Name`

**Role and Technology bullets:**
- `- **Role:** one sentence stating what this component does and what it produces`
- `- **Technology:**` followed by 2–4 sub-bullets. Each bullet names a specific library, model, framework, or managed service — with version or variant where known.

**Input block:** bold heading `**Input**` followed by a fenced `json` code block. Keys are field names; values are strings in the format `"type — description"`.

**Output block:** bold heading `**Output**` followed by a fenced `json` code block in the same format.

**Interactions table:** bold heading `**Component interactions**` followed by a Markdown table with columns: Direction / Component / What it provides. List every upstream and downstream partner. Direction values: `receives from` or `sends to`.

**Notes:** one or more bullets in the format `- **Note (<short label>):** one sentence`. Notes explain *why* — a design constraint, rationale, or non-obvious invariant. Not a restatement of Role.

---

### Interaction Flow Section

Open with `## Interaction Flow`.

Produce a Mermaid `sequenceDiagram` with `autonumber`. Show the primary happy-path from system input to output, stepping through the product's actual chosen layers and components in data-flow order. (For a `perception → planning → execution → persistence → background` product this is ingest → plan/decide → execute → persist → background feedback loop, the loop compressed to 1–2 exchanges; include only the phases this product actually has.)

After the diagram, write `### Worked Example: <concrete scenario title>` with a specific, realistic event for this product.

Break into 4–6 steps using `**Step N — <phase name>**` headings. Each step has 1–2 sentences and, where appropriate, a fenced `json` block with realistic domain values — not placeholder strings.

---

## Step 7 — Output

Print the complete document to the conversation.

Then determine the save path `planning/<kebab-product-name>-architecture.md` and check whether a file already exists there.

**If it does not exist**, print:

```
──────────────────────────────────────────────────────────────────
Save to planning/<kebab-product-name>-architecture.md? (yes / no)
──────────────────────────────────────────────────────────────────
```

**If it already exists**, print instead:

```
──────────────────────────────────────────────────────────────────
planning/<kebab-product-name>-architecture.md already exists — overwrite? (yes / no)
──────────────────────────────────────────────────────────────────
```

---

## Step 8 — Save (if confirmed)

If the user answers `yes` or `y`:
1. Create the `planning/` directory if it does not exist.
2. Write the document to `planning/<kebab-product-name>-architecture.md`, where the filename is `PRODUCT_NAME` lowercased with spaces and punctuation replaced by hyphens.
3. Confirm: `Saved to planning/<kebab-product-name>-architecture.md`

If the user answers `no` or `n`: stop.

---

## Rules

- Generate every section in full — never stub, truncate, or use placeholder text.
- Component count must be between 5 and 15. Split if a component has two clearly distinct responsibilities; merge if two components only ever communicate with each other.
- When `PRD_CONTEXT` is available, every component must trace back to at least one requirement or user story from it. When `RESOURCE_CONTEXT` is available, ground at least one technology choice, one JSON field name, and one worked-example value per major component in the reference material.
- Do not import domain knowledge from unrelated domains unless `PRODUCT_INPUT` is explicitly in that domain.
- If a technology is genuinely unknown for a component, choose a widely-used sensible default and add `- **Note (default tech):** <choice> used as a default; replace with your stack's equivalent.`
- All Mermaid blocks must be syntactically valid. Check mentally: every `subgraph` has a closing `end`, edge labels with spaces are quoted, node IDs contain no spaces or reserved characters.
- JSON blocks in Input/Output sections use `"key": "type — description"` format. JSON blocks in the Worked Example use realistic domain values.
- The interactions table must be complete — every upstream and downstream edge in the architecture diagram has a corresponding row in the component's interactions table.
