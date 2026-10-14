# SAP Project — Multi-Agent Claude Code Setup

This repo ships a [Claude Code](https://claude.com/claude-code) setup that gives every SAP
developer on the team the same roster of specialized agents (product, architecture, UX, QA,
security, release, deploy, docs) wired into two pipelines: `/sap-plan` and `/sap-ship`. See
`CLAUDE.md` for the full agent roster and stack conventions.

Every agent's checklist is grounded in
[`docs/sap-fullstack-practical-guide.md`](docs/sap-fullstack-practical-guide.md) — the SAP
Full-Stack Development Practical Guide v4.0 (the original `.docx` is also kept in the repo root
for reference). If you're new to the team, skim it once: it covers the RAP-vs-CAP decision
framework, clean-core extensibility levels (A/B/C/D), domain/service design rules, the test
pyramid, and release/operations checklists that every agent below cites by section number.

Not using Claude Code? The same guide, role checklists, and pipeline steps are also available as
an [MCP server](https://github.com/veereshkumarv/sap-fullstack-mcp-server) that any MCP-capable AI
harness (Claude Desktop, Cursor, your own agent loop) can connect to.

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

## The practical guide (`docs/sap-fullstack-practical-guide.md`)

This is the source of truth behind every agent's checklist — if an agent's behavior and the guide
ever disagree, the guide wins and the agent file should be fixed, not worked around. A few things
worth knowing up front:

- **Normative language**: the guide uses **MUST** (mandatory, no merge/release without an
  approved exception), **SHOULD** (expected default; deviation needs a written reason), and
  **MAY** (optional).
- **Clean-core levels**, used throughout architecture/security/release review: **Level A**
  (released API/extension point, preferred), **Level B** (SAP-classified classic API), **Level C**
  (internal object, exception only, needs an owner), **Level D** (modifications/implicit
  enhancements/unsupported direct writes — forbidden for new work, and an automatic blocker in
  security review).
- It's organized by SDLC phase (Discover → Choose Architecture → Design Contracts → Engineering
  Foundation → Build RAP/CAP/Fiori → Integrate S/4HANA → Verify → Release/Deploy →
  Operate/Retire) — each agent references the specific section(s) it draws from (e.g.
  `exec-reviewer` cites §1.2 and §4.4, `security-reviewer` cites §10-11).
- If you update the guide (new version, corrected guidance), re-check the agent files that cite
  the changed section numbers so they don't drift out of sync.

## Customizing

- Add a new role: create `.claude/agents/<role>.md` following the existing frontmatter pattern
  (`name`, `description`, `tools`, `model`) and reference it from the roster table in `CLAUDE.md`.
- Add a new pipeline: create `.claude/skills/<pipeline-name>/SKILL.md` describing which agents to
  invoke and in what order.
- Tighten or loosen the destructive-command guard: edit the hook in `.claude/settings.json`.

Keep changes to this scaffold in their own commit/PR so the team can review process changes
separately from actual app code.
