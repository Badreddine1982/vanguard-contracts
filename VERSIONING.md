# Contract Versioning Policy

Status: proposed governance policy; applies to contract candidates in this repository.

## Version format

Use Semantic Versioning: MAJOR.MINOR.PATCH.

- **MAJOR**: incompatible change to required fields, dependencies, error semantics, execution guarantees, or observable behavior.
- **MINOR**: backward-compatible capability or optional field addition.
- **PATCH**: backward-compatible clarification or correction that does not change the contract's observable behavior.

## Before adoption

A contract marked `proposed` is not an adopted interface. Nevertheless, once a versioned candidate has been referenced by another repository or used as evidence, do not silently rewrite its meaning under the same version. A breaking candidate revision receives a new major version and versioned filename; retain the previous file as historical evidence.

## Adoption boundary

A passing test or merge does not itself adopt a contract. Adoption requires an explicit, recorded decision after compatibility, implementation, and evidence gates pass. `automatic_adoption` remains false.

## Runtime execution API decision

The proposed v1 API is retained unchanged as historical candidate evidence. Requiring `ActionExecutor` alongside `AuditSink`, and defining explicit timeout/cancellation outcomes, is treated as a breaking change. The new candidate is therefore `runtime.execution` **2.0.0**, stored in `contracts/runtime/execution.v2.yaml` with schema `contracts/runtime/execution.v2.schema.yaml`.

The v2 timeout contract distinguishes:
- `EXECUTION_TIMEOUT`: the executor confirms cancellation;
- `EXECUTION_STATE_UNKNOWN`: the deadline expired but completion/cancellation is unconfirmed.

A timeout must never be interpreted as proof that no side effect occurred. The runtime implementation and long-running-action tests must satisfy this contract before the candidate can be considered verified or adopted.
