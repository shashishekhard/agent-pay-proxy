# Architecture Decision Record: Agentic Payment Authorization Proxy

## Context & Threat Model
Autonomous AI agents executing payments introduce failure modes that traditional checkout flows do not handle:
1. **Prompt Injection / Scope Creep:** A compromised agent attempts to spend above user intent or at an unauthorized merchant.
2. **Hallucinated Retries:** An LLM reasoning loop fires duplicate tool calls concurrently or after network timeouts.
3. **Credential Exposure:** Giving an agent raw PANs or reusable merchant tokens creates unbounded blast radius.

## Core Design Invariants
* **Zero-Trust Delegation:** Agents never receive payment credentials. They receive short-lived (2–5 min) Ed25519-signed capability tokens bound to `max_amount_minor`, `allowed_mccs`, `allowed_merchants`, and a SHA-256 `intent_hash` of the user-approved cart.
* **Fail-Closed Control Plane:** If Redis (replay store) or PostgreSQL (idempotency/audit store) is unreachable, the proxy immediately returns `503 Service Unavailable`. It never degrades to "allow."
* **3-Layer Replay Protection:**
  1. **Transport:** Rejects requests with timestamp drift > 30 seconds.
  2. **Token Consumption:** Atomic Redis `SET jti:{id} "consumed" NX EX {ttl}` guarantees single-use tokens cannot be spent twice under concurrency.
  3. **Ledger Idempotency:** PostgreSQL `UNIQUE(idempotency_key)` state machine (`PENDING` -> `AUTHORIZED` | `FAILED`) ensures timeout retries return the cached decision without double-charging.
