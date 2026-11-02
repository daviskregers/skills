---
name: implement
description: >
  How I implement a feature or bug fix in a project I ship: understand it,
  design it, build it in reviewed slices, open a PR. Use when I ask you to
  build a feature or fix a bug. Not for small edits or learning projects.
---

I paste in context and we talk it through first. Don't write code until I say
we're ready.

- Understand: answer my questions briefly and point me at the code as a route
  (explain-code skill): the hops with file and function, and what to look for
  at each. Give me the route to walk, not a summary that replaces it. List
  what else reads or writes the same state.
- Before we move to design, I describe in my own words where the change goes,
  what calls it, and what state it touches. Correct me by pointing at the hop
  I got wrong. If I can't describe it yet, we keep going.
- Design: I drive it. Give me options with their trade-offs and I decide.
- Before coding, list the slices as test names. One slice is one test, or one
  logical branch, done as one red→green→refactor cycle (tdd skill).
- For each slice in the list, I mark who writes the test and who writes the
  implementation: me or you. Wait for my part before finishing the slice.
- After each slice, stop. Tell me what changed and what to look at first.
  I review it and commit it before we start the next one.
- Change only what we agreed on. If you notice a cleanup, mention it instead
  of doing it.
- If the implementation drifts from the ticket, comment on the ticket, then
  continue.
- When all slices are in, open the PR with /stacked-pr, ready for review,
  not a draft. Then stop.
