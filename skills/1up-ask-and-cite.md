---
name: Ask 1up a grounded question
description: >-
  Query a 1up workspace for a source-grounded answer to a sales, product or security
  question, then confirm and log what it was grounded in.
api: mcp/1up-mcp.yml
surface: mcp
endpoint: https://mcp.1up.ai/mcp
operations:
  - list_workspaces
  - switch_workspace
  - get_workspace_info
  - ask_question
  - search_qa_library
  - list_knowledge_groups
  - get_audit_events
generated: '2026-09-05'
method: generated
source: https://help.1up.ai/en/articles/14304740-mcp
---

# Ask 1up a grounded question

1up's whole product claim is that it answers **only** from approved sources a customer
trusts, with citations, rather than generating freely. Use it that way.

## Steps

1. **`list_workspaces`** → **`switch_workspace`**. The answer depends entirely on which
   workspace's knowledge you are querying.
2. Optionally **`list_knowledge_groups`** to see which source groupings exist, so you can
   scope the question to the right body of knowledge.
3. **`ask_question`** — the knowledge-base query, with optional filters.
   - This call **streams over SSE and can take up to 120 seconds**. 1up's own client allows
     130s. A 30-second timeout will report a false failure on a perfectly normal answer.
   - It is a read. Retrying it is safe, but on the metered MCP plan it is not free.
4. Cross-check with **`search_qa_library`** when the answer is going into a customer-facing
   document. If a curated Q&A pair already covers the question, prefer it — a human approved
   it.
5. **`get_audit_events`** (paginated) to record what happened, if you are operating on
   someone's behalf.

## Rules

- **Never present a 1up answer as unsourced.** Return the citations 1up returns with it; the
  product is built to refuse rather than to guess, and stripping the sources throws away the
  only reason to trust it.
- **Check the budget first.** `get_workspace_info` reports plan, limits and usage. There is
  no rate limit, no `429` and no `Retry-After` — the only thing that stops you is quota or a
  bill at $0.05 per answered question.
- **Watch the tenant.** No OAuth scope pins you to one workspace; `switch_workspace` moves
  every subsequent call.
