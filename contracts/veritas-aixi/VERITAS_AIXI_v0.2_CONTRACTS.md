# VERITAS-AIXI v0.2 — Core Contract Set

**Status:** INSTALLED / EXPERIMENTAL BASELINE + LONG-HORIZON UPDATE
**Purpose:** تثبيت الحد الأدنى من العقود التي تفصل القرار عن التمثيل والتنفيذ، مع الحفاظ على VERITAS-AIXI كنواة مستقلة.

## 1. StateContract
يمثل الحالة الداخلية القابلة للتعريف والمقارنة، ولا يحتوي على تنفيذ خارجي.

Required semantics:
- stable `state_id`
- immutable snapshot semantics
- latent representation may be opaque
- decoder/rendering state is not part of the decision authority
- state transitions must be traceable

Conceptual form:
`observation -> belief/latent_state -> StateContract`

## 2. PolicyContract
يحوّل الحالة إلى **abstract action intent** فقط.

Required semantics:
- receives StateContract
- may rank/select candidate actions
- MUST NOT execute tools directly
- MUST NOT own permissions
- MUST remain replaceable

Conceptual form:
`latent_state -> policy -> abstract_action`

## 3. VerificationContract
يتحقق من نتيجة الفعل ويعيد evidence-oriented feedback.

Required semantics:
- accepts before/after state and action result
- returns validity/evidence/failure information
- may reject a transition
- does not mutate protected state
- supports rollback/recovery reporting

Conceptual form:
`state + action + observation -> verification -> reward/evidence`

## 4. Architectural separation

```text
                 VERITAS-AIXI
                       |
                 StateContract
                       |
                  latent_state
                       |
                PolicyContract
                       |
                abstract_action
                       |
              Action Decoder
                       |
             Action Renderer
                       |
               execution
                       |
              Observation
                       |
           VerificationContract
                       |
              evidence/reward
                       |
                 next state
```

The decoder translates an abstract action into an executable representation. The renderer performs the concrete side effect. They are adapters, not part of the latent decision state.

## 5. Reversibility invariant
Every mutating renderer SHOULD expose an inverse operation or a compensating transaction when a true mathematical inverse is impossible.

`render(action) -> effect + inverse`

The inverse is selected against the recorded pre-state, not reconstructed from guesswork.

## 6. Reordering invariant
Actions are represented as independent transitions with explicit preconditions/postconditions. A scheduler may reorder only actions whose preconditions remain satisfied and whose dependency constraints do not conflict.

`candidate sequence -> dependency check -> reordered sequence -> execute -> verify`

## 7. State separation
The contracts preserve the existing VERITAS/VANGUARD boundary: runtime state may evolve during an episode, but it does not silently overwrite Knowledge State or Master State. This is consistent with the existing library architecture.

## 8. Acceptance criteria
- State can be snapshotted and restored.
- Policy can be swapped without changing execution code.
- Decoder can be swapped without changing policy semantics.
- Renderer can be swapped without changing latent-state semantics.
- A sequence can be reversed deterministically in the toy environment.
- Independent actions can be reordered according to state/activity.
- Failed transitions produce evidence rather than silent state mutation.
## 9. LongHorizonPlanner
The v0.2 reversible example is promoted from a single ordered demo to a bounded
long-horizon planner.

Responsibilities:
- maintain a finite Runtime-State trajectory;
- explore alternative action sequences without permanently mutating the world;
- use renderer inverses/snapshots to rollback speculative branches;
- reorder candidates only when dependency/precondition constraints permit it;
- verify every speculative transition;
- commit only the selected trajectory and verify it again;
- preserve failed branches as evidence.

Conceptual loop:

`state -> candidates -> branch -> render -> verify -> evidence -> rollback -> next branch`

The planner is a search component, not a permission authority and not a second
ORCHESTRATO policy engine.

## 10. Evidence Memory
`EvidenceMemory` is the minimal local experience substrate for the example.
It stores:
- state fingerprint;
- abstract action;
- next-state fingerprint when available;
- verification reward;
- verification status;
- failure signature when the transition fails.

Evidence is reusable experience, not automatically trusted Knowledge State.
Promotion to durable knowledge remains subject to the existing verification and
internalization lifecycle.

## 11. AIXI-Style Decision Use
AIXI is used as a **decision principle** over the recorded interaction history,
not as a database, executor, renderer, or permission system.

The practical v0.2 implementation uses a finite-horizon empirical approximation:

`history -> empirical transition/reward expectation -> action preference -> search`

`AIXIStylePredictor`:
- reads EvidenceMemory;
- estimates expected value for a candidate action in the current state;
- gives bounded novelty value to unseen actions;
- orders search branches by predicted value;
- does not execute actions or mutate state.

This is explicitly **AIXI-style**, not a claim to implement exact Solomonoff
induction or computationally intractable exact AIXI.

## 12. Learning From Rollback and Reordering
Rollback is not information loss.

A speculative branch produces:

`state + action + observation/result + verification + reward + rollback outcome`

That record remains available to subsequent planning. Therefore the planner can
avoid repeating known failures and prefer verified trajectories while still
exploring unknown alternatives.

The intended learning loop is:

`Search -> Execute Speculatively -> Verify -> Record -> Rollback -> Reorder/Branch -> Search`

## 13. Extended Acceptance Criteria
In addition to the original criteria, v0.2 experimental acceptance requires:
- `LongHorizonPlanner` can search beyond one transition;
- speculative branches restore the exact pre-branch latent state;
- failed branches remain in EvidenceMemory after rollback;
- AIXI-style value estimates influence branch ordering without gaining execution authority;
- selected plans are re-verified during commit;
- reordering and rollback do not cross Runtime/Knowledge/Master state boundaries;
- no external model, database, RAG system, or runtime service is required by the toy implementation.

## 13. Superposition of Search Hypotheses

In the VERITAS-AIXI experimental planner, **superposition is an architectural/search term, not quantum computation**. It denotes a finite set of alternative trajectories represented simultaneously as MCTS nodes while only one concrete runtime branch is materialized at a time.

```text
                 latent state
                      |
             candidate actions
                      |
          +-----------+-----------+
          |           |           |
        A→B→C       A→C→B       B→A→C
          |           |           |
       hypothesis  hypothesis  hypothesis
          +-----------+-----------+
                      |
                 verMCTS value
                      |
              select / reorder
```

The tree is therefore a reversible hypothesis space. A branch is not committed merely because it was explored.

## 14. Nano verMCTS Integration

`NanoVerMCTS` is the minimal concrete bridge between `LongHorizonPlanner`, `EvidenceMemory`, and the verification-aware search principle.

Its loop is:

`select → materialize state → expand → speculative render → verify → rollout → backpropagate → exact restore`

The implementation remains deliberately small and model-agnostic. It uses UCT-style exploration for tree search and `AIXIStylePredictor` as an empirical prior for branch ordering. Neither component owns permission or execution authority.

### Seed use

A search seed is an optional deterministic control for the Nano MCTS exploration process. It is not stored as trusted knowledge. The seed may reproduce a search episode, compare alternative search policies, and support controlled experiments around Fold/Unfold or reordered trajectories.

`seed → deterministic search trajectory → evidence → verify → compare`

Different seeds may expose different branches without changing the protected state semantics.

## 15. Integrated Long-Horizon Loop

The combined experimental architecture is:

```text
Latent State
    ↓
Policy / candidate generation
    ↓
Superposed hypotheses
    ↓
Nano verMCTS
    ├── AIXI-style empirical prior
    ├── reorder
    ├── branch
    └── bounded exploration
    ↓
Decoder
    ↓
Renderer
    ↓
Observation
    ↓
Verification
    ├── success → value update
    └── failure → evidence + rollback
    ↓
EvidenceMemory
    ↓
AIXI-style prediction update
    ↓
next search
```

Rollback therefore changes the **information available to the next search**, not merely the runtime state. This is the intended coupling between reversibility and learning.

## 16. Experimental Boundary

The Nano verMCTS/superposition implementation is an experimental extension of v0.2. It does not claim exact AIXI, quantum superposition, lossless compression, or general AGI planning. It remains subject to the existing Runtime/Knowledge/Master state separation and verification/internalization lifecycle.

The existing EXP-001 result supports the narrower hypothesis that MCTS-derived search representations can be useful experimental memory objects before Fold/Unfold. The present extension tests the complementary hypothesis that reversible long-horizon search can reuse verified branch evidence.

## 17. Extended Acceptance Criteria

- `LongHorizonPlanner` delegates multi-step branch selection to Nano verMCTS.
- Search branches are represented simultaneously as immutable hypotheses.
- Only materialized branches mutate Runtime State.
- Every speculative mutation is verified and exactly restored.
- Failed branches remain in EvidenceMemory.
- AIXI-style empirical value changes branch preference without granting execution authority.
- Search seeds provide reproducible bounded experiments.
- Reordering remains constrained by state/preconditions/dependencies.
- Selected trajectories are verified again during commit.
- No external model, database, RAG system, or runtime service is required by the toy implementation.