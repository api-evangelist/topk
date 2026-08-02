---
name: Ingest documents and get grounded answers (RAG)
description: Upload unstructured document files to a TopK dataset, then retrieve passages and get evidence-backed, cited answers via the Dataset API, CLI, or MCP server.
api: topk:dataset-api
operations:
  - datasets.ingest
  - datasets.search
  - datasets.ask
source: https://docs.topk.io/datasets/ask
auth: Bearer TOPK_API_KEY (region-scoped)
---

# Ingest documents and get grounded answers on TopK

Use this skill to give an agent grounded, citable context from complex documents (PDF, Markdown, HTML, and more) using TopK's Dataset API — document parsing, semantic search, and question answering in one API.

## Prerequisites
- A TopK API key from `console.topk.io`, exported as `TOPK_API_KEY`.
- A region (`aws-us-east-1-elastica` or `aws-eu-central-1-monstera`).
- The CLI (`brew tap topk-io/topk && brew install topk`) or an SDK.

## Steps

1. **Ingest files** into a dataset (`datasets.ingest`). With the CLI:
   ```bash
   topk upload '*.pdf' --dataset my-dataset
   topk upload 'docs/**/*.md' --dataset my-dataset
   ```

2. **Search** for the most relevant passages (`datasets.search`):
   ```bash
   topk search "my query" --dataset my-dataset --top-k 10
   ```

3. **Ask** a natural-language question and get a grounded, evidence-backed answer with citations (`datasets.ask`):
   ```bash
   topk ask "What does the contract say about termination?" --dataset my-dataset --show-refs
   ```
   Modes: `auto` (default), `summarize`, `research`.

4. **From an AI agent**, connect the hosted MCP server instead of shelling out. Add the region endpoint (`https://elastica.api.topk.io/mcp` or `https://monstera.api.topk.io/mcp`) with `Authorization: Bearer <TOPK_API_KEY>`, then use the `list_datasets` tool to discover data and the `ask` tool to query it in natural language. See `mcp/topk-mcp.yml`.

## Rules
- Authenticate every call with the API key (`topk login`, `TOPK_API_KEY`, or `Authorization: Bearer`).
- Keep the dataset and client in the same region.
- Prefer `--show-refs` / grounded answers so responses stay citable and evidence-backed.
- Handle transient failures via SDK retries and typed errors (`topk_sdk.error`).
