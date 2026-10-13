# SAP Project — Multi-Agent Claude Code Setup

This repo ships a [Claude Code](https://claude.com/claude-code) setup that gives every SAP
developer on the team the same roster of specialized agents (product, architecture, UX, QA,
security, release, deploy, docs) wired into two pipelines: `/sap-plan` and `/sap-ship`. See
`CLAUDE.md` for the full agent roster and stack conventions.

## Getting started (new team members)

1. **Install Claude Code**, if you haven't already — see https://claude.com/claude-code.
2. **Clone this repo** and open Claude Code with it as your working directory:
   ```
   git clone https://github.com/veereshkumarv/Multi-Agent-Claude-Code-Setup-for-SAP-dev.git sap-project
   cd sap-project
   claude
   ```
   Nothing to configure — Claude Code automatically picks up `CLAUDE.md`,
   `.claude/agents/`, `.claude/skills/`, and `.claude/settings.json` from the repo root the
   moment you open a session here.
3. **Install the UI5 MCP plugin** if your Claude Code install doesn't already have it (several
   agents call `get_guidelines`, `get_api_reference`, `run_ui5_linter`, and
   `run_manifest_validation` from it). Ask a teammate or check your Claude Code plugin settings if
   you're not sure it's installed.
4. **Drop your actual UI5/CAP/ABAP project into this repo** (or point Claude Code at your
   existing project directory and copy the `.claude/` folder and `CLAUDE.md` into it) — this
   scaffold is stack-agnostic until you add real app code.

## Day-to-day usage

- Starting a new feature: `/sap-plan "<describe the feature>"` — runs the planning pipeline
  (product → exec → architecture → UX → devex review) and produces `PLAN.md`.
- Implementing: once a plan is approved, ask Claude to implement it — it will use the
  `implementer` agent automatically (or invoke it by name: "use the implementer agent to build
  this").
- Readying for release: `/sap-ship` — runs code review → QA → security review → release
  engineering → docs, producing `SHIP-REPORT.md`. This never deploys on its own.
- Deploying: only on explicit request ("use deploy-coordinator to deploy to dev") — it always
  confirms the exact command before running it.
- After a ship cycle: `/sap-retro` summarizes what each stage caught, into `RETRO.md`.

## Conventions

All agents follow the SAP/UI5 stack conventions documented in `CLAUDE.md` (JavaScript over
TypeScript, CAP folder layout, form/control guidelines, etc.). If your team's conventions change,
update `CLAUDE.md` — every agent reads it as shared context, so you only need to change it in one
place.

## Customizing

- Add a new role: create `.claude/agents/<role>.md` following the existing frontmatter pattern
  (`name`, `description`, `tools`, `model`) and reference it from the roster table in `CLAUDE.md`.
- Add a new pipeline: create `.claude/skills/<pipeline-name>/SKILL.md` describing which agents to
  invoke and in what order.
- Tighten or loosen the destructive-command guard: edit the hook in `.claude/settings.json`.

Keep changes to this scaffold in their own commit/PR so the team can review process changes
separately from actual app code.
