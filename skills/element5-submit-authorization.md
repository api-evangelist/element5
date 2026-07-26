---
name: Submit a prior authorization with Element5
description: Upload supporting documents and submit a prior-authorization request, then track it.
api: openapi/element5-openapi-original.json
operations:
- uploadFileObject
- submitAuthorizationRequest
- getSubmitAuthorizationRequestStatus
---

# Submit a prior authorization with Element5

Use this skill to submit a prior-authorization workflow and monitor it to completion.

## Authentication

Send `X-API-Key` on every request (key issued at onboarding). Test against `https://api-qa.e5.ai`
(files via `https://blob-qa.e5.ai`); go live against `https://api.e5.ai` (`https://blob.e5.ai`).

## Steps

1. If the authorization needs supporting documents, upload each with `uploadFileObject`
   (`PUT /store/v1/file-object`) and keep the returned object id. A `413` means the file is too large.
2. Submit the authorization with `submitAuthorizationRequest`
   (`POST /wf/v2/workflows/submit-authorization`), referencing any uploaded document ids. The response
   returns a `task-id` and status `Queued`. (This operation is marked Draft in the spec — confirm the
   current contract with Element5.)
3. Track completion via polling `getSubmitAuthorizationRequestStatus`
   (`GET /wf/v2/workflows/submit-authorization/{task-id}`) or via `E5TaskEvents` webhook callbacks.
4. On `Success`, read the authorization result; on `Failed`, read the `reason`.

## Rules

- Resolve everything by `task-id`; the submit call is asynchronous.
- To fetch a stored document later, use `getFileObject` (`GET /store/v1/file-object/{object-id}`),
  which may `302` redirect to a signed blob URL.
- Error envelope is `{ code, message }`; see `errors/element5-problem-types.yml`.
