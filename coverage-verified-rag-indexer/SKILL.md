---
name: coverage-verified-rag-indexer
description: >-
  Build a local RAG document indexer that NEVER silently drops chunks — block-structured stages with
  per-stage error attribution, a coverage check (stored count == chunk count) before a job is marked done,
  OCR-capable extraction, token-based chunking, and an atomic job claim so two workers can't double-process.
  Use when building or debugging a document → embeddings → vector DB pipeline, or when retrieval quality is
  bad because indexing failed silently. Trigger on "build a RAG indexer", "embeddings worker", "chunks
  missing / duplicated", "Milvus/Qdrant ingest", or "reindex pipeline".
---

# Coverage-Verified RAG Indexer

Most RAG indexers fail silently — a crash mid-batch, a dropped insert, a duplicate run — and you only find
out when answers are bad. This pattern makes failures **loud and impossible to miss.**

## Pipeline (block-structured stages)
`resolve_file → extract (OCR) → chunk → embed (batched+retry) → upsert → verify_coverage → mark_complete`

Each stage raises `StageError(stage, msg)`. On failure, record **which stage** + the message into the job
row (`error_stage`, `error_msg`). No more "it just hangs" — you always know where it broke.

## The key idea: coverage verification
After insert + flush, query the vector DB for `count(doc, version)` and assert it **equals the number of
chunks**. Mismatch → the job **FAILS** (it never marks `complete`, so a corrupt index is never served):
```python
stored = count_in_vectordb(doc, version)
if stored != len(chunks):
    raise StageError("verify", f"coverage mismatch: stored {stored} of {len(chunks)}")
```
This single check catches dropped chunks, partial inserts, and duplication (e.g. a 2× count from two workers).

## Extraction — handle real documents
Use an OCR-capable parser (e.g. **Docling**) so scanned/image PDFs and `.docx` tables don't yield empty text.
If extraction returns near-nothing, fail at `extract` (flag `needs_ocr`) instead of embedding a blank doc.

## Chunking — measure in TOKENS, not characters
Chunk with the **embedding model's own tokenizer**, target **500–1000 tokens** (e.g. 800) with ~20% overlap,
or use a structure-aware chunker (Docling HybridChunker) that respects headings. Char-based chunking lies:
1200 chars ≈ 300 tokens, far below target.

## Concurrency — claim jobs atomically
A polling worker MUST claim jobs atomically, or two instances grab the same job and double-insert (the classic
2× coverage failure). With Postgres:
```sql
UPDATE jobs SET status='processing'
WHERE id = (SELECT id FROM jobs WHERE status='queued' ORDER BY id LIMIT 1 FOR UPDATE SKIP LOCKED)
RETURNING ...;
```

## Operational gotchas
- **Auto-ingest** by watching a folder + content hash (only re-index new/changed files); dedup against a
  **clearable** state table, NOT an append-only audit ledger (or you can't reset).
- **Run one instance** under launchd/systemd (the atomic claim is the real safety net).
- **Derive doc identity from the real filename** (slug), not a hand-typed map — spacing/casing mismatches bite.
- Vector DB query windows are capped (e.g. Milvus offset+limit ≤ 16384) — count via iteration for big corpora.

## Result
Every indexed doc is provably complete (count-verified), failures name their stage, and duplication can't slip
through. That's the difference between a demo RAG and one you can trust.
