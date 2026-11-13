# Dataset Card: curatorkit_run

## Summary

- **Run timestamp**: 2026-11-11T08:09:46.008189Z
- **Wall clock time**: 75.0s
- **Pipeline config hash**: `4b1468739825f0ab`

## Source Documents

  - Unknown

## Pipeline Stages

| Step | Input | Passed | Recovered | Rejected | Forward |
|------|-------|--------|-----------|----------|---------|
| JSONLReader | 3902 | 3902 | — | 0 | 3902 |
| SchemaGate | 3902 | 3878 | 0 | 24 | 3878 |
| ExactDeduplicator | 3878 | 3878 | — | — | 3878 |
| MinHashDeduplicator | 3878 | 3556 | — | — | 3556 |
| TextCleaner | 3556 | 3556 | — | — | 3556 |
| MessagesExporter | — | 3556 | — | — | 3556 |

## Gate Pass Rates and Rejection Breakdown

- **JSONLReader**: 3902/0 passed (N/A), 0 rejected
- **SchemaGate**: 3878/3902 passed (99.4%), 24 rejected


**Rejection reasons:**
  - `below_min_tokens:9`: 8
  - `below_min_tokens:6`: 6
  - `below_min_tokens:8`: 4
  - `below_min_tokens:2`: 2
  - `below_min_tokens:5`: 2
  - `below_min_tokens:7`: 2

## Deduplication

  - near_duplicates_removed: 322
  - surviving_count: 3556

## Known Limitations

- MinHash threshold used: **0.85** — this is a starting point, not a universal answer. Adjust per domain (legal, medical, and other domain-specific corpora may need a lower threshold to avoid losing structurally similar but semantically valid examples).
- Table extraction requires `extract_tables: true` on the PDF reader and MinerU with GPU support. Disabled by default.
- Token counting uses a whitespace tokenizer by default. For exact LLM context window
  enforcement, enable `use_tiktoken: true` in the SchemaGate config.
- Both within-run (MinHash/Diversity) and cross-run (EmbeddingDeduplicator) deduplication
  are available. Cross-run dedup persists an embedding index across runs; use
  `embedding_reset_index: true` to start fresh.

## Reproduction

```bash
pip install curatorkit
curatorkit run pipeline.yaml --output-dir ./out/
```

## Tool Versions

- **curatorkit**: 0.1.23
- **python**: 3.12.6
