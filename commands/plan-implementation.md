---
description: Turn a technical architecture into a phased, vertical-slice implementation plan, deciding technology choices interactively one at a time in dependency order.
argument-hint: <architecture-folder-or-url>
---

# /plan-implementation — Implementation Planner

**Goal:** Read a product's technical architecture and produce a detailed implementation plan built from phased, vertical slices — while resolving every technology choice interactively, one decision at a time, in dependency order, using each answer to inform the next.

---

## Step 1 — Help / No Arguments

If `$ARGUMENTS` is empty or `--help`, print the following and stop:

```
/plan-implementation — Implementation Planner
──────────────────────────────────────────────────────────────────
Turn a technical architecture into a phased, vertical-slice implementation plan.

USAGE
  /plan-implementation <architecture-folder-or-url>

ARGUMENTS
  architecture-folder-or-url   Required. Either:
                                 • a local folder path containing the technical
                                   architecture (markdown, diagrams, docs), or
                                 • a URL pointing to the architecture document.

BEHAVIOUR
  Reads the architecture, then walks you through technology choices ONE AT A
  TIME in dependency order — recommending an option and its reasoning, and
  waiting for your decision before the next. Finally produces a phased
  implementation plan of end-to-end vertical slices.

OUTPUT
  Interactive technology decisions, then a full implementation plan.
  Optionally saved to planning/<kebab-product-name>-implementation-plan.md.

EXAMPLES
  /plan-implementation planning/
  /plan-implementation planning/architecture-planner.md
  /plan-implementation https://notion.so/our-architecture
──────────────────────────────────────────────────────────────────
```

---

## Step 2 — Parse Arguments

From `$ARGUMENTS`, extract:
- `ARCH_INPUT` — the first argument (may be a quoted path if it contains spaces). It is either a local folder/file path or a URL.

If `ARCH_INPUT` is empty or missing, print:

```
Error: architecture-folder-or-url is required.
Usage: /plan-implementation <architecture-folder-or-url>
```

Then stop.

---

## Step 3 — Load the Architecture

Route on `ARCH_INPUT`:
- If it starts with `http://` or `https://` → **URL branch**.
- Otherwise resolve it on the filesystem: if it is a directory (`test -d`) → **folder branch**; if it is a file (`test -f`) → **file branch**; if neither exists, print `Error: No readable architecture content found at <path>.` and stop.

**URL branch:**
- Fetch the page content using the web fetch tool.
- Extract the main body text; discard navigation, headers, and boilerplate.
- If the fetch fails: print `Error: Could not fetch architecture from <url>. Check the link and try again.` then stop.
- Note: private or authenticated pages (e.g. many Notion/Confluence docs) are often unfetchable. If the fetch returns a login wall or empty body, tell the user and suggest exporting the doc to a local file or folder and re-running.

**Folder branch:**
- List all files (recurse one level deeper if fewer than 3 files are found at the top level).
  - Skip: `*.png`, `*.jpg`, `*.gif`, `*.svg`, `node_modules/`, `.git/`, `target/`, `dist/`, `build/`
  - Cap at 20 files. If more exist, prefer `.md`, `.txt`, and files with "arch" in the name first, then others by ascending file size.
- Read each file. For files > 500 lines, read the first 400 lines and note truncation.

**File branch:**
- Read it in full.

Store the loaded content as `ARCH_CONTEXT`. If nothing readable was found, print `Error: No readable architecture content found at <path>.` then stop.

Derive `PRODUCT_NAME` from the architecture's title or the input path (2–5 words).

---

## Step 4 — Build the Technology Decision Queue

Read `ARCH_CONTEXT` and identify every technology choice the implementation genuinely requires — and **only** those the architecture actually calls for. Typical categories (include only what applies):

- Language / runtime
- Package manager / build tool
- Application framework(s) — backend and/or frontend
- Data store(s) — primary database, and any cache / search / blob store
- Data-access layer — ORM / query builder / driver
- Messaging / queue / streaming (only if the architecture has async or event flows)
- Authentication / authorization
- API style / transport (REST / gRPC / GraphQL / WebSocket)
- External integrations / SDKs named in the architecture
- Testing stack
- Deployment / hosting / infrastructure
- CI/CD
- Observability (logging, metrics, tracing)

**Order the queue by dependency** — a choice must come after anything it depends on and before anything that depends on it (e.g. language → framework → data-access → auth; hosting after runtime). Store this ordered list as `QUEUE`. Initialize `DECISIONS = []`.

**Pre-fixed technologies:** if the architecture already commits to a specific technology (e.g. it names "PostgreSQL" or "React"), treat that as a decision already made — seed it into `DECISIONS` with the note "fixed by architecture", do **not** ask the user about it, and remove it from `QUEUE`. Still use it as an input when reasoning about later choices.

Then print the roadmap so the user sees what's ahead — the ordered list of decision categories still in `QUEUE`, numbered, plus a one-line note of any pre-fixed choices already seeded — followed by:

```
I'll walk through these one at a time. For each I'll give a recommendation and
why, then wait for your call before the next.
```

---

## Step 5 — Interactive Technology Decisions (one at a time)

This is a strict loop. **Resolve exactly ONE decision at a time.** Present a single decision, ask it with one AskUserQuestion call, and only after that answer returns do you compute and present the next one. Never precompute or present two pending decisions at once. Never skip ahead to Step 6 until `QUEUE` is empty. (The AskUserQuestion tool blocks for the user's real input on every call, so each choice is genuinely made before the next is framed — the invariant that matters is that each recommendation is derived from the *already-answered* decisions, never batched up front.)

For the next category in `QUEUE` (call it decision `i`, where `n` = `i` + the number still remaining in `QUEUE`, recomputed each step since the queue can change):

1. Print a header: `### Decision <i>/<n>: <category>`
2. **Recommended:** state your recommended choice in one line.
3. **Why:** 2–4 sentences of reasoning. Ground it in (a) what `ARCH_CONTEXT` requires, and (b) every prior answer in `DECISIONS` — name the specific constraints driving the recommendation.
4. **Alternatives:** 1–2 realistic alternatives, each with a one-line trade-off (when you'd pick it instead).
5. Ask the user to decide using the **AskUserQuestion** tool:
   - List your recommended option **first**, labelled with `(Recommended)`.
   - Include the 1–2 alternatives as options.
   - The tool always lets the user supply their own answer, so do not add an "Other" option yourself.
   - Make exactly ONE such call — do not chain a second decision's question before this one is answered.

When the user answers:
- Append `{category, choice, rationale}` to `DECISIONS`.
- **Re-evaluate the remaining `QUEUE`** in light of the new choice — a decision can add, remove, reorder, or constrain later ones. Examples: a batteries-included framework may absorb the ORM, auth, and API-style decisions (remove them); choosing a managed platform may remove the CI/CD or hosting decision; picking a language may narrow the framework field. If the queue changes, briefly note what changed and why **before** framing the next decision.
- Then move to the next category and repeat from step 1 with recommendations recomputed from the updated `DECISIONS`.

Repeat until `QUEUE` is empty. Then print a compact **Chosen stack** summary table (Category / Choice) from `DECISIONS` and proceed to Step 6.

---

## Step 6 — Generate the Implementation Plan

Using `ARCH_CONTEXT` and the finalized `DECISIONS`, produce the implementation plan. Structure it as follows.

### Overview
2–3 sentences: what is being built and the delivery approach. State the guiding constraints explicitly: **vertical slices**, **minimal complexity**, **efficient by default** (no needless layers, round-trips, or indirection), and **standard engineering principles**.

### Chosen Stack
A table of the `DECISIONS` (Category / Choice / One-line reason).

### Guiding Principles
A short bulleted list actually tailored to this project — e.g. YAGNI (build only what the current slice needs), separation of concerns across the architecture's layers, dependency inversion at the boundaries that matter, prefer composition over inheritance. Name any design patterns that fit specific spots in this architecture (e.g. Repository at the data-access seam, Strategy for a pluggable rule, Adapter for an external SDK) — and state where each applies. Do **not** list patterns for their own sake; each entry must earn its place.

### Phases
Break delivery into ordered phases. **Every phase is a vertical slice**: it delivers working, end-to-end functionality that cuts through all relevant layers (e.g. UI → API → domain → data), not an isolated horizontal component ("all models", then "all controllers"). Phase 1 should be the thinnest end-to-end walking skeleton that proves the architecture's spine.

For each phase, provide:

- **Phase N — <name>** (one-line goal)
- **Milestone:** the observable, demoable outcome that marks this phase done.
- **User-facing capability:** the end-to-end behaviour a user (or caller) can exercise after this phase.
- **Scope:** in-scope work across each layer it touches; explicitly list what is deferred.
- **Key components / files:** the concrete pieces built or changed, grounded in the architecture's components.
- **Design patterns applied:** only those relevant to this phase, with the seam each addresses.
- **Definition of done:** testable acceptance criteria (including the tests to write).
- **Depends on:** prior phases required first.

### Sequencing Rationale
2–4 sentences on why the phases are ordered this way — what each unlocks and why the walking skeleton comes first.

### Cross-Cutting Concerns
A short list of things handled continuously across phases (error handling, config/secrets, logging/observability, security, testing strategy) — with the phase where each is first established.

---

## Step 7 — Output & Save

Print the complete plan to the conversation.

Then determine the save path `planning/<kebab-product-name>-implementation-plan.md` and check whether a file already exists there.

**If it does not exist**, print:

```
──────────────────────────────────────────────────────────────────
Save to planning/<kebab-product-name>-implementation-plan.md? (yes / no)
──────────────────────────────────────────────────────────────────
```

**If it already exists**, print instead:

```
──────────────────────────────────────────────────────────────────
planning/<kebab-product-name>-implementation-plan.md already exists — overwrite? (yes / no)
──────────────────────────────────────────────────────────────────
```

If the user answers `yes` or `y`:
1. Create the `planning/` directory if it does not exist.
2. Write the plan (filename is `PRODUCT_NAME` lowercased, spaces and punctuation replaced by hyphens).
3. Confirm: `Saved to planning/<kebab-product-name>-implementation-plan.md`

If the user answers `no` or `n`: stop.

---

## Rules

- **One decision at a time in Step 5.** Present a single decision, ask it with exactly one AskUserQuestion call, and compute the next only after that answer returns. Batching decisions, or precomputing later recommendations before the user has answered the current one, defeats the purpose.
- Every recommendation must cite the specific architecture requirement and prior decisions that drive it — not generic pros/cons.
- Re-evaluate the remaining queue after each answer; a good choice often removes downstream decisions rather than adding them.
- **Every phase is a vertical slice** — working end-to-end functionality, never an isolated technical layer. If a "phase" only builds one horizontal tier, merge or re-cut it.
- Favor the simplest, most efficient design that satisfies the current phase — no needless layers or round-trips. Do not introduce abstractions, indirection, or patterns for anticipated future needs (YAGNI). A pattern appears in the plan only where it removes real complexity at a named seam.
- Ground the plan in the architecture's actual components and the chosen stack — no invented services or requirements.
- Do not write any files until the user confirms in Step 7.
