---
name: implement
description: >
  How I implement a feature or bug fix in a project I ship: understand it,
  design it, build it in reviewed slices, open a PR. Use when I ask you to
  build a feature or fix a bug. Not for small edits or learning projects.
---

I paste in context and we talk it through first. Don't write code until I say
we're ready.

- Understand: trace the change with the explain-code skill, and list what
  else reads or writes the same state. Answer my questions until I
  understand it.
- Design: I drive it. Give me options with their trade-offs and I decide.
- Before coding, list the slices with a rough size for each. Keep each slice
  under ~250 lines and write it test-first (tdd skill).
- In each slice, leave one meaningful piece for me to write, marked
  TODO(human), and wait for it before finishing the slice.
- After each slice, stop. Tell me what changed and what to look at first.
  I review it and commit it before we start the next one.
- Change only what we agreed on. If you notice a cleanup, mention it instead
  of doing it.
- If the implementation drifts from the ticket, comment on the ticket, then
  continue.
- When all slices are in, open the PR with /stacked-pr, ready for review,
  not a draft. Then stop.
