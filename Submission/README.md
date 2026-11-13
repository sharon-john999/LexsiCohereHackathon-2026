# Hackathon submission package

This folder is a local, GitHub-ready package for reviewing and reproducing the ATC practice-model experiment. 

## Track and problem statement

- **Tracks:** Track 1 — Fine-tuning. Data curation is supporting work; the official problem-statement number is not recorded in this package.
- **Toolkits:** CuratorKit and AuditKit are used in the documented workflow. AlignTune is installed by a notebook dependency cell, but no feature use is evidenced.
- **Problem statement:** Can a compact language model be adapted with curated aviation communications data to provide a more domain-aligned, reproducible text-based ATC practice experience than its base model, while measuring phraseology and critical-token retention?

The intended use is classroom-style text practice and phraseology exploration. This model is not for live aviation communications or operational decisions.

## Package contents

- `data_to_adapter-working.ipynb` — dataset cleaning, curation, SFT, and evaluation workflow.
- `evaluation.ipynb` — base-versus-adapter evaluation and plots. The final GGUF conversion cell has been replaced with a note because this package excludes the merged model and GGUF.
- `HF_ATC_ARTICLE_DRAFT.md` — write-up draft with evaluation results and limitations.
- `Adapters/tiny-aya/` — adapter, tokenizer files, saved checkpoints, trainer state, and TensorBoard event logs.
- `data/`, `dataset/`, `test/`, `ground_train.jsonl`, `ground_test.jsonl` — local source, curated, synthetic, and evaluation data snapshots with their cards and provenance files.
- `Evaluation/eval/` — saved AuditKit runs, per-sample CSV, plots, and provenance.
- `outputs/` — additional local training log artifacts.
- `SHA256SUMS.txt` — SHA-256 checksums for the packaged files (excluding the manifest itself).

The base model weights are not bundled. Evaluation requires access to [`CohereLabs/tiny-aya-global`](https://huggingface.co/CohereLabs/tiny-aya-global); reviewers must obtain access and accept the publisher's license terms.

The merged `Final_model` and GGUF exports are intentionally omitted. The repository still includes the PEFT adapter needed for evaluation.

## Reproducing evaluation

1. Use Python 3.12.6 and install the notebook dependencies (`torch`, `transformers`, `peft`, `auditkit`, `numpy`,`curatorkit`, `pandas`, `aligntune`, and `matplotlib`).
2. Obtain access to the base model and authenticate with Hugging Face using the reviewer's own credentials.
3. Open `evaluation.ipynb` with this directory as the working directory and run the code cells in order. It reads `data/test.jsonl` and `Adapters/tiny-aya/`, and saves run results under `Evaluation/eval/`.
4. The saved comparison used five prompts, selected from 172 candidate controller turns with prompt-pool seed 0. Although the setup cell declares 25 samples, a later evaluation cell caps the run at 5 because evaluation compute was limited.


## Reproducing training and curation

`data_to_adapter-working.ipynb` expects the Parquet files under `dataset/sft/`, which are included. Run its cleaning, curation, and training sections in order. It saves a new adapter under `outputs/aya-ground-controller/`; copy the resulting adapter into `Adapters/tiny-aya/` before running the evaluation notebook if you want to test newly trained weights.

The adapter README records PEFT 0.21.2, TRL 1.17.1, Transformers 5.22.2, PyTorch 2.13.0, Datasets 5.2.0, and Tokenizers 0.23.3 for training. The saved local evaluation was run with Python 3.12.6, Transformers 4.57.6, PEFT 0.21.2, and AuditKit 1.1.6. There is no single pinned environment lockfile yet; pin compatible dependencies before claiming a bit-for-bit reproduction.

## Data and licensing notes

- The local dataset card lists CC-BY-NC-SA 4.0 and describes both synthetic Gulf Coast examples and real transcribed ATC exchanges. Check each upstream source's terms before redistributing any data or derivatives.
- The Tiny Aya base model is subject to its own license and acceptable-use terms. A public adapter repository would still need to comply with those terms.
- Evaluation logs include prompts and generated text; TensorBoard event files may include local environment metadata. Review and sanitize artifacts before any public release.
