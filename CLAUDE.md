# SAP Project — Multi-Agent Development Setup

> Template scaffold for SAP application development (SAPUI5/Fiori, CAP, and/or ABAP/RAP), using  specialized agents wired into 
> pipelines. Replace this paragraph with a real project description once the app is defined. Trim the CAP or ABAP sections below 
> once the backend stack is decided.

## Primary reference

Every role below is grounded in
[`docs/sap-fullstack-practical-guide.md`](docs/sap-fullstack-practical-guide.md) — the SAP
Full-Stack Development Practical Guide v4.0. If an agent's checklist and the guide ever disagree,
the guide wins; update the agent, don't work around it. The guide uses normative language that
applies across every role:

- **MUST** — mandatory; a change that breaks the rule does not merge or release without an
  approved exception.
- **SHOULD** — expected approach; deviation needs a written reason and evidence the alternative
  is safe.
- **MAY** — optional; use only when it adds value without unnecessary complexity.

It also defines clean-core extensibility levels used throughout architecture, security, and
release review: **Level A** (released API/extension point — preferred), **Level B** (SAP-classified
classic API, when Level A doesn't cover it), **Level C** (internal object, exception only, with an
owner and remediation path), **Level D** (modifications/implicit enhancements/unsupported direct
writes — forbidden for new work).

## Stack conventions (apply regardless of role)

- UI5 apps: prefer JavaScript. Never use global `sap.*` access — use `sap.ui.define`/ES imports.
  Always use data binding, never inline `<script>`, always use `ComponentSupport` to bootstrap.
- Forms: use `sap.ui.layout.form.Form` + `ColumnLayout` (2/3/4 columns at M/L/XL) — never
  `SimpleForm` unless explicitly requested.
- Event handlers (JavaScript): use the generic `sap/ui/base/Event`; read parameters via
  `event.getParameter("...")` with the parameter name from the control's API reference
  (`get_api_reference`).
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
