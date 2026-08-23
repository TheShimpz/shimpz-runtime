# Repository working rules

These rules apply to every contributor and coding agent working in this repository.

## Current status and authority

- This repository contains a proposed Shimpz Runtime specification. A Draft describes a target contract only: it
  authorizes no implementation, integration, deployment, or conformance claim.
- This repository is not current Shimpz architecture authority. While it is evaluated for integration, the umbrella
  `AGENTS.md` and `.context/ARCHITECTURE.md` remain authoritative. An owner-approved ADR is required before this
  repository, a root mount, or a cross-domain contract becomes part of Shimpz architecture.
- Do not implement runtime code, add build configuration, select dependencies, or mount this repository as a
  submodule until the owner explicitly promotes the specification and requests implementation.

## Delivery

- Work in the smallest independently reviewable task that produces a useful result.
- After a microtask succeeds, run the smallest relevant checks, commit it immediately, and push it immediately.
- Never batch unrelated successful microtasks into one commit.
- Write artifacts and commit messages in English. Use a clear conventional commit subject.
- Preserve unrelated work and keep every Draft status, limitation, and changelog honest.

## Specification

- Define observable behavior, accepted values, state transitions, failures, limits, and security properties before
  implementation mechanisms.
- Mark examples as illustrative unless the specification explicitly makes their syntax normative.
- Use normative requirement keywords only for the target contract and never to imply that a Draft is available.
- Reject unknown or invalid current-contract input explicitly; do not add compatibility paths for an earlier Draft.
- Do not introduce current Team, Developers, Services, Account, Admin, Brain, or Assistant protocol authority here.
  Cross-domain integration remains a separately admitted contract owned by its natural producer.

## Security

- Preserve Team isolation, least privilege, fail-closed validation, immutable provenance, bounded execution, and
  secret and protected-payload redaction.
- Treat authored source, build inputs, dependencies, callers, payloads, and artifacts as hostile.
- Never present line count, AST size, linting, or model review as an execution-security boundary.
- Do not introduce retired Shimpz identifiers or aliases, including `App`, `AppSpec`, `Captain`, `Driver`, actor
  `operator`, `accounts`, `brain_id`, `team.brain`, or `team.model`.

## Validation and review

- This Draft-only repository currently has no implementation or local test harness. At minimum, run
  `git diff --check` and inspect the rendered document structure and links before each documentation commit.
- When this repository is mounted by the umbrella, run the umbrella-selected impact and architecture gates in
  addition to repository-local checks.
- Classify work through the umbrella `.agents/skills/claude-sidekick-review` rules. Every mutation, protocol,
  architecture, authority, or security judgment is Tier 1 and requires one read-only Claude Opus 5 high-effort
  session for pre-action debate and post-result audit; Codex retains final authority.
