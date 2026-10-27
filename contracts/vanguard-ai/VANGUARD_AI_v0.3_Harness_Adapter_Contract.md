# VANGUARD AI v0.3 — Harness Adapter & Experimental Execution Contract

**Status:** PROPOSED / MASTER-RECONCILIATION REQUIRED
**Version:** 0.3
**Scope:** Harness integration boundary + experimental execution runtime
**Parent:** VANGUARD AI v0.2 Knowledge Infrastructure & Autonomous Execution Contract
**Change Type:** ADD

## 1. Purpose

This section adds a replaceable Harness Adapter boundary to VANGUARD AI without making any specific agent harness part of the intelligence substrate or protected core.

The first implementation candidate is DeepSeek Harness, used as an external/experimental execution runtime. Its role is execution, delegation, tool composition, and runtime experimentation; it is not the decision engine, security authority, knowledge authority, or architectural authority.

## 2. Architectural Placement

The intended separation is:

```text
VANGUARD
    │
    ├── VERITAS-AIXI candidate decision kernel
    │
    └── ORCHESTRATO control plane
             │
             ├── Model Registry
             ├── Agent Registry
             ├── Capability Registry
             ├── Policy Engine
             ├── Session State
             └── Harness Adapter
                      │
                      ├── DeepSeek Harness
                      ├── Claude Code / compatible harness
                      ├── Codex / compatible harness
                      └── Future Local/Remote Runtimes
```

DeepSeek Harness MUST remain below the control/policy boundary and MUST NOT become a dependency of VERITAS-AIXI decision logic.

## 3. Harness Adapter Contract

The abstract boundary SHOULD expose only the minimum execution semantics required by ORCHESTRATO:

- `start_session`
- `execute`
- `delegate`
- `observe`
- `inspect`
- `stop`
- capability discovery
- execution result reporting
- failure reporting

The contract MUST remain provider-neutral. The implementation name MUST NOT leak into higher-level decision contracts.

## 4. Relationship to Existing Adapter Principle

v0.2 already establishes that external capabilities are adapters rather than hard dependencies and lists RAG, Lean/formal verification, Browser, compute/local clusters, model providers, and code execution as planned adapters.

Harness is therefore an additional execution adapter, not a replacement for those capabilities.

The resulting principle is:

`Decision → Policy → Capability → Harness/Provider → Tool → Observation → Verification`

The Harness MUST NOT own global architectural decisions.

## 5. Relationship to ORCHESTRATO

ORCHESTRATO remains the lightweight control plane.

Existing registries and controls SHOULD be reused rather than duplicated. Harness integration MUST NOT introduce a second model registry, permission system, session authority, or audit authority.

The preferred structure is:

```text
ORCHESTRATO
    │
    ├── Model Registry
    ├── Agent Registry
    ├── Capability Registry
    ├── Policy Engine
    ├── Session State
    ├── Audit
    │
    └── Harness Adapter
             │
             └── selected runtime
```

## 6. Subagent Delegation

A Harness implementation MAY provide subagent execution/delegation.

Delegation remains subordinate to ORCHESTRATO policy and task routing. A child agent MUST inherit only explicitly granted capabilities and permissions.

A subagent MUST NOT acquire architectural authority merely because it is delegated execution.

## 7. Hooks and Enforcement Boundary

Harness lifecycle hooks MAY be used as enforcement integration points for:

- session start
- task submission
- pre-tool checks
- post-tool observation
- stop/failure handling
- subagent lifecycle events

Hooks are integration points, not the security authority.

Security and permission enforcement remain outside the model and outside any untrusted harness implementation.

## 8. Security Model

The following boundary is mandatory:

```text
Model / Harness
      ↓
Capability Request
      ↓
ORCHESTRATO Policy Engine
      ↓
Permission Decision
      ↓
Execution Adapter
      ↓
Observation / Audit
      ↓
Verification
```

No Harness integration may grant unrestricted Internet access, unrestricted filesystem access, unrestricted shell execution, or direct mutation authority over the protected core.

Every mutating action MUST have an auditable identity, permission context, verification requirement, and rollback path where applicable.

## 9. Experimental Lab Role

DeepSeek Harness is initially classified as an **Experimental Execution Runtime**.

It may be used to evaluate:

- agent delegation
- plugin composition
- tool execution
- hooks
- session behavior
- model/runtime interchangeability
- coding workflows
- browser/tool workflows
- failure and recovery behavior

Experimental results SHOULD be recorded as observations/evidence and MUST NOT automatically modify Master State or trusted Knowledge State.

## 10. Internalization Boundary

Harness output follows the existing VANGUARD knowledge lifecycle:

```text
External/Runtime Observation
        ↓
Candidate Artifact
        ↓
Reconstruction
        ↓
Verification / Reproduction
        ↓
Evaluation
        ↓
Internalization Candidate
        ↓
Knowledge Store
```

Execution output is evidence, not trusted knowledge by default.

## 11. State Separation

Harness state belongs to Runtime State.

It MUST NOT directly overwrite:

- Master State
- trusted Knowledge State
- protected Core source/configuration

Any durable promotion follows the existing Change Request and knowledge trust lifecycle.

## 12. Controlled Code Evolution

Harness-based coding agents MAY operate only inside the existing controlled evolution lifecycle:

`Current Version → Proposed Change → Sandbox → Tests → Formal/Static Checks → Benchmark → Regression Gate → Security Gate → Approval → New Version`

The Harness does not bypass any gate.

## 13. VERITAS-AIXI Relationship

VERITAS-AIXI remains separable from the VANGUARD core as defined by v0.2.

Its decision layer may select an execution capability abstractly, but it MUST NOT depend on DeepSeek Harness APIs.

The intended direction is:

```text
VERITAS-AIXI
     │
 Decision / Value / Verification
     ↓
Abstract Capability
     ↓
ORCHESTRATO Policy + Routing
     ↓
Harness Adapter
```

This preserves replaceability and prevents runtime implementation details from contaminating the decision kernel.

## 14. Non-Goals

This proposal does NOT:

- make DeepSeek the permanent model
- make DeepSeek Harness part of the protected core
- replace ORCHESTRATO
- replace the Policy Engine
- replace the Audit system
- create a second permission system
- grant unrestricted autonomy
- grant unrestricted Internet access
- claim AGI
- require multi-agent execution

## 15. Minimal Implementation Boundary

The minimum implementation SHOULD consist of:

1. `HarnessAdapter` interface/contract
2. one experimental `DeepSeekHarnessAdapter`
3. ORCHESTRATO registration entry
4. permission/context propagation
5. normalized execution result
6. audit event mapping
7. failure/rollback reporting
8. integration tests

No additional service should be introduced unless the existing runtime cannot provide the required function.

## 16. Change Request

**ID:** VANGUARD-CR-0003

**Source Section:** Harness / External Execution Integration

**Type:** `ADD`

**Target:** VANGUARD Adapter Boundary + ORCHESTRATO execution integration

**Current State:** v0.2 defines replaceable adapters but has no explicit Harness adapter contract.

**Proposed State:** Add a provider-neutral Harness Adapter boundary with DeepSeek Harness as the first experimental implementation.

**Rationale:** Reuse an existing agent-runtime pattern for execution/delegation while preserving VANGUARD's model-agnostic architecture, ORCHESTRATO's control-plane role, and VERITAS-AIXI's separation from execution details.

**Dependencies:** v0.2 Adapter Principle; ORCHESTRATO capability/policy/session/audit contracts; controlled code-evolution lifecycle.

**Affected Modules:** Adapter boundary, ORCHESTRATO integration, experimental lab, audit mapping, runtime state.

**Conflicts:** No direct conflict identified with the reviewed v0.1/v0.2 contracts. Master Page reconciliation is still required before global architectural approval.

**Backward Compatibility:** Additive. Existing RAG, Lean, Browser, compute, model, and code-execution adapters remain valid.

**Security Impact:** Medium/high because the Harness can mediate tool execution. Security boundaries remain external to the model/harness.

**Resource Impact:** Limited initially; one adapter and test surface only.

**Verification Plan:** Contract tests, permission propagation tests, isolation tests, audit tests, failure/recovery tests, model/runtime switching tests, and regression tests.

**Rollback Plan:** Disable/remove the Harness adapter registration; no protected-core mutation is permitted by the integration.

## 17. Master Reconciliation Gate

This document is a proposal. It does not modify Master State by itself.

Required transition:

`PROPOSAL → CONSISTENCY CHECK → MASTER APPROVAL → IMPLEMENTATION → VERIFICATION → COMMIT`

Until that transition is completed, the Harness integration remains experimental and non-authoritative.