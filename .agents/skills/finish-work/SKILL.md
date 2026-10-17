---
name: finish-work
description: Validate an Issue change, arrange independent fixed-revision review, and finish or hand off integration and owned-resource cleanup. Review-only requests do not perform writer actions.
---

# Finish work

Use docs/workflow.md and the Issue acceptance; for context changes also docs/context.md. Review-only requests return findings/evidence without new Issues or writer mutations.

Run `python3 scripts/check.py` for the harness and relevant real project checks, using docs/workflow.md's existing-hook conditions to avoid duplicate execution. Include mixed/unknown impact and explicit build/GUI requirements; a skip is not a test pass. For instruction/method changes, reconcile active consumers under docs/context.md while preserving historical evidence.

For CI-related changes, review docs/setup/ci.md's measurement/reuse/omission decision, preserved guarantees, actual results and regression/rollback owner. A new document or passing check alone does not establish operational improvement.

Freeze HEAD/base and obtain separate-session review. Self-review or a script declaring success is not independent approval. Resolve concrete defects before integration. New sessions must satisfy actual user authorization and client execution policy.

Before stopping or handing off unfinished work, apply docs/workflow.md's bounded stop assessment; immediate human/safety/permission/resource stops take precedence. On handoff stop the writer and record owner, SHA/base, dirty state, checks, remaining acceptance and next action. Keep paths/hosts/tokens private. Do not overwrite registry or resume PAUSED jobs. Use prompts/coordinator.md only for enrolled work with a real execution route.

Read current CI, PR merge and Issue acceptance back. Close only completed scope; transfer pending verification with bidirectional links, steps, expectations and owner. Merge does not prove release, GUI pass or cleanup. Remove only owned, stopped, clean resources after checking remote/local/tracking refs and worktree use. Never delete main/master/develop. Record retained resources and resumption conditions.
