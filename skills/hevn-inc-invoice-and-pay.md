---
name: Create an invoice and pay it
description: >-
  Add a payout contact, create an invoice, then settle it with a HEVN transfer
  against the invoice context.
api: openapi/hevn-inc-openapi-original.json
operations:
  - get_contacts_api_v1_user_contacts_get
  - create_invoice_api_v1_documents_contracts_invoices_post
  - list_invoices_api_v1_documents_contracts_invoices_get
  - get_invoice_api_v1_documents_contracts_invoices__invoice_id__get
  - transfer_api_v1_mcp_transfer_post
---

# Create an invoice and pay it

## Auth
HEVN API key (`hvn_...`) as a bearer token or `X-Api-Key` header. See
`authentication/hevn-inc-authentication.yml`.

## Steps
1. **Find or list contacts** — `get_contacts_api_v1_user_contacts_get` (GET
   `/api/v1/user/contacts?limit=100&offset=0`) to locate the counterparty.
2. **Create the invoice** — `create_invoice_api_v1_documents_contracts_invoices_post`
   (POST `/api/v1/documents/contracts/invoices`) with the line items and
   contact/contract reference.
3. **Verify it** — `get_invoice_api_v1_documents_contracts_invoices__invoice_id__get`
   (GET `/api/v1/documents/contracts/invoices/{invoice_id}`) and/or
   `list_invoices_api_v1_documents_contracts_invoices_get` to confirm state.
4. **Pay it** — `transfer_api_v1_mcp_transfer_post` (POST `/api/v1/mcp/transfer`)
   resolving amount and recipient from the invoice context. Pass a stable
   `Idempotency-Key`.

## Rules
- Paginate lists with `limit` + `offset`.
- Idempotency-Key on the transfer prevents double payment on retry.
- Handle 422 (validation) and 404 (missing invoice) per
  `errors/hevn-inc-problem-types.yml`.
