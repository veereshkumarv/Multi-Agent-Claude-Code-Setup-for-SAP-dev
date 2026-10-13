---
name: sap-retro
description: Summarize what each role flagged during the last /sap-ship cycle and how it was resolved, producing RETRO.md. Use after a ship cycle completes, when the user wants a retrospective on how the pipeline went, e.g. "/sap-retro".
---

This skill does not invoke other agents — it synthesizes what already happened in the most recent
`/sap-plan` and `/sap-ship` cycles directly.

1. Read `SPEC.md`, `PLAN.md`, and `SHIP-REPORT.md` if present, plus any git history for this
   feature (`git log` on recently changed files).
2. For each stage that ran (product, exec, architecture, UX, devex, code review, QA, security,
   release), note: what it flagged, whether it was resolved before shipping or deferred, and
   whether resolving it earlier (e.g. in `/sap-plan` instead of `/sap-ship`) would have saved
   rework.
3. Identify 1-3 concrete process changes for next time — e.g. "architecture review missed the
   authorization annotations that security review later caught; architecture-reviewer should check
   this going forward."
4. Write `RETRO.md` with sections: What Shipped, What Each Stage Caught, What Could Move Earlier,
   Action Items.

Keep it short and concrete — this is meant to improve the next cycle, not to document history for
its own sake.
