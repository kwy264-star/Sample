# 주장 체계와 논리 구조

## 목적

이 문서는 연구에서 주장할 수 있는 것과 주장하면 안 되는 것을 구분한다. 연구계획서와 발표에서 논리적 허점을 줄이는 것이 목적이다.

## 중심 주장

본 연구의 중심 주장은 다음이다.

> LithoBench의 mask-to-printed image pair를 활용하면 AI surrogate model로 printed resist gray image를 예측할 수 있으며, 예측된 printed contour와 reference pattern의 mismatch를 분석해 simulation-based printability risk candidate를 도출할 수 있다.

이 주장은 세 부분으로 나뉜다.

1. AI model이 printed image를 예측할 수 있는가
2. 예측된 printed image가 contour/mismatch 분석에 쓸 만큼 충분한가
3. mismatch 분석이 공정 관점의 risk screening으로 해석 가능한가

## Claim 1. AI는 printed resist image를 예측할 수 있다

### 주장

Mask image를 입력으로 사용하면 AI model이 corresponding printed resist gray image를 예측할 수 있다.

### 근거

- Lithography simulation task는 image-to-image mapping으로 구성할 수 있다.
- LithoBench는 mask, litho/aerial, printed/resist image pair를 제공한다.
- U-Net 또는 LithoBench baseline 계열 모델은 spatial image translation에 적합하다.

### 필요한 증거

- Test set에서 predicted printed image와 ground-truth printed image의 MAE/MSE/SSIM
- Error map 시각화
- Edge-zone error

### 약한 지점

전체 image metric이 좋아도 contour 주변이 부정확하면 risk analysis에는 쓸 수 없다.

### 방어 방법

Edge-zone metric과 contour-derived metric을 별도로 보고한다.

## Claim 2. Printed contour mismatch는 printability risk proxy가 될 수 있다

### 주장

Printed gray image에서 develop threshold contour를 추출하고 reference pattern과 비교하면, printability risk candidate를 정의할 수 있다.

### 근거

- Printed resist image는 binary pattern이 아니라 continuous gray response map이다.
- 공정적으로 의미 있는 비교를 위해 effective printed boundary가 필요하다.
- Target/reference 대비 printed pattern의 extra/missing 영역은 over-print 또는 under-print risk를 나타낼 수 있다.

### 필요한 증거

- Ground-truth printed image 기반 mismatch map
- Predicted printed image 기반 mismatch map
- 두 mismatch map의 agreement
- Extra-print, missing-print, mixed-risk tag의 예시

### 약한 지점

Threshold 값이 임의적으로 보일 수 있다.

### 방어 방법

0.4, 0.5, 0.6 threshold sensitivity를 수행하고, high-risk ranking이 크게 흔들리는지 확인한다.

## Claim 3. Corner condition에서 risk가 증가하는 pattern은 공정 취약 pattern으로 볼 수 있다

### 주장

Nominal condition에서는 안정적이지만 dose/focus corner에서 mismatch가 증가하는 pattern은 process-corner-sensitive pattern으로 해석할 수 있다.

### 근거

- 실제 공정에서는 nominal 성능보다 process margin이 중요하다.
- LithoBench simulator는 nominal/max/min 또는 focus/defocus condition을 제공할 수 있다.
- Corner condition에서 mismatch가 증가하면 printed pattern robustness가 낮다고 볼 수 있다.

### 필요한 증거

- Nominal vs corner mismatch metric 비교
- Process sensitivity score
- Corner condition에서만 hotspot candidate가 되는 sample 시각화

### 약한 지점

LithoBench condition은 실제 장비의 full focus-dose matrix가 아니다.

### 방어 방법

"실제 fab process window"가 아니라 "LithoBench simulator corner condition"이라고 표현한다.

## Claim 4. 본 연구는 실제 defect prediction이 아니라 risk screening이다

### 주장

본 연구는 실제 wafer defect를 예측하는 것이 아니라, simulator-derived printability risk candidate를 생성한다.

### 근거

- 실제 wafer inspection label이 없다.
- Bridge/open/necking type label이 공식적으로 제공된다고 가정할 수 없다.
- Risk label은 reference와 printed contour mismatch에서 생성한 pseudo-label이다.

### 필요한 증거

- Label 생성 절차 명시
- 실제 defect claim을 하지 않는 limitation section
- Risk tag와 defect class의 차이 설명

### 약한 지점

"hotspot"이라는 단어가 실제 defect처럼 들릴 수 있다.

### 방어 방법

항상 "simulation-based hotspot candidate" 또는 "printability risk candidate"라고 쓴다.

## 주장 강도 분류

### 강하게 주장 가능

- AI model로 printed image prediction을 시도했다.
- Ground-truth 대비 prediction quality를 정량 평가했다.
- Printed contour mismatch 기반 risk map을 생성했다.
- Nominal/corner condition에서 mismatch 변화를 비교했다.

### 조건부로 주장 가능

- AI model이 lithography simulation을 빠르게 대체할 수 있다.
- Predicted-derived risk map이 screening에 유용하다.
- 특정 pattern이 process-sensitive하다.

조건:

- prediction metric과 risk map agreement가 충분해야 한다.

### 주장하면 안 됨

- 실제 wafer defect를 예측했다.
- 실제 fab hotspot을 검출했다.
- bridge/open/necking을 supervised classification했다.
- 실제 장비 조건의 full process window를 재현했다.

## 연구의 최종 논리 구조

```text
LithoBench mask/printed pair
        |
        v
AI surrogate printed image prediction
        |
        v
Prediction quality validation
        |
        v
Contour-based effective printed mask
        |
        v
Reference vs printed mismatch
        |
        v
Simulation-based printability risk candidate
        |
        v
Nominal/corner condition process sensitivity analysis
```

