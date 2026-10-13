# SAP Full-Stack Development Practical Guide

RAP • CAP with Node.js • SAPUI5 • Fiori elements • SAP BTP
A guided SDLC handbook for developers, reviewers, technical leads and architects.
Version 4.0 | October 2026 | Learning and delivery edition | Owner: SAP Engineering Practice

> This is the canonical reference every agent in `.claude/agents/` is grounded in. If an agent's
> checklist and this document ever disagree, this document wins — update the agent, not the other
> way around.

## Document Control

| Field | Value |
|---|---|
| Title | SAP Full-Stack Development Practical Guide |
| Version | 4.0 |
| Effective date | October 2026 |
| Audience | Junior and senior developers, reviewers, technical leads, architects and delivery stakeholders |
| Scope | RAP, CAP Node.js, JavaScript, SAPUI5, Fiori elements, SAP HANA Cloud, SAP BTP Cloud Foundry and S/4HANA integration |
| Review cycle | Annual and after material SAP platform, security or clean-core changes |
| Project supplement | Add target versions, naming rules, SLAs, approved services, security classification and exception owners |

## 1. How to Read and Apply This Guide

A full-stack SAP application is not just a user interface plus an OData service. It is a chain of
decisions about transaction ownership, data contracts, user experience, authorisation,
persistence, integration, deployment and support. A weak decision early in the project usually
appears later as difficult testing, upgrade risk or production instability. This guide therefore
follows the order in which those decisions should be made.

Junior developers should read the lifecycle chapters in sequence for a new project. Experienced
developers can use the decision sections as reference points. Architects and reviewers should use
the phase exit criteria to judge whether a project is ready to move forward.

> **Working rule** — Do not start with "Which technology do I want to use?" Start with "Who owns
> the business transaction and data, who consumes it, and what must remain stable over time?"

### 1.1 Normative language

- **MUST** is mandatory. A change that breaks the rule does not merge or release without an
  approved exception.
- **SHOULD** is the expected approach. Deviation needs a written reason and evidence that the
  alternative is safe.
- **MAY** is optional. Use it only when it adds value and does not introduce unnecessary
  complexity.

### 1.2 When a junior developer should escalate

The guide is intended to make everyday engineering decisions self-service. Escalation is still
appropriate when the decision changes ownership, security, data integrity or operational
exposure.

| Escalate when | Why specialist review is needed |
|---|---|
| A standard S/4HANA table or internal object is proposed | This affects clean-core classification and upgrade risk. |
| A BAPI or legacy API commits internally | This can break RAP save-sequence consistency. |
| A new multitenant SaaS design is proposed | Tenant isolation, subscription and upgrade lifecycle are architectural concerns. |
| Personal, payroll, health or highly restricted data is involved | Privacy, retention, masking and audit rules require security and data-owner input. |
| A public endpoint, anonymous route or internet-facing integration is required | Threat modelling and platform security controls must be reviewed. |
| A microservice split is proposed | Service boundaries add deployment, consistency, monitoring and support cost. |
| Performance remains outside agreed targets after evidence-based tuning | Database, UI and platform specialists may need to interpret traces and plans. |

## 2. The SAP Full-Stack Mental Model

A useful mental model is to separate the system into responsibilities. Each responsibility should
have one clear owner. When responsibilities are mixed, the same rule is often reimplemented in
several places.

| Responsibility | Where it normally belongs | Why |
|---|---|---|
| Business transaction and invariants | RAP when the transaction belongs to S/4HANA; CAP domain/service layer for side-by-side capability | The system of record must remain authoritative. |
| Consumer contract | RAP projection/service or CAP service projection | Consumers should not depend on persistence details. |
| User interaction | Fiori elements or SAPUI5 | The UI coordinates interaction but does not enforce business security. |
| Authentication | SAP BTP identity and application-router services | Applications should not implement passwords, sessions or cryptography. |
| Authorisation | RAP DCL/behaviour authorisation or CAP @requires/@restrict | Authorisation must be enforced server-side. |
| Persistence | S/4HANA-owned objects for RAP; SAP HANA Cloud for CAP-owned data | Ownership determines transaction and lifecycle. |
| Integration | Released APIs, destinations and events | Stable interfaces reduce coupling and support clean core. |
| Operations | Cloud Foundry/Kyma platform controls, logs, metrics and runbooks | The application must be supportable after development ends. |

```
User
  -> Fiori elements or SAPUI5
  -> Application Router / service binding
  -> RAP or CAP service contract
  -> Authorisation and business behaviour
  -> Owned persistence or released remote API
  -> Structured messages and targeted UI refresh
```

## 3. Phase 1: Discover the Problem Before Designing the Solution

**Why this phase matters** — Discovery prevents the team from solving the wrong problem or
reproducing a standard SAP capability in custom code. It also reveals the system of record, user
roles and non-functional constraints that drive the framework decision.

**When it is complete** — A product owner can describe the in-scope business outcome, users, data
ownership, exceptions and success measures without relying on a proposed technology.

**How to work through it** — Run short workshops with business, process, security, data and
support representatives. Capture facts and open decisions separately. Validate whether a standard
SAP application or configuration already covers the need.

**Expected output** — Problem statement, process boundary, user/persona list, data inventory,
integration inventory, preliminary NFRs and fit-gap record.

### 3.1 Questions to answer

- What business outcome changes if the application is delivered?
- Which users perform the process, and what may each role view or change?
- Which system is authoritative for each business object?
- Is the process an extension of an S/4HANA transaction, or a new cross-system capability?
- How many records, users and requests are expected at normal and peak load?
- What happens if S/4HANA, a remote API or SAP HANA Cloud is temporarily unavailable?
- What personal, confidential or regulated information is processed?
- Can the requirement be met by a standard SAP application, in-app extensibility or SAP Build
  configuration?

### 3.2 Worked example: framing the capability

Suppose a team asks for "a supplier approval application." That phrase does not identify the
transaction owner. Discovery should reveal whether the application approves a supplier master
change inside S/4HANA, orchestrates risk information from several systems, or merely provides a
new screen over existing data. The first is likely RAP, the second is likely CAP, and the third
may require no custom application at all.

> **Phase exit test** — If the team cannot identify data ownership and the authoritative
> transaction, do not select RAP or CAP yet.

## 4. Phase 2: Choose the Architecture and Framework

**Why this phase matters** — Framework selection determines transaction semantics, deployment
lifecycle, clean-core exposure, developer skills and operational ownership. Reversing the
decision after the data and service model are built is expensive.

**When it is complete** — The team can explain the selected framework using business ownership
and lifecycle facts, not personal preference.

**How to work through it** — Use the decision path below, record an Architecture Decision Record
(ADR), and test uncertain assumptions with a small technical spike.

**Expected output** — Approved framework, UI, persistence, integration, tenant and deployment
decisions.

### 4.1 RAP or CAP: decision path

Ask where the business transaction belongs. If it is an S/4HANA-owned object and must participate
in the S/4HANA logical unit of work, start with RAP. If the capability combines SAP and non-SAP
systems, requires independent deployment, uses BTP services, or owns its own side-by-side data,
start with CAP. If both are true, use a hybrid design: keep S/4HANA rules in RAP and cross-system
orchestration in CAP. If neither is clear, return to discovery — technology cannot resolve unknown
ownership.

| Signal | RAP is stronger when | CAP is stronger when |
|---|---|---|
| Transaction ownership | S/4HANA owns the business object and save sequence. | The application owns a side-by-side capability or orchestrates systems. |
| Data | Primary data is in S/4HANA and follows SAP LUW semantics. | Application data is in SAP HANA Cloud or gathered from several sources. |
| Lifecycle | Release follows S/4HANA transport and upgrade cadence. | Release must be independent of S/4HANA. |
| Platform services | Limited need outside ABAP platform. | Needs messaging, AI, document, workflow, multitenancy or Node.js libraries. |
| Integration | Mostly within S/4HANA. | Multiple SAP/non-SAP APIs and events. |
| Clean core | ABAP Cloud and released objects on-stack. | Side-by-side decoupling through released APIs/events. |

### 4.2 Managed RAP, unmanaged save or unmanaged RAP

Managed RAP is the default because the framework supplies standard interaction, buffering and
save behaviour. The developer then concentrates on business rules. Unmanaged RAP transfers much
more responsibility to the developer, including buffer, CRUD, locks and save handling. Use that
responsibility only when existing business logic genuinely requires it.

| Situation | Choose | Reason |
|---|---|---|
| New custom BO with application-owned tables | Managed RAP | Standard CRUD, locks, ETags and draft can be provided by the framework. |
| Managed interaction but final write must use an approved API/BAPI | Managed with unmanaged save | Keep framework-managed interaction while delegating final persistence, after proving save compatibility. |
| Legacy BO owns read, lock, numbering, buffer and save | Unmanaged RAP | The existing transaction cannot be represented safely by the managed provider. |
| Read-only database-backed report | CDS query/read-only service | No transactional behaviour is needed. |
| Read-only non-database source | Unmanaged query/custom entity | Use only when standard SQL pushdown cannot serve the source. |

> Before selecting managed with unmanaged save for a BAPI, confirm whether the API performs an
> internal commit, how late numbering is returned, how partial failures are mapped, and whether
> the complete save can be rolled back. If those answers are uncertain, a technical spike and
> senior ABAP review are required.

### 4.3 Fiori elements or freestyle SAPUI5

Fiori elements is not a "low-code" compromise. It is the preferred way to build common enterprise
experiences because the service metadata drives consistent floorplans, messages, navigation and
adaptation. Freestyle SAPUI5 is appropriate when the interaction itself is genuinely specialised.

| Question | Choose Fiori elements when | Choose freestyle when |
|---|---|---|
| Page pattern | List Report, Object Page, worklist or analytical floorplan fits. | The interaction is canvas-like, highly visual or cannot be assembled from supported building blocks. |
| Customisation | A few custom actions, sections or building blocks are sufficient. | Most of the page behaviour must be custom. |
| Maintenance | Consistency and upgrade efficiency are priorities. | The project accepts full accessibility, performance and regression ownership. |
| Team skill | Team understands OData annotations and Fiori patterns. | Team has strong SAPUI5 and web engineering skills. |

### 4.4 Other decisions beginners often overlook

| Decision | Default | Change the default only when |
|---|---|---|
| OData version | V4 | A target release or existing contract prevents V4. |
| CAP handler | Generic provider | A real domain command or orchestration is required. |
| Service decomposition | Modular monolith | Independent ownership, scale and failure boundaries are proven. |
| Integration style | Synchronous API | Eventual result is acceptable and fan-out/recovery is designed. |
| Tenant model | Single tenant for client-specific app | The solution is a SaaS product with tenant subscription and isolation lifecycle. |
| Application router | Managed router | Custom runtime routing/session behavior cannot be met otherwise. |

> **Decision record template** — Context: what facts drive the decision? Options: what
> alternatives were considered? Decision: what was selected? Consequences: what becomes easier or
> harder? Evidence: what spike, documentation or measurement supports it? Review trigger: what
> future change would reopen it?

## 5. Phase 3: Design the Domain, Service and Data Contracts

**Why this phase matters** — A good design makes business meaning explicit before code grows. It
keeps consumers independent from persistence and prevents UI-specific decisions leaking into the
core model.

**When it is complete** — Entities, keys, lifecycle ownership, service operations, authorisation,
error semantics and data migration are understood before feature implementation starts.

**How to work through it** — Model business language with CDS, review with process and data
owners, then create consumer-specific projections. Walk through create, read, change, delete and
failure scenarios.

**Expected output** — Domain model, service contract, behaviour design, role matrix, error
catalogue and migration plan.

### 5.1 Model the business, not the screen

Screens change more often than the business meaning of data. A field named "InputBox3" or a model
that copies every column from a source table is not a domain model. Use business names,
deliberate types and explicit lifecycle relationships.

- Use compositions when the child exists only as part of the parent lifecycle.
- Use associations when the target has its own lifecycle and is referenced.
- Choose immutable technical keys for identity; keep mutable business numbers as alternate
  identifiers where needed.
- Define length, precision, scale, mandatory status and valid ranges deliberately.
- Keep UI annotations in consumption/service layers rather than reusable interface models.

### 5.2 Service contract design

A service is a promise to consumers. Expose only what the consumer needs, use actions for
business commands and preserve compatibility. Persistence entities are implementation details;
projections are the contract boundary.

```cds
using { cuid, managed } from '@sap/cds/common';
namespace sample.orders;

entity Orders : cuid, managed {
  orderNumber : String(20) not null;
  status      : String(20) default 'DRAFT';
  currency    : String(3);
  total       : Decimal(15,2);
  items       : Composition of many Items on items.parent = $self;
}

entity Items : cuid {
  parent   : Association to Orders not null;
  product  : String(40) not null;
  quantity : Decimal(13,3);
  unit     : String(3);
}
```

```cds
using { sample.orders as db } from '../db/schema';

service OrderService @(requires: 'authenticated-user') {
  @restrict: [
    { grant: 'READ', to: 'OrderViewer' },
    { grant: '*', to: 'OrderManager' }
  ]
  entity Orders as projection on db.Orders;
  action approve(ID: UUID) returns Orders;
}
```

In this neutral example, the domain model owns persistence and the service projection controls
exposure. The `approve` operation is an action because approval is an explicit business command
with side effects.

### 5.3 Design authorisation before UI

Start from business roles and data boundaries. For each entity and action, define who can read,
create, change, delete or execute it, and whether access depends on company code, sales
organisation, plant, owner or tenant. The UI may hide unavailable actions, but the backend must
reject unauthorised requests.

### 5.4 Design errors as part of the contract

A useful error tells the consumer what failed, where it failed and what can be done next. Use
stable error keys, a target field or entity where possible, and a safe user-facing message. Log
diagnostic detail separately with a correlation ID. Do not return database messages, stack traces
or remote credentials.

## 6. Phase 4: Establish the Engineering Foundation

**Why this phase matters** — A project becomes difficult to improve once inconsistent structure
and manual quality practices spread. Establishing the baseline early makes the desired path the
easiest path.

**When it is complete** — A new developer can clone, configure and run the project using the
README; the first pipeline already runs lint, tests, build and security checks.

**How to work through it** — Generate supported projects, pin runtimes, create branch rules,
configure local and cloud profiles, and commit a working skeleton before building features.

**Expected output** — Repository, README, environment profiles, pipeline, initial deployment and
coding conventions.

### 6.1 RAP project baseline

- Use ABAP Development Tools and the ABAP language version required by the target clean-core
  model.
- Create packages with clear transport ownership and dependency direction.
- Configure central ATC variants before feature growth.
- Create the CDS root, behaviour definition, service definition/binding and a minimal test slice
  to prove the path.

### 6.2 CAP project baseline

```sh
cds init sample-app --nodejs
cd sample-app
cds add hana
cds add mta
npm install
cds watch
```

Use the Node.js LTS version supported by the project's CAP release. Commit the lockfile. Separate
local mocked configuration from production service bindings, and never commit production
credentials.

```
app/                         # Fiori elements / SAPUI5
db/                          # domain model and HANA artifacts
srv/                         # service contracts and logic
test/                        # unit, integration, auth, contract, E2E
package.json
mta.yaml
xs-security.json
README.md
```

### 6.3 First pipeline

```sh
npm ci
npx cds lint
npx ui5 lint
npx eslint app srv db test
npm test
gitleaks detect --source . --redact
cds build --production
mbt build
```

Add deeper SAST, software-composition analysis, image scanning and SBOM generation as the
delivery environment matures. The important point is to start with a working gate before the
codebase becomes large.

## 7. Phase 5A: Build with RAP

**Why this phase matters** — RAP provides a consistent business-object contract for transactional
applications and APIs. Using its framework semantics reduces custom transaction code and makes
behaviour predictable for Fiori and other consumers.

**When it is complete** — A vertical slice can create/read/change the BO, enforce authorisation
and validation, return messages correctly and pass unit/integration tests.

**How to work through it** — Build from CDS interface model to behaviour, projection and service.
Add one business rule at a time and verify through EML or the service binding.

**Expected output** — Working RAP BO, tests, service binding, UI annotations and traceable rules.

### 7.1 Layering explained

| Layer | What belongs here | What should not be here |
|---|---|---|
| Persistence | Application-owned tables or an approved API-owned save path. | Direct writes to SAP standard tables. |
| Interface/basic CDS | Reusable semantic fields and associations. | UI-only annotations and scenario-specific filters. |
| Composite CDS | Reusable joins or enrichment with clear semantics. | A pass-through layer with no meaning. |
| Projection/consumption | Consumer-specific fields, behaviours and annotations. | Unneeded internal or sensitive fields. |
| Behaviour | CRUD, actions, validations, determinations, locks, ETags and authorisation. | UI-only convenience logic. |
| Service definition/binding | Published contract and protocol. | Persistence exposure by accident. |

### 7.2 Decide where a rule belongs

| Rule type | Use | Example |
|---|---|---|
| Validation | Data may be rejected. | Amount must be positive before save. |
| Determination | A value is derived automatically. | Default status on creation. |
| Action | User explicitly requests a business transition. | Approve, reject or release. |
| Feature control | Availability depends on instance state. | Approve enabled only for submitted records. |
| Authorisation | User permission controls the operation. | Only managers may approve. |
| Side effect | Action changes fields the UI should reread. | Approval updates status and approver. |

### 7.3 Transaction and concurrency

ETags protect users from overwriting changes made by someone else. Locks coordinate transactional
changes. Draft supports longer edit sessions where users need to pause or resume. Choose these
based on the user journey, not because they are available. Do not issue `COMMIT WORK` or
`ROLLBACK WORK` inside behaviour handlers; RAP owns the transaction boundary.

### 7.4 Managed RAP example

```abap
managed implementation in class zbp_i_sampleorder unique;
strict ( 2 );
with draft;

define behavior for ZI_SampleOrder alias Order
persistent table zsample_order
draft table zsample_order_d
lock master total etag LastChangedAt
authorization master ( global, instance )
etag master LocalLastChangedAt
{
  create; update; delete;
  field ( readonly ) OrderUUID, CreatedAt, CreatedBy, LastChangedAt;
  validation validateAmount on save { field Amount, Currency; create; update; }
  determination setDefaults on modify { create; }
  action ( features : instance ) approve result [1] $self;
  draft action Edit;
  draft action Activate optimized;
  draft action Discard;
  draft action Resume;
}
```

The example shows intent, not a complete application. The framework owns standard persistence,
while the developer supplies rules and commands. Keep implementation methods set-based and map
failures to the affected instance.

## 8. Phase 5B: Build with CAP Node.js

**Why this phase matters** — CAP turns CDS models into service behaviour through generic
providers. The main design skill is knowing when the model is sufficient and when custom code
adds genuine domain value.

**When it is complete** — Standard CRUD works without custom handlers, business actions are thin
and testable, remote services are configured, and the application remains tenant-aware and
stateless.

**How to work through it** — Start with model and service projections. Add custom handlers only
for behaviour that cannot be expressed declaratively. Move complex decisions to pure domain
functions.

**Expected output** — CAP domain model, service contracts, thin handlers, integration adapters
and automated tests.

### 8.1 Understand generic providers

CAP can automatically serve standard CRUD, paging, filtering and OData behaviour from the model.
Rewriting this in `on READ` or `on CREATE` usually removes framework guarantees and creates extra
tests. Use custom handlers for business commands, orchestration or special integration behaviour,
not to prove that code can be written.

### 8.2 What belongs in each handler

| Handler | Use it for | Avoid |
|---|---|---|
| before | Request validation and defaults that depend on request context. | Long-running orchestration. |
| on | Custom actions/functions or deliberate replacement of generic handling. | Rewriting ordinary CRUD. |
| after | Presentation-only enrichment that does not affect filtering, sorting or paging. | Calculated list fields consumers must query. |
| error | Safe mapping and structured diagnostics. | Returning stack traces or raw upstream payloads. |

### 8.3 Thin-handler pattern

```js
const cds = require('@sap/cds');

module.exports = class OrderService extends cds.ApplicationService {
  async init() {
    const { Orders } = this.entities;

    this.before(['CREATE', 'UPDATE'], Orders, request => {
      if (request.data.total < 0) {
        request.error(400, 'TOTAL_MUST_BE_POSITIVE', 'total');
      }
    });

    this.on('approve', async request => {
      const tx = cds.tx(request);
      const order = await tx.run(SELECT.one.from(Orders).where({ ID: request.data.ID }));
      if (!order) request.reject(404);
      if (order.status !== 'SUBMITTED') request.reject(409, 'INVALID_STATE');
      await tx.run(UPDATE(Orders).set({ status: 'APPROVED' }).where({ ID: order.ID }));
      return { ...order, status: 'APPROVED' };
    });

    return super.init();
  }
};
```

The handler validates context, uses a request-bound transaction and performs one command. If
approval rules become complex, move the decision into a pure domain module and unit-test it
independently.

### 8.4 JavaScript rules that prevent production defects

- Use `async/await` consistently and await every operation that affects the response.
- Do not keep mutable request or user data in module-level variables; multiple requests and
  instances run concurrently.
- Connect to configured services through `cds.connect.to`; do not create a new ad-hoc client for
  every request.
- Use CQL/CQN with bound values; do not concatenate user input into SQL or expressions.
- Avoid synchronous file I/O and CPU-heavy loops in request paths.
- Centralise and validate configuration at startup rather than reading `process.env` throughout
  the code.

## 9. Phase 5C: Build the Fiori User Experience

**Why this phase matters** — The UI is where users experience the quality of the whole system,
but it should not become a second business-logic engine. A metadata-driven design keeps frontend
and backend behaviour aligned.

**When it is complete** — Critical journeys work with keyboard and assistive technology, OData
requests are bounded, messages are actionable and no backend security rule exists only in the UI.

**How to work through it** — Start from user tasks and Fiori patterns, implement
annotations/service metadata, then add the smallest supported extension needed. Validate network
and accessibility behaviour continuously.

**Expected output** — Fiori elements or SAPUI5 app, annotations, i18n, automated journeys and
accessibility evidence.

### 9.1 Fiori elements first

Use Fiori elements for standard list, object, analytical and worklist experiences. It provides
reusable floorplans and building blocks driven by OData metadata. Add supported custom sections
or actions only where the standard experience has a genuine gap. Choose freestyle SAPUI5 when the
interaction itself is specialised, not because the team is more familiar with controllers.

### 9.2 Modern SAPUI5 practices

- Use XML views/fragments, stable IDs, manifest-first configuration and i18n.
- Use `sap.ui.define` for declared dependencies and asynchronous loading for components, routes
  and fragments.
- Use only public APIs. Do not access underscore-prefixed members, generated IDs or UI5-managed
  DOM directly.
- Match the OData model to the service version. V2 model methods are not valid patterns for V4
  bindings.
- Use side effects or targeted binding refresh instead of refreshing the entire model.
- Destroy programmatically created dialogs, timers and global subscriptions in lifecycle hooks.

### 9.3 Draft, messages and side effects

Draft is useful for long or collaborative edit flows because an interim version can be saved
while the active version remains stable. It is unnecessary for a one-field quick update. For both
RAP and CAP, design the backend draft semantics first and let the OData V4/Fiori flow consume
them. Messages should identify the affected field or object, and actions should declare side
effects so the UI updates only what changed.

### 9.4 UI performance

Start with the browser Network and Performance tools. Count OData requests, compare payload sizes
and inspect long main-thread tasks. A slow page is often caused by an unbounded table, deep
`$expand`, repeated formatter work or broad refresh rather than a single slow backend method.
Build and deploy the optimised `dist` output with preload and minification.

Starting targets (engineering triggers, not universal promises): 50-100 rows per interactive
page, investigate browser tasks above 50ms, list pages normally below 2 seconds under agreed
conditions, explicit remediation when a save/action exceeds 5 seconds unless designed as
asynchronous work.

## 10. Phase 5D: Integrate S/4HANA and Other Systems Safely

**Why this phase matters** — Integration is where clean-core, transaction, security and
availability concerns meet. A technically reachable table is not automatically a valid business
interface.

**When it is complete** — Every dependency has an approved contract, authentication method,
timeout, failure behaviour, owner and clean-core level.

**How to work through it** — Evaluate released APIs first, then governed data products, then
approved classic APIs. Use federation only for bounded read-only cases. Design retries and events
around business semantics.

**Expected output** — Integration catalogue, destination/service bindings, clean-core record,
contract tests and resilience design.

### 10.1 Clean-core decision order

- **Level A** — Use a released API or extension point; classify the extension as Level A where
  possible.
- **Level B** — If Level A does not cover the requirement, evaluate SAP-classified classic APIs
  and recommended technologies.
- **Level C** — Use internal objects only as an exception, with upgrade checks, an owner and a
  remediation path.
- **Level D (forbidden)** — Do not build new solutions using modifications, implicit
  enhancements or unsupported direct writes to SAP tables.

### 10.2 API, event, virtual table or replication

| Need | Best starting choice | Why |
|---|---|---|
| Immediate authoritative business result | Synchronous released API | The caller needs success/failure now and business semantics remain in the source. |
| Fan-out after a completed fact | Event | Consumers can react independently and eventual consistency is acceptable. |
| Small current read-only dataset | Released API, then approved virtual table if necessary | Federation avoids copying but creates runtime source dependency. |
| Large or repeatedly scanned dataset | Replication/governed data product | Local data gives predictable performance and source independence. |
| Write to S/4HANA | Released transactional API/RAP/BAPI as approved | A virtual table must not bypass application logic. |
| Object in another local HDI container | Synonym plus grants | The object is local to the HANA landscape, not a remote federation case. |

### 10.3 Virtual table architecture

```
Approved source view
  -> SAP HANA Cloud remote source
  -> .hdbvirtualtable
  -> .hdbview mapping/restriction layer
  -> @cds.persistence.exists + @readonly facade
  -> authorised service projection with query limit
  -> Fiori elements / SAPUI5
```

A virtual table stores metadata locally and accesses the remote data at query time. That means
source latency and outages become application behaviour. Before approval, prove selective filter
pushdown, deterministic paging, acceptable source load and safe handling of schema drift. **Do
not expose a raw virtual table or standard S/4HANA table as OData.**

```sql
VIRTUAL TABLE "VT_REMOTE_REFERENCE"
AT "REMOTE_SOURCE"."REMOTE_DB"."REMOTE_SCHEMA"."APPROVED_REFERENCE_VIEW"
```

```cds
namespace sample.remote;

@cds.persistence.exists
@readonly
entity V_REMOTE_REFERENCE {
  key BusinessKey : String(40);
      Description : String(120);
      ChangedAt   : Timestamp;
}
```

```cds
using { sample.remote as remote } from '../db/remote-facade';

service ReferenceService @(requires: 'authenticated-user') {
  @readonly
  @cds.query.limit: { default: 100, max: 500 }
  @restrict: [{ grant: 'READ', to: 'ReferenceViewer' }]
  entity References as projection on remote.V_REMOTE_REFERENCE;
}
```

### 10.4 Remote calls and events

- Set connect/read timeouts and response-size limits.
- Retry only transient, safe or idempotent operations with bounded exponential backoff and
  jitter.
- Do not keep a database transaction open while waiting for a slow remote system unless the
  consistency design explicitly requires it.
- For events, include event ID, source, time, tenant/subject and schema version. Assume
  duplicates and possible reordering.
- Use a transactional outbox or equivalent if the database change and event intent must be
  atomic.

## 11. Phase 6: Verify Quality, Security and Performance

**Why this phase matters** — Testing is not only proof that the happy path works. It is evidence
that the chosen architecture behaves correctly under denied access, realistic volume,
concurrency, dependency failure and deployment change.

**When it is complete** — Mandatory tests and scans pass, agreed NFRs are measured, known risks
have owners and the release candidate can be deployed and rolled back.

**How to work through it** — Build the test pyramid while features are developed. Run specialised
HANA, browser, authorisation and resilience tests in environments that resemble production.

**Expected output** — Test reports, traces, scan results, defect decisions and release
recommendation.

### 11.1 Test each responsibility at the right level

| Test layer | What it proves | Typical tools |
|---|---|---|
| Unit | Pure business decisions and mappings. | ABAP Unit; Node test runner/Jest/Mocha. |
| RAP BO/service integration | CRUD, actions, messages, concurrency and authorisation. | EML tests, service tests. |
| CAP service integration | Transactions, actions, paging, errors and auth. | cds.test and HTTP/OData tests. |
| Contract | Metadata and backward compatibility. | Snapshot/contract comparison and consumer tests. |
| UI journey | Critical user tasks and messages. | OPA5, wdi5 or Playwright. |
| Accessibility | Keyboard, labels, focus and screen-reader behaviour. | Automated scanner plus manual checks. |
| HANA/native artefact | HDI grants, synonyms/virtual tables and pushdown. | HANA Cloud integration environment and SQL plan tools. |
| Performance | Latency, throughput, resource use and source load. | Browser tools, STATS_FE, ST05/SAT, HANA expensive statements, load testing. |
| Resilience | Timeout, retry, outage, duplicate event and recovery. | Fault injection and controlled dependency failure. |

### 11.2 Security verification

For each role, test what is allowed and what is denied. Include instance-level restrictions,
organisational boundaries and cross-tenant cases. Scan for secrets, vulnerable dependencies and
unsafe code. Confirm that logs and error responses do not disclose credentials, sensitive
payloads or internal topology.

### 11.3 Performance method

1. Define the journey, data volume, user role, network condition and cache state.
2. Measure the browser or caller first to identify request count, payload and perceived latency.
3. Correlate to the service and database using a correlation ID, STATS_FE, CAP logs, ST05/SAT or
   HANA analysis as appropriate.
4. Change one dominant cause, repeat the same test and retain before/after evidence.

Use starting targets as engineering triggers, not universal promises: 50-100 rows per interactive
page, investigation of browser tasks above 50ms, list pages normally below 2 seconds under agreed
conditions, and explicit remediation when a save/action exceeds 5 seconds unless designed as
asynchronous work.

## 12. Phase 7: Release and Deploy Safely

**Why this phase matters** — A successful build is not the same as a successful release. Release
includes schema change, target environment, service bindings, health, smoke tests, observability
and rollback.

**When it is complete** — The same immutable artefact tested in pre-production is promoted,
migration succeeds, smoke tests pass and rollback has been rehearsed or made credible.

**How to work through it** — Use controlled transports for RAP and an automated pipeline for
CAP/UI. Verify the deployment trigger, target, services, versions and support access.

**Expected output** — Release record, artefact versions, migration evidence, smoke result,
monitoring links and rollback decision.

### 12.1 RAP release

- Use controlled transport or gCTS and preserve dependency order across CDS, behaviour, service
  and UI artefacts.
- Run central ATC and activation checks before import.
- Provide conversion and rollback steps for persistence changes.

### 12.2 CAP and UI release

- Build and promote an immutable MTA or approved container artefact.
- Use SAP HANA Cloud schema evolution with backward-compatible expand/deploy/migrate/contract
  changes.
- Deploy the optimised UI5 `dist` output, not the project source tree.
- Keep Cloud Foundry services stateless; use multiple instances where continuity requires it.
- Provide readiness/liveness behaviour and graceful shutdown.

### 12.3 Release-day verification

- Confirm the main-branch merge actually triggered the expected pipeline.
- Confirm the target subaccount, Cloud Foundry org/space or S/4HANA system.
- Confirm service bindings, destinations, roles and certificates are valid.
- Run database/tenant upgrade steps and smoke tests.
- Confirm logs, metrics, alerts and support access.
- Record the decision to continue or roll back.

## 13. Phase 8: Operate, Improve and Retire

**Why this phase matters** — Most application cost is incurred after the first release.
Operations provides the feedback needed to improve reliability, performance and upgrade
readiness. Retirement prevents unused services and access from becoming security debt.

**When it is complete** — Support ownership, dashboards, alerts and runbooks are active;
restore/replay/rollback procedures are tested; lifecycle reviews occur on schedule.

**How to work through it** — Review incidents and metrics, patch dependencies, reassess
clean-core and deprecated APIs, and remove obsolete roles, services, subscriptions and data.

**Expected output** — Operational acceptance, improvement backlog, upgrade assessment and
retirement record.

### 13.1 What to monitor

| Signal | What it tells you | Example action |
|---|---|---|
| Latency and throughput | Whether user experience and capacity remain healthy. | Inspect dependency or database bottleneck before scaling blindly. |
| Error rate by operation | Which business journeys are failing. | Correlate to deployment, role or upstream changes. |
| Memory, CPU and restarts | Whether instances are saturated or leaking. | Profile code and right-size instances. |
| Remote dependency latency/timeouts | Whether integrations threaten capacity. | Adjust timeout, circuit breaker or degradation path. |
| Event backlog/dead letters | Whether asynchronous work is keeping up. | Fix consumer, replay safely and increase capacity if justified. |
| Authorisation denials | Whether role design or misuse needs attention. | Review expected roles and suspicious patterns. |
| Deprecated APIs and vulnerabilities | Whether technical debt threatens upgrades or support. | Patch or plan replacement within SLA. |

### 13.2 Retirement

Retirement is not just deleting code. Remove service routes, roles, destinations, remote-source
grants, subscriptions, schedulers, secrets and dashboards. Archive or delete data according to
retention policy, notify consumers and leave an auditable record of the final state.

## 14. Guided Decision Playbooks

### 14.1 "I need to display S/4HANA data in a new app"

1. Check whether a standard SAP application already satisfies the need.
2. Find the released API or released CDS/data product.
3. If the user journey is a standard list/object page, use Fiori elements.
4. If the transaction remains in S/4HANA, prefer RAP or consume the released service directly.
5. Use CAP when the app combines systems or owns side-by-side data/process logic.
6. Consider a virtual table only for approved, read-only, bounded federation after API and
   replication options are rejected.

### 14.2 "I need custom logic in CAP"

1. Ask whether CDS constraints, service projections or the generic provider already express the
   rule.
2. If the rule validates request/state, use `before`.
3. If it is a command or replaces processing, use `on`.
4. If it only enriches the returned representation, use `after`.
5. If the logic contains several business decisions, move them into a domain module and unit-test
   them.

### 14.3 "I need a BAPI in RAP"

1. Confirm that the BAPI/API is approved for the target clean-core level.
2. Check commit behaviour, late numbering, failure mapping and rollback semantics.
3. If interaction is standard and only final persistence is custom, assess managed with unmanaged
   save.
4. If the legacy engine owns read, lock, buffer and save, use fully unmanaged RAP.
5. Do not add `BAPI_TRANSACTION_COMMIT` mechanically inside behaviour handlers.

### 14.4 "The page is slow"

1. Measure the browser request count, payload and rendering before changing backend code.
2. Identify whether time is in UI, network, service, database or remote dependency.
3. Check paging, `$select`, `$expand`, `$count`, formatters and broad refreshes.
4. Use STATS_FE/ST05/SAT for RAP and CAP/HANA telemetry for CAP.
5. Retest the same scenario and retain evidence.

## 15. Phase Exit Checkpoints

The following checkpoints are summaries, not substitutes for the preceding guidance.

| Phase | Ready to proceed when |
|---|---|
| Discover | Business outcome, users, data ownership, scope and NFRs are understood. |
| Decide | Framework, UI, persistence, integration, tenant and deployment decisions are recorded with consequences. |
| Design | Domain, service, authorisation, errors, data migration and testing strategy are agreed. |
| Establish | A new developer can run the skeleton and the pipeline enforces basic quality. |
| Build | Vertical slices use framework semantics, tests are included and no layer owns another layer's responsibility. |
| Verify | Functional, security, accessibility, performance and resilience evidence meets the project NFRs. |
| Release | Immutable artefact, migration, smoke, monitoring and rollback are ready. |
| Operate | Ownership, alerts, runbooks, patching and lifecycle review are active. |
| Retire | Consumers, access, services and data are removed or archived safely. |

## 16. Common Failure Patterns and How to Correct Them

| Pattern | Why it happens | Correction |
|---|---|---|
| RAP or CAP chosen by familiarity | Team starts from skills rather than transaction ownership. | Return to system-of-record and lifecycle decision. |
| Fully unmanaged RAP for new tables | Classic ABAP habits make custom control feel safer. | Use managed RAP and implement only business rules. |
| Custom CAP CRUD everywhere | Developers do not trust or understand generic providers. | Start model-first and remove handlers that duplicate framework behaviour. |
| Business rules in SAPUI5 controllers | Immediate UI change seems convenient. | Move authoritative rule to RAP/CAP and keep optional pre-validation in UI. |
| Raw S/4HANA table federation | Database access appears faster than API discovery. | Use released API/data product; treat virtual table as approved read-only exception. |
| One large service file | Features are added without capability boundaries. | Group handlers by capability and move business decisions to domain modules. |
| Microservices too early | Independent deployment is assumed to equal good architecture. | Begin with modular monolith until ownership and scale boundaries are proven. |
| Retry every error | Retries are added as a generic resilience feature. | Retry only transient, safe/idempotent operations with bounds and jitter. |
| Increase memory for every slowdown | Platform symptoms are treated as sizing problems. | Measure database, event loop and dependency causes first. |
| Checklist-only review | Rules are checked without understanding the design. | Review the ADR, data ownership and failure scenarios before code details. |

## 17. Learning Path for Junior Developers

The learning path follows the same order as delivery. Do not try to learn every framework feature
before building a small vertical slice.

| Step | What to learn | Practice outcome |
|---|---|---|
| 1. Business and architecture basics | System of record, transaction boundary, API contract and clean core. | Explain why a sample belongs in RAP or CAP. |
| 2. CDS modelling | Entities, keys, associations, compositions, projections and annotations. | Build a small domain and service without custom handlers. |
| 3. Transaction behaviour | RAP managed behaviour or CAP generic provider and transactions. | Implement create/change plus one validation and action. |
| 4. Authorisation | DCL/behaviour auth or @requires/@restrict. | Write one allowed and one denied test. |
| 5. Fiori elements | OData V4 metadata, List Report, Object Page, actions and messages. | Create a working standard app before custom UI. |
| 6. Integration | Destination/remote services, timeouts, clean core and events. | Consume one mocked remote API and handle failure. |
| 7. Deployment | Transport or MTA, service bindings, health and logs. | Deploy a test artefact and run a smoke test. |
| 8. Quality | Unit/integration/UI tests, traces and security scans. | Attach evidence to a pull request. |

## 18. Primary References

- ABAP RESTful Application Programming Model: https://help.sap.com/docs/abap-cloud/abap-rap/managed-and-unmanaged-scenarios
- RAP business object implementation types: https://help.sap.com/docs/abap-cloud/abap-rap/business-object-implementation-types
- RAP business object contract: https://help.sap.com/docs/abap-cloud/abap-rap/rap-business-object-contract
- CAP getting started: https://cap.cloud.sap/docs/get-started/
- CAP core concepts: https://cap.cloud.sap/docs/get-started/concepts
- CAP Node.js SDK: https://cap.cloud.sap/docs/node.js/
- CAP security: https://cap.cloud.sap/docs/guides/security/
- CAP native HANA artifacts: https://cap.cloud.sap/docs/guides/databases/hana-native
- CAP deployment: https://cap.cloud.sap/docs/guides/deploy/
- UI5 modern application guidance: https://ui5.github.io/docs/02_Read-Me-First/developing-apps-with-modern-concepts-guidance-for-developers-4cb54eb.html
- UI5 developer best practices: https://ui5.github.io/docs/03_Get-Started/best-practices-for-developers-28fcd55.html
- SAP Fiori Design Guidelines: https://experience.sap.com/fiori-design-web/
- SAP HANA Cloud virtual tables: https://help.sap.com/docs/HANA_CLOUD_DATABASE/c2b99f19e9264c4d9ae9221b22f6f589/00340d4ede0a43ceb0a67f36bbc46a3a.html?locale=en-US
- SAP clean-core extensibility update: https://community.sap.com/t5/technology-blog-posts-by-sap/abap-extensibility-guide-clean-core-for-sap-s-4hana-cloud-august-2025/ba-p/14175399
