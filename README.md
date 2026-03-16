# ASDM — Adaptive Source-Drift Monitor

[![Paper](https://img.shields.io/badge/IEEE_TNNLS-Under_Review-blue)]()
[![Python](https://img.shields.io/badge/Python-3.9+-green)]()
[![License](https://img.shields.io/badge/License-MIT-yellow)]()

Source-attributed feature drift detection for multi-source healthcare ML pipelines.
Detects drift, attributes it to the responsible upstream source, and classifies
the mechanism — in 130 ms per batch.

---

## Abstract

Production healthcare machine learning (ML) systems assemble feature stores from
multiple independent upstream sources—electronic health record (EHR) vendors,
health information exchange (HIE) aggregators, Medicaid extracts, and claims
processors. Conventional drift monitoring detects that something shifted but cannot
identify which source or why. We address this source attribution problem by
separating three tasks that prior work conflates: (1) detecting drift, (2)
attributing it to a specific upstream source, and (3) classifying the drift
mechanism. Applying the additivity of Kullback-Leibler (KL) divergence under
product measures yields a per-source localization signal; we formalize an
identifiability condition and present a five-type taxonomy of healthcare drift
mechanisms derived from 87 production incidents spanning 18 months and an
estimated 500 million patient feature vectors. The Adaptive Source-Drift Monitor
(ASDM) implements this framework as four components. A three-tier evaluation on
production incidents, MIMIC-IV (190,835 admissions), and standard synthetic
benchmarks shows 91.6% recall at 3.2% false-positive rate and a mean detection
delay of 287 instances versus 1,847 for ADWIN. Mean time to root-cause
identification fell from 4.1 hours to 23 minutes—a 91% reduction sustained over
12 months of deployment.

---

## The Problem

Healthcare ML systems assemble features from multiple independent sources.
Conventional drift monitors detect *that* something shifted — not *which source*
or *why*. Every alert becomes a 4-hour investigation.

ASDM separates three tasks prior work conflates:
1. **Detect** — did any upstream source shift from its baseline?
2. **Attribute** — which specific source is responsible?
3. **Classify** — what kind of drift is it, and what is the fix?

---

## Key Results

| Metric | ASDM | Best Baseline |
|--------|------|---------------|
| Recall | 91.6% | 79.6% |
| False-positive rate | 3.2% | 5.2% |
| Detection delay | 287 instances | 918 instances |
| Mean time to root cause | 23 min | 4.1 hours |
| Localization accuracy | 87.7% | 68.5% |

Validated on 71 production incidents (500M+ patient feature vectors),
MIMIC-IV (190,835 admissions), and 5 synthetic benchmarks (500K instances each).

---

## Drift Taxonomy

| Type | Origin | Change Level | Temporal Shape | Remediation |
|------|--------|--------------|----------------|-------------|
| SED | EHR schema upgrade | Schema | Sudden step | ETL patch / rollback |
| ECD | ICD/CPT revision | Value | Gradual ramp | Coding calendar audit |
| SOD | New source onboarding | Schema + Value | Step at registry date | Demographic recalibration |
| SPD | Seasonal utilization | Value | Periodic, year-over-year | No action (expected) |
| NCD | NULL sentinel change | Schema | Missingness spike | Sentinel remap |

---

## Architecture

```
Multi-Source Ingestion: EHR · HIE · Medicaid · Claims · Laboratory
                              ↓
          ① Source Distribution Fingerprinter (SDF)
             KS statistic · JS divergence · Attribution ratio R̂s
                              ↓
          ② Adaptive Drift Classifier (ADC)
             XGBoost · 14 features · 5 classes · macro-F1 = 0.831
                              ↓
          ③ SHAP-Weighted Impact Estimator (SIE)
             Risk(f,s) = D̂(f,s) × SHAP_norm(f)
                              ↓
          ④ HIPAA Audit Logger (HAL)
             Zero PHI · append-only · self-labeling feedback loop
```

End-to-end latency: **130 ms per batch** (<1% of mean ingestion latency)

---

## Theoretical Foundation

KL divergence decomposes additively under product measures:

> D_KL(P^t ‖ P^0) = Σ_s D_KL(P_s^t ‖ P_s^0)

Any monitor operating only on the aggregate discards per-source terms.
Moving monitoring to the source-table boundary — before any join — recovers them.

**Proposition 2** establishes sufficient conditions for unique attribution.
**Corollary 1** characterizes exactly when simultaneous multi-source shifts
make attribution provably underdetermined — confirmed empirically (all 6
misattributions are SOD events, as predicted by theory before experiments ran).

---

## Datasets

| Dataset | Scale | Role |
|---------|-------|------|
| Environment B (production) | 500M+ patient feature vectors, 18 months | Primary evaluation |
| MIMIC-IV v2.2 | 190,835 admissions, 2008–2019 | External validation |
| MOA synthetic benchmarks | 500K instances × 5 generators | Detection delay benchmarking |
| Prospective injection study | 180 injections on staging replica | Survivorship bias control |

---

## Citation

```bibtex
@article{kapu2026asdm,
  title   = {Source-Attributed Feature Drift Detection in
             Multi-Source Healthcare ML Pipelines},
  author  = {Kapu, Dileep Kumar Reddy},
  journal = {IEEE Transactions on Neural Networks and Learning Systems},
  year    = {2026},
  note    = {Under Review}
}
```

---

## Author

**Dileep Kumar Reddy Kapu**  
M.S. Computer Engineering, University of New Mexico  
