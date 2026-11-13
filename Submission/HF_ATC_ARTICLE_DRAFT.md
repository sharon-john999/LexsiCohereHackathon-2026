# Fine-tuning Tiny Aya for General-Aviation ATC Practice

**Claim:** Due to compute shortage it was tested only on five test samples, LoRA supervised fine-tuning increased reference-text overlap over the base model; this is an early phraseology-practice experiment, not evidence of operationally reliable air-traffic control.

> **Content and safety note:** The corpus includes simulated aviation emergencies, and some generated examples contain incorrect or contradictory clearances. Do not use the model or examples for live communications, flight planning, or operational decisions.

## Track and problem statement

- **Track 1 — Fine-tuning and Data Curation** Official problem-statement number: **1. Domain Assistants for Regulated Industries.**
## Abstract

This project adapts CohereLabs/tiny-aya-global, a 3.35B-parameter multilingual model, to generate the next controller-side message in general-aviation radio conversations. The intended use is a lightweight text practice aid for student pilots to explore radio phraseology—not an air-traffic-control system. A LoRA adapter is trained with supervised fine-tuning (SFT); the training notebook uses 4-bit NF4 loading, double quantization, and gradient checkpointing. CuratorKit is used in saved data-curation runs, and AuditKit records a small comparison of the base and adapted models. On five sampled prompts, mean token F1 rose from 0.167 to 0.399, with increases in several other reference-overlap metrics. However, the comparison is too small to support reliable performance claims: decoding is sampled, no aviation experts reviewed outputs, and the automatic scores do not establish procedural or safety correctness. Dataset snapshots and recorded curation counts also differ, so the exact training data cannot be identified confidently from this package. The model is a research prototype for offline study only.

## Headline result

**Token F1, five prompts:** base **0.167** → adapted **0.399** (**+0.231 absolute**; rounded from the saved AuditKit runs).

## Base model and method

**CohereLabs/tiny-aya-global (3.35B) + one-epoch SFT with a LoRA adapter (rank 16, alpha 32); the training notebook specifies 4-bit NF4 with double quantization and gradient checkpointing.**

## For judges: one minute

- **What:** Generate the next controller-side turn from a pilot/controller conversation.
- **Why:** Give student pilots an accessible way to rehearse phraseology in text.
- **How:** Curate conversational data, SFT a compact multilingual base model with LoRA, and compare the same five prompts before and after adaptation.
- **Result:** Token F1 0.167 → 0.399; critical-token recall 0.297 → 0.706. These are small-sample lexical-overlap results, not proof of correct clearances.
- **Caveat:** Do not use for live aviation. The outputs can invent or contradict instructions; human expert review and a much larger evaluation are still needed.

## Problem and approach

Radio phraseology is concise and context-dependent. A general-purpose model may not reliably reproduce the role-specific wording, callsign details, and turn-taking patterns that learners want to practice. This project tests whether domain-specific SFT can move a small model toward those patterns.

The task is next-controller-turn generation: the input is the conversation context before an assistant/controller message, and the target is that next message. The training notebook cleans and normalizes conversational roles, applies curation, and trains an SFT adapter. The evaluation notebook compares the base model with the same base plus adapter using identical selected prompts.

### Why this use case

Text-based scenario practice could let a learner explore routine radio exchanges and discuss phraseology without requiring a live radio session. The potential value is practice accessibility and a way to inspect example exchanges—not replacing an instructor, published procedures, or an operational controller.

### Research question and hypotheses

- **Research question:** Does SFT on the available ATC conversation data increase reference overlap and retention of critical lexical details versus the unadapted base model on selected controller-turn prompts?
- **Exploratory expectation:** The adapted model will score higher on token overlap, critical-token recall, and phraseology overlap.
- **Threat model:** Not a security or adversarial robustness study. This package does not test prompt injection, deliberate misuse, or attacks on an operational system. It does identify ordinary model error—such as invented, contradictory, or malformed instructions—as a safety concern.

## Related work

The project applies parameter-efficient fine-tuning using LoRA.

## Hackathon repositories and pipeline

- **CuratorKIT** The training notebook calls Curator to clean, schema-check, deduplicate, and export conversational data. A saved run records 546 input examples and 508 exported examples after 38 MinHash near-duplicates were removed (threshold 0.85). A separate saved curation card records a different run, 3,902 → 3,556, including 24 schema rejections and 322 near-duplicates removed.
- **AuditKIT** Used for base-versus-adapter generation evaluation, metric scoring, run JSON, and summary artifacts.

**Connection:** source/snapshot data → notebook cleaning and role normalization → CuratorKIT filtering/export → SFT/LoRA adapter → AuditKit comparison and saved results. Tried to use AlignTune, but were facing issues.

## Models and licenses

- **Base:** `CohereLabs/tiny-aya-global`, 3.35B parameters, loaded from the publisher's gated Hugging Face repository. The adapter config does not pin a model revision. Obtain access and follow the current upstream license and acceptable-use terms; this package does not reproduce or reinterpret those terms.
- **Task adapter:** PEFT LoRA adapter in [`Adapters/tiny-aya/`](./Adapters/tiny-aya/).
- **Training config:** 4-bit NF4 with double quantization, gradient checkpointing, LoRA rank/alpha 16/32, dropout 0.05, one epoch, maximum length 1,024. The training notebook sets `target_modules="all-linear"` while the saved adapter config lists seven projection modules (`q_proj`, `k_proj`, `v_proj`, `o_proj`, `gate_proj`, `up_proj`, `down_proj`); treat that as a configuration/provenance discrepancy.
- **Merged or quantized deployment model:** Find it in ./Final_model .

## Dataset, splits, and licensing

The intended upstream source is the [Gulf Coast GA ATC Communications dataset](https://huggingface.co/datasets/starlineventures/gulf-coast-ga-atc-communications). Its packaged dataset card describes synthetic Gulf Coast scenarios and real transcribed ATC exchanges, including Czech-airport sources. It reports 5,401 conversations in its overview.

| Evidence in this package | Count | Interpretation |
|---|---:|---|
| `./data/train.jsonl` | 546 conversations | Local cleaned training snapshot |
| `./data/curated/sft_sharegpt.jsonl` | 508 conversations | Output of a saved CuratorKit run from 546 inputs |
| Separate curation record under `./test/curated/` | 3,902 → 3,556 | A distinct saved run; not evidence of the final adapter's exact training input |
| `./data/test.jsonl` | 54 conversations | Local evaluation source |
| Candidate assistant turns in the evaluation notebook | 172 | Prompt pool before random selection |
| Saved base-versus-adapter comparison | 5 prompts | Shuffled with Python seed 0, then capped at five |


The dataset is tagged English; the package does not report a per-language distribution or language-specific evaluation. The base is multilingual.


## Metrics and evaluation design

The saved comparison uses the same five selected prompts for the base and adapted model. The evaluation notebook uses `do_sample=True`, temperature 0.4, top-p 0.85, repetition penalty 1.15, no-repeat 4-gram constraint, and up to 60 new tokens. Prompt selection uses seed 0; a generation seed is not fixed, so repeated outputs can differ.

| Metric | Direction | What it measures / caveat |
|---|---|---|
| Token F1 | Higher | Word overlap with the reference |
| BLEU | Higher | N-gram overlap |
| ROUGE-L | Higher | Longest-common-subsequence overlap |
| chrF | Higher | Character n-gram overlap |
| Levenshtein similarity | Higher | Normalized string similarity |
| Critical-token recall | Higher | Reference numbers and NATO-alphabet tokens also present in output; lexical only |
| Phraseology match | Higher | Jaccard overlap of selected phrase/clearance strings |
| Exact match | Higher | Strict normalized token equality |
| Length ratio | Higher, toward 1 | Similarity of output and reference word counts |
| Word error rate (WER) | Lower | Reported as zero for both models, but unresolved and excluded; the saved run marks its direction as maximize and the values do not match visible output/reference differences |

These are reference-text diagnostics, not aviation-procedure checks. A matching runway number, callsign token, or phrase does not establish that it is correct in context. No automatic-judge model or human-label agreement study was used. No confidence intervals, bootstrap analysis, or significance tests were run. No safety/general-capability retention metrics were collected.

## Results

Means from the two saved AuditKit runs; differences are adapted minus base. Scores are rounded to three decimals.

| Metric (higher is better unless noted) | Base | Adapted | Change |
|---|---:|---:|---:|
| Token F1 | 0.167 | 0.399 | +0.231 |
| BLEU | 0.000 | 0.134 | +0.134 |
| ROUGE-L | 0.091 | 0.377 | +0.286 |
| chrF | 0.185 | 0.384 | +0.199 |
| Levenshtein similarity | 0.203 | 0.242 | +0.039 |
| Critical-token recall | 0.297 | 0.706 | +0.409 |
| Phraseology match | 0.117 | 0.520 | +0.403 |
| Exact match | 0.000 | 0.000 | 0.000 |
| Length ratio | 0.524 | 0.370 | -0.154 |
| WER (lower is better) | 0.000* | 0.000* | unresolved |

The apparent overlap gains are promising only as a small exploratory signal. Exact match stayed at zero, and the lower length ratio indicates that adapted responses were often much shorter than references. The generations can also contain contradictory content: for one saved departure prompt, the reference is a takeoff clearance with wind, while the adapted output includes additional traffic and touch-and-go wording. This is an example of a serious task error, not a successful controller response. No output in this experiment was expert-validated.


### What did not work / what remains unknown

- Five prompts are insufficient to estimate robust performance or generalization.
- Sampled decoding was not seeded, so the comparison is not fully repeatable.
- Lexical metrics cannot assess clearance correctness, contextual consistency, or safety.
- The model sometimes adds unrelated or contradictory phrases; longer or more fluent text is not necessarily better.
- Conflicting dataset/curation snapshots prevent a definitive statement of the adapter's exact training-data size.

## Method and reproduction

The primary workflow is [`data_to_adapter-working.ipynb`](./data_to_adapter-working.ipynb) for cleaning, curation, and SFT, and [`evaluation.ipynb`](./evaluation.ipynb) for the base-versus-adapter run. See [`README.md`](./README.md) for package layout and run instructions.

Recorded SFT settings: one epoch; per-device batch size 1; gradient accumulation 8; learning rate 2e-4; max sequence length 1,024; LoRA rank 16, alpha 32, dropout 0.05; gradient checkpointing enabled. The adapter README lists PEFT 0.21.2, TRL 1.17.1, Transformers 5.22.2, PyTorch 2.13.0, Datasets 5.2.0, and Tokenizers 0.23.3. The saved local evaluation environment is recorded as Python 3.12.6, Transformers 4.57.6, PEFT 0.21.2, and AuditKit 1.1.6. There is no single lockfile, and training/evaluation version differences mean bit-for-bit reproduction is not assured.

The evaluation notebook declares 25 samples but later caps the saved run at five. The curation and training notebooks contain install cells and Colab-oriented paths; access to the gated base model and suitable GPU memory are required for practical reproduction. Training hardware, accelerator-hours, wall-clock training time, and monetary cost are not documented. A notebook comment references a T4 for a quantized evaluation run; this is not sufficient provenance for the saved adapter's training hardware.

## Artifacts and project status

The local package includes the adapter, notebooks, dataset snapshots, evaluation JSON, per-sample CSV, plots, and curation provenance. Key files:

- Adapter: [`Adapters/tiny-aya/`](./Adapters/tiny-aya/)
- Training and curation: [`data_to_adapter-working.ipynb`](./data_to_adapter-working.ipynb)
- Evaluation: [`evaluation.ipynb`](./evaluation.ipynb)
- Base run: [`Evaluation/eval/atc_base.json`](./Evaluation/eval/atc_base.json), AuditKit run `7adb04ff0b43870c`
- Adapted run: [`Evaluation/eval/atc_finetuned.json`](./Evaluation/eval/atc_finetuned.json), AuditKit run `ff15c850bc778fb6`
- Per-sample outputs: [`Evaluation/eval/per_sample_finetuned.csv`](./Evaluation/eval/per_sample_finetuned.csv)
- Curation record: [`data/curated/dataset_card.md`](./data/curated/dataset_card.md)

No project HF model collection, HF dataset collection, GitHub URL, demo/Space, video, or slides are identified in the package. The upstream dataset and base model links above are not project-published artifacts.

## Contributions, acknowledgements, and disclosure

- **LLM usage:** AI assistance was used to organize and draft this write-up from the packaged artifacts.
- **Changelog:** Initial submission write-up. For any resubmission, record each version and its changes here.

## References

1. Cohere Labs, [Tiny Aya Global model](https://huggingface.co/CohereLabs/tiny-aya-global) (base model; check current access and license terms).
2. Starline Ventures, [Gulf Coast GA ATC Communications dataset](https://huggingface.co/datasets/starlineventures/gulf-coast-ga-atc-communications) (dataset card and source references).
3. Hu et al., [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685), 2021.
4. Dettmers et al., [QLoRA: Efficient Finetuning of Quantized LLMs](https://arxiv.org/abs/2305.14314), 2023.
5. Zuluaga-Gomez et al., [ATCO2 corpus](https://arxiv.org/abs/2211.04054), 2022.
6. [CuratorKIT](https://github.com/Lexsi-Labs/CuratorKIT) and [AuditKIT](https://github.com/Lexsi-Labs/AuditKIT), used as recorded in the notebooks.