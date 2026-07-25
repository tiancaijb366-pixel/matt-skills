---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets. Follow this workflow:

### 1. Seams

Read the spec/ticket, identify the test seams. Present them to the user for confirmation before writing any code.

### 2. /tdd

Red-green at agreed seams. Drive TDD as:

- **Red** — write one failing test at the agreed seam.
- **Green** — write minimal code to pass it.
- **Diagnose** — run `lens_diagnostics mode=all` (or `lsp_diagnostics` on changed
  files) to catch lint/type/security issues introduced by this slice. Fix findings
  before the next slice.

Repeat. Typecheck + run single-test-file regularly.

### 3. /ponytail-review — opencode subagent

Use the opencode agent in its dedicated herdr pane:

```
herdr agent prompt opencode \
  "Ponytail-review the latest diff for over-engineering.
   Check: reinvented stdlib, speculative abstractions,
   unneeded deps, dead flexibility, boilerplate.
   Output findings as bullet points. Under 300 words.
   Diff: $(git diff HEAD~1..HEAD)" \
  --wait --timeout 120000
```

If opencode writes findings to a file under `architecture/inbox/to/pi/`, read it;
otherwise read its terminal output via `herdr pane read <pane-id>`.

Fix findings. Re-run `lens_diagnostics mode=all` to confirm no regressions.

### 4. /code-review — two parallel opencode subagents

Spawn a second opencode pane, run Standards + Spec in parallel, then fix
all findings together and diagnose once.

```bash
# 1. Split a new pane for the second opencode
SECOND=$(herdr pane split --pane MAIN_PANE --direction right \
  | jq -r '.result.pane.pane_id')
herdr agent start cr-spec --kind opencode --pane "$SECOND"

# 2. Prompt both in parallel
# Pane A (existing opencode) — Standards
herdr agent prompt opencode \
  "Review the latest git diff for Standards violations:
   - repo coding standards (check docs/ if any)
   - code smells (Mysterious Name, Duplicated Code,
     Feature Envy, Speculative Generality, etc.)
   Distinguish hard violations from judgement calls.
   Diff: $(git diff HEAD~1..HEAD)" \
  --wait --timeout 120000 &

# Pane B (new opencode) — Spec
herdr agent prompt cr-spec \
  "Review the latest git diff against the spec:
   - requirements missing or partial
   - behaviour not asked for (scope creep)
   - requirements that look wrong
   Diff: $(git diff HEAD~1..HEAD)" \
  --wait --timeout 120000 &

wait  # wait for both

# 3. Collect results
herdr pane read "$SECOND"
herdr pane close "$SECOND"
```

### 5. Fix & diagnose

Fix all findings from both reviews. Then run diagnostics once:

- `lens_diagnostics mode=all` — no blocking errors.
- `lsp_diagnostics path=. severity=error` — no LSP errors.
- Typecheck + full test suite (if applicable).

### 6. Commit

```bash
git add -A && git commit -m "<descriptive message>"
```
