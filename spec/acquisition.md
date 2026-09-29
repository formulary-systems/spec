# Procedure Manifests v0.0.3 — Acquisition Boundary and Authority Eligibility

**Author:** @chugarchugarr  
**Status:** Normative v0.0.3 candidate  
**Builds on:** v0.0.2 closure invariant and adjudication state machine

## 1. Purpose

v0.0.2 closes the post-observation transformation boundary: given the same
committed manifest and the same complete run observations, conforming
implementations must derive the same authorized remedy.

v0.0.3 closes the immediately preceding boundary: which observations are
permitted to acquire contractual authority in the first place.

A procedure can admit the wrong observation while preserving every v0.0.2
transformation rule. In particular:

1. **Selective rerun / equivocation:** a provider or executor can obtain
   multiple authentic executions for one logical slot and submit the preferred
   one.
2. **Withholding:** an unfavorable completed result can be suppressed and
   represented as timeout or transport failure if terminality is not
   independently verifiable.
3. **Request substitution:** a receipt can bind the right slot to the wrong
   prompt, evidence root, model profile, or sampling parameters unless the exact
   execution request is committed.
4. **Namespace regeneration:** a fresh executor-selected dispute namespace can
   recreate authorized run slots after results exist.
5. **Scope escalation:** an observation can be authentic and uniquely acquired
   while establishing less than the contractual consequence requires.

These are instances of one rule:

> **Authenticity is necessary for authority, but it is not sufficient for authority.**

## 2. Strengthened closure invariant

> **Every outcome-relevant transformation MUST be committed by the procedure;
> every observation admitted to those transformations MUST originate from a
> uniquely authorized execution; and every condition deciding whether that
> observation is sufficient to carry contractual authority MUST itself be
> committed and machine-evaluable.**

The v0.0.2 rule remains intact. v0.0.3 inserts acquisition and
authority-eligibility semantics before the existing authoritative-result path.

The complete normative boundary is:

```text
evidence
  ↓
authorized execution
  ↓
attested observation
  ↓
authority eligibility
  ↓
authoritative result
  ↓
within-judge aggregation
  ↓
across-panel aggregation
  ↓
requirement resolution
  ↓
contract outcome
  ↓
authorized remedy
  ↓
manifested consequence
```

Only an **eligible** observation may enter the v0.0.2 authoritative-result
path.

For an observation `o`:

```text
eligible(o) =
    authorized_execution(o)
  ∧ exact_request_binding(o)
  ∧ unique_terminal_execution(o)
  ∧ sufficient_scope(o)
```

If any term is false or cannot be established under the committed provenance
mechanism, the observation MUST NOT acquire authority as a PASS/FAIL result.
The run MUST resolve `UNRESOLVED`.

## 3. Preauthorized run slots

Each logical run slot MUST be identifiable before its result exists.

A conforming profile MUST define a deterministic derivation equivalent in
semantics to:

```text
run_id = H(
  manifest_hash,
  dispute_id,
  requirement_id,
  judge_id,
  run_index
)
```

`run_id` MUST NOT contain the result or any value chosen after observing a
result.

The dispute namespace MUST be committed or derived from authoritative
pre-execution state before any provider admission or result production can
occur. The provenance profile MUST identify the mechanism that establishes
both:

1. the authority of the predecessor state from which the namespace is derived;
   and
2. the ordering fact that this predecessor state existed before the execution
   acquired authority.

A self-asserted timestamp, local clock value, executor field, or
unauthenticated ordering marker — including a bare `committed_at` value —
MUST NOT by itself establish predecessor authority or temporal precedence.

Cryptographic binding can preserve an authoritative ordering fact supplied by
an underlying mechanism; it cannot manufacture that fact. If the committed
provenance profile cannot establish the predecessor authority/order relation,
`authorized_execution` is `cannot_establish` and the run MUST resolve
`UNRESOLVED`.

`dispute_id` MUST NOT be executor-regenerable after observation.

## 4. Submission is not admission

A requester-signed object and a provider-signed object make different claims
and MUST remain separate.

```text
submission_commitment
  = requester proves "I sent this request"

claim_receipt
  = provider proves "I accepted this request into this authorized execution slot"
```

A `submission_commitment` MAY be preserved as evidence.

It MUST NOT prove provider admission.

A `claim_receipt` MUST be signed or otherwise attested by the provenance
mechanism that has authority to admit the execution.

Silence after a requester submission MUST NOT be interpreted as admission or as
`ATTESTED_NO_RESULT` unless the committed provenance mechanism makes that
absence independently verifiable.

## 5. Exact request binding

The claim MUST bind the exact execution request, not merely the slot.

A canonical request digest MUST commit every outcome-relevant execution input,
including at minimum:

```text
request_hash = H(
  evidence_root,
  prompt_or_material_digest,
  judge_execution_profile,
  model_version_or_weights_pin,
  sampling_parameters,
  transformation/provenance inputs required by the manifest,
  any other manifest-declared outcome-relevant execution input
)
```

The concrete encoding is provenance-profile-defined, but the invariant is
normative:

> Two executions that can differ in any outcome-relevant input MUST NOT share
> the same `request_hash`.

## 6. Acquisition lifecycle

The logical run lifecycle is:

```text
AUTHORIZED
    ↓
CLAIMED
    ↓
TERMINAL_RESULT
  | TERMINAL_UNRESOLVED
  | ATTESTED_NO_RESULT
```

### AUTHORIZED

The run slot exists under the committed manifest but no execution mechanism has
yet acquired it.

### CLAIMED

The provenance mechanism has accepted exactly one execution attempt for the
authorized slot and issued a claim binding at least:

```text
run_id
attempt_id
request_hash
accepted_at
ordering_proof
provenance_profile
```

The claim MUST be attributable to the provider/provenance mechanism.

`accepted_at` is evidence about the provider's reported acceptance time. It
MUST NOT substitute for the authoritative `ordering_proof` required by C16.
The `ordering_proof` MUST be interpreted under the committed provenance
profile and MUST establish the predecessor-state/order relation independently
of a caller-local or provider-self-asserted clock value.

For each authorized `attempt_id`, the provenance profile MUST provide one of:

1. provider-enforced claim uniqueness; or
2. independently detectable claim equivocation with a committed fail-closed
   consequence.

Two distinct authentic claims for the same authorized `attempt_id` MUST NOT
create an implementation-defined choice. Claim uniqueness is part of
`authorized_execution(o)`: if the committed provenance profile cannot
establish a unique authoritative claim for the attempt, then
`authorized_execution(o)` is not `true`, neither claim may acquire
contractual authority, and the run MUST resolve `UNRESOLVED`.

### TERMINAL_RESULT

The claimed execution produced a conforming terminal observation.

The terminal record MUST bind the claim, terminal status, and output:

```text
terminal_receipt = Attest(
  claim_receipt_hash,
  terminal_status = RESULT,
  output_hash
)
```

### TERMINAL_UNRESOLVED

The claimed execution completed, but its terminal result is explicitly
unresolved according to the committed execution/result contract.

This consumes the logical run exactly as `TERMINAL_RESULT` does.

### ATTESTED_NO_RESULT

The provenance mechanism independently establishes that the claimed attempt
produced no terminal result under the precommitted failure semantics.

Only this state — or an equivalent independently verifiable absence condition
committed by the provenance profile — MAY authorize a retry.

A caller-local timeout, dropped connection, or executor assertion is
insufficient.

## 7. Retry semantics

`TERMINAL_RESULT` and `TERMINAL_UNRESOLVED` consume the logical run.

They MUST NOT open another attempt merely because the result is unfavorable.

A retry MAY occur only after `ATTESTED_NO_RESULT` or another
manifest-committed, independently verifiable retry condition.

If retries are supported, all permitted attempts MUST be enumerable or
derivable before their results exist. For example:

```text
attempt_id = H(run_id, attempt_index)
```

with a manifest-committed maximum attempt count and retry predicate.

Failed attempts remain part of transcript lineage.

## 8. Terminal uniqueness and equivocation

An authentic terminal receipt is not enough if multiple conflicting authentic
terminal receipts can exist for the same claimed attempt.

For every `CLAIMED` attempt, the committed provenance profile MUST provide one
of:

1. provider-enforced terminal uniqueness; or
2. independently detectable terminal equivocation with a committed fail-closed
   consequence.

A conforming implementation MUST NOT silently choose among multiple valid
terminal observations for the same claimed attempt.

If conflicting terminal records exist and the provenance profile does not
establish a unique authoritative terminal state, the run MUST resolve
`UNRESOLVED`.

Claim uniqueness (§6) and terminal uniqueness are separate requirements. A
profile that proves one does not thereby prove the other.

## 9. Authority scope

Provenance answers **where/how this observation came from**.

Authority scope answers **what this observation is sufficient to establish**.

These MUST remain distinct.

Free-form limitations MAY remain evidence-only.

Any limitation capable of altering contractual state MUST enter the
authoritative projection as a structured, machine-evaluable field.

The manifest MUST commit:

- the `required_scope` for each requirement or result class; and
- the exact predicate by which an observation scope satisfies that requirement.

If `observation_scope` does not satisfy `required_scope`, the run MUST
resolve `UNRESOLVED`.

A VERIFIED/authentic result therefore cannot silently escalate into authority
for a consequence outside the scope it establishes.

## 10. Manifest and record surface

A conforming v0.0.3 manifest MUST commit an execution-acquisition/provenance
section containing, by whatever concrete schema names the profile adopts, the
semantics of:

```json
{
  "acquisition": {
    "dispute_id_rule": "...",
    "predecessor_authority_rule": "...",
    "ordering_proof_rule": "...",
    "run_id_rule": "manifest+dispute+requirement+judge+run_index",
    "provenance_profile": "...",
    "claim_semantics": "...",
    "terminal_semantics": "...",
    "retry_policy": {
      "allowed_after": ["ATTESTED_NO_RESULT"],
      "max_attempts": 1
    }
  }
}
```

Requirement/result configuration MUST additionally commit `required_scope`
and the scope-satisfaction predicate wherever scope can affect authority.

A claim record MUST minimally bind:

```json
{
  "run_id": "...",
  "attempt_id": "...",
  "request_hash": "...",
  "accepted_at": "...",
  "ordering_proof": "...",
  "provenance_profile": "...",
  "attestation": "..."
}
```

A terminal record MUST minimally bind:

```json
{
  "claim_receipt_hash": "...",
  "terminal_status": "RESULT | UNRESOLVED | NO_RESULT",
  "output_hash": "...",
  "attestation": "..."
}
```

`output_hash` is omitted only when the committed terminal status does not
carry an output.

## 11. Conformance rules C16–C21

### C16 — preauthorized execution slots and authoritative predecessor ordering

Every LLM-judge run MUST have a deterministic `run_id` derivable from
committed pre-result state. `dispute_id` MUST be committed or derived before
provider admission or result production and MUST NOT be executor-regenerable
after observation.

The committed provenance profile MUST identify an authoritative predecessor
state/commitment source and an independently verifiable ordering mechanism
sufficient to establish that predecessor state before execution acquired
authority. A bare timestamp, `committed_at`, executor assertion, or local clock
comparison MUST NOT satisfy C16 by itself.

If predecessor authority or ordering cannot be established, the execution is
not authority-eligible and the run MUST resolve `UNRESOLVED`.

### C17 — exact request binding

Every claimed execution MUST bind a `request_hash` covering every
outcome-relevant execution input committed by the manifest.

### C18 — claim authority and uniqueness

A claimed execution MUST carry provider/provenance attestation proving
admission into the authorized slot. Requester submission evidence alone MUST
NOT satisfy this rule.

For a given authorized `attempt_id`, the provenance profile MUST also
establish at most one authoritative claim or make multiple authentic claims
detectable as equivocation with a committed fail-closed consequence. Claim
uniqueness is part of `authorized_execution(o)`; if uniqueness cannot be
established, `authorized_execution(o)` is not `true`. A conforming
implementation MUST NOT select among conflicting authentic claims.

### C19 — unique terminal execution

The provenance profile MUST establish a unique terminal state for each claimed
attempt, or make terminal equivocation detectable with a committed fail-closed
consequence.

### C20 — retry closure

Retry MUST be authorized only by a manifest-committed, independently verifiable
terminal absence/failure state. Caller-local timeout or executor assertion MUST
NOT authorize another attempt.

### C21 — authority scope

Every scope/limitation capable of changing contractual state MUST be structured
and machine-evaluable. The manifest MUST commit the required scope and exact
satisfaction predicate.

## 12. Required semantic regression fixtures

This boundary MUST NOT merge without executable fixtures for at least the
following cases.

### F1 — authentic double terminal

Same `run_id`, same claimed attempt, two authentic conflicting terminal
outputs.

**Expected:** no implementation may select one silently. If terminal uniqueness
cannot be established, run resolves `UNRESOLVED`.

### F1b — authentic double claim

Same authorized `run_id` and same enumerated `attempt_id`, but the provider
issues two distinct authentic claim receipts. The claims may bind different
request hashes, ordering records, or downstream terminal receipts.

**Expected:** no implementation may select or order one claim into authority.
Unless the committed provenance profile proves a unique authoritative claim,
claim equivocation is fail-closed and the run resolves `UNRESOLVED`.

### F2 — suppressed unfavorable result

A claimed attempt has a real terminal result, but the executor presents a local
timeout and requests retry.

**Expected:** retry refused unless the committed provenance mechanism attests
`NO_RESULT`.

### F3 — right slot, wrong request

A terminal result is authentic for the authorized `run_id` but the
`request_hash` differs in an outcome-relevant input.

**Expected:** observation ineligible; run `UNRESOLVED`.

### F4 — authentic but insufficient scope

A uniquely acquired authentic result carries scope narrower than the manifest's
`required_scope`.

**Expected:** observation ineligible; run `UNRESOLVED`.

### F5 — submission without admission

Requester provides a valid `submission_commitment`; no provider claim exists.

**Expected:** attempted submission is preserved as evidence, but execution is
not `CLAIMED`, and no result/admission authority follows.

### F6 — namespace regeneration

Executor attempts to create a fresh `dispute_id` after observing an
unfavorable terminal result.

**Expected:** refused because the dispute namespace was committed or derived
from authoritative predecessor state before execution.

### F6b — unauthoritative predecessor ordering

The supplied `dispute_id` derivation is internally consistent, and a local
`committed_at` value numerically precedes the claim, but the committed
provenance profile supplies no authoritative predecessor-state or ordering
mechanism that can establish the precedence claim.

**Expected:** `authorized_execution = cannot_establish`; run
`UNRESOLVED`. Numeric timestamp comparison alone is insufficient.

## 13. Semantic conformance property for v0.0.3

v0.0.2 requires:

> Given the same committed manifest and the same complete run observations, two
> independent conforming implementations MUST derive the same requirement
> resolutions, the same contract outcome, and the same authorized remedy.

v0.0.3 strengthens this to:

> **Given the same committed manifest and the same acquisition/transcript
> evidence, two independent conforming implementations MUST admit the same
> observations as authoritative and MUST derive the same requirement
> resolutions, contract outcome, and authorized remedy.**

This moves semantic closure one boundary earlier: not merely deterministic
consumption of frozen observations, but deterministic authority over which
observations are permitted to enter the consuming state machine.

## 14. Layer boundary

This specification does **not** require every execution provider to support the
same provenance mechanism.

It requires the manifest to commit one whose guarantees are sufficient for the
authority it is being asked to confer.

If a provider cannot establish authoritative predecessor ordering, single-use
admission, exact request binding, claim uniqueness, terminal uniqueness,
independently verifiable absence, or required scope, the correct result is
refusal or `UNRESOLVED` at that boundary — not an implementation-specific
guess.

Likewise, a requester-side submission commitment cannot manufacture provider
admission, and cryptographic structure cannot manufacture an authoritative
wall-clock or predecessor state that the underlying mechanism does not provide.

Historical evidence may be preserved without inheriting authority. Authority
must be re-established under the successor state's committed admissibility
conditions.

## Provenance

This v0.0.3 acquisition boundary was developed publicly by @chugarchugarr in
the Ethereum Magicians Procedure Manifests RFC thread after the v0.0.1 closure
counterexample. It formalizes the authority boundary immediately before the
v0.0.2 adjudication state machine.

The operative design rule is:

> **Preserve evidence. Re-earn authority.**
