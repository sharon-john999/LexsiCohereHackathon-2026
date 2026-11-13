# Dataset Card: curatorkit_run

## Summary

- **Run timestamp**: 2026-11-09T11:08:55.907406Z
- **Wall clock time**: 13.8s
- **Pipeline config hash**: `2d750e1357aa677a`

## Source Documents

  - Unknown

## Pipeline Stages

| Step | Input | Passed | Recovered | Rejected | Forward |
|------|-------|--------|-----------|----------|---------|
| JSONLReader | 546 | 546 | — | 0 | 546 |
| SchemaGate | 546 | 546 | 0 | 0 | 546 |
| ExactDeduplicator | 546 | 546 | — | — | 546 |
| MinHashDeduplicator | 546 | 508 | — | — | 508 |
| TextCleaner | 508 | 508 | — | — | 508 |
| ShareGPTExporter | — | 508 | — | — | 508 |

## Gate Pass Rates and Rejection Breakdown

- **JSONLReader**: 546/0 passed (N/A), 0 rejected
- **SchemaGate**: 546/546 passed (100.0%), 0 rejected


**Rejection reasons:**
  - None

## Deduplication

  - near_duplicates_removed: 38
  - surviving_count: 508

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
