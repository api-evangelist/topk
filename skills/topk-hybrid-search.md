---
name: Build a collection and run hybrid search
description: Create a TopK collection with keyword and semantic indexes, upsert documents, and run a hybrid (vector + multi-vector + BM25 + filter) query.
api: topk:collection-api
operations:
  - collections.create
  - collection.upsert
  - collection.query
source: https://docs.topk.io/sdk/topk-py/overview
auth: Bearer TOPK_API_KEY (region-scoped client)
---

# Build a collection and run hybrid search on TopK

Use this skill to stand up structured search over your own documents with TopK's Collection API. TopK combines dense-vector, multi-vector (late-interaction), sparse-vector, and BM25 keyword search plus metadata filtering in a single query — with no separate embedding pipeline, vector store, or reranking service to maintain.

## Prerequisites
- A TopK API key from `console.topk.io`, exported as `TOPK_API_KEY`.
- A region (e.g. `aws-us-east-1-elastica` for US East, `aws-eu-central-1-monstera` for EU Central).
- `pip install topk-sdk` (Python 3.9+) — or use `topk-js`, `topk-rs`, or a PostgreSQL client via `topk-sql`.

## Steps

1. **Create the client** with your API key and region.
   ```python
   import os
   from topk_sdk import Client
   client = Client(api_key=os.environ["TOPK_API_KEY"], region="aws-us-east-1-elastica")
   ```

2. **Create a collection** (`collections.create`) with a typed schema — annotate text fields with `keyword_index()` for BM25 and `semantic_index()` for multi-vector semantic retrieval.
   ```python
   from topk_sdk.schema import text, keyword_index, semantic_index
   client.collections().create("books", schema={
       "title": text().required().index(keyword_index()),
       "content": text().index(semantic_index()),
   })
   ```

3. **Upsert documents** (`collection.upsert`). Every document is keyed by `_id`; re-upserting the same `_id` replaces it (data-level idempotency — see `conventions/topk-conventions.yml`).
   ```python
   client.collection("books").upsert([
       {"_id": "1", "title": "1984", "content": "It was a bright cold day in April ...", "author": "George Orwell"},
   ])
   ```

4. **Query** (`collection.query`) — combine keyword, semantic/vector, and metadata filters, and set a top-k limit.

## Rules
- Always send the API key as `Authorization: Bearer <TOPK_API_KEY>`; keep the region consistent with where the collection lives.
- Use upsert-by-`_id` for safe re-ingestion; there is no request Idempotency-Key header.
- Handle errors via the SDK's typed error classes (`topk_sdk.error`); the SDK retries transient failures automatically.
- Respect the production limits at https://docs.topk.io/limits.
