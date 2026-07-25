---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets. Follow this workflow:

1. /tdd — Red-green at pre-agreed seams. Run typechecking + tests regularly.

2. /ponytail-review — Subagent (agent: "reviewer") to check for over-engineering. Fix findings.

3. /code-review — Two parallel subagent calls (agent: "reviewer") — Standards + Spec. Fix findings.

4. Commit to the current branch with a descriptive message.
