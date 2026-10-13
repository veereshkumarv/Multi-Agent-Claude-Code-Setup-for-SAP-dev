---
name: sap-plan
description: Run the full planning pipeline for a new SAP feature request (product -> exec -> architecture -> ux -> devex review), producing PLAN.md. Use when the user describes a new feature/app to build and wants it planned before implementation, e.g. "/sap-plan add a vendor approval screen".
---

Run this pipeline sequentially — each stage reads the output of the previous one. Do not
parallelize these; each review depends on the prior stage's conclusions.

1. Invoke the `product-interrogator` agent with the user's feature request. It will ask
   clarifying questions (via AskUserQuestion) and write `SPEC.md`.
2. Invoke the `exec-reviewer` agent to review `SPEC.md`. It appends an Exec Review section.
   If its verdict is "reconsider", stop and surface that to the user before continuing.
3. Invoke the `architecture-reviewer` agent to design the technical approach based on the
   reviewed spec. It creates/appends to `PLAN.md` with an Architecture section.
4. Invoke the `ux-reviewer` agent to audit the planned screens/flows against Fiori guidelines.
   It appends a UX Review section to `PLAN.md`.
5. Invoke the `devex-reviewer` agent to flag any build/test/tooling friction and confirm the
   plan is sequenced for incremental verification. It appends a DevEx Review section.

After all five stages, read `PLAN.md` yourself, summarize the final plan in 3-5 sentences to the
user, and surface any open questions or blockers raised by any stage via AskUserQuestion before
treating the plan as approved for implementation (hand off to the `implementer` agent).
