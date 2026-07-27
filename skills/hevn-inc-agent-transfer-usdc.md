---
name: Transfer USDC with an MCP agent
description: >-
  Resolve a recipient, check balance, and send USDC on Base through HEVN's
  agent-first MCP surface, using an idempotency key so retries are safe.
api: openapi/hevn-inc-openapi-original.json
operations:
  - get_mcp_balance_api_v1_mcp_get_balance_get
  - resolve_address_api_v1_mcp_resolve_address_get
  - transfer_api_v1_mcp_transfer_post
  - transfer_to_user_api_v1_mcp_transfer_to_user_post
  - list_transfers_api_v1_mcp_transfers_get
---

# Transfer USDC with an MCP agent

HEVN exposes a dedicated agent surface under `/api/v1/mcp/*`. Install it once with
`hevn mcp install` (Claude Code / Codex / Cursor) or call the REST endpoints directly.

## Auth
Send your HEVN API key (`hvn_...`) as `Authorization: Bearer hvn_...`, or as
`X-Api-Key: hvn_...` when the deployment sets `HEVN_API_KEY_HEADER=X-Api-Key`.
Money movement is bounded by the spend permission attached to your API key.

## Steps
1. **Check balance** — `get_mcp_balance_api_v1_mcp_get_balance_get` (GET
   `/api/v1/mcp/get_balance`) returns the USDC balance on Base. Confirm funds
   cover the amount before moving on.
2. **Resolve the recipient** — if you only have an email, call
   `resolve_address_api_v1_mcp_resolve_address_get` (GET
   `/api/v1/mcp/resolve_address`) to get the smart wallet address.
3. **Send** — call `transfer_api_v1_mcp_transfer_post` (POST
   `/api/v1/mcp/transfer`) using invoice/contact/quote context, or
   `transfer_to_user_api_v1_mcp_transfer_to_user_post` to pay a HEVN user via
   spend permission. Always pass an `Idempotency-Key` header (or
   `idempotency_key`) with a stable operation id so a retry never double-sends.
4. **Confirm** — `list_transfers_api_v1_mcp_transfers_get` (GET
   `/api/v1/mcp/transfers`) to verify the transfer landed.

## Rules
- Idempotency is mandatory for automation: reuse the same key on retries (see
  `conventions/hevn-inc-conventions.yml`).
- A 422 means validation failed — read `detail[]` and fix inputs; do not retry
  blindly (see `errors/hevn-inc-problem-types.yml`).
- From the CLI, use `--dry-run` to preview a mutating transfer and `--yes` only
  when the user has already approved the action.
