# SAP Project — Multi-Agent Development Setup

> Template scaffold for SAP application development (SAPUI5/Fiori, CAP, and/or ABAP/RAP), using a
> gstack-style roster of specialized agents wired into pipelines. Replace this paragraph with a
> real project description once the app is defined. Trim the CAP or ABAP sections below once the
> backend stack is decided.

## Stack conventions (apply regardless of role)

- UI5 apps: prefer TypeScript. Never use global `sap.*` access — use `sap.ui.define`/ES imports.
  Always use data binding, never inline `<script>`, always use `ComponentSupport` to bootstrap.
- Forms: use `sap.ui.layout.form.Form` + `ColumnLayout` (2/3/4 columns at M/L/XL) — never
  `SimpleForm` unless explicitly requested.
- Event handlers (UI5 >= 1.115.0, TypeScript): import the specific `<Control>$<Event>Event` type,
  not the generic `sap/ui/base/Event`.
- **CAP projects**: UI5 apps live under `app/` in the CAP project root. Install
  `cds-plugin-ui5` in the CAP root. Run everything via `cds watch` from the CAP root — never a
  separate `ui5 serve`/`npm start`, never a `ui5-middleware-simpleproxy`. Use CDS tooling (or
  `cds compile '*'` / `cds compile '*' --to serviceinfo`) to inspect services/entities before
  building UI on top of them.
- **ABAP projects**: backend changes go through RAP/BOPF business objects exposed as OData V4
  services; UI5 apps consume them the same way as CAP-exposed services.
- Before any UI5-specific work, call the `get_guidelines` MCP tool once per session. Use
  `get_api_reference` for control/API lookups and `run_ui5_linter` / `run_manifest_validation`
  before considering UI5 work done.

## Agent roster

| Role | Agent file | Invoked by | Purpose |
|---|---|---|---|
| Product Interrogator | `.claude/agents/product-interrogator.md` | `/sap-plan` | Pressure-tests a feature request into a clear `SPEC.md` |
| Exec Reviewer | `.claude/agents/exec-reviewer.md` | `/sap-plan` | Business value / scope / buy-vs-build check on the spec |
| Architecture Reviewer | `.claude/agents/architecture-reviewer.md` | `/sap-plan` | CAP/ABAP/UI5 data flow & design review |
| UX Reviewer | `.claude/agents/ux-reviewer.md` | `/sap-plan` | Fiori guidelines + UI5 best-practice + a11y audit |
| DevEx Reviewer | `.claude/agents/devex-reviewer.md` | `/sap-plan` | Flags build/test/tooling friction in the plan |
| Implementer | `.claude/agents/implementer.md` | manual / ad hoc | Writes the actual UI5/CAP/ABAP code against an approved plan |
| Code Reviewer | `.claude/agents/code-reviewer.md` | `/sap-ship` | Staff-engineer correctness/robustness pass |
| QA Tester | `.claude/agents/qa-tester.md` | `/sap-ship` | OPA5/QUnit + manual exploratory testing |
| Security Reviewer | `.claude/agents/security-reviewer.md` | `/sap-ship` | OWASP/STRIDE + SAP-specific (XSUAA, CSRF, destinations) |
| Release Engineer | `.claude/agents/release-engineer.md` | `/sap-ship` | Verifies tests/lint/manifest, assembles release notes, opens PR |
| Deploy Coordinator | `.claude/agents/deploy-coordinator.md` | manual, after `/sap-ship` | Prepares BTP/CF/MTA deploy steps; never pushes without explicit confirmation |
| Doc Writer | `.claude/agents/doc-writer.md` | `/sap-ship` or manual | Keeps docs in sync with shipped behavior |

## Pipelines

- `/sap-plan "<feature request>"` — runs Product Interrogator → Exec Reviewer → Architecture
  Reviewer → UX Reviewer → DevEx Reviewer in sequence, producing/updating `PLAN.md`. Stops to ask
  the user about any open decision before implementation starts.
- `/sap-ship` — runs Code Reviewer → QA Tester → Security Reviewer → Release Engineer (+ Doc
  Writer) over the current changes, producing `SHIP-REPORT.md`. Does **not** deploy — hand off to
  Deploy Coordinator manually once you're ready to actually push to an environment.
- `/sap-retro` — summarizes what each role flagged during the last `/sap-ship` cycle and how it
  was resolved, into `RETRO.md`.

## Safety

Destructive commands (`rm -rf`, `DROP TABLE`/`DROP SCHEMA`, `cf delete*`/`cf unbind*`,
`git push --force`) are flagged by a `PreToolUse` hook (see `.claude/settings.json`) — treat any
warning from it as a prompt to double-check before proceeding, not something to bypass.
