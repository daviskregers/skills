---
name: documents
description: >
  How to write documents I'll read: plans, reviews, longer write-ups, PR and
  issue descriptions, diagrams and tables. Use when writing any of them.
---

- Write plans, reviews and other longer write-ups I'll come back to into a
  file in ./.dk-notes/{current-project}/, not into the chat. In chat, tell me what
  you wrote and what to look at first.
- In PR and issue descriptions, keep the part for people short and put the
  detailed evidence (file references, tool output, ruled-out options) in a
  collapsible AI Context section at the end.
- For diagrams use a Mermaid `flowchart TD` with a subgraph per boundary,
  not Mermaid's C4 diagrams: they have no layout, so arrows and labels land
  on top of each other. Keep it under ~12 nodes and edge labels to ~3 words.
  Say what each box is in its text, like `API<br/>[Service]`.
- Add a diagram only when the thing spans more than one component. A
  diagram is never the whole answer: after it, walk through what happens at
  each step (for code, the file and function).
- Keep tables narrow: at most 4 columns and rows under ~120 characters, or
  they wrap in my editor. Longer detail goes under the table. Never a table
  of sentences.
