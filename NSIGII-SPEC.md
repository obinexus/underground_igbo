# NSIGII Protocol Specification

## Network / Signal Integrity & Governance Interface

**Status:** Draft Specification
**Architecture:** Verification-first
**Primary Trident:** TRANSMIT / RECEIVE / VERIFY

---

# 1. Purpose

NSIGII defines a protocol for transmitting, receiving, verifying, classifying and auditing information in systems where message integrity, identity, state consistency and uncertainty matter.

NSIGII is designed for environments in which:

* information may be incomplete;
* communication may be unreliable;
* participants may disagree;
* messages may arrive out of order;
* systems may become disconnected;
* malicious or corrupted data may exist;
* human welfare may depend upon correct interpretation.

The protocol therefore treats verification as a first-class operation.

---

# 2. Core Trident

The minimum NSIGII interaction contains three operations:

```text
TRANSMIT
RECEIVE
VERIFY
```

## 2.1 TRANSMIT

A participant constructs and sends a NSIGII message.

A transmitted message SHOULD contain enough information to identify:

```text
sender
message type
timestamp
payload
context
integrity information
```

TRANSMIT means:

> A claim has entered the system.

It does not mean:

> The claim is true.

---

## 2.2 RECEIVE

A receiver accepts the message into a controlled processing boundary.

RECEIVE MUST NOT imply verification.

A receiver may:

* reject malformed input;
* quarantine unknown messages;
* normalise encoding;
* check message size;
* record transport metadata;
* establish receipt time.

---

## 2.3 VERIFY

VERIFY determines whether the received message satisfies the required integrity and authenticity conditions.

Verification may include:

* message digest verification;
* signature verification;
* identity verification;
* canonical representation validation;
* replay detection;
* context validation;
* authorization checks;
* ledger continuity checks.

Verification produces a result.

---

# 3. Verification States

The canonical NSIGII verification states are:

```text
YES
NO
MAYBE
TAMPERED
```

## YES

Required verification conditions passed.

## NO

Required verification conditions failed.

## MAYBE

There is insufficient information to determine YES or NO.

MAYBE is not failure.

It represents unresolved uncertainty.

## TAMPERED

Evidence indicates that the message or its integrity context has been modified inconsistently.

---

# 4. Signal Classification

NSIGII may independently classify communication state as:

```text
SIGNAL
NOISE
NOSIGNAL
NONOISE
```

These values describe the information channel rather than truth.

A message can therefore be:

```text
SIGNAL + MAYBE
SIGNAL + TAMPERED
NOISE + YES
NOSIGNAL
```

Implementations MUST define the exact semantics of their classification rules.

---

# 5. Extended Lifecycle

A complete NSIGII event lifecycle is:

```text
TRANSMIT
    ↓
RECEIVE
    ↓
CANONICALISE
    ↓
VERIFY
    ↓
CLASSIFY
    ↓
TRANSITION
    ↓
RESPOND
    ↓
AUDIT
```

---

# 6. Canonicalisation

Equivalent information may have multiple binary, textual or representational forms.

NSIGII therefore requires canonicalisation before certain security decisions.

Conceptually:

```text
many representations
        ↓
canonical representation
        ↓
single policy evaluation
```

Canonicalisation SHOULD address:

* text normalization;
* key ordering;
* whitespace rules;
* number representation;
* time representation;
* encoding;
* duplicate fields;
* case rules;
* schema version.

Security-sensitive verification MUST NOT rely upon an ambiguous representation.

---

# 7. Message Envelope

A minimal conceptual NSIGII envelope is:

```json
{
  "nsigii": "1.0",
  "id": "message-id",
  "type": "EVENT",
  "sender": "actor-id",
  "timestamp": "timestamp",
  "context": "context-id",
  "payload": {},
  "integrity": {
    "algorithm": "implementation-defined",
    "digest": "digest-value"
  }
}
```

An implementation MAY extend the envelope.

Additional fields may include:

```text
receiver
sequence
previous
signature
nonce
expiry
classification
confidence
location
evidence
permissions
schema
```

---

# 8. Message Identity

Every operationally significant message SHOULD have a stable message identifier.

The identifier is used for:

* deduplication;
* replay detection;
* acknowledgement;
* audit linkage;
* causal ordering.

An implementation MUST NOT assume network delivery order equals event order.

---

# 9. Evidence Model

NSIGII distinguishes between:

```text
CLAIM
OBSERVATION
EVIDENCE
INFERENCE
DECISION
ACTION
```

## Claim

Something asserted by an actor.

## Observation

Something reported as directly observed.

## Evidence

Material used to support or challenge a claim.

## Inference

A conclusion derived from evidence or observations.

## Decision

A policy-selected outcome.

## Action

A real-world or system operation.

These categories MUST NOT be silently collapsed.

---

# 10. State Transition Model

NSIGII models verified state change.

Let:

```text
S(t)
```

represent the system state at time `t`.

A verified transition is:

```text
ΔS = S(t+1) - S(t)
```

The protocol is interested not only in raw values but in meaningful changes.

Example:

```text
SHELTER_AVAILABLE
        ↓
SHELTER_FULL
```

is operationally more important than repeatedly receiving:

```text
occupancy=100
occupancy=100
occupancy=100
```

without context.

---

# 11. Bayesian Belief Layer

Implementations MAY maintain a belief state describing uncertainty about the current system state.

Let:

```text
B(t) = probability distribution over possible states
```

New evidence may update that belief.

Conceptually:

```text
prior belief
    +
verified observation
    ↓
updated belief
```

NSIGII does not require a specific Bayesian implementation.

The mathematical model MUST remain distinguishable from verified facts.

A probability estimate is not the same thing as an observed event.

---

# 12. Dimensional State Model

A complex system may be decomposed into multiple dimensions.

For MMUCO civil resilience, example dimensions include:

```text
FOOD
WATER
SHELTER
HEALTH
COMMUNICATION
POWER
TRANSPORT
SAFETY
```

Each dimension may have independent states.

Example:

```text
WATER:
STABLE → LOW → CRITICAL → RESTORING → STABLE
```

Combined state may form a vector:

```text
V(t) = [
  food,
  water,
  shelter,
  health,
  communications
]
```

The purpose is to detect imbalance and changing need.

---

# 13. Recovery-First Invariant

When:

* human life may be at risk;
* uncertainty is high;
* multiple actions are possible;
* escalation would increase irreversible risk;

the system SHOULD prefer:

```text
PRESERVE
    ↓
RECOVER
    ↓
VERIFY
    ↓
REASSESS
```

unless a more appropriate lawful emergency response is clearly established.

This is the **Recovery-First Invariant**.

---

# 14. Distress Protocol

A distress message SHOULD describe:

```text
actor
need
severity
location
time
evidence
confidence
```

Conceptual example:

```json
{
  "type": "DISTRESS",
  "need": "MEDICAL",
  "severity": "CRITICAL",
  "confidence": 0.94
}
```

The receiver SHOULD produce an acknowledgement.

```text
DISTRESS
    ↓
RECEIVED
    ↓
VERIFIED
    ↓
ACKNOWLEDGED
```

The acknowledgement MUST NOT falsely imply that help has arrived.

Possible states SHOULD distinguish:

```text
RECEIVED
VERIFIED
QUEUED
DISPATCHED
ARRIVED
RESOLVED
```

---

# 15. Stable Nodes

A Stable Node is a trusted or verified support endpoint.

Examples include:

```text
clinic
shelter
water station
food point
communications hub
coordination centre
```

Stable Nodes MUST have state.

Example:

```json
{
  "node": "clinic-21",
  "state": "AVAILABLE",
  "verified_at": "timestamp"
}
```

A stale Stable Node record MUST NOT be treated as current indefinitely.

---

# 16. Acknowledgement

NSIGII distinguishes:

```text
delivery
receipt
verification
acceptance
completion
```

These are separate events.

A message being received does not mean that it was verified.

A message being verified does not mean that its requested action was completed.

---

# 17. Ledger

NSIGII MAY maintain an append-oriented incident or trust ledger.

The ledger may store:

```text
message identifiers
verification results
state changes
acknowledgements
policy decisions
actions
errors
resource allocations
```

A ledger entry SHOULD include causality where possible.

Example:

```text
event A
    ↓ caused
decision B
    ↓ caused
action C
```

---

# 18. Tamper Evidence

Implementations SHOULD be capable of detecting inconsistent modification.

Possible mechanisms include:

* cryptographic hashes;
* signatures;
* authenticated data structures;
* append-only logs;
* sequence numbers;
* linked event hashes.

NSIGII does not prescribe a single cryptographic primitive.

---

# 19. C-S-H Security Requirements

NSIGII adopts:

```text
CORRECTNESS
SOUNDNESS
HARDNESS
```

## Correctness

Legitimate valid input produces the expected successful verification result.

## Soundness

Invalid input does not incorrectly pass required verification.

## Hardness

Breaking the security guarantees should be computationally infeasible under the declared threat model and chosen primitives.

Claims about complexity or security MUST be justified by the actual cryptographic construction.

---

# 20. Threat Model

Each NSIGII deployment SHOULD define its threat model.

Potential threats include:

```text
message modification
message replay
identity spoofing
duplicate events
out-of-order delivery
malformed input
encoding ambiguity
stale data
false claims
compromised endpoints
network partition
resource exhaustion
clock disagreement
```

Not every deployment needs to address every threat equally.

---

# 21. Failure Model

NSIGII prefers explicit failure states.

Examples:

```text
MALFORMED
UNVERIFIED
UNKNOWN_IDENTITY
EXPIRED
REPLAYED
CONTEXT_MISMATCH
SIGNATURE_INVALID
TAMPERED
UNSUPPORTED_VERSION
INSUFFICIENT_EVIDENCE
```

Errors SHOULD be machine-readable.

---

# 22. Human Needs Model

For MMUCO integration, NSIGII treats essential needs as operational state:

```text
FOOD
WATER
SHELTER
INTEGRITY
```

An implementation MAY extend this to:

```text
MEDICAL
POWER
SANITATION
COMMUNICATION
TRANSPORT
SAFETY
```

Example:

```text
WATER = CRITICAL
SHELTER = STABLE
MEDICAL = DEGRADED
```

This allows resource conditions to participate in state-transition logic.

---

# 23. Policy Gate

Verification alone does not determine action.

After verification:

```text
VERIFIED EVENT
      ↓
POLICY GATE
      ↓
PERMITTED RESPONSE
```

Policy may consider:

```text
authorization
urgency
confidence
resource availability
human safety
legal constraints
privacy
consent
```

---

# 24. Privacy

NSIGII implementations SHOULD minimize unnecessary personal information.

Sensitive fields SHOULD support:

* redaction;
* access control;
* encryption;
* retention limits;
* pseudonymous identifiers.

Location information is particularly sensitive.

The system SHOULD reveal only the precision necessary for the response.

---

# 25. Human Override

Automated analysis MAY recommend or classify.

Human decision-makers MUST be able to inspect:

```text
what was observed
what was verified
what was inferred
why the recommendation was produced
```

Critical civilian-protection decisions SHOULD remain reviewable.

---

# 26. Parser Requirements

A NSIGII parser MUST safely handle:

```text
empty input
invalid encoding
unknown version
unknown fields
duplicate fields
oversized payloads
truncated messages
invalid signatures
unexpected nesting
malformed numeric values
```

Malformed input MUST NOT crash the parser.

---

# 27. Fuzzing Requirements

The reference implementation SHOULD include fuzzing for:

```text
parser
canonicaliser
decoder
state-transition engine
verification boundary
```

Fuzzing success MUST NOT be presented as proof of correctness.

It is one testing technique among several.

---

# 28. Reference API

A future implementation may expose:

```text
nsigii.parse()
nsigii.canonicalise()
nsigii.transmit()
nsigii.receive()
nsigii.verify()
nsigii.classify()
nsigii.transition()
nsigii.respond()
nsigii.audit()
```

---

# 29. Example Flow

```text
Person reports:
"No drinking water remains at Node A."

        ↓

TRANSMIT

        ↓

RECEIVE

        ↓

VERIFY SOURCE

        ↓

VERIFY NODE

        ↓

CHECK RECENT SENSOR / REPORT DATA

        ↓

CLASSIFY
WATER = CRITICAL

        ↓

STATE TRANSITION
LOW → CRITICAL

        ↓

POLICY GATE

        ↓

RESOURCE RESPONSE

        ↓

ACKNOWLEDGEMENT

        ↓

AUDIT
```

---

# 30. Constitutional Rules

NSIGII follows these protocol rules:

```text
1. Receipt is not truth.
2. Authority is not verification.
3. Uncertainty must be represented.
4. State change must be traceable.
5. Evidence and inference must remain distinguishable.
6. Human life takes priority in ambiguous emergency conditions.
7. Automated systems must fail safely.
8. Important decisions must be auditable.
9. Sensitive data must be minimized.
10. Verification must precede irreversible action where reasonably possible.
```

---

# 31. Versioning

Protocol versions SHOULD follow semantic versioning where practical.

Example:

```text
NSIGII 1.0.0
```

Breaking message-format or semantic changes require a major version change.

---

# 32. Status

This document defines a **draft protocol architecture**.

It does not claim that the repository currently contains a complete conforming implementation.

Conformance should only be claimed after the implementation passes reproducible tests defined by the specification.

---

# 33. Summary

NSIGII reduces to a simple principle:

```text
DO NOT TRUST BECAUSE IT ARRIVED.
VERIFY.
```

Its operational architecture is:

```text
TRANSMIT
RECEIVE
VERIFY
CLASSIFY
TRANSITION
RESPOND
AUDIT
```

Its human-centred invariant is:

```text
WHEN UNCERTAIN:
PRESERVE → RECOVER → VERIFY → REASSESS
```
