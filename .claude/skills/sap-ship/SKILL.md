---
name: sap-ship
description: Run the full pre-release pipeline (code review -> QA -> security review -> release engineering -> docs) over the current changes, producing SHIP-REPORT.md. Use when a feature is functionally complete and the user wants it readied for release, e.g. "/sap-ship" or "ship this feature".
---

Run this pipeline sequentially — each stage should only proceed once the prior stage's blockers
are resolved (or explicitly accepted by the user as deferred).

1. Invoke the `code-reviewer` agent over the current diff/changes. If it reports blockers, stop
   and report them to the user — do not continue to QA with open blockers unless the user
   explicitly says to proceed anyway.
2. Invoke the `qa-tester` agent to write/run automated tests and manually exercise the feature.
   Bugs found go back to the user (and implementer, if they want fixes made now) before continuing.
3. Invoke the `security-reviewer` agent for an OWASP/STRIDE + SAP-specific audit. Blockers here
   are treated the same way as step 1.
4. Invoke the `release-engineer` agent to verify tests/lint/manifest validation all pass and
   assemble `SHIP-REPORT.md`.
5. Invoke the `doc-writer` agent to bring README/CHANGELOG/app docs in line with what actually
   shipped.

This pipeline stops short of deployment. After it completes, tell the user `SHIP-REPORT.md` is
ready and that the `deploy-coordinator` agent is available on explicit request for the next step —
do not invoke it automatically.
