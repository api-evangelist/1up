---
name: Automate a questionnaire with 1up
description: >-
  Upload an RFP, DDQ or security questionnaire to a 1up workspace, generate answers,
  review and approve them, and export the completed document — over the 1up MCP server.
api: mcp/1up-mcp.yml
surface: mcp
endpoint: https://mcp.1up.ai/mcp
operations:
  - get_workspace_info
  - upload_questionnaire
  - list_questionnaires
  - get_questionnaire
  - get_questionnaire_questions
  - regenerate_answer
  - update_questionnaire_answer
  - approve_question
  - bulk_approve_questions
  - request_questionnaire_export
generated: '2026-09-05'
method: generated
source: https://help.1up.ai/en/articles/14304740-mcp
---

# Automate a questionnaire with 1up

Every operation named here is a tool 1up documents on its own MCP server. Nothing is
invented. The live `inputSchema` for each tool is OAuth-gated, so parameter names are not
given below — read them from `tools/list` after authenticating.

## Before you start

1. Connect to `https://mcp.1up.ai/mcp` over Streamable HTTP. Authentication is OAuth 2.1
   (authorization code + PKCE `S256`, dynamic client registration at
   `https://mcp.1up.ai/register`). The token goes in the `Authorization` header.
2. Call **`list_workspaces`**, then **`switch_workspace`** to pin the right tenant.
   *This matters more here than on most APIs*: no OAuth scope pins you to a workspace, so
   every write after this point lands wherever `switch_workspace` last pointed.
3. Call **`get_workspace_info`** and read `plan`, `limits` and `usage` **before** you
   generate anything. On the MCP plan 1up bills **$0.05 per question answered**, and on the
   Free plan you get 50 answers a month. There is no rate limit and no `429` — this call is
   your only budget signal.

## Run the questionnaire

4. **`upload_questionnaire`** — XLSX, DOCX, PDF or CSV. Do not retry this on a timeout:
   1up documents no idempotency key, and its own client deliberately never retries writes.
   If a call times out, use **`list_questionnaires`** to check whether the upload landed
   before trying again.
5. **`list_questionnaires`** / **`get_questionnaire`** — confirm the upload and read
   processing state. A `423 Locked` means the questionnaire is busy; wait and retry — 423 is
   the one status where 1up passes the server's own message through verbatim, so read it.
6. **`get_questionnaire_questions`** — pull questions with their generated answers.
   Answer generation is slow by design: allow up to ~120 seconds per streamed answer, not
   the 30 seconds you would normally budget.

## Review before you approve

7. For any answer that is wrong or thin, either **`regenerate_answer`** (re-run the model)
   or **`update_questionnaire_answer`** (write the text yourself).
8. Where a question needs a human, **`assign_question`** to a teammate and
   **`add_question_comment`** with what you need from them. Do not approve on their behalf.
9. **`approve_question`** for individual answers. Use **`bulk_approve_questions`** only when
   a human has actually reviewed the batch — approval is the human-accountability step in
   1up's workflow, and bulk-approving unreviewed model output defeats the purpose of the
   product.

## Export

10. **`request_questionnaire_export`** — XLSX, DOCX or CSV.

## Error handling

| Status | Meaning | Do |
|---|---|---|
| 400 | Bad request, or no workspace selected | Fix the payload, or `switch_workspace` |
| 401 | Token missing/expired | Re-run the OAuth flow |
| 403 | RBAC denial **in this workspace** | Check role, or switch workspace |
| 404 | Wrong ID, **or right ID in the wrong workspace** | Verify workspace before assuming it is gone |
| 423 | Resource locked, likely mid-processing | Read the server message, wait, retry |
| 5xx | Server error | Safe to retry reads; never blind-retry writes |

Errors are `{"error", "error_description"}` on MCP and `{"detail": ...}` on the platform
API. They are **not** RFC 9457 problem+json and carry no stable error codes.
