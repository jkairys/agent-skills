---
name: save-intent
description: Compress a discovery conversation into a structured intent.md at the path provided. Use after grilling has reached shared understanding, to write the durable bridge artifact for engineering. Do not use mid-grilling.
---

Compress the discovery conversation that just happened into the file at the path the orchestrator specifies (typically `specs/<feature-name>/intent.md`). Use the template at @templates/intent.md.

# Compression rules

- Skip dead-ends. If grilling explored an option then rejected it, only mention the rejection if the rationale is non-obvious.
- Skip transitional turns ("good question", "let me check"). Capture decisions and rationale only.
- Aim for 30-80 lines total. If longer, you're not compressing enough.
- The reader is an engineer who wasn't in the conversation. They should be able to brief implementation work from this alone.

# Sections (follow the template exactly)

- **What** — one paragraph in plain language. The feature, in the user's voice.
- **Why** — whose problem this solves, the user need. One paragraph.
- **Key decisions** — bullets, each formatted as `**<Decision>**: <summary>. Rationale: <why this won>.` Include codebase findings as decisions (e.g. "Extend `UserService` rather than create a new one because grilling found `UserService` already handles the related concerns").
- **Open questions** — deliberately deferred. Mark `→ engineer` if engineering should resolve, `→ user` if user input is still needed.
- **Out of scope** — explicit non-goals surfaced during grilling. Prevents scope creep in engineering.
- **Related** — code paths, prior specs, external refs.

# Output

Write the file. Then return:

- The file path
- A 2-line summary the orchestrator can use for Gate 1
