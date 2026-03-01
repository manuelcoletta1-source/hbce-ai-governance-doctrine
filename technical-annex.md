# Technical Annex (Normative)

## Scope

This annex provides a normative technical reference for implementing deterministic, fail-closed AI execution attestation under the HBCE doctrine.

It specifies:

- Required fields (minimal event)
- Hashing rule
- Chain rule
- Signature rule (Ed25519)
- Deterministic verification procedure (PASS/FAIL)

---

## 1. Minimal Event Object (Normative)

An implementation SHALL produce an event object containing at minimum:

### 1.1 Required Top-Level Fields

- `spec` (string)
- `event_id` (integer)
- `ts` (string, ISO 8601)
- `ipr_ai` (string)
- `ipr_operator` (string)
- `policy.policy_pack_id` (string)
- `input.canonical` (string)
- `input.sha256` (string, hex)
- `output.canonical` (string)
- `output.sha256` (string, hex)
- `chain.prev` (string, hex)
- `chain.entry` (string, hex)
- `chain.algo` (string)

If policy enforcement requires signatures, the following SHALL also be present:

- `sign.alg` (string, `"ED25519"`)
- `sign.key_id` (string)
- `sign.pub_ref` (string)
- `sign.sig` (string, base64)

---

## 2. Canonicalization (Normative)

Implementations MUST define deterministic canonicalization for:

- `input.canonical`
- `output.canonical`

Canonicalization MUST be stable and reproducible.

Hashing SHALL be computed over the canonical content bytes only.

---

## 3. Hashing Rule (Normative)

input_sha256  = sha256(input.canonical) output_sha256 = sha256(output.canonical)

Output SHALL be lowercase hex.

`input.sha256` and `output.sha256` MUST equal recomputed hashes under verification.

---

## 4. Chain Rule (Normative)

### 4.1 Concatenation Order

A string `base` SHALL be constructed exactly as:

base = prev | input_sha256 | output_sha256 | policy_pack_id | ts | ipr_ai | ipr_operator

Where `|` denotes a literal pipe character as delimiter.

### 4.2 Entry Computation

chain.entry = sha256(base)

Output SHALL be lowercase hex.

### 4.3 Continuity Constraint

`chain.prev` SHALL equal the prior event's `chain.entry`.

If continuity fails, verification SHALL FAIL.

---

## 5. Signature Rule (Normative)

If signature is required by policy:

### 5.1 Payload Definition

payload = ascii(chain.entry)

Where `ascii(chain.entry)` is the raw ASCII bytes of the 64-character hex string.

Payload length SHALL be 64 bytes.

### 5.2 Algorithm

signature_algorithm = ED25519

### 5.3 Signature Encoding

The signature SHALL be stored as:

- base64 of the 64-byte Ed25519 signature.

### 5.4 Verification Rule

Verification SHALL compute:

ED25519.verify(pubkey, payload, signature) == true

If false, verification SHALL FAIL.

---

## 6. Deterministic Verification Procedure (Normative)

A verifier SHALL perform the following steps in order:

1. Load policy pack and determine enforcement parameters.
2. Load event data and validate required fields exist.
3. Recompute `input_sha256` and compare to `input.sha256`.
4. Recompute `output_sha256` and compare to `output.sha256`.
5. Recompute `base` with the exact concatenation order and delimiter.
6. Recompute `chain.entry` and compare to stored `chain.entry`.
7. Validate `chain.prev` continuity against known prior entry (when available).
8. If policy requires signature:
   - Load public key referenced by `sign.pub_ref`
   - Verify Ed25519 signature over `ascii(chain.entry)`
9. Return result as:
   - PASS, or
   - FAIL

No partial states are permitted.

---

## 7. PASS/FAIL Output Contract (Normative)

A compliant verifier SHALL output one of:

- `PASS`
- `FAIL`

Implementations MAY provide forensic diagnostics,
but diagnostics MUST NOT change the binary result.

---

## 8. Security Notes (Non-Normative)

- Private key protection and operational key rotation are out of scope.
- Canonicalization quality is an implementation-critical risk surface.
- Fail-closed posture is required for accountability-critical deployments.

---

## Annex Closing Statement

This annex defines the minimal deterministic primitives required
to transform AI execution into an attestable operational event.

Governance is enforced through identity binding, deterministic hashing,
append-only chaining, and optional cryptographic signature under policy.


---


