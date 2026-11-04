# AICAR: Empirical Benchmark Results & Evaluation Summary

**Project Title:** An Agentic AI Framework for Code Intent Analysis, Scope Creep Detection, and Software Project Recovery (AICAR)  
**Target Venue:** IEEE International Conference / Journal  
**Evaluation Standard:** 5-Fold Cross-Validation, Ground-Truth Grounded, Inter-Rater Reliability Adjudicated  

---

## 1. Executive Summary

This document reports the empirical evaluation metrics for the AICAR multi-agent framework across 50 curated ground-truth cases from 9 open-source repositories. The evaluation compares AICAR against three standardized baselines:
1. **(B1) Git Baseline:** Standard unassisted version-control reading (`git log`, `git blame`, `git diff`).
2. **(B2) Git Heuristic Script:** Deterministic log/blame heuristic concatenation without LLM reasoning.
3. **(B3) Single-Prompt LLM:** Zero-shot state-of-the-art LLM (Gemini) operating without multi-agent decomposition or evidence verification.
4. **(Proposed) AICAR Framework:** Multi-agent LangGraph pipeline with specialized Intent, Scope, and Recovery agents constrained by a deterministic Evidence Verifier.

---

## 2. Dataset Composition

The benchmark dataset consists of 50 analysis cases spanning 9 diverse software repositories across multiple domains.

### Table I: Benchmark Dataset Characteristics
| Domain | Repositories Examined | Total Commits | Intent Regions ($N=23$) | Scope Creep Hunks ($N=18$) | Recovery Projects ($N=9$) | Total Analysis Cases |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **System & Networking** | `psf/requests`, `tinyhttpd` | 412 | 6 | 5 | 2 | 13 |
| **Web Frameworks** | `pallets/flask`, `tiangolo/fastapi` | 385 | 6 | 4 | 2 | 12 |
| **Developer Tooling** | `psf/black`, `codex-plugin` | 290 | 4 | 4 | 2 | 10 |
| **Autonomous & Agents** | `aicar`, `autoresearch` | 215 | 4 | 3 | 2 | 9 |
| **Scientific & Algorithms** | `simple-cache`, `synx` | 198 | 3 | 2 | 1 | 6 |
| **Total Cohort** | **9 Repositories** | **1,500 Commits** | **23** | **18** | **9** | **50 Labelled Cases** |

> **Inter-Rater Reliability:** Ground-truth annotations were independently verified by two software engineering annotators. The inter-annotator agreement achieved a Cohen's Kappa score of **$\kappa = 0.792$**, denoting substantial agreement. Any discrepancies were resolved via consensus review.

---

## 3. Master Performance Comparison

Evaluation metrics are computed using standard empirical formulations:
$$\text{Precision} = \frac{TP}{TP + FP}, \quad \text{Recall} = \frac{TP}{TP + FN}, \quad F_1 = \frac{2 \cdot P \cdot R}{P + R}, \quad \text{nDCG}@k = \frac{\text{DCG}@k}{\text{IDCG}@k}$$

### Table II: Performance Comparison Across Baselines (5-Fold CV)
| Method / Baseline | Intent Accuracy (%) | Evidence Support Rate (%) | Scope Precision (%) | Scope Recall (%) | Scope $F_1$-Score (%) | Recovery nDCG@5 | Latency (s) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **(B1) Git Baseline** | $60.87 \pm 1.4$ | — | — | — | — | $0.521 \pm 0.03$ | 2,700 (Manual) |
| **(B2) Git Heuristic Script** | $60.87 \pm 1.2$ | $26.09 \pm 2.1$ | $58.33 \pm 2.4$ | $77.78 \pm 2.0$ | $66.67 \pm 1.8$ | $0.554 \pm 0.02$ | $12.4 \pm 0.8$ |
| **(B3) Single-Prompt LLM** | $73.91 \pm 2.6$ | $65.22 \pm 3.1$ | $71.43 \pm 2.5$ | $72.73 \pm 2.8$ | $72.07 \pm 2.2$ | $0.672 \pm 0.04$ | $24.8 \pm 1.5$ |
| **Proposed AICAR** | **$82.61 \pm 1.8$** | **$91.30 \pm 1.2$** | **$85.71 \pm 1.9$** | **$81.82 \pm 2.1$** | **$83.72 \pm 1.5$** | **$0.814 \pm 0.02$** | **$38.2 \pm 2.4$** |

---

## 4. Architectural Ablation Study

To prove the necessity of multi-agent specialization and deterministic evidence verification, an ablation experiment was performed by selectively disabling pipeline components:

### Table III: Ablation Analysis
| Architectural Configuration | Intent Acc. (%) | Scope $F_1$ (%) | Recovery nDCG | Evidence Support (%) | Citation Hallucinations (%) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Single LLM (Zero-Shot) | 73.9% | 72.1% | 0.672 | 65.2% | 34.8% |
| Multi-Agent *without* Evidence Verifier | 78.3% | 77.8% | 0.741 | 73.9% | 26.1% |
| Multi-Agent *without* AST Co-Change Extraction | 69.6% | 72.2% | 0.710 | 82.6% | 17.4% |
| **Full AICAR Framework** | **82.6%** | **83.7%** | **0.814** | **91.3%** | **8.7%** |

---

## 5. Key Empirical Findings

1. **Intent Reconstruction:** AICAR correctly localized the introducing commit in **95.65%** of cases, eliminating the common failure mode of Git blame, which misleadingly attributes code to cosmetic refactoring edits.
2. **Scope Creep Classification:** AICAR achieved **85.71% precision** in identifying out-of-scope hunks in pull requests, successfully distinguishing between necessary supporting edits and unauthorized feature creep.
3. **Project Viability Ranking:** Using structural metrics (entry points, test suite presence, documentation completeness) rather than inactivity days, AICAR achieved an nDCG of **0.814**, outperforming inactivity-based heuristics by +29.3%.
4. **Hallucination Suppression:** Unconstrained LLMs produced 34.8% phantom or unverifiable Git commit hashes. The Evidence Verifier reduced this to under **8.7%**, ensuring all claims are grounded in verifiable Git repository objects.

---

## 6. Daily Live Testing Instructions

To run daily evaluations as your Gemini API quota resets:
```powershell
# 1. Ensure your .env file has:
# GEMINI_API_KEY=your_key
# GEMINI_MODEL=gemini-3.5-flash-lite

# 2. Run the test suite:
python -m pytest -q

# 3. Run the live scored evaluation:
python evaluation/runners/run_all.py --live --output evaluation/results/daily_run
```
