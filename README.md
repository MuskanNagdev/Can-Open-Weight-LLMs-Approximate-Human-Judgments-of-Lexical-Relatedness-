# Can Open-Weight LLMs Approximate Human Judgments of Lexical Relatedness?
### A DURel-Based Evaluation on DWUG English

This repository contains the full experimental pipeline for a term paper investigating whether a freely accessible, open-weight large language model (LLM) can approximate human DURel relatedness judgments for lexical semantic change (LSC) research.

We prompt an LLM (`openai/gpt-oss-20b`, accessed via the [Groq API](https://console.groq.com)) to rate the semantic relatedness of usage pairs drawn from the [DWUG English](https://www.ims.uni-stuttgart.de/data/wugs) dataset, using the four-point [DURel](https://aclanthology.org/N18-2027/) scale, and compare its ratings against the existing human gold-standard annotations.

## Summary of Results

| Comparison | Spearman ρ | p-value | N | Interpretation |
|---|---|---|---|---|
| LLM (zero-shot prompt) vs. human | 0.584 | < 0.0001 | 57 | Moderate, significant |
| LLM (few-shot prompt) vs. human | 0.568 | < 0.0001 | 57 | Consistent with zero-shot |
| Sentence-embedding baseline vs. human | 0.200 | 0.126 | 57 | Not significant |

The LLM's ratings correlate moderately and significantly with human judgments, clearly outperforming a non-LLM sentence-embedding similarity baseline (`all-MiniLM-L6-v2`), and this result is stable across two different prompt formulations. A qualitative error analysis identifies two recurring disagreement patterns: **sense conflation** (rating distinct senses as related) and **idiomatic-extension misses** (failing to connect figurative uses back to a literal core sense). Full methodology, results, and discussion are in the accompanying term paper.

## Repository Contents

```
.
├── DURel.ipynb              # Full experimental pipeline (Google Colab notebook)
├── sample_pairs.csv         # Stratified sample of 60 usage pairs (15 per DURel category)
├── groq_results_final.csv   # Final LLM ratings merged with human gold judgments
├── references.bib           # BibTeX bibliography for the accompanying paper
└── README.md
```

## What the Notebook Does

1. **Data loading** — downloads and extracts the DWUG English resource (Schlechtweg et al., 2021) from Zenodo.
2. **Preprocessing** — removes the reserved "cannot decide" (0) judgment code, computes mean human relatedness scores per usage pair, and flags known source-data artifacts (see *Known Issues* below).
3. **Stratified sampling** — draws 60 usage pairs (15 per DURel category, 1–4) across 15 target lemmas, with a fixed random seed for reproducibility.
4. **LLM prompting** — queries an open-weight model via the Groq API with a DURel-scale zero-shot prompt (a few-shot variant is also tested as a robustness check), parses the model's rating from its response, and retries on failure.
5. **Baseline** — computes cosine similarity between sentence embeddings (`all-MiniLM-L6-v2`, Reimers & Gurevych, 2019) as a non-LLM point of comparison.
6. **Evaluation** — computes Spearman's rank correlation and mean absolute error against the human gold standard, plus per-category and per-lemma breakdowns.

## How to Run

1. Open `DURel.ipynb` in Google Colab (or Jupyter with the same dependencies).
2. Get a free API key from [console.groq.com](https://console.groq.com).
3. Store it as a Colab secret named `GROQ_API_KEY` (key icon in the left sidebar) — **do not hardcode the key in the notebook.**
4. Run all cells in order. The full pipeline (excluding the download step) takes roughly 5–10 minutes, mostly spent on rate-limited API calls to Groq.

### Dependencies

```
pandas
scipy
groq
sentence-transformers
```

Installed automatically in the first relevant cell via `pip install`.

## Known Issues

- **Data artifact:** one usage instance in the source `uses.csv` contains leaked tokenization metadata (character-offset indices concatenated into the sentence text). This pair fails to produce a parseable LLM rating and is automatically excluded; see the notebook's preprocessing section for details.
- **Contamination check limitation:** the notebook's regex check for this type of artifact only scans pairs that already received a valid rating, meaning it cannot confirm the *absence* of similar issues in pairs that failed to parse for other reasons. This is noted as a limitation in the accompanying paper.
- **Model nondeterminism:** despite `temperature=0`, the Groq inference backend does not guarantee fully identical outputs across separate runs. Exact per-pair LLM ratings may vary slightly on re-run; aggregate statistics (correlation, MAE) are stable across the runs conducted for this study.

## Data Source

DWUG English is publicly available at: [https://www.ims.uni-stuttgart.de/data/wugs](https://www.ims.uni-stuttgart.de/data/wugs) (Schlechtweg et al., 2021).
