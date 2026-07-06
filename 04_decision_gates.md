# 의사결정 게이트와 보완 계획

## 목적

이 문서는 연구 진행 중 언제 다음 단계로 넘어갈지, 언제 범위를 줄이거나 해석을 낮출지 결정하는 기준을 정의한다.

## 전체 의사결정 흐름

```text
Gate 1. 데이터 pairing이 가능한가?
        |
        v
Gate 2. AI printed image prediction이 충분한가?
        |
        v
Gate 3. Predicted-derived mismatch map이 신뢰 가능한가?
        |
        v
Gate 4. Corner condition에서 유의미한 risk 변화가 보이는가?
        |
        v
Gate 5. 최종 claim을 어느 강도로 말할 수 있는가?
```

## Gate 1. 데이터 pairing 가능성

### 질문

Mask, printed image, target/reference pattern을 같은 sample 단위로 안정적으로 연결할 수 있는가?

### 통과 기준

- Mask와 printed image의 1:1 matching이 가능하다.
- Image size와 pixel range가 일관된다.
- Reference pattern을 확보하거나, reference 부재 시 대체 기준을 명확히 정의한다.

### 실패 시 대응

1. 공식 LithoBench split 또는 dataset loader를 우선 사용한다.
2. Target/reference matching이 불가능하면 mask-to-printed prediction까지만 1차 범위로 축소한다.
3. Mismatch risk 분석은 ground-truth printed image와 available reference가 확인된 subset에서만 수행한다.

### Claim 조정

통과 시:

> Mask-to-printed prediction과 reference mismatch 분석을 수행했다.

실패 시:

> Mask-to-printed printed image prediction pipeline을 구축했고, reference 기반 risk 분석은 데이터 pairing 확인 후 확장한다.

## Gate 2. AI printed image prediction 품질

### 질문

AI model의 predicted printed image가 downstream contour 분석에 쓸 만큼 정확한가?

### 통과 기준

- Test set에서 MAE/MSE/SSIM이 baseline 대비 개선되거나 합리적인 수준이다.
- Error map이 전체적으로 random noise 수준에 가깝고, 큰 구조적 shift가 적다.
- Edge-zone error가 지나치게 크지 않다.

### 실패 신호

- Printed image가 blur되어 contour 위치가 불명확하다.
- Edge 주변 error가 크다.
- Pattern shift가 반복적으로 발생한다.
- Ground-truth와 비교했을 때 printed structure가 무너진다.

### 실패 시 대응

1. 학습 epoch, resolution, loss function을 조정한다.
2. Edge-zone loss를 추가한다.
3. 더 작은 subset에서 overfit test를 수행해 pipeline bug를 확인한다.
4. 그래도 약하면 AI model claim을 낮춘다.

### Claim 조정

통과 시:

> AI surrogate model은 printed image prediction에 충분한 가능성을 보였다.

실패 시:

> AI surrogate model은 baseline 수준에서 구현했으나, reliable risk screening에는 추가 성능 개선이 필요하다.

## Gate 3. Predicted-derived mismatch map 신뢰성

### 질문

Predicted printed image에서 만든 mismatch/risk map이 ground-truth printed image 기반 risk map과 유사한가?

### 통과 기준

- Hotspot candidate IoU가 의미 있는 수준이다.
- High-risk top-k overlap이 random보다 높다.
- Precision/recall 중 최소 하나가 연구 목적에 맞는 수준이다.

### 실패 신호

- Image metric은 괜찮지만 hotspot map agreement가 낮다.
- 작은 contour error가 큰 mismatch label 변화를 만든다.
- Predicted-derived risk ranking이 ground-truth-derived ranking과 다르다.

### 실패 시 대응

1. Risk map을 predicted-derived 중심이 아니라 ground-truth-derived 중심으로 보고한다.
2. AI model은 acceleration 가능성으로만 설명한다.
3. Contour threshold sensitivity를 더 강조한다.
4. Risk score 기준을 단순화한다.

### Claim 조정

통과 시:

> Predicted printed image 기반 risk map은 ground-truth-derived risk map과 유사해 screening proxy로 활용 가능성을 보였다.

실패 시:

> Printed image prediction은 가능했지만, predicted-derived risk map의 신뢰성은 추가 개선이 필요하다.

## Gate 4. Process corner sensitivity

### 질문

Nominal condition 대비 corner condition에서 mismatch risk가 증가하는 pattern을 선별할 수 있는가?

### 통과 기준

- 일부 sample에서 corner mismatch가 nominal mismatch보다 유의미하게 증가한다.
- High-sensitivity sample이 시각적으로도 납득 가능하다.
- Corner-only hotspot candidate가 관찰된다.

### 실패 신호

- Nominal과 corner metric 차이가 거의 없다.
- 차이가 noise인지 process sensitivity인지 구분하기 어렵다.
- Corner condition 정의가 데이터셋에서 불명확하다.

### 실패 시 대응

1. Corner condition 사용 가능 여부를 다시 확인한다.
2. Nominal-only printed image prediction project로 범위를 축소한다.
3. Process corner 분석은 보완 계획으로 둔다.

### Claim 조정

통과 시:

> LithoBench simulator corner condition에서 process-sensitive pattern을 선별했다.

실패 시:

> Corner condition 분석은 데이터 availability와 simulator setting 확인 후 확장할 계획이다.

## Gate 5. 최종 claim 강도 결정

### 질문

최종 발표/자소서에서 어느 강도로 말할 수 있는가?

## Claim Level A: 강한 결과

조건:

- Printed image prediction이 안정적이다.
- Predicted-derived risk map과 ground-truth-derived risk map이 유사하다.
- Corner condition에서 process sensitivity가 관찰된다.

사용 문장:

> LithoBench 기반 mask-to-printed surrogate model을 학습하고, predicted printed contour mismatch를 통해 process-corner-sensitive printability risk candidate를 선별했다.

## Claim Level B: 중간 결과

조건:

- Printed image prediction은 가능하다.
- Risk map agreement는 일부 sample에서만 의미 있다.
- Corner analysis는 제한적으로 가능하다.

사용 문장:

> AI printed image prediction pipeline과 contour mismatch 기반 risk analysis framework를 구축했고, 일부 sample에서 process-sensitive mismatch 증가를 확인했다.

## Claim Level C: 보수적 결과

조건:

- Model prediction이 약하다.
- Risk map agreement가 낮다.
- Corner condition 분석이 불명확하다.

사용 문장:

> LithoBench 데이터를 활용해 mask-to-printed prediction과 contour mismatch 분석 pipeline을 구축했으며, 실제 screening 성능 확보를 위해 edge-aware model과 data pairing 개선이 필요함을 확인했다.

## 보완 계획

### 보완 1. Edge-aware model 개선

문제:

Risk map은 edge 위치에 민감하다.

계획:

- Edge-zone weighted loss 추가
- Contour-aware metric 추가
- Edge 주변 crop oversampling

### 보완 2. Threshold sensitivity 강화

문제:

Effective printed mask는 threshold 선택에 영향을 받는다.

계획:

- Threshold 0.4, 0.5, 0.6 비교
- High-risk ranking stability 확인
- Threshold에 민감한 sample을 별도 failure case로 분석

### 보완 3. CD/EPE metrology 확장

문제:

Mismatch area만으로는 실제 공정 계측 느낌이 부족할 수 있다.

계획:

- Vertical line/space sample을 선별한다.
- Contour crossing 기반 sub-pixel edge 위치를 계산한다.
- Printed CD와 target/reference CD를 비교한다.

단, 이 보완은 1차 MVP 이후에 진행한다.

### 보완 4. Via/contact 확장

문제:

MetalSet만으로는 contact/via 공정 risk를 다루지 못한다.

계획:

- ViaSet에서 component area, missing via, merge risk를 분석한다.
- MetalSet과 ViaSet의 sensitivity 차이를 비교한다.

단, ViaSet은 component matching 난이도가 높으므로 2차 연구로 둔다.

## 최종 연구계획서용 요약

본 연구는 실제 wafer defect label이 없는 LithoBench 환경에서 real hotspot classification을 주장하지 않는다. 대신 AI surrogate model을 통해 printed resist image를 예측하고, printed contour mismatch를 기반으로 simulation-based printability risk candidate를 정의한다. 연구의 핵심 검증은 printed image prediction quality, predicted-derived mismatch agreement, process corner sensitivity의 세 단계로 구성된다. 각 단계의 결과에 따라 최종 claim 강도를 조정한다.

