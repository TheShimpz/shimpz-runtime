# Shimpz Runtime Specification

| Field | Value |
|---|---|
| Status | Draft 0.1 |
| Target | SPEC v1 |
| Authority | Non-authoritative deliberation |
| Implementation | Unavailable; this document authorizes no implementation |
| Owner | Juliano |
| Umbrella evidence | `TheShimpz/shimpz@afaee04e9163c1810e5ab02ceac6071fe6b7197d` |
| Updated | 2026-08-22 |

## 1. Problem and intended outcome

Shimpz needs a durable way for Brain to propose Rust functionality without choosing files, Cargo configuration,
build commands, deployment, routing, workers, credentials, WebSocket mechanics, or infrastructure.

Brain may revise functionality but never gains authority to admit, bind, publish, invoke, update, or delete it.

This Draft decides ontology, authority, the first vertical, data custody, and execution trust before later design.

## 2. Evidence and coverage

The private umbrella was inspected at the pinned revision. Evidence uses pinned `path:line` references.

- Brain reasons over admitted Assistant capabilities and does not execute Actions or acquire Assistant authority
  (`.context/ARCHITECTURE.md:69-82`).
- The durable product mechanism says programmed Actions execute work instead of AI generating a new implementation
  per task (`.context/PRODUCT.md:117-120`).
- The shipped Brain prompt rejects code execution, dependencies, shell, and filesystem access
  (`brain/agent_runtime.py:305-316`).
- Assistant Spec v1 is Python, direct-process, bounded to one JSON result and an eight-second deadline; it admits no
  authored servers (`.context/decisions/0012-file-backed-assistant-spec-v1.md:20-40`).
- Team owns authorization, binding, Actions, and human-request lifecycle (`AGENTS.md:98-103`,
  `.context/ARCHITECTURE.md:118-133`). Current continuation replays an immutable Action rather than restoring an
  async process stack.
- PostgreSQL binds to Hosted Team lifecycle; no Runtime Project or Assistant Service-binding contract exists
  (`.context/ARCHITECTURE.md:547-561`).
- The previous root `runtime/` was retired and is rejected by repository-shape admission
  (`.context/ARCHITECTURE.md:347-356`, `.scripts/repo-shape.py:32-70`).
- Existing umbrella Rust workspaces remain pinned to 1.97.1. Rust 1.98.0 was released on 2026-08-20
  ([official announcement](https://blog.rust-lang.org/2026/08/20/Rust-1.98.0/)).

The remote had no observable refs and `runtime/` was not locally mounted. No implementation existed to inspect.

## 3. Classification

### Inherited invariants

- Team authorization and machine identity remain distinct from Brain reasoning.
- Cross-Team access fails closed and hides another Team's existence.
- Protocols belong to their natural producer; consumers pin exact mirrors and validate shared vectors.
- Protected payloads and secrets never enter audit.
- Deletion succeeds only after all owned residue is absent.
- A builder or run-once job has measured CPU, memory, swap-disabled, PID, and health bounds
  (`.context/ARCHITECTURE.md:522-523`).
- Separate runtime authorities do not collapse merely because roles share an image.

### Owner-proposed constraints

- `TheShimpz/shimpz-runtime` is proposed for future mount at root `runtime/`; its name reserves no public vocabulary.
- All new Runtime and workload source is Rust-only with Rust 1.98.0 as the owner-set MSRV baseline. The exact
  toolchain artifact and checksum must be admitted before build implementation; existing 1.97.1 workspaces are not
  silently upgraded.
- Brain uses semantic, typed operations and never receives filesystem, shell, Cargo, builder, or deployment control.
- Runtime Project is the candidate compilation and publication unit.
- One exported operation per source file is the candidate navigation rule.

### Provisional

- Brain-authored source becomes a durable proposal, not per-invocation generated code.
- Builds are offline and isolated; artifacts are immutable and addressed by digest.
- Runtime source is protected Team data, subject to the open custody decision in P0-1.
- A narrow Runtime Control API may own source validation, build orchestration, artifact lifecycle, and—only if
  admitted—execution.

Repository-local working rules are absent. A separate governance microtask is required before any repository work
beyond this first Draft.

## 4. Conflict register

| Conflict | Current constraint | Required closure |
|---|---|---|
| Brain authors Rust | Product and live prompt reject generated code execution | Amend Product, Architecture, and live prompt or reject the proposal |
| Capability admission | Assistant → Action uses Developers → Store → Team | Map Runtime to that path or admit a genuinely separate Team-private class |
| Root path | `runtime/` is retired and gate-enforced absent | Accept an owner-approved supersession before mounting |
| Authoring contract | Assistant Spec v1 is Python and direct-process | Decide replacement, extension, or disjoint contract |
| Control authority | Account, Admin, Developers, Team, and Services own distinct authority | Runtime must not absorb peer ownership |
| PostgreSQL | No Project/Assistant Service binding exists | Defer DB or separately approve a binding |
| Human continuation | Current replay is bounded and immutable | Defer new HITL or define a superseding state machine |
| Chains | Durable `after_*` is a workflow engine and overlaps reserved Routine semantics | Defer pending a separate ontology and protocol decision |
| Frontend | The current site lane uses Svelte/Tailwind | Keep its Svelte/Node build separate or explicitly reopen the boundary |
| Dependencies | Native crates, macros, FFI, egress, and secrets widen the trust class | Start fixed and offline; admit wider classes separately |
| WebSocket | `shimpz.chat.v4` is the existing Admin/retained Hosted browser chat control-plane contract | A Runtime data plane needs a separate producer-owned protocol |

## 5. Working terminology

These terms are internal and provisional.

- **Shimpz Runtime:** proposed Team-bound substrate for durable code artifacts. It is not Brain, Team Controller,
  Admin, Developers, Assistant runtime, PostgreSQL, or static delivery.
- **Runtime Project:** candidate compilation/version unit containing operations.
- **Operation:** candidate exported, typed entry point. It is not public vocabulary and does not automatically mean
  Action.
- **Proposal:** Brain-authored source and metadata with no execution authority.
- **Artifact:** admitted immutable build output identified by digest.
- **Binding:** Team authorization connecting an exact artifact/operation to an invocation boundary.

`App` remains retired and `Routine` remains reserved. This Draft introduces no new public actor named User, Agent,
Service, Function, Project, or Operation.

## 6. Authority and data matrix

| Transition/data | Proposed owner | Constraint |
|---|---|---|
| Request functionality | Authenticated Team authority | Human intent is not machine identity |
| Propose source | Brain through an admitted structured capability | Proposal grants no authority |
| Review/admit source | Open in P0-1 | Must be attributable and independent of Brain |
| Build artifact | Runtime build boundary | Offline, no secrets, no production data |
| Bind artifact | Team | Exact Team, derived artifact digest, operation, and policy |
| Invoke | Team-authorized caller | Identity derives from authenticated evidence, never caller metadata |
| Execute | Runtime execution boundary if P0-2/B is selected | Exact admitted digest and bounded capabilities |
| Update/rollback | Team plus Runtime lifecycle | Atomic active digest; prior version remains bounded |
| Delete | Team-authorized lifecycle; Runtime removes owned residue | Source, artifact, state, and caches included |
| Audit | Owning domain of each transition | Metadata and outcome only; never protected payload |
| Source custody | Open in P0-1; provisionally Team-scoped | No cross-Team deduplication or existence oracle |

Required changes in Team, Developers, Admin, Account, or Services are peer requests. This SPEC cannot author their
normative protocol.

## 7. Threat model and required properties

Assume malicious source, build inputs, dependencies, callers, payloads, artifacts, and authoring instructions.
Prompt-injected Team or Assistant content may induce durable malicious source; provenance and review remain
independent of Brain.

Consider cross-Team access, artifact substitution, stale authorization, exhaustion, exfiltration, secret exposure,
partial lifecycle failure, and deletion residue. Any executable candidate requires:

- no ambient authority;
- authenticated, request-bound identity and exact Team/artifact binding;
- fail-closed schema and capability validation;
- isolated, networkless build with builder authority separated from gateway/executor;
- per-job and per-invocation limits;
- immutable provenance from reviewed source/toolchain/SDK to artifact digest;
- patchable sandbox/runtime;
- no protected payload in audit or logs;
- complete idempotent deletion.

Reviewability is separate from security. File length, AST size, and complexity may help Brain edit code but are not
a sandbox.

## 8. P0-1: ontology, admission, and source custody

| Option | Summary | Main advantage | Main cost |
|---|---|---|---|
| A | Runtime Project/Operation implements Assistant/Action and uses Developers → Store → Team | Reuses one admission path | Creator publication does not naturally fit private Brain iteration |
| B | New Team-private Runtime capability, never represented or admitted as Assistant/Action | Matches private iteration and preserves Action provenance | Requires new ontology and admission protocol |
| C | Brain builds artifacts but cannot invoke Runtime Operations | Smallest authority change | Does not deliver an AI-native execution runtime |

Option B is rejected if a Runtime Operation becomes Brain-invocable on the same footing as an Action. Renaming an
Assistant-equivalent capability cannot bypass Developers → Store → Team (`AGENTS.md:98-103`).

P0-1 must name lifecycle owners and close source custody, profile-specific at-rest behavior, retention, readers,
backup treatment, no-log/no-audit handling, and residue-complete deletion. No source persistence precedes closure.

**Provisional recommendation:** B only if the owner explicitly replaces the product mechanism for this new
trust class. Team remains authorization/binding owner; Brain remains proposal-only. Reuse of `runtime/` is part of
this supersession.

**Status:** Open.

## 9. P0-2: first vertical

The common provisional prefix is:

`proposal → attributable non-Brain review/admission → isolated offline build → immutable digest → metadata-only audit → idempotent rollback/deletion`

| Option | Branch | What it proves | Limitation |
|---|---|---|---|
| A: static artifact | `publish → static delivery` | Safest landing-page path; no authored server execution | Requires the separate current Svelte/Node build lane and may not prove a Runtime |
| B: pure operation | `Team binding → typed input → typed output → Outcome` | Smallest real execution proof | Requires P0-1 admission and P0-3 isolation |
| C: original slice | HTTP + Postgres + catalog + chain + HITL + audit | Many eventual capabilities at once | Not independently reviewable; rejected without extraordinary evidence |

**Provisional recommendation:** keep the product site-first in its own lane and choose B for Runtime. Do not build a
hybrid v1.

**Status:** Open.

## 10. P0-3: executable artifact and isolation

| Option | Advantage | Cost/risk |
|---|---|---|
| A: WebAssembly Component plus outer sandbox | Closed imports/capabilities and small invocation unit | Restricts crates and native integration |
| B: native Rust inside a Team-bound outer sandbox such as gVisor | Broad Rust compatibility | Larger trusted surface and stronger worker lifecycle burden |
| C: native process with seccomp/namespaces only | Lowest initial machinery | Insufficient default evidence for hostile machine-authored code |

Evaluate capability closure, Rust compatibility, blast radius, density, cold start, patching, egress, secrets,
provenance, cross-Team leakage, and measured builder/job limits. Existing gVisor investment is evidence, not
inherited authority for this Runtime.

**Provisional recommendation:** A for the first pure operation; B remains a separate wider trust class. C is
rejected absent equivalent adversarial evidence.

**Status:** Open.

## 11. P0-4: profile scope

| Option | Advantage | Cost |
|---|---|---|
| Hosted-first implementation of one Local+Hosted concept | Fastest hostile-tenant proof without false topology equality | Local arrives later |
| Local-first | Faster developer iteration | Current Local does not claim hostile multi-tenant isolation |
| Both in v1 | Early conceptual symmetry | Doubles lifecycle and assurance evidence |

**Provisional recommendation:** one Local+Hosted concept, with a Hosted-first proof and later Local profile.

**Status:** Open.

## 12. Candidate v1 and non-goals

If P0-1/B, P0-2/B, P0-3/A, and Hosted-first are accepted, candidate v1 is limited to:

- structured create/describe/update/delete for one Rust Operation in a closed Runtime Project template;
- one exported operation per file, with measured rather than guessed reviewability bounds;
- fixed vendored SDK/dependency set and no build network;
- deterministic project build and immutable artifact digest;
- exact Team-private binding and one typed request/response boundary;
- progressive operation discovery;
- bounded execution, version selection, drain, rollback, audit, and deletion.

No artifact becomes bindable until an attributable non-Brain authority selected by P0-1 approves the reviewed
source and the binding admits only the digest provably derived from that source, toolchain, and SDK. Source
persistence also requires the complete P0-1 custody contract.

Acceptance requires deterministic repeated builds, rejection of undeclared imports/capabilities, cross-Team
denial, resource-limit enforcement, output validation, exact-version invocation, safe rollback, metadata-only
audit, and absence of all owned residue after deletion.

Original-prompt concepts deferred, not roadmap commitments: arbitrary dependencies, DB, secrets, egress,
filesystem, WebSocket, events/jobs, `after_*`, durable DAGs, new HITL, inter-project calls, SSR, RBAC/ReBAC, Control
Plane UI, MCP, multi-role or distributed scale, and other languages.

Rejected for v1: monolithic administration authority, caller-supplied actor/delegation/metadata, builder sharing
authority with executor, line count as a security boundary, and networked arbitrary-dependency builds.

## 13. Failure, version, audit, and deletion

An invocation, if admitted, resolves one exact artifact digest before execution. A replacement never changes an
in-flight invocation. Unknown, stale, unauthorized, malformed, timed-out, or over-limit work fails closed without
committing a success outcome.

Audit and observability are distinct. Audit may record bounded principal identity, Team, operation, artifact
digest, lifecycle state, timing, and outcome class. It never records source, prompt, input, output, secrets, or
protected payload. Hashing low-entropy payloads is not automatically safe.

Deletion succeeds only after Runtime-owned source, artifact, binding projections, state, and AOT/cache residue are
absent. Partial failure remains incomplete and names its retry owner.

## 14. Open decision ledger

| ID | Decision | Owner | Promotion gate |
|---|---|---|---|
| P0-1 | Assistant/Action mapping, Team-private capability, or artifact-only; admission and source custody | Juliano | Explicit option and complete authority/data-lifecycle decision |
| P0-2 | Static artifact or pure operation first vertical | Juliano | Explicit vertical selection |
| P0-3 | WebAssembly Component or sandboxed native execution | Juliano | Threat model and artifact/sandbox decision |
| P0-4 | Hosted-first, Local-first, or both | Juliano | Profile scope and assurance statement |
| P1-1 | Exact project/operation naming | Future Draft | Vocabulary decision |
| P1-2 | Rust reviewability bounds | Future Draft | Measured Brain authoring evidence |
| P1-3 | Exact toolchain artifact/checksum and SDK provenance | Future Draft | Required before build implementation |
| P1-4 | Runtime-owned protocol surface | Future Draft | P0-1/P0-2 closure |

## 15. Promotion criteria

Draft 0.x may become SPEC v1 only after:

1. all P0 decisions and the complete ledger are explicitly approved by the owner;
2. a new-domain/new-repository ADR admits `shimpz-runtime` and the repository registry;
3. current `runtime/` retirement is explicitly superseded through the Architecture/ADR-0018 runtime boundaries,
   ADR-0019 registry, repository-shape gates, and a passing `.scripts/validate-architecture` before any mount;
4. Product, Brain prompt, Assistant/Action, and Developers → Store → Team impacts are resolved;
5. threat model, artifact format, builder boundary, profile scope, data lifecycle, and protocol ownership are
   normative;
6. repository-local governance exists before subsequent repository work;
7. no text confuses SPEC authority with implemented availability.

A later implementation request is a separate task.

## 16. Questions for the owner

1. Should Runtime Operations be a new Team-private capability (P0-1/B), as provisionally recommended?
2. Should Runtime's first vertical execute one pure Rust operation (P0-2/B), while static sites remain separate?
3. Should v1 use a WebAssembly Component inside an outer sandbox (P0-3/A)?
4. Should the concept cover Local and Hosted while the first execution proof is Hosted-first (P0-4)?

## Changelog

- **Draft 0.1 — 2026-08-22:** Established current conflicts, proposed boundaries, four P0 decisions, candidate v1,
  deferrals, and promotion gates. No implementation is authorized.
