# Shimpz Runtime Draft Decisions

| Field | Value |
|---|---|
| Status | Open deliberation for Draft 0.2 |
| Authority | Non-authoritative coordination record |
| Implementation | Unavailable; this document authorizes no implementation |
| Owner | Juliano |
| Umbrella evidence | `TheShimpz/shimpz@afaee04e9163c1810e5ab02ceac6071fe6b7197d` |
| Updated | 2026-08-23 |

This file preserves unresolved architecture, authority, admission, and implementation-order decisions outside the
functional contract in [`SPEC.md`](SPEC.md). Describing target behavior in the SPEC does not close any decision in
this file, admit a first vertical, or change current Shimpz architecture.

The informative [`DESIGN.md`](DESIGN.md) is an alternative reduced-v1 visual proposal for discussion. It carries no
authority to amend this decision record or `SPEC.md` Draft 0.2; the questions it raises are recorded below.

## 1. Current evidence

- Brain reasons over admitted Assistant capabilities and does not execute Actions or acquire Assistant authority
  (`shimpz/.context/ARCHITECTURE.md:69-82`).
- The current product mechanism executes programmed Actions instead of generating a new implementation for each
  task (`shimpz/.context/PRODUCT.md:117-120`).
- The shipped Brain prompt rejects code execution, dependencies, shell, and filesystem access
  (`shimpz/brain/agent_runtime.py:305-316`).
- Team owns authorization, binding, Actions, and the current human-request lifecycle
  (`shimpz/.context/ARCHITECTURE.md:118-133`).
- PostgreSQL has no current Runtime Project or Assistant Service-binding contract
  (`shimpz/.context/ARCHITECTURE.md:547-561`).
- The previous root `runtime/` was retired and is rejected by repository-shape admission
  (`shimpz/.context/ARCHITECTURE.md:347-356`, `shimpz/.scripts/repo-shape.py:32-70`).
- Existing umbrella Rust workspaces are pinned to 1.97.1. Rust 1.98.0 is the proposed Runtime baseline, but an exact
  toolchain artifact and checksum have not been admitted.
- Umbrella repository governance requires Node.js 24 for every Node.js workload
  (`shimpz/AGENTS.md:80`).

## 2. Boundary carried by Draft 0.2

- The SPEC describes a target Runtime-local contract, not current availability or current umbrella authority.
- It does not define Team HTTP or challenge protocols, Developers publication or source-package protocols,
  `shimpz.chat.v4`, PostgreSQL Service binding, or repository admission.
- A capability that needs another domain remains unavailable until that domain supplies an admitted binding.
- Runtime Project and Operation are document-local concepts, not new current Shimpz actors.
- Runtime human input is a proposed Runtime-local state machine. It does not modify the current Action human-request
  contract.
- Runtime-local `after_*` chains are not Routine. Routine remains reserved and unchanged.
- The functional surface is execution-model-neutral. It does not select WebAssembly Component or native execution.
- Describing the complete target surface does not select it as the first implementation vertical. The previously
  rejected hybrid first vertical remains rejected.
- No Product, Architecture, or shipped Brain-prompt amendment has occurred.

## 3. Conflict register

| Conflict | Current constraint | Required closure |
|---|---|---|
| Brain authors Rust | Product and the live prompt reject generated-code execution | Amend those authorities or reject the proposal |
| Capability admission | Assistant to Action uses Developers to Store to Team | Map Runtime to that path or admit a separate Team-private class |
| Root path | `runtime/` is retired and gate-enforced absent | Supersede the retirement before any mount |
| Authoring contract | Assistant Spec v1 is Python and direct-process | Decide replacement, extension, or disjoint contract |
| Control authority | Account, Admin, Developers, Team, and Services own distinct authority | Keep Runtime from absorbing peer ownership |
| PostgreSQL | No Runtime Service binding exists | Admit a binding before `ctx.db()` is available |
| Human continuation | Current Action continuation is bounded replay | Admit a separate or superseding Runtime state machine |
| Chains | Durable `after_*` can overlap reserved Routine language | Keep the concepts disjoint and admit separately |
| Frontend | Static sites use the Svelte and Tailwind lane | Decide whether an immutable Runtime Project version may instead contain static frontend assets and Rust endpoint code |
| Browser safety | Draft 0.2 assigns escaping, sanitized raw HTML, and restrictive response policy to Runtime page rendering | Decide how an isolated frontend build plus non-weakenable Gateway policy preserves the same security intent |
| Dependencies | Native packages, frontend packages, macros, FFI, egress, and secrets widen trust | Start with fixed offline Rust and frontend sets; admit wider classes separately |
| Reduced surface | Draft 0.2 includes pages, events, project calls, chains, human input, database, storage, secrets, outbound HTTP, authenticated principals, and sessions | Decide whether v1 contains only static HTTP, anonymous public Rust HTTP, and anonymous request/response WebSocket |
| Authoring surface | Draft 0.2 exposes separate source, validation, build, publication, rollback, dependency, invocation, and human-response methods | Decide whether authoring is reduced to `project.inspect`, `project.change`, and `change.status` |
| Change lifecycle | Draft 0.2 exposes separate revision and version states plus build cancellation | Decide whether one Change, four terminal outcomes, and five observable internal phases are sufficient |
| Activation authority | Draft 0.2 activates a validated artifact through Runtime publication | Decide which external authority permits one exact digest and how Project Control proves it cannot self-authorize |
| WebSocket | Draft 0.2 preserves Runtime-owned connections across publication and `shimpz.chat.v4` is the existing browser chat contract | Decide whether reduced v1 may close old-version connections and admit a separate producer-owned protocol |
| Rollback | Draft 0.2 rolls back to one exact retained version without rebuilding or approximate fallback | Decide whether rollback is deferred from reduced v1 |
| Project deletion | Draft 0.2 requires residue-complete deletion | Assign retention and deletion authority before deciding whether deletion enters reduced v1 |
| Serving roles | Draft 0.2 permits multiple roles of one binary | Decide whether v1 exposes one trusted Runtime product surface with separate internal Gateway and Control authorities |
| Public origin | No Runtime Project hostname namespace or browser-isolation authority exists | Admit origin ownership and isolate both Shimpz control surfaces and sibling Projects before serving public content |

## 4. P0-1: ontology, admission, and source custody

| Option | Meaning |
|---|---|
| A | Runtime Project and Operation implement Assistant and Action through Developers to Store to Team |
| B | Runtime is a separate Team-private capability with its own admitted lifecycle |
| C | Brain may produce artifacts, but cannot invoke Runtime Operations |

The selected option must name source custody, readers, retention, backup treatment, profile-specific storage,
admission authority, deletion owner, and residue-complete deletion. Renaming an Assistant-equivalent capability
cannot bypass the standard Assistant path.

**Provisional recommendation:** B, only if the owner explicitly admits the new trust class.

**Status:** Open.

## 5. P0-2: first vertical

| Option | Vertical |
|---|---|
| A | Produce an immutable static artifact for the separate site-delivery lane |
| B | Execute one pure typed Rust operation with no database, realtime, events, pages, chains, or human input |
| C | Implement HTTP, database, catalog, chains, human input, and audit together |

**Provisional recommendation:** B. Option C remains rejected as a first vertical because it is not independently
reviewable. The complete surface in the SPEC is a target contract, not an implementation sequence.

**Status:** Open.

## 6. P0-3: execution substrate

| Option | Meaning |
|---|---|
| A | WebAssembly Component inside an outer isolation boundary |
| B | Native Rust inside a Team-bound outer sandbox |
| C | Native process with operating-system isolation alone |

The decision must evaluate capability closure, Rust compatibility, blast radius, density, cold start, patching,
egress, secrets, provenance, and measured build and execution limits. The SPEC remains neutral between A and B.
Option C remains rejected without equivalent adversarial evidence.

**Status:** Open.

## 7. P0-4: profile scope

| Option | Meaning |
|---|---|
| A | One Local and Hosted concept with a Hosted-first implementation proof |
| B | Local-first implementation proof |
| C | Implement Local and Hosted together |

**Provisional recommendation:** A.

**Status:** Open.

## 8. Promotion ledger

| ID | Decision | Promotion gate |
|---|---|---|
| P0-1 | Ontology, admission, source custody, and deletion | Explicit owner choice and complete lifecycle contract |
| P0-2 | First implementation vertical | Explicit owner choice |
| P0-3 | Artifact format and execution substrate | Threat model and measured isolation decision |
| P0-4 | Profile implementation order | Explicit owner choice and assurance statement |
| P1-1 | Exact public naming | Vocabulary decision |
| P1-2 | Rust reviewability limits | Measured authoring evidence |
| P1-3 | Toolchain, SDK, build-cache, and builder-attestation provenance | Exact artifacts, checksums, cache verification, attestation trust, and provenance before building |
| P1-4 | Cross-domain protocol surfaces | Producer-owned contracts after P0 closure |
| P1-5 | Public origin namespace and browser isolation | Exact authority, ownership conflict handling, cookie isolation, and profile-specific threat model |
| P1-6 | Dependency Catalog governance | Admission evidence, bounded discovery, vulnerability response, notification, blocking, and revocation |

Draft 0.x can become SPEC v1 only after all P0 decisions are owner-approved; an ADR admits the new repository and
domain and updates the ADR-0019 repository registry; root retirement and repository-shape gates are superseded;
the umbrella `.scripts/validate-architecture` passes before any mount; Product, Brain, Assistant, Developers,
Store, Team, and Services impacts are resolved; artifact format, builder boundary, profile scope, complete data
lifecycle, and protocol ownership are normative; and no text confuses target behavior with implemented
availability.

## Changelog

- **Discussion update — 2026-08-23:** Reduced the visual proposal to Project, Member, Change, three authoring
  operations, two diagrams, and six security invariants; added its principal conflicts with SPEC Draft 0.2.
- **Draft 0.2 — 2026-08-22:** Moved deliberative material out of the functional SPEC, preserved all open P0
  decisions, and made the target-contract boundary explicit.
- **Draft 0.1 — 2026-08-22:** Established conflicts, proposed boundaries, four P0 decisions, candidate first
  vertical, deferrals, and promotion gates.
