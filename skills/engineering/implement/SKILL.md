---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets. Follow this workflow:

1. /tdd — Red-green loop at pre-agreed seams. Run typechecking regularly, single test files regularly, and the full test suite once at the end.

2. **Review (ponytail)** — Use the subagent tool (single mode, agent: "reviewer"). Task:
   > Review the changes for over-engineering: find code that should be deleted (yml, dead flexibility, unnecessary abstractions), replaced with stdlib, or simplified. Check `git diff`.

3. **Apply feedback (ponytail)** — Fix every finding from step 2: delete over-engineering, replace with stdlib, simplify. If changes are large, delegate to the "worker" subagent.

4. **Review (code)** — Use the subagent tool (single mode, agent: "reviewer"). Task:
   > Review the implementation for correctness, bugs, security issues, code smells. Check `git diff`. Output: Files Reviewed / Critical (must fix) / Warnings / Suggestions / Summary.

5. **Apply feedback (review)** — Fix all Critical and Warning issues from the review.

6. **Commit** — Commit the work to the current branch with a descriptive message.
