---
description: Resolve open checklist items by updating specs and plans incrementally, using best practices and escalating real decisions to the user.
---

## Purpose

Transform checklist findings into concrete, minimal improvements to the feature specification and implementation plan.

This command:

1. Reads a specific checklist file under `specs/<branch>/checklists/`, where <branch> relates to folder name containing the checklist and is the same name as the current local git branch we are on.
2. For each unchecked item, determines whether it can be resolved:
   - Directly, by updating `spec.md` / `plan.md` / related docs using established best practices and existing project conventions; or
   - Only via explicit product/engineering decision from the user.
3. Applies targeted edits to the spec/plan to resolve safe items, keeping to the structure and sections already established.
4. Marks the corresponding checklist items as `[x]` once addressed.
5. Leaves genuinely open questions as `[ ]` with clear notes, and asks the user to choose between concrete options.

It MUST NOT recreate, wholesale rewrite, or discard existing specs/plans. It operates incrementally on top of human-authored content.

## Non‑Goals and Write Policy

- Do NOT generate any new code or scripts. No new files anywhere in the repo.
- Do NOT modify source code or automation (for example `src/`, `scripts/`, `.specify/`).
- The ONLY files you may edit are:
  - The feature’s `spec.md`
  - The feature’s `plan.md`
  - The target checklist file itself (to tick items and append short notes)
- You may READ other files (for example `data-model.md`, `research.md`) for context, but MUST NOT write to them.

## Usage

Invoke this prompt as a simple slash command with optional positional arguments.

Supported patterns:

- `/speckit-custom.resolve-checklist`
- `/speckit-custom.resolve-checklist mychecklist.md`
- `/speckit-custom.resolve-checklist mychecklist1.md mychecklist2.md`

Rules:

- Treat everything after the command on the same line as raw arguments text.
- Do NOT define or require named parameters such as checklist_name, feature_dir, arg_checklist, or similar.
- Do NOT echo or inject any boilerplate with named placeholders appended to the command (for example, platform‑added parameter stubs).

If your environment auto-injects such placeholders (for example, `feature_dir=""`), ignore them completely when parsing input and never include them in your responses.

Argument handling:

1. No arguments:
   - Run `.specify/scripts/bash/check-prerequisites.sh --json` to obtain the feature directory path.
   - List `*.md` files under the feature directory’s `checklists/`.
   - If exactly one exists: select it automatically.
   - If none exist: report an error explaining that no checklists are available.
   - If more than one exists: list candidates and ask the user to choose; do not guess.

2. One or more arguments:
   - For each argument, attempt to resolve it as:
     - A filename under the feature directory’s `checklists/`, or
     - A relative path from the feature directory.
   - If an argument matches multiple files, show the options and ask the user to pick.
   - If an argument matches no files, report that argument as unresolved and continue with others that resolved successfully.

In all cases:
- Do not mutate the original command line.
- Do not require any key=value style named parameters.

## Workflow Overview

1. Identify the target checklist.
2. Load the relevant feature context.
3. Analyse unchecked checklist items.
4. Classify each item (AUTO_SPEC_PATCH / DECISION_NEEDED / OUT_OF_SCOPE).
5. Apply minimal patches for AUTO_SPEC_PATCH items and tick them.
6. Propose explicit options (without guessing) for DECISION_NEEDED items.
7. Summarise all changes and outstanding decisions.

## Step 1: Identify target checklist

1. Run:

   ```bash
   .specify/scripts/bash/check-prerequisites.sh --json
   ```

   Parse JSON to derive:

   - the feature directory path
   - available checklists under that feature (list of `*.md` files with absolute paths).

2. Resolve arguments_text (derived from arguments) using this precedence:

   1. If arguments is a simple filename (for example `sql-datatypes-classification.md`), search:
      - `feature_dir/checklists/arguments_text`
   2. If arguments is a relative path, normalise it from repo root to an absolute path.
   3. If arguments is empty:
      - If there is exactly one `*.md` file in the feature directory’s `checklists/`:
        - Automatically select that checklist (no prompt required).
      - If there are zero `*.md` files:
        - ERROR with a clear message and exit.
      - If there are more than one `*.md` files:
        - Do NOT guess. List all candidate files and ask the user which one to use.

3. If multiple candidates match a non-empty arguments_text:

   - Present a short, numbered list of absolute paths.
   - Ask the user which path to use.
   - Abort with a clear message if no valid selection is made.

4. If no match is found for a provided arguments_text:

   - ERROR with:
     - The requested value (arguments_text)
     - Where you searched
     - A hint on expected location (for example `specs/&lt;feature&gt;/checklists/&lt;name&gt;.md`).

All reasoning about files SHOULD use absolute paths internally.

## Step 2: Load feature context

Given the resolved checklist path, infer the feature directory as the parent feature folder (for example `specs/007-column-processing-controls`).

From the feature directory, load (if present):

- `spec.md`
- `plan.md`
- `data-model.md` (read‑only; do not modify)
- `research.md` (read‑only; do not modify)
- `contracts/` (filenames only, unless needed to resolve a specific item; read‑only)

Context loading rules:

- Only load content necessary to evaluate checklist items.
- Prefer summarising relevant sections over copying entire files.
- NEVER overwrite or drop existing sections.

## Step 3: Analyse unchecked checklist items

For each line in the target checklist:

```markdown
- [ ] CHK### Question text [tags and references]
```

extract:

- `CHK_ID` (e.g. CHK031)
- Question / requirement text
- Quality tags: `[Completeness]`, `[Clarity]`, `[Consistency]`, `[NFR]`, `[Gap]`, `[Assumption]`, `[Conflict]`, etc.
- References (e.g. `[Spec §X]`, `[Data Model §Y]`, `[Gap]`)

Then classify each unchecked item into one of:

1. `AUTO_SPEC_PATCH`
2. `DECISION_NEEDED`
3. `OUT_OF_SCOPE`

### Classification rules

`AUTO_SPEC_PATCH` when ALL are true:

- The item is consistent with existing direction and language of the feature.
- A safe, widely-accepted practice can satisfy it (for example clarifying case-insensitive matching, documenting deep-merge semantics, logging requirements).
- Implementing it does not:
  - Introduce a breaking change, or
  - Imply strong external commitments (SLAs, legal guarantees) beyond current scope.

`DECISION_NEEDED` when ANY are true:

- Multiple valid patterns exist with different trade-offs.
- Choice affects:
  - External contracts, SLAs, data retention, or risk posture.
- The spec currently leaves this intentionally open or ambiguous.

`OUT_OF_SCOPE` when ANY are true:

- The item clearly belongs to another feature or layer.
- It conflicts with project constraints or agreed scope.
- Addressing it here would be misleading.

When in doubt between AUTO_SPEC_PATCH and DECISION_NEEDED, choose DECISION_NEEDED and ask.

## Step 4: Apply minimal patches for AUTO_SPEC_PATCH

For each `AUTO_SPEC_PATCH` item:

1. Determine the correct target document and section:

   - Behavioural and requirements clarifications:
     - `spec.md` under:
       - Clarifications
       - Edge cases
       - Requirements
       - Assumptions
   - Implementation or testing guidance:
     - `plan.md`
   - Note: Do NOT modify `data-model.md` or other documents.

2. Draft a concise addition or refinement that:

   - Uses the project’s existing terminology and style (section headings, bullet format, FR/SC style).
   - States the rule or requirement explicitly.
   - References related checklist IDs or sections only if helpful.

3. Apply the patch surgically (limited to `spec.md` and `plan.md` only):

   - Add bullets or short paragraphs.
   - Avoid large rewrites.
   - Do not delete human-written content.
   - Do not change semantics already clearly established unless the checklist identifies a true inconsistency.

4. After updating the spec/plan (and only those files):

   - Update the corresponding checklist item:

     ```markdown
     - [x] CHK### Original text [Resolved in Spec §.../Plan §...]
     ```

   - Preserve the CHK ID and question text; only change `[ ]` → `[x]` and optionally append a brief reference note.

## Step 5: Handle DECISION_NEEDED items (ask first)

For each `DECISION_NEEDED` item:

1. Propose 2–3 concrete options based on best practices and existing context.

   Each option MUST include:

   - A short label (e.g. “A. Strict support matrix”).
   - 1–2 bullets on impact and trade-offs.
   - Alignment with current project norms where possible.

2. Present options to the user:

   - Use a compact, explicit format.
   - Example:

     ```text
     CHK031: SQL Server version support strategy

     A) Strict LTS versions only (2019/2022) - simpler guarantees.
     B) Best-effort for 2012+ with warnings - broader coverage, more edge cases.
     C) Defer to separate infrastructure spec.

     Select A/B/C or specify a custom policy.
     ```

3. Until the user chooses:

   - DO NOT modify spec/plan for that CHK.
   - Keep the checklist item unchecked:

     ```markdown
     - [ ] CHK031 ... [Decision pending: see options in last resolve-checklist run]
     ```

4. After the user provides a decision:

   - Apply a targeted spec/plan patch encoding the selected option.
   - Update the checklist entry to `[x]` with reference.

## Step 6: Handle OUT_OF_SCOPE items

For each `OUT_OF_SCOPE` item:

1. Do NOT force-fit it into the current spec.

2. Add a concise note inline:

   ```markdown
   - [ ] CHK0XX ... [Out of scope for this feature; consider separate spec XYZ]
   ```

3. Optionally propose creating a dedicated checklist/spec for that domain.

The command MUST NOT silently drop out-of-scope items.

## Step 7: Reporting and Idempotence

At the end of execution, output a structured summary:

1. Target checklist:

   - Full absolute path.
   - Inferred feature directory.

2. Results:

   - `resolved_count`: number of items changed from `[ ]` → `[x]`.
   - `pending_decisions_count`: number of CHKs still `[ ]` classified as DECISION_NEEDED.
   - `out_of_scope_count`: number of CHKs marked as out-of-scope.

3. Patch log (succinct):

   For each resolved CHK:

   - `CHK### → file_path:section_name — one-line description`

   Example:

   ```text
   CHK009 → specs/007-column-processing-controls/spec.md:Clarifications — Added explicit rule for case-insensitive datatype matching.
   ```

4. Decision prompts:

   - For each DECISION_NEEDED CHK, restate:
     - CHK ID and summary
     - Options A/B/C (or similar)

5. Reminder:

   - Specs and plans were updated incrementally.
   - Checklist items corresponding to applied patches are now `[x]`.
   - Re-running `/speckit-custom.resolve-checklist` is safe:
     - It SHOULD re-scan,
     - Skip already resolved `[x]` items,
     - Only propose new edits for remaining `[ ]` or newly added items.

## Guardrails

- NEVER call `/speckit.specify` or `/speckit.plan` from this prompt.
- NEVER regenerate or wholesale overwrite:
  - `spec.md`
  - `plan.md`
  - `data-model.md`
  - Existing checklists
- ALWAYS:
  - Use minimal diffs.
  - Preserve authorship and narrative style.
  - Honour existing FR/SC patterns and terminology.
- When uncertain:
  - Ask explicit questions instead of guessing.
  - Classify as DECISION_NEEDED, not AUTO_SPEC_PATCH.

Additional prohibitions:

- Do NOT create new files (including scripts, CLIs, or helpers). This workflow only edits `spec.md`, `plan.md`, and the checklist file.
- Do NOT modify any files under `src/`, `scripts/`, `.specify/`, or elsewhere outside the feature directory’s `spec.md`/`plan.md` and the target checklist.
