# Shimpz Runtime Functional Specification

| Field | Value |
|---|---|
| Status | Draft 0.2 |
| Target | SPEC v1 |
| Contract | Proposed Runtime-local functional contract |
| Availability | Unavailable |
| Conformance | No conformance claim is admissible against this Draft |
| Implementation | Not authorized |
| Owner | Juliano |
| Updated | 2026-08-22 |

## Abstract

Shimpz Runtime is a proposed Rust execution runtime operated through a small typed interface. A code-authoring Brain
provides a function name, description, schemas, Rust code, and intended effects. The Runtime owns placement,
formatting, validation, dependency resolution, build, publication, routing, execution, realtime connections,
durable continuation, limits, and telemetry.

This document specifies the target functionality, accepted values, state transitions, errors, limits, and minimum
observable behavior. It does not choose a sandbox substrate, define another Shimpz domain's protocol, or authorize
implementation. Open admission and implementation decisions are tracked in
[`DRAFT-DECISIONS.md`](DRAFT-DECISIONS.md).

## 1. Status, scope, and authority

The requirement keywords in this Draft describe the intended SPEC v1 contract. They do not describe functionality
that is currently available. No implementation, integration, deployment, root mount, or conformance claim is
authorized by Draft 0.2.

No Product, Architecture, or shipped Brain-prompt amendment admitting Brain-authored Rust has occurred. This Draft
describes the proposed target only; the required authority changes remain open in
[`DRAFT-DECISIONS.md`](DRAFT-DECISIONS.md).

This specification owns only the proposed Runtime-local surface:

- structured source authoring;
- Runtime Project validation, build, version, and publication lifecycle;
- canonical invocation and outcome semantics;
- Runtime-local HTTP, realtime, event, page, and internal-call adapters;
- Runtime-owned connections, jobs, chains, and human-input state;
- capability handles exposed to Rust functions;
- operation discovery, errors, limits, audit events, and security behavior.

This specification does not define Team HTTP or human-challenge wire contracts, Developers publication or
source-package protocols, browser chat, PostgreSQL Service binding, Account identity, Admin UI, Store discovery, or
repository admission. A feature that depends on one of those domains MUST remain `unavailable` until an admitted
binding from its owning domain exists.

`Runtime Project`, `Runtime Operation`, and `Runtime Function` are document-local concepts. They are not current
public Shimpz actors. Runtime realtime is not `shimpz.chat.v4`. Runtime chains are not Routine. Static marketing
sites remain in the separate Svelte and Tailwind build lane; Runtime pages are dynamic server-rendered interfaces.

Describing all target features does not select a hybrid first implementation vertical. The candidate first vertical
remains one pure typed Rust operation. The execution substrate and Local-versus-Hosted implementation order remain
open. The functional behavior below MUST be preserved regardless of the selected conforming substrate.

## 2. Conventions

### 2.1 Requirement language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**,
**RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described
in [BCP 14](https://www.rfc-editor.org/info/bcp14) when, and only when, they appear in all capitals.

### 2.2 Data representation

- Structured requests and results use UTF-8 JSON.
- Timestamps use RFC 3339 UTC strings unless a field explicitly says `milliseconds since Unix epoch`.
- Identifiers are lowercase ASCII. A local name MUST match `^[a-z][a-z0-9_]{0,62}$`.
- A qualified operation name is `<project>.<operation>`.
- Digests use `sha256:<64 lowercase hexadecimal characters>`.
- Decimal and money amounts use base-10 strings. JSON floating-point numbers MUST NOT represent money.
- Protocol integers MUST fit a signed 64-bit integer. Larger exact numbers use decimal strings defined by schema.
- Generated schemas use JSON Schema Draft 2020-12 with unknown object properties rejected by default.
- A field is optional only when this document marks it optional. Unknown fields MUST be rejected.
- `null` MUST be rejected unless the field schema explicitly admits `null`.
- Examples are illustrative. They do not establish an implemented API or override normative prose and tables.

### 2.3 Retired identifiers

New Runtime paths, fields, APIs, types, variables, labels, artifacts, and examples MUST NOT introduce the retired
identifiers `App`, `AppSpec`, `Captain`, `Driver`, actor `operator`, `accounts`, `brain_id`, `team.brain`, or
`team.model`. Implementations MUST NOT conceal a retired identifier through splitting, concatenation, or encoding.

## 3. Functional model

| Term | Functional meaning |
|---|---|
| Runtime | The control and execution substrate defined by this document |
| Runtime Project | Source, schemas, dependencies, generated code, and versions compiled as one unit |
| Runtime Function | One Rust callable stored in one Runtime-owned source file |
| Runtime Operation | A discoverable typed function that can be invoked through an admitted adapter |
| Proposal | Unadmitted source and metadata; it has no invocation authority |
| Artifact | Immutable build output plus manifest, addressed by digest |
| Version | Exact artifact digest and generated catalog for one successful project build |
| Binding | External authorization connecting one Team to exact Runtime capabilities and artifacts |
| Invocation | One durable attempt to execute one exact operation version |
| Outcome | One terminal success or error returned for an invocation |
| Capability | One bounded function authority from the closed set in Section 11 |

The Runtime MUST expose `runtime.describe`. Its result MUST contain the effective protocol version, Rust baseline,
feature states, and limits.

```json
{
  "protocol_version": "1",
  "rust_baseline": "1.98.0",
  "features": {
    "backend": { "state": "available" },
    "realtime": { "state": "unavailable", "reason": "binding_missing" },
    "events": { "state": "available" },
    "pages": { "state": "available" },
    "database": { "state": "unavailable", "reason": "binding_missing" }
  },
  "limits": {
    "input_bytes": 524288,
    "output_bytes": 524288,
    "default_deadline_ms": 8000
  }
}
```

Every feature has exactly one state:

| State | Meaning |
|---|---|
| `available` | The feature and every required external binding are usable |
| `unavailable` | The feature cannot be used; `reason` is REQUIRED |
| `degraded` | Existing work may complete, but new work can be rejected; `reason` is REQUIRED |

The complete feature set is:

| Feature | Functionality |
|---|---|
| `backend` | Typed Runtime Operations and the canonical invocation pipeline |
| `catalog` | Progressive operation discovery |
| `build` | Validation, dependency, build, artifact, and publication lifecycle |
| `realtime` | Runtime-owned WebSocket connections and frames |
| `events` | Durable publish, retry, ordering, and dead letter |
| `pages` | Dynamic server-rendered pages, forms, sessions, assets, and head metadata |
| `database` | Bound project database handle |
| `outbound_http` | Policy-controlled HTTPS client handle |
| `storage` | Bound object storage handle |
| `secrets` | Scoped non-serializable secret handles |
| `project_calls` | Typed calls between Runtime Projects |
| `chains` | Durable `after_*` relationships |
| `human_input` | One durable typed human response per invocation |

Feature `reason` has exactly these values:

| Reason | Meaning |
|---|---|
| `not_implemented` | Target behavior has no admitted implementation |
| `binding_missing` | A required producer-owned binding is absent |
| `policy_disabled` | Static policy disables the feature |
| `dependency_unavailable` | A required admitted dependency is unavailable |
| `unhealthy` | No healthy serving instance is available |
| `maintenance` | New work is temporarily paused |
| `capacity_exhausted` | Bounded capacity cannot accept new work |
| `unsupported_profile` | The current Local or Hosted profile does not implement the feature |

## 4. Runtime Project and source model

### 4.1 Compilation unit

A Runtime Project MUST be the unit of validation, compilation, publication, and rollback. Changing one function
MUST rebuild only its project. It MUST NOT rebuild the Runtime or another project.

The Runtime owns all paths and generated files. The authoring client MUST NOT select a path, edit Cargo files,
invoke a build command, or access the filesystem.

The logical layout is:

```text
src/
├── operations/<name>.rs
├── functions/<name>.rs
├── pages/<name>.rs
├── events/<name>.rs
├── realtime/<name>.rs
└── generated/
    ├── registry.rs
    └── schemas.rs
```

This layout is Runtime-owned. It is not a client request format.

### 4.2 Function kinds

| Kind | Discoverable | Invocable externally | Purpose |
|---|---:|---:|---|
| `operation` | Yes | When an adapter is declared and bound | Typed query or command |
| `function` | No | No | Small internal helper |
| `page` | Yes | HTTP `GET` and `HEAD` only | Server-rendered page |
| `event` | Yes | Event delivery only | Durable event consumer |
| `realtime` | Yes | Realtime frame only | Connection event consumer |

An `operation` has one semantic kind:

| Operation kind | State changes allowed | Retry expectation |
|---|---:|---|
| `query` | No | Safe to retry |
| `command` | Yes | Requires Runtime idempotency |

### 4.3 One function per file

Each Runtime Function MUST occupy one Runtime-owned file and MUST contain exactly one primary callable. Shared types
are generated from the structured input and output schemas. Generated registries and schemas are excluded from
authoring limits.

The Runtime MUST apply `rustfmt` before measuring source. Initial reviewability limits are:

| Measure | Maximum |
|---|---:|
| Formatted lines in the primary callable | 50 |
| Parameters excluding `Context` | 8 |
| Nested block depth | 5 |
| Branch points | 12 |
| Cyclomatic complexity | 12 |
| AST nodes in the primary callable | 2,000 |

Only static Runtime policy may change these maxima. A project or authoring client MAY request stricter values and
MUST NOT raise them. Exceeding a limit returns `source_limit_exceeded` with the measured value and a suggested split.
These limits improve navigation and review; they are not an execution-security boundary.

## 5. Structured control protocol

### 5.1 Envelope

Every control request has this shape:

```json
{
  "protocol_version": "1",
  "request_id": "018f3a8c-7c15-7b2a-9ef1-1d2cd5f35c41",
  "method": "function.describe",
  "params": {
    "project": "orders",
    "name": "create_order"
  }
}
```

`request_id` MUST be a lowercase UUIDv7 and is the idempotency identity for a mutating control request. Repeating the exact
request returns the original result. Reusing it with different canonical params returns `idempotency_conflict`.

A successful response contains `result`; an unsuccessful response contains `error`. It MUST NOT contain both.
Protocol framing, authentication, authorization, and request-admission failures use envelope `error`. Once an
Invocation is durably admitted, its accepted state or terminal Outcome is always returned inside envelope `result`,
including a failed, rejected, cancelled, or timed-out Outcome that contains its own `error`.

```json
{
  "protocol_version": "1",
  "request_id": "018f3a8c-7c15-7b2a-9ef1-1d2cd5f35c41",
  "result": {}
}
```

### 5.2 Methods

| Method | Mutates | Required params | Result |
|---|---:|---|---|
| `runtime.describe` | No | none | Features, versions, and effective limits |
| `projects.list` | No | optional `cursor`, `limit` | Bounded project summaries and next cursor |
| `project.create` | Yes | `name`, `description` | Project revision |
| `project.describe` | No | `project` | Project, revision, functions, dependency state |
| `project.delete` | Yes | `project`, `expected_revision` | Deletion state |
| `function.create` | Yes | function request in Section 5.3 | Function revision |
| `function.update` | Yes | function request plus `expected_revision` | Function revision |
| `function.delete` | Yes | `project`, `name`, `expected_revision` | Project revision |
| `function.describe` | No | `project`, `name` | Source metadata, schema, relationships, validation |
| `function.test` | No | `project`, `name`, `cases` | Per-case structured outcomes |
| `project.validate` | Yes | `project`, `revision` | Validation report |
| `project.build` | Yes | `project`, `revision` | Build identifier and state |
| `build.describe` | No | `build_id` | Current build state and bounded report |
| `build.cancel` | Yes | `build_id`, optional `reason` | Cancellation state |
| `project.publish` | Yes | `project`, `artifact_digest` | Version and activation state |
| `project.rollback` | Yes | `project`, `artifact_digest` | Activation state |
| `project.versions` | No | `project`, optional `cursor`, `limit` | Immutable versions |
| `dependency.list` | No | optional `project` | Available or selected packages |
| `dependency.describe` | No | `name`, optional `version` | Approved versions, features, and risk |
| `dependency.request` | Yes | request in Section 7.2 | Approval request and dependency state |
| `project.operations` | No | `project`, optional `cursor`, `limit` | Bounded operation summaries |
| `operation.describe` | No | `project`, `operation` | Full catalog entry |
| `operation.invoke` | Yes | invocation request in Section 8.1 | Accepted state or terminal Outcome |
| `invocation.describe` | No | `invocation_id` | Current state and safe metadata |
| `invocation.cancel` | Yes | `invocation_id`, optional `reason` | Cancellation state |
| `human.respond` | Yes | response in Section 18.3 | Resumed invocation state |

`human.respond` is available only to an admitted trusted human-response adapter. It MUST NOT be exposed as a Brain
authoring tool.

List methods sort by stable identifier, default to `limit = 50`, accept `limit = 1..=200`, and return an opaque
`next_cursor` only when more results exist. A cursor is bound to the authenticated Team, method, filters, and exact
snapshot; mismatch or expiry returns `invalid_request`.

### 5.3 Function create and update

```json
{
  "project": "orders",
  "name": "create_order",
  "kind": "operation",
  "operation_kind": "command",
  "description": "Creates an order",
  "input_schema": { "$ref": "CreateOrder" },
  "output_schema": { "$ref": "OrderCreated" },
  "required_permissions": ["orders.create"],
  "required_capabilities": ["database", "events"],
  "effects": ["database_write", "event_publish"],
  "timeout_ms": 8000,
  "code": "#[operation]\npub async fn create_order(ctx: Context, input: CreateOrder) -> Result<OrderCreated> { /* ... */ }"
}
```

Required fields for every kind are `project`, `name`, `kind`, `description`, and `code`. Schema requirements are:

| Kind | Input schema | Output schema |
|---|---|---|
| `operation` | Required | Required |
| `function` | Optional; derived from the Rust signature when absent | Optional; derived from the Rust signature when absent |
| `page` | Optional; defaults to an empty object | Fixed by the Runtime `Page` type and MUST be omitted |
| `event` | Required event payload schema | Fixed to unit and MUST be omitted |
| `realtime` | Required frame payload schema | Fixed to `RealtimeEffect` and MUST be omitted |

`operation_kind` is REQUIRED only for `operation`. Optional arrays default to empty. `timeout_ms` defaults to the
Runtime deadline and can only lower it. Update additionally requires `expected_revision`.

The complete declared effect set is:

| Effect | Required capability |
|---|---|
| `database_read` | `database` |
| `database_write` | `database` |
| `outbound_http_read` | `outbound_http` |
| `outbound_http_write` | `outbound_http` |
| `realtime_send` | `realtime` |
| `event_publish` | `events` |
| `storage_read` | `storage` |
| `storage_write` | `storage` |
| `secret_use` | `secrets` |
| `project_query` | `project_calls` |
| `project_command` | `project_calls` |
| `human_input` | `human_input` |
| `session_read` | `session` |
| `session_write` | `session` |

A query MAY declare only `database_read`, `outbound_http_read`, `storage_read`, `secret_use`, `project_query`, and
`session_read`. Unknown effects are rejected. Declaring an effect without its capability, or using a capability
effect not declared here, fails validation.

The Runtime MUST derive the source path, module, imports, registry, schemas, Cargo metadata, formatting, tests,
build, and artifact manifest. It MUST reject code whose declared function name, kind, signature, or effects differ
from the structured request.

### 5.4 Test cases

Each `function.test` case contains `name`, `input`, optional capability fixtures, and exactly one expectation:
`output`, `error_code`, or `suspends_for_human`. Tests MUST run in the isolated build/test boundary without
production secrets or production data.

### 5.5 Adapter declarations

Adapter declaration is closed and kind-specific:

| Function kind | Declaration | Generated catalog adapters |
|---|---|---|
| `operation` | `#[operation(...)]` | `control`; optional `project_call` when `project_call = true`; optional `http` when `#[http(...)]` is present |
| `function` | `#[function]` | none |
| `page` | `#[page(path = "...")]` | `http` |
| `event` | `#[event(name = "...")]` | `event` |
| `realtime` | `#[websocket(event = "...")]` | `realtime` |

An adapter is callable only when its feature and external binding are available. Unknown attributes, duplicate
adapters, and an adapter that does not match the function kind fail source validation.

## 6. Project lifecycle

### 6.1 Revisions and versions

A project revision is an exact source snapshot. A project version is an immutable successful artifact. A revision
MUST pass validation before build. An artifact MUST pass build and health validation before publication.

Revision states are:

| State | Meaning | Allowed next states |
|---|---|---|
| `draft` | Unvalidated source snapshot | `validating`, `deleted` |
| `validating` | Deterministic validation is running | `valid`, `invalid` |
| `invalid` | Validation failed | `draft`, `deleted` |
| `valid` | Revision can build | `building`, `draft`, `deleted` |
| `building` | Isolated build is running | `build_succeeded`, `build_failed`, `build_cancelled` |
| `build_failed` | Build failed | `draft`, `building`, `deleted` |
| `build_cancelled` | Build cancellation committed | `deleted` |
| `build_succeeded` | Immutable artifact was created | `deleted` |
| `deleted` | Revision-owned residue is absent | none |

A transition shown from `invalid`, `valid`, or `build_failed` to `draft` creates a new successor revision; it does
not mutate the prior source snapshot.

A successful build creates a version in `built`. Version states are:

| State | Meaning | Allowed next states |
|---|---|---|
| `built` | Immutable artifact exists | `starting`, `deleted` |
| `starting` | Candidate worker is health checked | `healthy`, `start_failed` |
| `start_failed` | Candidate could not become healthy | `starting`, `deleted` |
| `healthy` | Candidate may activate | `active`, `inactive`, `deleted` |
| `active` | Receives new invocations | `draining` |
| `draining` | Receives no new invocations | `inactive` |
| `inactive` | Retained for rollback or deletion | `active`, `deleted` |
| `deleted` | Version-owned residue is absent | none |

Only one version per project MAY be `active`. Publication and rollback MUST atomically select an exact artifact
digest for new invocations. An in-flight invocation MUST keep its selected digest. Drain timeout defaults to the
60-second maximum synchronous invocation deadline. At expiry, unfinished execution attempts lose capability commit
authority and move to `retry_pending` against the same exact digest unless their invocation deadline has elapsed.
The serving version becomes `inactive`, but remains loadable for previously pinned invocations and chain nodes.

### 6.2 Deletion

Deletion is idempotent. It succeeds only when Runtime-owned source, generated files, artifacts, version pointers,
invocations subject to retention, jobs, outbox entries, human requests, and caches are absent. Partial deletion MUST
remain incomplete, identify the retry owner, and MUST NOT report success. Unknown content outside Runtime ownership
MUST be preserved.

## 7. Validation, dependencies, and build

### 7.1 Validation pipeline

The Runtime MUST execute these steps in order:

1. validate the structured request and schemas;
2. parse Rust source;
3. apply `rustfmt`;
4. enforce one primary callable and source limits;
5. compare signature, declaration, permissions, effects, and capabilities;
6. reject prohibited imports and APIs;
7. generate registries and schemas;
8. run `cargo check`;
9. run Clippy and Runtime lints;
10. inspect dependency metadata and advisories;
11. run function and project tests;
12. build without network in an isolated builder;
13. run isolated integration and health tests;
14. produce an immutable artifact and provenance manifest.

An optional model-based security review MAY report suspicious logic. It MUST NOT replace any deterministic step or
serve as the primary security boundary.

### 7.2 Dependencies

The authoring client can list and describe admitted packages and can request a new exact package selection. It
cannot edit Cargo files, run Cargo, choose a registry or Git source, update a lockfile, or install a package.

```json
{
  "project": "analytics",
  "name": "polars",
  "version": "0.51.0",
  "features": ["lazy"],
  "reason": "Aggregate uploaded datasets"
}
```

An exact package version has availability `available` or `unavailable`. A dependency change request has this state
machine:

| State | Meaning | Allowed next states |
|---|---|---|
| `requested` | Exact add, update, or removal request exists | `analyzing` |
| `analyzing` | Metadata, advisories, and transitive tree are resolved without build | `waiting_human`, `blocked` |
| `waiting_human` | Exact resolution awaits mandatory approval | `approved`, `denied`, `blocked` |
| `approved` | Exact resolution is authorized | `selected`, `blocked` |
| `denied` | Human denied the request | none |
| `blocked` | Static Runtime policy prohibited the request | none |
| `selected` | Exact approved resolution is in the project lock | none |

Every add, update, or removal MUST enter `waiting_human`; project policy cannot disable that gate. Updating or
removing a selected dependency creates a new request and does not mutate the completed request.

Risk has these values:

| Risk | Examples | Result |
|---|---|---|
| `low` | Rust-only package with no build script | Human approval still required |
| `medium` | Build script, procedural macro, or material `unsafe` | Human approval plus expanded report |
| `high` | FFI, native library, or external source | Human approval plus static Runtime allow decision |
| `critical` | Severe advisory, unverifiable source, or prohibited behavior | Runtime MUST block even after human approval |

Approval MUST bind package, exact version, features, lockfile digest, transitive-tree digest, requester, approver,
and expiry. Any resolution change invalidates approval.

### 7.3 Build isolation and provenance

The builder MUST have no production secrets, production database access, cross-Team project access, or ambient
credentials. Its filesystem is temporary; network is denied during compilation; CPU, memory, PIDs, storage, and
time are bounded. Only expected artifacts and a bounded report may leave it.

The provenance manifest MUST bind canonical source digest, generated-code digest, exact dependency lock digest,
SDK digest, toolchain artifact digest, build policy revision, test result digest, and output artifact digest.
Rust 1.98.0 is the proposed baseline, but no exact toolchain artifact or checksum is admitted by this Draft.

## 8. Invocation and Outcome

### 8.1 Caller request versus canonical Invocation

An `operation.invoke` request accepts only:

```json
{
  "project": "orders",
  "operation": "create_order",
  "input": { "customer_id": "cus_123", "total": "99.90" },
  "idempotency_key": "checkout_018f3a8c",
  "deadline_unix_ms": 1787421600000,
  "response_mode": "wait"
}
```

`idempotency_key`, `deadline_unix_ms`, and `response_mode` are optional. `response_mode` is `wait` or `accepted` and
defaults to `wait`. `accepted` returns the durable invocation id and current state immediately. `wait` returns a
terminal Outcome, except that it returns an accepted non-terminal result when the Invocation reaches
`waiting_human` or the control-request wait expires. Those cases use `pending_reason = human_required` and
`pending_reason = wait_timeout`, respectively. A wait timeout does not change or cancel the Invocation.

```json
{
  "invocation_id": "inv_018f3a8c",
  "state": "waiting_human",
  "pending_reason": "human_required"
}
```

A caller MUST NOT supply Team identity, principal, delegation, artifact digest, permissions, trusted metadata, or
capability bindings.

The Runtime derives a canonical internal Invocation from authenticated evidence and the active version:

```rust
pub struct Invocation {
    pub id: InvocationId,
    pub root_id: InvocationId,
    pub parent_id: Option<InvocationId>,
    pub team_id: TeamId,
    pub project: ProjectId,
    pub operation: OperationId,
    pub artifact_digest: ArtifactDigest,
    pub input: serde_json::Value,
    pub principal: RuntimePrincipal,
    pub delegation: Vec<DelegationEvidence>,
    pub source: InvocationSource,
    pub deadline_unix_ms: i64,
    pub idempotency_key: IdempotencyKey,
}
```

The Rust shape is illustrative. The separation between untrusted caller fields and derived fields is normative.

`InvocationSource` has the values `http`, `realtime`, `event`, `project_call`, and `control`.

`RuntimePrincipal.kind` has exactly these values:

| Kind | Source |
|---|---|
| `account` | Authenticated Hosted identity |
| `assistant` | Admitted Assistant machine identity |
| `service` | Admitted Service machine identity |
| `system` | Narrow Runtime-owned lifecycle authority |
| `anonymous` | Public page request with no authenticated subject |

`anonymous` is a Runtime-local principal value, not a Shimpz actor. It is allowed only for a page declaring
`auth = none`, remains bound to the page's owning Team, and has no caller permissions, delegation, or command
authority. The page MAY use only explicitly admitted read effects. Function-level `session_write` is denied; the
Runtime MAY issue its own bounded anti-CSRF session state without exposing session-write authority to the function.

### 8.2 State machine

| State | Meaning | Allowed next states |
|---|---|---|
| `pending` | Invocation exists durably | `authorizing`, `cancelled`, `timed_out` |
| `authorizing` | Identity, schema, and policy checks run | `waiting_human`, `queued`, `rejected`, `cancelled`, `timed_out` |
| `waiting_human` | A durable typed response is required | `authorizing`, `rejected`, `cancelled`, `timed_out` |
| `queued` | Eligible for execution | `running`, `cancelled`, `timed_out` |
| `running` | Exact artifact is executing | `succeeded`, `retry_pending`, `failed`, `cancelled`, `timed_out` |
| `retry_pending` | A retryable attempt is delayed | `queued`, `cancelled`, `timed_out`, `failed` |
| `succeeded` | Valid output committed | none |
| `failed` | Terminal execution failure | none |
| `rejected` | Authorization or policy denied execution | none |
| `cancelled` | Cancellation committed | none |
| `timed_out` | Deadline elapsed | none |

Terminal states are `succeeded`, `failed`, `rejected`, `cancelled`, and `timed_out`. Exactly one terminal state MUST
be committed. Invalid transitions return `state_conflict`.

### 8.3 Outcome

```json
{
  "invocation_id": "inv_018f3a8c",
  "state": "succeeded",
  "output": { "order_id": "ord_123" },
  "metadata": {
    "artifact_digest": "sha256:0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
    "duration_ms": 18
  }
}
```

A successful Outcome has `output` and no `error`. Any other terminal Outcome has `error` and no `output`.
Metadata MUST be Runtime-generated, bounded, and free of protected payloads and secrets.

### 8.4 Idempotency, retry, cancellation, and deadlines

- Idempotency scope is Team, project, operation, and authenticated principal.
- A command key is derived by source: control uses `request_id`; HTTP requires an admitted `Idempotency-Key` header;
  realtime uses connection identity plus `message_id`; event uses `event_id` plus handler; and project call uses
  parent invocation id plus a stable child ordinal. A missing HTTP command key returns `invalid_request`.
- Repeating a key with the same canonical input returns the existing invocation or Outcome.
- Repeating a key with different canonical input returns `idempotency_conflict`.
- Execution and event delivery are at least once. Function code MUST tolerate replay; Runtime capability writes MUST
  use the invocation idempotency key where supported.
- Only one terminal Outcome is committed, even when an attempt runs more than once.
- A child deadline is the minimum of parent remaining time, operation timeout, and static Runtime maximum.
- Cancellation is cooperative during execution and mandatory before starting queued work. A result racing with
  cancellation is resolved by the first terminal state committed.
- A timed-out or cancelled attempt MUST lose authority to commit new capability effects after the terminal state.

## 9. Mandatory invocation pipeline

Every source protocol MUST converge on the same canonical Invocation and execute this ordered pipeline:

1. framing, size, version, and unknown-field validation;
2. Team binding and Runtime principal derivation from trusted evidence;
3. input schema validation;
4. static Runtime policy;
5. admitted Team and project policy;
6. function-declared permission, effect, and capability restrictions;
7. optional durable human gate;
8. bounded execution;
9. output schema validation;
10. outcome commit and metadata-only audit.

Every stage is mandatory even when its result is allow. A stage MAY short-circuit only with a structured terminal
error. Audit runs for success, failure, rejection, cancellation, and timeout.

Policy precedence is:

```text
static Runtime policy > admitted Team/project policy > function declaration
```

A lower layer MAY restrict more and MUST NOT relax a higher layer.

## 10. Operation catalog

Every successful project build MUST generate a catalog for every discoverable function. Each entry contains:

| Field | Required | Meaning |
|---|---:|---|
| `name` | Yes | Qualified semantic name |
| `description` | Yes | Human and model-readable purpose |
| `kind` | Yes | `query`, `command`, `page`, `event`, or `realtime` |
| `input_schema` | Yes | Exact JSON Schema |
| `output_schema` | Yes | Exact JSON Schema |
| `permissions` | Yes | Required permission identifiers |
| `effects` | Yes | Declared effect identifiers |
| `capabilities` | Yes | Required Runtime capabilities |
| `adapters` | Yes | Any of `http`, `realtime`, `event`, `project_call`, `control` |
| `previous` | Yes | Incoming chain relationships, possibly empty |
| `next` | Yes | Outgoing chain relationships, possibly empty |
| `human_input` | No | Typed human request declaration |
| `examples` | Yes | Bounded valid examples |
| `artifact_digest` | Yes | Exact producing version |

Discovery is progressive: `projects.list` to `project.operations` to `operation.describe` to `operation.invoke`.
The Runtime MUST NOT inject the full catalog into model context by default.

## 11. Context capabilities

Rust functions receive a `Context`. A capability is available only when declared by the function, admitted by
static policy, authorized by the Team binding, and present at invocation time.

Feature names describe Runtime subsystems; capability names describe per-function authority. They are separate
namespaces. The complete capability set is `database`, `outbound_http`, `realtime`, `events`, `project_calls`,
`storage`, `secrets`, `session`, and `human_input`.

| Handle | Operations | Prohibited behavior |
|---|---|---|
| `ctx.db()` | typed query, execute, transaction | Raw host, port, TLS material, credentials, or pool control |
| `ctx.http()` | allowed-host outbound request | Arbitrary destination, ambient proxy, or raw credential access |
| `ctx.realtime()` | send, publish, close | Socket ownership, heartbeat, or connection placement |
| `ctx.events()` | publish durable event | Queue or worker administration |
| `ctx.projects()` | typed query or command | Cross-Team discovery or transport selection |
| `ctx.storage()` | bounded object get, put, delete | Host filesystem access |
| `ctx.secrets()` | obtain a non-serializable scoped secret handle | Listing, logging, exporting, or receiving unrelated secrets |
| `ctx.session()` | typed get, set, and delete for the bound page session | Raw cookie or another session's access |

`human_input` has no Context handle; its admitted answer is delivered only through the typed function parameter in
Section 18. An unavailable bound handle returns `capability_unavailable` before performing an effect. An undeclared
or policy-denied handle returns `permission_denied`. Raw environment variables, ambient network, host filesystem,
and ambient credentials MUST NOT be function capabilities.

## 12. Backend and HTTP adapter

HTTP exists for APIs, pages, webhooks, assets, SEO, and traditional clients. It is not replaced by realtime.

An operation MAY declare one HTTP adapter:

```rust
#[operation(description = "Creates an order", kind = "command")]
#[http(method = "POST", path = "/orders", success = "created")]
pub async fn create_order(
    ctx: Context,
    input: CreateOrder,
) -> Result<OrderCreated> {
    // Functionality only.
}
```

Supported method values are `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, and `DELETE`. `OPTIONS` is generated by the
Runtime. A query MAY use `GET` or `HEAD`. A command MUST use `POST`, `PUT`, `PATCH`, or `DELETE`.

`#[http]` accepts exactly `method`, `path`, and optional `success`. Success values are `ok` (`200`, the default),
`created` (`201`), and `no_content` (`204`). `created` requires a non-unit output; when that schema contains a
validated optional `location` URI field, the Runtime emits it as `Location`. `no_content` requires a unit output.
Queries MUST use `ok`.

Path parameters, query parameters, admitted headers, and body are decoded against the operation's generated schema.
Unknown path or query fields and non-admitted headers are not passed to function code. JSON request bodies use
`application/json`. Page results use `text/html; charset=utf-8`. Assets use their registered media type.
Every HTTP command requires a valid `Idempotency-Key`; the Runtime consumes it and does not expose it to function
code. HTTP queries MAY include the header.

HTTP defaults to `response_mode = wait`. `Prefer: respond-async` selects `accepted`; `Prefer: wait=N` sets an HTTP
wait ceiling of `0..=60` seconds. When both are present, `respond-async` takes precedence. An honored preference is
returned in `Preference-Applied`. Reaching `waiting_human`, honoring `respond-async`, or reaching the HTTP wait
ceiling returns `202 Accepted` with the accepted result from Section 8.1, an admitted status URI in `Location`, and
an optional bounded `Retry-After`. Unknown preference tokens follow RFC 7240 and do not change strict JSON parsing.

The HTTP adapter maps Runtime outcomes as follows:

| Outcome or error code | HTTP status |
|---|---:|
| `success = ok` | `200` |
| `success = created` | `201` |
| accepted non-terminal Invocation | `202` |
| `success = no_content` | `204` |
| `invalid_request`, `unknown_field`, `unsupported_protocol_version`, `schema_invalid` | `400` |
| `authentication_failed` | `401` |
| `permission_denied`, `policy_denied` | `403` |
| `not_found` | `404` |
| `idempotency_conflict`, `state_conflict`, `human_invalidated`, `human_expired`, `cancelled` | `409` |
| `payload_too_large` | `413` |
| `semantic_validation_failed`, `operation_failed` | `422` |
| `rate_limited` | `429` |
| `feature_unavailable`, `capability_unavailable`, `artifact_unavailable`, `upstream_unavailable` | `503` |
| `deadline_exceeded` | `504` |
| `call_cycle`, `output_invalid`, `internal_error` | `500` |

The adapter derives identity from its admitted external binding. Function code MUST NOT parse authentication headers.
Route conflicts, ambiguous parameter shapes, or duplicate method and path pairs fail project validation.
Source, dependency, build, and publication errors are control-protocol errors and MUST NOT be emitted by an active
HTTP operation adapter. If invalid active state would otherwise expose one, the adapter emits `internal_error`.

## 13. Realtime adapter

The Runtime owns physical WebSocket connections, authentication binding, heartbeat, session association,
backpressure, fan-out, serialization, and connection placement. A function receives events and returns effects; it
never owns a socket. No process or worker is pinned to one connection.

```rust
#[websocket(event = "chat.message")]
pub async fn receive_message(
    ctx: Context,
    message: ChatMessage,
) -> Result<RealtimeEffect> {
    ctx.realtime().publish("chat.responses", &message).await?;
    Ok(RealtimeEffect::None)
}
```

### 13.1 Client frames

Text frames contain one strict JSON object. Binary frames are unsupported in v1.
`message_id` MUST be a lowercase UUIDv7. A topic contains `1..=8` local-name segments separated by `.`, is at most
128 bytes, and admits no wildcard.

| `type` | Required fields | Meaning |
|---|---|---|
| `invoke` | `message_id`, `handler`, `input` | Invoke one `#[websocket]` handler |
| `subscribe` | `message_id`, `topic` | Subscribe after authorization |
| `unsubscribe` | `message_id`, `topic` | Remove a subscription |
| `ping` | `message_id` | Request `pong` |

### 13.2 Server frames

| `type` | Required fields | Meaning |
|---|---|---|
| `outcome` | `message_id`, `invocation_id`, `state`, `output` or `error` | Terminal invocation result |
| `event` | `event_id`, `topic`, `payload` | Published realtime event |
| `error` | `message_id`, `error` | Frame-level failure |
| `pong` | `message_id` | Ping response |

Frames are ordered per connection in the sequence accepted by the Runtime. Delivery to an active socket is at most
once. Reconnect does not replay realtime frames; durable replay requires the Events capability.

A realtime function MAY return exactly these effects: `none`, `send_connection`, `publish_topic`, or `close`.
Publish scopes are `connection`, `session`, `team`, and `topic`; every scope is authorization checked.

Default heartbeat is 30 seconds with a 10-second response grace. Effective values are discoverable and static policy
may lower them. When an outbound queue exceeds either 128 frames or 1 MiB, the Runtime MUST close the connection;
it MUST NOT silently drop an accepted frame.

Application close codes are:

| Code | Meaning |
|---:|---|
| `4400` | Invalid frame |
| `4401` | Authentication failed |
| `4403` | Authorization denied |
| `4408` | Heartbeat or invocation timeout |
| `4409` | State or idempotency conflict |
| `4429` | Rate, payload, or backpressure limit |
| `4500` | Internal Runtime error |
| `4503` | Realtime feature unavailable or no healthy version available |

Project publication MUST NOT close Runtime-owned connections. Each accepted `invoke` frame captures the active
artifact digest at dispatch; a later frame can use a newer active digest.

## 14. Events

```rust
#[event(name = "orders.created")]
pub async fn handle_order_created(
    ctx: Context,
    event: OrderCreated,
) -> Result<()> {
    // Durable consumer.
}
```

An event contains `event_id`, `name`, `schema_version`, `producer_project`, `producer_operation`, `artifact_digest`,
`occurred_at`, optional `partition_key`, and `payload`.

An event name is `<namespace>.<event>` and each segment follows the local-name grammar in Section 2.2. Event names
and qualified operation names are interpreted only in their typed field and are never interchangeable.
`event_id` MUST be a lowercase UUIDv7. `schema_version` is a positive 32-bit integer. `partition_key`, when present,
is `1..=128` bytes of UTF-8 and is treated as opaque.

Delivery is at least once. Consumers MUST be idempotent by `event_id`. Events with the same non-empty
`partition_key` are delivered in publish order to one consumer group; no ordering is guaranteed across partitions
or when the key is absent.

Retry policy has these options:

| Field | Values | Default |
|---|---|---|
| `strategy` | `fixed`, `exponential` | `exponential` |
| `max_attempts` | `1..=20` | `8` |
| `initial_delay_ms` | `100..=60000` | `1000` |
| `max_delay_ms` | `1000..=3600000` | `300000` |

After the final attempt, the job enters `dead_letter`. Requeue requires an authorized lifecycle request and preserves
the original event id. Publishing an event alongside a database mutation MUST use a transactional outbox so either
both commit or neither commits.

A minimal implementation MAY use Postgres jobs, invocations, outbox, dead-letter rows, and
`FOR UPDATE SKIP LOCKED`. That storage choice is illustrative and does not define the PostgreSQL Service binding.

## 15. Server-rendered pages

Runtime pages cover dynamic dashboards, administration interfaces, forms, and agent interfaces. They do not build
static Svelte and Tailwind marketing sites.

```rust
#[page(path = "/orders")]
pub async fn orders_page(ctx: Context) -> Result<Page> {
    let orders = ctx.db().query_as("SELECT id, status FROM orders").await?;
    Ok(page!("orders", { orders }))
}
```

A page declaration supports:

| Option | Values or shape |
|---|---|
| `path` | Absolute route with typed path parameters |
| `render` | `full`, `fragment`; default `full` |
| `cache` | `no_store`, `private`, `public`; default `no_store` |
| `auth` | `required`, `optional`, `none`; default `required` |
| `realtime` | Optional authorized topic bindings |

For `auth = required`, missing or invalid evidence fails. For `auth = optional`, valid evidence derives its normal
principal, absence derives `anonymous`, and invalid supplied evidence fails. For `auth = none`, the page always uses
`anonymous` and no caller credential is passed to function code.

`Page` can contain layout, typed components, forms, assets, session effects, status, safe response headers, and head
metadata. Head metadata supports `title`, `description`, `canonical`, `robots`, `hreflang`, `open_graph`, `twitter`,
and validated JSON-LD. This is the complete v1 SEO and answer-engine metadata set.

Text is escaped by default. Raw HTML MUST require a separately admitted sanitized type and MUST NOT be constructed
from untrusted strings. The Runtime MUST generate a restrictive Content Security Policy, immutable digest-named
assets, width and height for media where known, and no inline executable script by default. Forms use generated
schemas, CSRF protection where applicable, and the same invocation pipeline as other operations.

## 16. Database and other capability behavior

### 16.1 Database

`ctx.db()` is available only through an admitted Team-to-Runtime database binding. The function receives no host,
port, TLS material, credentials, provisioning control, or cross-project database visibility.

The v1 handle supports `query`, `query_as`, `execute`, and `transaction`. Transactions support `read_only` and
`read_write`; default is `read_write` for commands and `read_only` for queries. Nested transactions are rejected.
The Runtime MUST apply the project-specific database role and bound pool automatically.

### 16.2 Outbound HTTP

`ctx.http()` accepts method, HTTPS URL, bounded headers, bounded body, and deadline. Supported methods are `GET`,
`HEAD`, `POST`, `PUT`, `PATCH`, and `DELETE`. The destination host, port, resolved address, redirect target, method,
and credential handle MUST all satisfy the admitted allow policy. Plain HTTP, arbitrary CONNECT, private-address
resolution, and undeclared redirects are denied by default.
Queries MAY use only `GET` and `HEAD`; commands and non-operation functions use the methods admitted by their effect
declaration and policy.

### 16.3 Storage

`ctx.storage()` supports `get`, `put`, `delete`, `head`, and bounded prefix `list` inside the invocation's bound
namespace. Keys are opaque relative names; absolute paths and parent traversal are rejected. Objects are isolated by
Team and Runtime Project.

### 16.4 Secrets

`ctx.secrets()` resolves only a declared secret identifier. Returned handles MUST redact debug output, refuse
serialization and cloning into an Outcome, and expire no later than the invocation. Secrets MUST NOT enter source,
builds, logs, traces, events, catalogs, audit, or error details.

## 17. Calls between Runtime Projects

A function calls another Runtime Operation only through:

```rust
ctx.projects().query("customers.find", input).await?;
ctx.projects().command("billing.create_invoice", input).await?;
ctx.events().publish("orders.created", &order).await?;
```

The three semantic choices are:

| Choice | Meaning |
|---|---|
| `query` | Request data without changing state |
| `command` | Request an authorized state change |
| `event` | Publish a durable fact that already occurred |

The Runtime selects transport and exact active artifact. The call becomes a child Invocation, inherits Team and
delegation evidence, uses the parent as `parent_id`, and receives the remaining parent deadline. Cross-Team calls
fail as `not_found`. A callee that did not declare `project_call = true` also fails as `not_found`, without revealing
that the operation exists. Maximum call depth is 16. A synchronous call cycle detected at build time or runtime
returns `call_cycle`.

This surface is Runtime-local. Any call that crosses a Runtime or another domain requires a separately admitted
producer-owned binding.

## 18. Durable chains and human input

### 18.1 Chain relationships

The supported triggers are:

| Trigger | Input to dependent function | Runs when |
|---|---|---|
| `after_success` | Predecessor output type | Predecessor succeeds; this is the default |
| `after_failure` | `Failure` | Predecessor reaches `failed` |
| `after_finish` | `Completion<T>` | Predecessor reaches any terminal state |

`Failure` contains safe `code`, `retryable`, and `attempts` fields and no predecessor input, output, or protected
error details. `Completion<T>` has exactly the variants `Succeeded(T)`, `Failed(Failure)`, `Rejected`, `Cancelled`,
and `TimedOut`.

```rust
#[operation(description = "Analyzes fraud", kind = "command")]
#[after_success(operation = "orders.create_order")]
pub async fn analyze_fraud(
    ctx: Context,
    order: OrderCreated,
) -> Result<FraudAnalysis> {
    // Typed output-to-input continuation.
}
```

The compiler MUST validate relationship names, trigger types, and output-to-input compatibility. The graph MUST be
acyclic. Multiple dependents of one predecessor are queued in parallel. A sequence is expressed by each function
pointing to its immediate predecessor.

The root invocation pins the complete chain graph and every project artifact digest used by that graph. Publication
does not rewrite an active chain. Completion persistence and creation of dependent invocations and jobs MUST commit
in one transaction. Delivery is at least once, while the invocation state machine commits one terminal Outcome per
node.

Runtime chains are local execution relationships and MUST NOT be named, exposed as, or treated as Routine.

### 18.2 Human input kinds

An operation MAY declare at most one human input. The supported kinds are:

| Kind | Answer |
|---|---|
| `confirm` | `true` or `false` |
| `choice` | Exactly one generated enum value |
| `choices` | One or more unique generated enum values |
| `text` | Bounded UTF-8 string |
| `form` | Object validated by generated JSON Schema |
| `file` | Admitted storage object reference, never inline bytes |
| `date` | RFC 3339 full-date |
| `datetime` | RFC 3339 timestamp |

```rust
#[operation(description = "Approves a refund", kind = "command")]
#[human(
    kind = "confirm",
    prompt = "Approve this refund?",
    required_permission = "refunds.review",
    ttl_seconds = 900
)]
pub async fn approve_refund(
    ctx: Context,
    refund: Refund,
    approval: Human<Confirm>,
) -> Result<RefundApproved> {
    // Runs only after the admitted response.
}
```

### 18.3 Suspension and response

The Runtime does not preserve an async stack. Before execution it persists a human request and moves the Invocation
to `waiting_human`. A valid answer returns the same Invocation to `authorizing`, then invokes the exact artifact with
the immutable typed answer in the declared `Human<T>` function parameter. That parameter counts toward the function
parameter limit.

```json
{
  "human_request_id": "human_018f3a8c",
  "answer": { "kind": "confirm", "value": true }
}
```

The request binds invocation, Team, required permission, prompt, answer schema, canonical input digest, operation
digest, artifact digest, creation time, expiry, and response authority. Changing input, schema, operation, artifact,
or binding invalidates the request. Every response is single-use and audited without prompt or answer payload.

This proposed state machine does not change the current Action human-request contract and cannot operate until its
Team-owned integration is separately admitted.

## 19. Structured policy conditions

Policy conditions are data, never executable text. The complete expression set is:

```json
{
  "and": [
    {
      "field": "input.total",
      "operator": "greater_than",
      "value": { "type": "money", "amount": "1000.00", "currency": "BRL" }
    },
    {
      "not": {
        "field": "principal.kind",
        "operator": "equals",
        "value": { "type": "enum", "value": "system" }
      }
    }
  ]
}
```

Nodes are comparison objects containing `field`, `operator`, and `value`, or logical objects containing exactly one
of `and`, `or`, or `not`. `and` and `or` contain `1..=32` nodes. `not` contains exactly one. Maximum expression
depth is 8.

| Operator | Accepted value types |
|---|---|
| `equals`, `not_equals` | all listed types |
| `greater_than`, `greater_or_equal`, `less_than`, `less_or_equal` | integer, decimal, money, date, datetime |
| `in`, `not_in` | string, integer, enum, UUID against a list of `1..=128` unique values |
| `contains`, `starts_with`, `ends_with` | string |

Value types are `boolean`, `string`, `integer`, `decimal`, `money`, `date`, `datetime`, `uuid`, and `enum`. Field
identifiers MUST resolve to a compiled schema or Runtime-derived principal field. Type mismatch rejects the policy.

Conditions MUST NOT contain eval, closures, SQL, scripts, function calls, network calls, or regular expressions.
Function declarations list required permissions, while the Runtime applies permission and relation decisions from
the admitted binding.

## 20. Publication, replacement, and rollback

Publication proceeds as:

```text
built artifact
→ isolated candidate start
→ health check
→ atomic active-digest swap
→ previous version drains
→ previous version becomes inactive
```

New invocations resolve the active digest once. Existing invocations and pinned chain nodes finish on their selected
digests. The Runtime owns WebSocket connections, so publication does not disconnect them. Each new realtime message
resolves the then-active digest.

Rollback activates one exact retained healthy digest through the same atomic swap. It MUST NOT rebuild source or
fall back to an approximate version. If the requested digest is missing, unauthorized, unhealthy, or incompatible
with the strict current protocol, rollback fails closed.

Protocol versions, schemas, and generated catalogs use exact matching. Unknown protocol versions and fields are
rejected. Draft-to-Draft aliases, old-format readers, dual reads or writes, and version fallback are prohibited.

## 21. Error model

```json
{
  "code": "schema_invalid",
  "message": "Input does not match CreateOrder",
  "retryable": false,
  "phase": "input_validation",
  "details": {
    "path": "input.total",
    "expected": "decimal_string"
  }
}
```

`message` and `details` are safe for the authenticated caller and MUST NOT contain source, secrets, credentials,
protected payload values, stack traces, filesystem paths, internal addresses, or cross-Team existence evidence.

| Code | Retryable | Meaning |
|---|---:|---|
| `invalid_request` | No | Malformed envelope or value |
| `unknown_field` | No | Field is not defined for this version |
| `unsupported_protocol_version` | No | Exact protocol version is unsupported |
| `not_found` | No | Missing, hidden, or cross-Team resource |
| `authentication_failed` | No | Trusted identity could not be derived |
| `permission_denied` | No | Bound identity lacks authority |
| `policy_denied` | No | Static or admitted policy denied work |
| `feature_unavailable` | Usually | Required feature is unavailable or degraded |
| `capability_unavailable` | Usually | A declared capability has no available admitted binding |
| `schema_invalid` | No | Input, output, event, or human answer fails schema |
| `payload_too_large` | No | A byte, field, item, or depth limit was exceeded |
| `semantic_validation_failed` | No | Validly encoded input violates declared domain constraints |
| `source_invalid` | No | Rust source or declaration is invalid |
| `source_limit_exceeded` | No | Reviewability limit exceeded |
| `dependency_waiting_human` | No | Exact dependency selection awaits approval |
| `dependency_denied` | No | Human or Runtime policy denied dependency |
| `dependency_blocked` | No | Critical dependency behavior is prohibited |
| `build_failed` | No | Validation, test, compile, provenance, or health failed |
| `artifact_unavailable` | Usually | Exact artifact cannot currently execute |
| `idempotency_conflict` | No | Key was reused with different canonical input |
| `state_conflict` | No | Requested lifecycle transition is invalid |
| `call_cycle` | No | Synchronous call cycle detected |
| `human_invalidated` | No | Bound request facts changed |
| `human_expired` | No | Response arrived after expiry |
| `rate_limited` | Yes | Rate or concurrency limit reached |
| `deadline_exceeded` | Sometimes | No time remains; retry needs a new invocation |
| `cancelled` | No | Invocation was cancelled |
| `operation_failed` | Declared | Function returned a safe domain failure |
| `output_invalid` | No | Function output failed its compiled schema |
| `upstream_unavailable` | Yes | Authorized dependency is temporarily unavailable |
| `internal_error` | Yes | Safe unclassified Runtime failure |

`Usually`, `Sometimes`, and `Declared` are documentation notes. Every emitted error contains a concrete Boolean
`retryable` decided from current state. A retryable error SHOULD include bounded `retry_after_ms`.

`phase` has exactly these values: `framing`, `identity`, `input_validation`, `static_policy`, `binding_policy`,
`function_policy`, `human_gate`, `execution`, `output_validation`, `lifecycle`, `dependency`, `build`, and `internal`.

## 22. Limits

These are target default maxima. Static Runtime policy MAY lower them. A function or project MAY request a lower
limit and MUST NOT raise one. Effective values are returned by `runtime.describe`.

| Limit | Default maximum |
|---|---:|
| Control request | 1 MiB |
| Invocation input | 512 KiB |
| Invocation output | 512 KiB |
| Event payload | 256 KiB |
| Realtime frame | 256 KiB |
| JSON nesting depth | 32 |
| JSON object fields | 1,024 |
| JSON array items | 10,000 |
| String value | 256 KiB UTF-8 |
| Default invocation deadline | 8 seconds |
| Maximum synchronous invocation deadline | 60 seconds |
| Delegation entries | 16 |
| Synchronous project-call depth | 16 |
| Chain nodes | 128 |
| Chain fan-out from one node | 32 |
| Policy expression depth | 8 |
| Human request TTL | 15 minutes default, 24 hours maximum |
| Realtime outbound queue | 128 frames and 1 MiB |

Every builder, worker, invocation, and job MUST additionally have measured CPU, memory, storage, PID, concurrency,
and wall-time limits, with swap disabled where applicable. Exact deployment values depend on the admitted substrate
and MUST be observable to the owning lifecycle authority before implementation is called conforming.

## 23. Observability and audit

For every invocation the Runtime records bounded metadata:

- invocation, root, and parent identifiers;
- Team, project, operation, and artifact digest;
- source protocol and safe principal reference;
- bounded delegation references;
- state transitions, attempt count, start, finish, and duration;
- safe error code and retry classification;
- measured CPU, memory, and output size;
- capability names used;
- child invocation and event identifiers;
- human-request lifecycle without prompt or answer;
- publication and rollback version transitions.

Audit MUST NOT contain source, prompts, HTTP bodies, function inputs or outputs, event payloads, human prompts or
answers, secrets, credentials, raw headers, stack traces, or provider values. Hashing low-entropy protected values
does not make them safe for audit. Internal idempotency digests MUST remain access-controlled and MUST NOT be exposed
as payload fingerprints.

Trace context is Runtime-derived and cannot carry arbitrary caller baggage. Observability is metadata-only and MUST
not promise retrieval of container stdout; a deployment may intentionally disable container logging.

## 24. Security requirements

The Runtime MUST assume hostile source, build inputs, dependencies, callers, payloads, artifacts, and authoring
instructions.

- Brain proposes functionality and source; it never receives shell, filesystem, Cargo, builder, deployment,
  binding, publication, invocation-authority, or secret-administration tools.
- Identity, Team, delegation, active digest, and capabilities derive from authenticated Runtime evidence, never
  caller fields.
- Authorization and schema validation fail closed and hide cross-Team existence.
- Build and execution authorities are separate. Neither has ambient access to another Team.
- Build is networkless and secretless after an exact dependency resolution is approved.
- Execution has bounded CPU, memory, time, PIDs, concurrency, input, output, filesystem, network, and capabilities.
- Filesystem is read-only except for an isolated bounded temporary area; durable storage uses `ctx.storage()`.
- Network is denied by default; authorized egress uses `ctx.http()` and exact destination policy.
- Database access uses the bound project role; function code never receives connection credentials.
- An artifact is invocable only by the exact digest proven from reviewed source, generated code, lockfile, SDK,
  toolchain, policy, and tests.
- Static analysis and model review are defense layers, not proof that code is safe.
- Publication drains exact versions; rollback selects an exact retained digest; deletion removes all owned residue.
- Security-sensitive denials, timeouts, partial failures, and cleanup failures remain errors and name their retry
  owner without exposing sensitive detail.

The selected WebAssembly or native sandbox design MUST satisfy these properties. This Draft does not select between
the two candidates.

## 25. Minimal end-to-end example

This example is illustrative and compresses asynchronous results.

### 25.1 Create a project

```json
{
  "protocol_version": "1",
  "request_id": "018f3a8c-0000-7000-8000-000000000001",
  "method": "project.create",
  "params": {
    "name": "orders",
    "description": "Order operations"
  }
}
```

### 25.2 Create one Rust operation

```rust
#[operation(description = "Creates an order", kind = "command")]
#[http(method = "POST", path = "/orders", success = "created")]
pub async fn create_order(
    ctx: Context,
    input: CreateOrder,
) -> Result<OrderCreated> {
    let order = ctx.db().transaction(|tx| async move {
        let order = tx.insert_order(input).await?;
        tx.outbox("orders.created", &order).await?;
        Ok(order)
    }).await?;

    Ok(OrderCreated { order_id: order.id })
}
```

The client sends this source through `function.create`. The Runtime chooses
`src/operations/create_order.rs`, formats it, generates schemas and registry, and validates the declared database
and event capabilities.

### 25.3 Validate, build, and publish

```text
project.validate(project=orders, revision=7)
→ project.build(project=orders, revision=7)
→ project.publish(project=orders, artifact_digest=sha256:...)
```

The builder has no network, secrets, production database, or another Team's source. Publication starts and health
checks the exact artifact, atomically activates it, and retains the previous healthy version for rollback.

### 25.4 Discover and invoke

```text
projects.list
→ project.operations(project=orders)
→ operation.describe(project=orders, operation=create_order)
→ operation.invoke(project=orders, operation=create_order, input={...})
```

HTTP can invoke the same operation:

```http
POST /orders HTTP/1.1
Content-Type: application/json
Idempotency-Key: checkout_018f3a8c

{"customer_id":"cus_123","total":"99.90"}
```

The HTTP adapter derives trusted identity and produces the same canonical Invocation used by realtime, events,
project calls, and control invocation.

### 25.5 Continue durably

If `analyze_fraud` declares `after_success(orders.create_order)`, the Runtime commits the `create_order` output and
the dependent job in one transaction. If `analyze_fraud` declares a human confirmation, its Invocation enters
`waiting_human`; an admitted response returns it to `authorizing`; the exact artifact then runs with the typed
answer. A crash between nodes cannot lose the continuation.

## 26. Minimal implementation model

The following Rust is illustrative. It shows the smallest control flow that preserves the contract; names are not
an SDK commitment.

```rust
async fn dispatch(
    wire: WireInvocation,
    evidence: TrustedEvidence,
    runtime: &Runtime,
) -> Outcome {
    let invocation = runtime.derive_invocation(wire, evidence).await?;
    runtime.store.create(&invocation).await?;

    let authorized = runtime.pipeline.authorize(invocation).await?;
    let artifact = runtime.versions.resolve_exact(&authorized).await?;
    let result = runtime.executor.run(artifact, authorized).await;

    runtime.commit_outcome_and_dependents(result).await
}
```

A minimal durable store needs these responsibilities, whether represented by tables or another transactional store:

| Record | Minimum fields |
|---|---|
| `project_revisions` | project, revision, canonical source digest, state |
| `artifacts` | digest, provenance manifest, health, creation time |
| `project_versions` | project, artifact digest, activation state |
| `operations` | artifact digest, name, schemas, permissions, effects, adapters |
| `invocations` | ids, exact artifact, safe identity refs, state, deadlines, attempts |
| `jobs` | invocation, availability time, lease, attempts |
| `outbox` | event id, producer invocation, name, payload reference, publish state |
| `human_requests` | invocation, type, binding digests, expiry, response state |
| `audit_events` | bounded identity refs, transition, safe outcome metadata, time |

An implementation MAY combine serving roles in one Rust binary. Role separation is an operational option, not a
second Runtime implementation:

```text
serve --role=all
serve --role=http
serve --role=realtime
serve --role=events
serve --role=executor
```

`--role=all` includes `http`, `realtime`, `events`, and `executor`. It MUST NOT include the isolated builder,
dependency resolver, human-response authority, or external control-plane authority. Separating serving roles MUST
preserve the same Invocation, Outcome, policy, version, idempotency, and audit semantics.

## 27. Normative references

- [RFC 2119: Key words for use in RFCs to Indicate Requirement Levels](https://www.rfc-editor.org/rfc/rfc2119)
- [RFC 8174: Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words](https://www.rfc-editor.org/rfc/rfc8174)
- [RFC 3339: Date and Time on the Internet](https://www.rfc-editor.org/rfc/rfc3339)
- [RFC 7240: Prefer Header for HTTP](https://www.rfc-editor.org/rfc/rfc7240)
- [RFC 8259: The JavaScript Object Notation Data Interchange Format](https://www.rfc-editor.org/rfc/rfc8259)
- [RFC 9562: Universally Unique IDentifiers](https://www.rfc-editor.org/rfc/rfc9562)
- [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12/json-schema-core)

## 28. Informative references

- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [Rust 1.98.0 announcement](https://blog.rust-lang.org/2026/08/20/Rust-1.98.0/)
- [`DRAFT-DECISIONS.md`](DRAFT-DECISIONS.md)

## Changelog

- **Draft 0.2 — 2026-08-22:** Replaced the conceptual layout with an RFC-style target contract; defined source,
  control, lifecycle, invocation, adapters, capabilities, chains, human input, policies, errors, limits, security,
  and minimum implementation behavior; moved unresolved deliberation to `DRAFT-DECISIONS.md`. No implementation is
  authorized.
- **Draft 0.1 — 2026-08-22:** Established current conflicts, candidate boundaries, open P0 decisions, and promotion
  gates.
