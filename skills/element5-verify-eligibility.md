---
name: Verify patient eligibility with Element5
description: Submit an eligibility/benefit verification request and track it to completion.
api: openapi/element5-openapi-original.json
operations:
- submitEligibilityRequest
- getSubmitEligibilityRequestStatus
---

# Verify patient eligibility with Element5

Use this skill to check a patient's insurance eligibility/benefits through the Element5 workflow API.

## Authentication

Every request must send the `X-API-Key` header with the key Element5 issued during onboarding. Use the
non-production host `https://api-qa.e5.ai` for testing and `https://api.e5.ai` for production.

## Steps

1. Submit the eligibility request with `submitEligibilityRequest`
   (`POST /wf/v2/workflows/submit-eligibility`). The response returns a `task-id` and an initial
   status of `Queued`.
2. Track completion one of two ways:
   - Poll `getSubmitEligibilityRequestStatus`
     (`GET /wf/v2/workflows/submit-eligibility/{task-id}`) until status is `Success` or `Failed`; or
   - Receive `E5TaskEvents` webhook callbacks (`task#succeeded` / `task#failed`) at your configured
     webhook URL instead of polling.
3. On `Success`, read the eligibility result from the status response. On `Failed`, read the `reason`
   field.

## Rules

- Long-running by design — do not expect a synchronous result from the submit call; always resolve via
  the `task-id`.
- Errors use a `{ code, message }` envelope. `401` means the API key is missing/invalid; `403` means
  the key is not authorized for this workflow. See `errors/element5-problem-types.yml`.
- No idempotency key is supported — avoid blind retries of the submit call; reconcile by `task-id`.
