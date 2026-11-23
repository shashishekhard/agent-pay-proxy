# Step-by-Step Walkthrough: `agent-pay-proxy`

## Request Lifecycle
When an AI agent sends `POST /v1/authorize`, the request passes through six strict gates:
1. **Transport Freshness:** Rejects requests with clock drift > 30 seconds.
2. **Cryptographic Verification:** Validates the Ed25519 signature and expiration (`exp`) of `X-Agent-Token`.
3. **Synchronous Policy Engine:** Enforces `max_amount_minor`, `currency`, `allowed_mccs`, `allowed_merchants`, and `intent_hash`.
4. **Idempotency Lock (Postgres):** Handles legitimate network retries by returning cached responses (`X-Idempotent-Replay: true`) or blocking in-flight duplicates (`409 Conflict`).
5. **Single-Use Token Consumption (Redis):** Atomically locks `jti:{token_id}` via `SET NX` to prevent concurrent reuse across distinct idempotency keys.
6. **PSP Execution & Audit:** Calls the downstream PSP, finalizes the idempotency record, and writes an immutable record to `dispute_audit_log`.
