---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets. Follow this workflow:

1. **Seams** — Read the spec/ticket, identify the test seams. Present them to the user for confirmation before writing any code.

2. /tdd — Red-green at agreed seams. Run typechecking + tests regularly.

3. /ponytail-review — Subagent (agent: "reviewer") to check for over-engineering. Fix findings.

4. /code-review — Two parallel subagent calls (agent: "reviewer") — Standards + Spec. Fix findings.

5. Commit to the current branch with a descriptive message.
