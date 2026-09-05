---
name: Curate the 1up Answer Library
description: >-
  Search, create, correct and retire the reusable Q&A pairs 1up answers from, and organise
  knowledge-base documents into knowledge groups that scope which sources an answer may use.
api: mcp/1up-mcp.yml
surface: mcp
endpoint: https://mcp.1up.ai/mcp
operations:
  - search_qa_library
  - get_qa_pair
  - create_qa_pair
  - update_qa_pair
  - delete_qa_pair
  - list_kb_items
  - get_kb_item
  - upload_kb_document
  - delete_kb_item
  - list_knowledge_groups
  - get_knowledge_group
  - create_knowledge_group
  - add_items_to_knowledge_group
  - remove_items_from_knowledge_group
  - delete_knowledge_group
generated: '2026-09-05'
method: generated
source: https://help.1up.ai/en/articles/14304740-mcp
---

# Curate the 1up Answer Library

The Answer Library is the reusable knowledge 1up answers from. Curating it well is what
makes generated answers correct; this is the highest-leverage and highest-blast-radius work
an agent does in 1up.

## Find before you create

1. **`search_qa_library`** (paginated) before every **`create_qa_pair`**. Duplicated Q&A
   pairs are the main way an answer library degrades — two contradictory pairs answering the
   same question produce a coin-flip, and 1up has no dedupe.
2. **`get_qa_pair`** to read one by ID. **`update_qa_pair`** to correct an existing pair in
   place. Prefer updating over creating a near-duplicate.

## Deletion is not uniform — read this before you delete anything

1up's three destructive tools behave differently, and it says so in its own tool
descriptions:

- **`delete_qa_pair`** — *"Archive a Q&A pair."* A soft delete. But **no restore tool is
  published and no retention window is stated**, so you cannot promise a human it is
  recoverable. Say "archived", not "deleted", and not "recoverable".
- **`delete_knowledge_group`** — *"Delete a group (items are ungrouped, not deleted)."*
  Safe: the documents and Q&A pairs survive. Rebuild with `create_knowledge_group` +
  `add_items_to_knowledge_group`.
- **`delete_kb_item`** — plain *"Delete a KB item."* No archive language. **Treat as
  unrecoverable** and get explicit human confirmation first.

Nothing in 1up is idempotent and no operation takes a dry-run flag. There is no preview.

## Knowledge base

3. **`list_kb_items`** with search/source filters, **`get_kb_item`** for detail.
4. **`upload_kb_document`** — PDF, DOCX, XLSX, CSV. Connected sources (Confluence, Notion,
   Google Drive, OneDrive, SharePoint, Box, Dropbox, Egnyte, Salesforce, Highspot, Seismic,
   Zendesk, Gong, GitBook, GitHub, websites) are wired in the 1up dashboard, not over MCP.

## Knowledge groups

5. **`list_knowledge_groups`** / **`get_knowledge_group`** — a group returns its documents
   *and* its Q&A pairs together.
6. **`create_knowledge_group`**, then **`add_items_to_knowledge_group`** /
   **`remove_items_from_knowledge_group`**. Groups scope which sources an answer may draw
   from, so grouping is an accuracy control, not just filing.

## Workspace hygiene

Confirm the active workspace with **`list_workspaces`** / **`switch_workspace`** before any
write. A `404` from 1up frequently means *right ID, wrong workspace* — its own error text
says so — so check tenancy before concluding something was deleted.
