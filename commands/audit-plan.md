---
description: Review a software design plan on its own merits. Identify architectural gaps, internal inconsistencies, missing decisions, and risks — without adding complexity for its own sake.
argument-hint: <planning-file>
---

# /audit-plan — Software Design Refinement

**Goal:** Review a software design plan and surface only what genuinely matters — architectural gaps, missing decisions, risks, and design improvements grounded in software engineering best practices. Not a rewrite prompt.

---

## Step 1 — Help / No Arguments

If `$ARGUMENTS` is empty or `--help`, print the following and stop:

```
/audit-plan — Software Design Refinement
──────────────────────────────────────────────────────────────────
Review a software design plan for gaps, risks, and missing decisions.

USAGE
  /audit-plan <planning-file>

ARGUMENTS
  planning-file    Path to your design plan (markdown, RFC, architecture doc)

EXAMPLES
  /audit-plan plan.md
  /audit-plan "Architecture Plan.md"

OUTPUT
  Inline design review with findings grouped by type, followed by an option
  to apply improvements directly to the planning file.
──────────────────────────────────────────────────────────────────
```

---

## Step 2 — Parse Arguments

From `$ARGUMENTS`, extract:
- `PLANNING_FILE` — the first argument (may be a quoted path if it contains spaces)

If missing, print:
```
Error: planning-file is required.
Usage: /audit-plan <planning-file>
```
Then stop.

---

## Step 3 — Read Plan

Read `PLANNING_FILE` in full. Store as `PLAN_CONTENT`.

If unreadable: print `Error: Cannot read planning file at <path>. Check the path and try again.` then stop.

---

## Step 4 — Review

Evaluate `PLAN_CONTENT` across these software design dimensions using engineering best practices as the reference.

Assign every finding a stable identity as you record it: `[<DIM>-<n>]` where `<DIM>` is the dimension tag (`GAP`, `INCONSIST`, `DECISION`, `PATTERN`, `RISK`) and `<n>` is a counter within that dimension — e.g. `[GAP-1]`, `[RISK-2]`. Carry this id and its dimension with the finding through every later step; they are what the critic loop, the output grouping, and the apply step key off. `SOLID` observations are not findings and need no id.

### 4.1 — Architectural Gaps
What system concerns does the plan leave unspecified that are necessary for implementation?
- Look for: missing components, unaddressed system boundaries, unclear ownership of responsibilities, absent data flow, unspecified integration points.
- Flag only gaps that would materially affect implementation or correctness.

### 4.2 — Internal Inconsistencies
Where does the plan contradict itself or make incompatible assumptions?
- Be specific: quote the conflicting claims and explain why they cannot both be true.
- Common examples: a component described as stateless elsewhere owns persistent state; a synchronous flow assumes non-blocking behaviour; a scaling claim conflicts with a stated constraint.

### 4.3 — Missing Design Decisions
What decisions are implicit or deferred that need to be made explicit before implementation can begin?
- Look for: data model not specified, API contracts undefined, failure modes unaddressed, scaling strategy absent, consistency/availability trade-offs not resolved, tech stack choices unjustified.

### 4.4 — Design Patterns and Improvements
What established architectural patterns or practices would strengthen the design?
- Ground suggestions in the plan's stated goals and constraints — no invented requirements.
- Explain concretely where in the plan the improvement would apply.

### 4.5 — Risks and Red Flags
What design choices introduce unnecessary coupling, single points of failure, scalability ceilings, or unscoped complexity?
- Flag only risks visible from the plan itself — not hypothetical future concerns.

### 4.6 — What's Already Solid
Explicitly name design decisions in the plan that are well-reasoned. This prevents unnecessary churn.

---

## Step 5 — Critic Loop (max 3 iterations)

Only run this step if at least one finding exists after Step 4.

Set `ITERATION = 1`. Set `FINDINGS` = all findings from Step 4.

Repeat until `ITERATION > 3` or the critic makes no changes:

### Spawn a critic agent

Spawn a single agent with `subagent_type: general-purpose` and no tools — the critic reasons only over the text passed inline (it reads no files and writes nothing). Give it the following context and instructions:

---

**Critic agent prompt:**

```
You are a skeptical software design critic. Your job is NOT to generate new ideas —
it is to challenge a set of proposed design improvements and make them sharper.

PLAN:
<PLAN_CONTENT>

PROPOSED IMPROVEMENTS (iteration <ITERATION>):
<FINDINGS>          # each prefixed with its id, e.g. [GAP-1], [RISK-2]

For each finding, decide:
  KEEP    — valid, non-trivial, would materially improve the design
  DROP    — unjustified, adds complexity without clear benefit, or already implicit in the plan
  REFINE  — valid concern but the framing or proposed fix is wrong — provide a sharper version

Preserve each finding's id verbatim in your response. For any finding you DROP or
REFINE, give a one-line reason.

After reviewing all findings, check: is there anything the review missed that
materially affects the design? Add it as a NEW finding only if you are confident
— do not pad. Give each new finding a fresh id using the right dimension tag
(GAP / INCONSIST / DECISION / PATTERN / RISK).

Return your response in this exact format:

KEPT:
- [id] finding as-is

DROPPED:
- [id] original finding — reason

REFINED:
- [id] original finding → improved version — reason

NEW:
- [id] finding — rationale

(Omit any section that has no entries.)
```

---

After the critic agent returns, reconcile by id:
- Remove every finding whose id appears under DROPPED
- Replace each finding whose id appears under REFINED with its improved version (keep the same id and dimension)
- Add all NEW findings, keeping their assigned ids
- If nothing changed: exit the loop early

Increment `ITERATION`. If `ITERATION > 3`, exit the loop.

Print a brief status after each iteration:
```
Critic pass <n>/3: <X> kept, <Y> dropped, <Z> refined, <W> new findings
```

Proceed to Step 6 with the final `FINDINGS`.

---

## Step 6 — Output

Group the final `FINDINGS` by their dimension tag (`GAP` → Architectural Gaps, `INCONSIST` → Internal Inconsistencies, `DECISION` → Missing Design Decisions, `PATTERN` → Design Patterns and Improvements, `RISK` → Risks and Red Flags). Format the response as:

---

### Design Review: `<planning-file>`

---

#### Architectural Gaps
> System concerns the plan leaves unspecified.

- **[Component / concern]** — [1–2 sentences: what is missing and why it matters]

*(If none: "No significant architectural gaps found.")*

---

#### Internal Inconsistencies
> Where the plan contradicts itself or makes incompatible assumptions.

- **[Claim A]** vs **[Claim B]** — [why they conflict and what needs to be resolved]

*(If none: "No internal inconsistencies found.")*

---

#### Missing Design Decisions
> Decisions that are implicit or deferred but need to be explicit before implementation.

- **[Decision]** — [what needs to be decided and the recommended direction]

*(If none: "No critical decisions appear to be missing.")*

---

#### Design Patterns and Improvements
> Established patterns or practices that would strengthen the plan.

- **[Pattern / improvement]** — [what it is, where in the plan it applies, why it fits]

*(If none: "Nothing significant to add.")*

---

#### Risks and Red Flags
> Design choices that introduce coupling, fragility, or unscoped complexity.

- **[Risk]** — [what the concern is and what to consider instead]

*(If none: "No significant risks identified.")*

---

#### What's Already Solid
> Design decisions that are well-reasoned.

- [Decision or section] — [why it holds up]

---

**Overall verdict:** [1 sentence: design is ready to implement / needs specific decisions resolved / has significant structural problems]

---

## Step 7 — Apply Improvements

Only run this step if at least one finding remains in `FINDINGS` after the critic loop.

After printing the review, ask:

```
Apply these improvements to <planning-file>? (yes / no / select)
  yes     — apply all findings
  no      — leave the file unchanged
  select  — choose which findings to apply
```

**If `no`:** stop. The review stands as-is.

**If `select`:** list each finding by its id as a numbered item and ask the user to enter the numbers they want applied (e.g. `1 3 4`). Then proceed with only those.

**If `yes` or after selection:** for each approved finding, route by its dimension tag:
- **`GAP` (Architectural Gap)** → add a new section, component description, or bullet where the missing concern belongs in the plan structure.
- **`INCONSIST` (Internal Inconsistency)** → edit one of the conflicting claims in-place to resolve the contradiction.
- **`DECISION` (Missing Design Decision)** → insert an explicit decision block (stating the decision, the rationale, and alternatives considered) at the relevant section.
- **`PATTERN` (Design Pattern / Improvement)** → insert a concise description of the pattern and where it applies at the most relevant point in the plan.
- **`RISK`** → add a risk note or constraint callout adjacent to the design choice that introduces it.

Rules for edits:
- Make targeted edits only — do not restructure or reformat unrelated sections.
- Preserve the author's voice and existing formatting style.
- Do not add more than the finding warrants. One finding = one focused addition or correction.
- Never invent design decisions on the author's behalf.

After writing, print a summary:

```
Updated <planning-file>:
  + [brief description of what was added/changed]
  + ...
```

---

## Rules

- Base every finding on `PLAN_CONTENT` and software engineering best practices only.
- Quote the plan when flagging an inconsistency or gap.
- Do not restructure or rewrite the plan — only surface findings and make targeted edits.
- Do not add complexity for its own sake.
- If the design is already solid, say so clearly and stop. Don't manufacture feedback to seem thorough.
