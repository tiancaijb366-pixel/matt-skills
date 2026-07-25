---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets. Follow this workflow:

1. **TDD** — Read the tdd skill's full content (loaded in context) and follow its instructions — red-green loop at the agreed seam, one slice at a time, no implementation-coupled tests. Run typechecking regularly, single test files regularly, and the full test suite once at the end.

2. **Ponytail-review** — Run the ponytail-review skill on the changes.

3. **Apply feedback (ponytail)** — Fix every finding from step 2: delete over-engineering, replace with stdlib, simplify. If changes are large, delegate to the "worker" subagent.

4. **Code review (subagent reviewer)** — Use the subagent tool (single mode, agent: "reviewer") to review the changes. Task:
   > Review the implementation of: {task description}. Run `git diff` to see changes. Check for bugs, security issues, code smells. Output: Files Reviewed / Critical (must fix) / Warnings / Suggestions / Summary.

5. **Apply feedback (review)** — Fix all Critical and Warning issues from the review.

6. **Commit** — Commit the work to the current branch with a descriptive message.
