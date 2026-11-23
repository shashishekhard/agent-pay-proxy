# Agentic Payment Authorization Proxy (`agent-pay-proxy`)

A fail-closed, zero-trust payment authorization proxy for autonomous AI agents featuring Ed25519-signed capability tokens, 3-layer replay protection, and an append-only dispute audit ledger.

## Quickstart & Chaos Tests

```bash
# 1. Run unit & fault-injection tests (no Docker required)
go test -v -race ./internal/proxy/...

# 2. Boot Postgres (auto-migrated) and Redis for full chaos tests
docker compose up -d postgres redis

# 3. Run the 50-goroutine concurrency storm, upsell block, and retry tests
go test -v -race ./tests/...

# 4. Run the proxy server
go run ./cmd/proxy
```

## Documentation
* `ARCHITECTURE.md` — 1-page Architecture Decision Record (ADR)
* `WALKTHROUGH.md` — File-by-file code and request-lifecycle explanation
* `COST_PROPOSAL.md` — AWS Pilot vs. Production HA cost and ROI breakdown
