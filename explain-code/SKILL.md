---
name: explain-code
description: >
  How to explain code to me. Use when explaining how code works, tracing a
  bug or a change, or pointing me at code to read.
---

I understand code by following it from where it starts.

- Start from the entrypoint: what a person or system actually touched
  (a URL, a click, a command, a job, a request), not the first function
  someone mentioned.
- Show the route: entrypoint → each hop → the line that matters, with the
  file and function at every hop. A bare file:line doesn't help me, because
  I can't tell what reaches it.
- A ticket's file:line list is where someone ended up, not where it starts.
  Find the entrypoint and trace forward to those lines.
- Order the explanation along that path, so each part only needs what came
  before it.
- When you tell me to read something, say what I'm looking for and why that
  file is on the path.
