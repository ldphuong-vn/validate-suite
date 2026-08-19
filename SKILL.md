---
name: validate-suite
description: Orchestrator for the validation pipeline. Use this whenever the user wants to validate an idea, product, feature, pivot, or a point decision (pricing, channel, kill/sunset) — to prepare for GTM or to run a full-lifecycle operating review. Triggers on "validate this", "should we build/ship/kill X", "stress-test this idea", "is this worth pursuing", "GTM readiness", or any request to vet a concept and produce a decision with evidence and critique. This skill runs Intake (classifies the case and routes the stages), executes the staged gates in order, streams progress to the console, and renders an HTML Validation Dossier at the end. Always use this rather than improvising an ad-hoc analysis, so the result is gated, evidence-based, and recorded in a reusable Dossier.
---

# Validate — Pipeline Orchestrator

This is the entry point. It does NOT do the analysis itself — it classifies the case, decides which
stages run, then hands each stage to its own skill, accumulating everything into one **Validation
Dossier** (the single source of truth). At the end it triggers the renderer.

Read `dossier-schema.md` before starting if the schema isn't in context. Hold these invariants:
stages only read/write the Dossier (never print HTML or talk to the user directly); the orchestrator
owns all user interaction and console output; thresholds/weights stay as numbers in the Dossier.

**Evidence discipline (mandatory):** every stage must follow `guardrails.md` — tag each gate's
evidence with an `evidence_type`, never fabricate figures (search, ask, or mark `UNKNOWN`), and let
weak evidence lower confidence. A confident verdict built on `ASSUMED`/`UNKNOWN` evidence is the
failure mode this whole pipeline exists to prevent.

## Step 0 — Initialize the Dossier
Create an in-memory Dossier with `meta.schema_version = "1.0.0"`, a `dossier_id`, `title`,
`one_liner`, timestamps. Everything below appends to it.

Set `meta.mode` (`STATEFUL` | `STATELESS`) — see `modes.md`. Default to the surface: private/with-backend →
`STATEFUL`; external one-shot → `STATELESS`.

> **Chạy dạng skill trong chat:** khi chưa nối một lớp lưu trữ (database lưu Dossier/outcome/
> tripwire/founder_profile + scheduled jobs), **buộc `STATELESS`**. Lý do: lớp mentor không có nơi
> ghi — chạy STATEFUL sẽ "hứa đã lưu / đã arm tripwire" trong khi không có store nào, đúng kiểu ảo
> giác mà `guardrails.md` cấm. Vẫn ghi Dossier ra file `*.dossier.json` + render HTML để xem lại.
> Bật STATEFUL chỉ sau khi đã nối backend lưu trữ.

The mode controls a memory/calibration layer on top of
the same core pipeline:
- **STATEFUL** (private/with-backend): before running, load `founder_profile` to personalize critique and use
  *calibrated* thresholds; after running, persist the Dossier to the ledger, schedule outcome tracking,
  and arm reversal_conditions as tripwires. See `mentor-layer.md`. Proactive nudges run on a scheduled job.
- **STATELESS** (external one-shot): no memory reads/writes, no calibration, no profile, no outcome
  tracking; use static default thresholds; retain nothing after the session; state the one-shot
  limitation transparently in the report (`modes.md`).

## Step 1 — Intake (classify + route)
Read the user's free-form description and **infer `case_type`**. Only ask the user directly if genuinely ambiguous; otherwise infer and confirm the routing map in Step 2.

| case | meaning | typical entry signal |
|---|---|---|
| `A_IDEA` | greenfield idea | "I have an idea for…", nothing built yet |
| `B_PRODUCT` | live product, audit | "is our product still healthy / should we double-down" |
| `C_FEATURE` | feature on existing product | "should we add / prioritize feature X" |
| `D_PIVOT` | pivot decision | "should we pivot from X to Y" |
| `E_DECISION` | point decision | pricing, new channel, new market, kill/sunset |

Also capture: `intake.objective` (what THIS run decides), and `intake.decision_makers`
(`[{name, role}]` using RAPID roles — the person with role `DECIDE` becomes `journal.decided_by`).

**Emit intent tags** (`intake.tags[]`) — the intent layer that prevents calling irrelevant lenses.
These are near-binary flags inferred cheaply from the objective/case (no heavy LLM step); ask one
question only if ambiguous. Vocabulary and the tag→lens mapping live in `lens-registry.md`:
`has_business_model`, `touches_pricing`, `touches_marketing`, `paid_acquisition`, `downstream_effects`
(`decision_gate` is always on when stage 6 runs). `paid_acquisition` implies `touches_marketing`.
Lenses to invoke = the union of `lens-registry.md` rows for the active tags — do NOT read every lens's
condition each time; match tags against the registry. Scope stays within business decisions.

**Default routing per case** (set `intake.routing` = `{stage_id: mode}`). Adjust to the specifics:

```
                 0 FRAME  1 GATE  2 PROB  3 SOL   4 MKT   5 BIZ   6 DEC   7 GTM   8 LIFE
A_IDEA           MUST     MUST    MUST    MUST    MUST    MUST    MUST    GO→MUST GO→MUST
B_PRODUCT        MUST     MUST    ASSERT  ASSERT  MUST    MUST    MUST    SKIP    MUST
C_FEATURE        MUST     MUST    ASSERT  MUST    SKIP    ASSERT  MUST    SKIP    ASSERT
D_PIVOT          MUST     MUST    MUST    MUST    MUST    ASSERT  MUST    GO→MUST GO→ASSERT
E_DECISION       MUST     SKIP    SKIP    SKIP   (only relevant stages) MUST    SKIP    SKIP
```
- `MUST` = run the gate. `ASSERT` = evidence already exists; record it (with a reference) without
  re-running the gate. `SKIP` = not relevant for this case.
- `GO→MUST` means: run only if stage 6 returns `GO`.
- **Plan mode:** for a live product (B_PRODUCT) or a GTM/marketing-prep request, the product's
  existence is the prior GO — route stage 7 to `MUST_RUN` regardless of a fresh verdict (see gtm-plan).
- **Low-n confidence flag:** when only 1–2 stages contribute a real score (common for repricing /
  point decisions), the averaged confidence rests on thin support — flag it ("confidence dựa trên ít
  tầng, đọc thận trọng") and lean on REVIEW rather than a falsely confident number.
- For `E_DECISION`, turn on just the stages the decision needs (e.g. pricing → 5 + 6; new market →
  4 + 6) and route straight to the decision gate.

## Step 2 — Print the route and confirm
Print the routing map to the console so the user sees what will run before any work happens, then
get a quick confirmation:

```
[Intake] case C_FEATURE · mục tiêu: <objective>
  → đường đi: 0,1,3,6 MUST_RUN · 2,5,8 ASSERT · 4,7 SKIP
  Xác nhận chạy? (hoặc chỉnh case/route)
```

## Step 3 — Run stages in order
Walk `stage_id` 0→8. For each stage per its `mode`:
- **MUST_RUN** → `read` the stage's SKILL.md (`stages/<name>/SKILL.md`) and execute it; it writes its
  record into `stages[]`.
- **ASSERT** → ask the user for / pull the existing evidence, write a stage record with
  `status: ASSERTED` and the evidence in `notes` (+ refs). Do not run new gates.
- **SKIP** → write a stage record with `status: SKIPPED`, empty gates.

Stage→skill map: 0 `framing`, 1 `hardgate`, 2 `problem-val`, 3 `solution-val`, 4 `market-gtm`,
5 `biz-model`, 6 `decision-gate`, 7 `gtm-plan`, 8 `lifecycle-ops`.

After each stage, print ONE console status line (verdict/score + a one-clause reason). Keep it
scannable; no long prose between stages.

## Step 4 — Honor the gates (stop early when you should)
- If stage 1 (`hardgate`) returns `FAILED`, **stop the pipeline**. Set `decision.verdict = NO_GO`
  with the failing gate as the reason, then still render the partial Dossier (the user gets a record).
- After stage 6, only continue to 7–8 if `decision.verdict == GO`. On `NO_GO`/`PARK`/`PIVOT`, stop
  after stage 6 (for `PIVOT`, note that re-entry starts at stage 0 carrying prior evidence).

## Step 5 — Render
When the run ends (completed or stopped), set `render.*` and trigger the renderer: inject the full
Dossier JSON into `render/template.html` (replace the `const DOSSIER = {…}` block) and write the
output HTML. Then present the file. The renderer is the only component that produces HTML.

Before injecting, populate `render.glossary`: read `glossary.md` and convert it into a
`{term: one-line plain-language definition}` map. The report's tooltips draw from that single
source — the template only keeps a static fallback for when a Dossier carries no glossary.

## Console style
One line per stage, machine-clean, Vietnamese to match the user. Lead each with `[Tầng N · <name>]`,
then verdict/score, then a single reason clause. Reserve longer explanation for the rendered report,
not the console.
