---
name: Always rebuild after tex changes
description: Run make after every tex edit and tail 5 lines to check for build errors
type: feedback
---

Always run `make` after making changes to any .tex file, and tail ~5 lines of output to catch build errors.

**Why:** User wants to ensure the document still compiles after every change, catching errors immediately rather than discovering them later.

**How to apply:** After any edit to .tex files in the thesis project, run `make 2>&1 | tail -5` to verify the build succeeds.
