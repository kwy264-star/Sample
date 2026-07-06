# 평가 지표와 실험 프로토콜

## 목적

이 문서는 연구 질문을 검증하기 위한 평가 지표와 실험 절차를 정의한다. 핵심은 AI 모델 성능과 공정 risk 분석을 분리해서 검증하는 것이다.

## 전체 평가 구조

평가는 네 단계로 나눈다.

```text
1. Data validity
2. Printed image prediction quality
3. Contour/mismatch agreement
4. Process corner sensitivity
```

각 단계가 통과되어야 다음 단계 해석이 가능하다.

## 1단계. Data Validity

### 목적

Mask, printed image, target/reference image가 올바르게 matching되는지 확인한다.

### 확인 항목

- Mask file 존재 여부
- Printed/resist image file 존재 여부
- Target/reference file 존재 여부
- Image size 일치 여부
- Pixel value range
- Basename 또는 metadata 기반 pair matching 가능 여부
- Train/validation/test leakage 여부

### 통과 기준

- 학습에 사용할 sample pair가 명확하다.
- Mask와 printed image의 spatial size가 일치한다.
- Reference pattern을 사용할 수 있거나, 사용할 수 없는 경우 그 한계를 명시한다.
- Split 기준이 문서화되어 있다.

## 2단계. Printed Image Prediction Metric

### 목적

AI surrogate model이 printed resist gray image를 얼마나 정확히 예측하는지 평가한다.

### 입력과 출력

```text
Input: mask image
Prediction: predicted printed resist gray image
Ground truth: LithoBench printed resist gray image
```

### 주요 지표

#### MAE

전체 pixel의 평균 절대 오차.

해석:

- 전체 gray image 예측 품질을 본다.
- 단독으로는 contour 품질을 보장하지 않는다.

#### MSE

큰 error에 더 민감한 pixel-wise error.

해석:

- 큰 prediction error가 있는 sample을 찾는 데 유용하다.

#### SSIM

구조적 유사도.

해석:

- printed pattern의 전체 구조가 보존되는지 본다.

#### Edge-zone MAE

Reference 또는 printed contour 주변 영역에서의 MAE.

해석:

- hotspot/risk 분석에 가장 중요한 지표 중 하나다.
- 전체 MAE가 낮아도 edge-zone MAE가 높으면 risk map은 신뢰하기 어렵다.

## 3단계. Contour and Mismatch Metric

### 목적

Predicted printed image에서 만든 mismatch map이 ground-truth printed image에서 만든 mismatch map과 얼마나 일치하는지 평가한다.

### Effective Printed Mask

Printed gray image `P`에 대해:

```text
effective_printed_mask = P >= threshold
```

기본 threshold:

```text
0.5
```

Sensitivity threshold:

```text
0.4, 0.5, 0.6
```

### Mismatch 정의

Reference pattern `R`, effective printed mask `E`:

```text
mismatch = abs(R - E)
extra_print = E == 1 and R == 0
missing_print = E == 0 and R == 1
```

### 주요 지표

#### Mismatch Area Ratio

```text
mismatch_area_ratio = mismatch pixel count / total pixel count
```

#### Extra-print Area Ratio

```text
extra_print_area_ratio = extra_print pixel count / total pixel count
```

#### Missing-print Area Ratio

```text
missing_print_area_ratio = missing_print pixel count / total pixel count
```

#### Largest Mismatch Component Area

Connected component analysis로 가장 큰 mismatch 영역의 면적을 측정한다.

해석:

- 작은 noise성 mismatch와 큰 hotspot candidate를 구분하는 데 필요하다.

#### Hotspot Candidate Agreement

Ground-truth-derived hotspot candidate와 predicted-derived hotspot candidate가 얼마나 일치하는지 본다.

지표:

- IoU
- precision
- recall
- top-k high-risk overlap

## 4단계. Process Corner Sensitivity

### 목적

Nominal condition과 corner condition에서 mismatch risk가 어떻게 달라지는지 평가한다.

### 기본 정의

```text
process_sensitivity = max(corner_mismatch) - nominal_mismatch
```

Corner condition은 LithoBench에서 제공되는 nominal/max/min 또는 focus/defocus output을 기준으로 한다.

### 주요 지표

- nominal mismatch area ratio
- corner mismatch area ratio
- process sensitivity score
- corner-only hotspot candidate count
- high-sensitivity sample ratio

### 해석

Process sensitivity가 높은 sample은 nominal condition에서는 비교적 안정적이지만 corner condition에서 printability risk가 증가하는 pattern으로 해석할 수 있다.

## 실험 프로토콜

### Protocol 1. 데이터 확인

1. LithoBench MetalSet에서 mask, printed, target/reference file 구조를 확인한다.
2. Pair matching rule을 정의한다.
3. 사용할 sample list를 만든다.
4. Train/validation/test split을 저장한다.
5. Sample visualization으로 pairing이 맞는지 확인한다.

### Protocol 2. Baseline 모델 학습

1. Mask image를 입력으로 사용한다.
2. Printed resist gray image를 target으로 사용한다.
3. U-Net 또는 LithoBench baseline을 학습한다.
4. Validation set에서 best checkpoint를 선택한다.
5. Test set에서 printed image prediction metric을 계산한다.

### Protocol 3. Contour mismatch 생성

1. Ground-truth printed image에서 effective printed mask를 만든다.
2. Predicted printed image에서도 effective printed mask를 만든다.
3. Reference pattern과 각각 비교해 mismatch map을 만든다.
4. Extra-print, missing-print, mixed-risk map을 생성한다.
5. Ground-truth-derived risk와 predicted-derived risk를 비교한다.

### Protocol 4. Corner condition 분석

1. Nominal condition printed image의 mismatch metric을 계산한다.
2. Corner condition printed image의 mismatch metric을 계산한다.
3. Corner-minus-nominal risk 증가량을 계산한다.
4. High-sensitivity sample을 선별한다.
5. 대표 sample을 시각화한다.

### Protocol 5. Threshold sensitivity 분석

1. Threshold 0.4, 0.5, 0.6에서 effective printed mask를 생성한다.
2. 각 threshold에서 mismatch metric을 계산한다.
3. High-risk ranking이 threshold에 얼마나 민감한지 확인한다.
4. 결과가 민감하면 threshold dependency를 limitation으로 보고한다.

## 최종 결과 표 구성

### Table 1. Printed Image Prediction

| Metric | Test Mean | Test Std | Note |
|---|---:|---:|---|
| MAE | | | 전체 gray image error |
| MSE | | | 큰 error에 민감 |
| SSIM | | | 구조적 유사도 |
| Edge-zone MAE | | | contour 주변 error |

### Table 2. Risk Map Agreement

| Metric | Value | Note |
|---|---:|---|
| Hotspot IoU | | predicted-derived vs ground-truth-derived |
| Precision | | predicted hotspot candidate 신뢰도 |
| Recall | | ground-truth-derived hotspot 포착률 |
| Top-10% overlap | | high-risk sample ranking |

### Table 3. Process Corner Sensitivity

| Metric | Value | Note |
|---|---:|---|
| Mean nominal mismatch | | nominal condition |
| Mean corner mismatch | | worst corner |
| Mean sensitivity | | corner - nominal |
| High-sensitivity ratio | | threshold 이상 sample 비율 |

