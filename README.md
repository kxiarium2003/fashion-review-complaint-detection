# Fashion Review Complaint Detection

패션 이커머스 리뷰에서 고객 불만을 자동 탐지하고, 불만이 존재할 경우 세부 불만 유형을 Multi-label 방식으로 분류하는 텍스트 마이닝 프로젝트입니다.

본 프로젝트는 LLM 기반 Weak Supervision을 활용하여 Silver Set을 구축하고, 이를 학습 데이터로 사용하여 고객 불만 분류 모델을 학습하였습니다. 

또한 무신사(Musinsa), 에이블리(Ably), 지그재그(Zigzag) 데이터를 활용하여 플랫폼 간 일반화 가능성을 검증하였습니다.

> - 본 레포지토리에는 제가 수행한 TF-IDF 기반 Baseline(Logistic Regression, LinearSVC) 실험 코드 및 결과만 포함되어 있습니다.
> - 프로젝트의 메인 모델(KLUE-RoBERTa, TAPT, Label-wise Threshold Tuning 등)은 팀 프로젝트에서 별도로 수행되었습니다.

---

## Project Overview

### Goal

리뷰 텍스트로부터

* 고객 불만 여부 탐지
* 불만 유형 분류
* 플랫폼 간 일반화 성능 검증

을 수행하는 모델을 구축 목표

### Example

**Input**

> 배송은 빨랐는데 생각보다 너무 얇고 마감이 아쉬워요.

**Output**

Multi-label Prediction
ex) quality, fit_dissatisfaction

---

## Dataset

### Raw Reviews

- 약 39만 건의 Musinsa 패션 리뷰
- 외부 검증용 리뷰: Ably(약 2만 5천 건), Zigzag(약 4만 건)

### Gold Set
* 팀원이 직접 라벨링한 평가 데이터
* 500 reviews
* 모델 평가 및 Silver Label 품질 검증

### Silver Set

* LLM(Qwen) 기반 Pseudo Label 데이터
* 30,000 reviews
* Student Model 학습

---

## Label Taxonomy

### Coarse Labels (8)

* delivery
* packaging
* size_fit
* quality
* fabric_comfort
* color_design
* price
* other_complaint_coarse

### Fine Labels (18)

#### Delivery

* late_delivery
* wrong_item
* missing_item
* damaged_on_arrival

#### Packaging

* packaging_issue

#### Size & Fit

* size_large
* size_small
* fit_dissatisfaction

#### Quality

* finish_defect
* durability
* contamination

#### Fabric & Comfort

* fabric_issue
* thickness
* poor_comfort

#### Color & Design

* color_mismatch
* design_mismatch

#### Price

* price_dissatisfaction

#### Other

* other_complaint

---

## Project Pipeline

```text
Raw Reviews (390K)
        │
        ▼
Data Cleaning & Schema Standardization
        │
        ▼
Gold Set Construction (500)
        │
        ├─────────────┐
        ▼             │
Teacher Model (LLM)   │
        │             │
        ▼             │
Silver Set (30K)      │
        │             │
        ▼             │
Student Training      │
(TF-IDF / RoBERTa)    │
        │             │
        └─────► Evaluation (Gold500)
```

---

## Baseline Models

### TF-IDF + Logistic Regression

* TF-IDF Vectorization
* One-vs-Rest Logistic Regression

**Result**

* Micro F1: 0.3347
* Exact Match Ratio: 0.5420

### TF-IDF + LinearSVC

* TF-IDF Vectorization
* One-vs-Rest LinearSVC

**Result**

* Micro F1: 0.5396
* Exact Match Ratio: 0.5500

### Baseline Comparison

| Model               | Micro F1 | Macro F1 | Subset Accuracy |
| ------------------- | -------: | --- | --- |
| Logistic Regression |   0.335 | 0.173 | 0.542 |
| LinearSVC           |   0.540 | 0.456 | 0.550 |

LinearSVC가 Logistic Regression 대비 Recall과 Micro F1-score 측면에서 우수한 성능을 보여 최종 TF-IDF Baseline 모델로 선정되었습니다.

---

## Main Model

### Backbone

* KLUE-RoBERTa-Large

### Task

* Fine 18-label Multi-label Classification

### Additional Techniques

* TAPT (Task-Adaptive Pretraining)
* Label-wise Threshold Optimization

---

## Final Results

### Fine 18-label Classification

| Metric          |  Score |
| --------------- | -----: |
| Micro F1        | 0.7977 |
| Macro F1        | 0.8030 |
| Subset Accuracy |  0.838 |

---

## Repository Structure

```text
fashion-review-complaint-detection
│
├── data/
│   ├── silver_set.csv
│   └── gold_set.csv
│
├── baseline/
│   └── tf-idf_baseline.ipynb
│
├── outputs/
│   ├── baseline_comparison.png
│   └── f1_by_label.png
│
├── requirements.txt
└── README.md
```

---

## Contributions
- Gold Set Coarse&Fine Label 설계 및 구축 참여
- TF-IDF Vectorization 기반 Baseline 구축 및 실험
- Multi-label 분류 성능 평가 및 분석
- Baseline 결과 시각화 및 보고서 작성

---

## Related Repositories

- [Musinsa Crawler](https://github.com/riex71/musinsa_crawler.git)
- TF-IDF Baseline (this repository)
- [BERT / TAPT Experiments](https://github.com/riex71/2026_tmpj_team4_bert.git)

