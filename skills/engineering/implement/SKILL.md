---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets. Follow this workflow:

### 0. Workspace setup

If the project isn't a git repo yet:

1. `cd` to the project directory.
2. Run `/setup-matt-pocock-skills` (or set up `docs/agents/` + `AGENTS.md` manually for a local workspace).
3. `git init && git add -A && git commit -m "init: <project>"`.

### 1. Seams

Read the spec/ticket, identify the test seams. Present them to the user for confirmation before writing any code.

### 2. /tdd

Red-green at agreed seams. Run typechecking + tests regularly.

### 3. pi-lens diagnostics

After each round of changes, run `lens_diagnostics mode=all` (or `lsp_diagnostics` on the changed files)
to catch lint/type/security issues early. Fix any findings before moving on.

### 4. /ponytail-review — herdr subagent

Spawn a dedicated pi agent in a new herdr pane to review for over-engineering:

```bash
# Split a new pane
herdr pane split --pane CURRENT_PANE --direction right
# Start pi agent (herdr assigns the new pane ID automatically)
herdr agent start ponytail-review --kind pi --pane NEW_PANE
# Prompt with the diff context
herdr agent prompt ponytail-review \
  "Review the latest changes for over-engineering.
   Check: reinvented stdlib, speculative abstractions,
   unneeded deps, dead flexibility, boilerplate.
   Output findings as bullet points. Under 300 words.
   Diff: $(git diff HEAD~1..HEAD)" \
  --wait --timeout 120000
# Read the result
herdr pane read NEW_PANE
# Close the pane
herdr pane close NEW_PANE
```

Fix the findings. Then re-run `lens_diagnostics` to confirm.

### 5. /code-review — two parallel herdr subagents

Spawn **two** panes in parallel for Standards and Spec review:

```bash
# Pane A — Standards
herdr pane split --pane CURRENT_PANE --direction right
herdr agent start cr-standards --kind pi --pane PANE_A
herdr agent prompt cr-standards \
  "Review this diff for Standards violations:
   - repo coding standards (see docs/ if any)
   - code smells (Mysterious Name, Duplicated Code,
     Feature Envy, Speculative Generality, etc.)
   Distinguish hard violations from judgement calls.
   Diff: $(git diff HEAD~1..HEAD)" \
  --wait --timeout 120000 &

# Pane B — Spec
herdr pane split --pane CURRENT_PANE --direction right
herdr agent start cr-spec --kind pi --pane PANE_B
herdr agent prompt cr-spec \
  "Review this diff against the spec:
   - requirements missing or partial
   - behaviour not asked for (scope creep)
   - requirements that look wrong
   Diff: $(git diff HEAD~1..HEAD)" \
  --wait --timeout 120000 &

wait  # wait for both
herdr pane read PANE_A
herdr pane close PANE_A
herdr pane read PANE_B
herdr pane close PANE_B
```

Fix findings. Re-run `lens_diagnostics`.

### 6. Pre-commit check

Before committing:

- `lens_diagnostics mode=all` — no blocking errors.
- `lsp_diagnostics path=. severity=error` — no LSP errors.
- Typecheck + full test suite (if applicable).

### 7. Commit

```bash
git add -A && git commit -m "<descriptive message>"
```
