# 연구 질문과 범위 정의

## 연구 제목

LithoBench 기반 AI Surrogate Lithography Simulation을 활용한 Printed Pattern 예측 및 Printability Risk 분석

## 연구 배경

반도체 포토 공정에서는 설계자가 의도한 pattern이 wafer 위에 그대로 형성되지 않는다. Mask, optical imaging, resist response, dose/focus variation 등의 영향으로 printed pattern은 target/reference pattern과 차이를 보인다. 이 차이가 커지면 bridge, necking, open, line-end shortening 같은 공정 취약 현상으로 이어질 수 있다.

하지만 모든 layout pattern을 고비용 lithography simulation 또는 실제 wafer inspection으로 평가하기는 어렵다. 따라서 빠르게 printed pattern을 예측하고, 공정조건 변화에 민감한 pattern을 선별하는 screening 방법이 필요하다.

본 연구는 LithoBench 데이터를 활용해 AI surrogate model이 printed resist image를 예측할 수 있는지 확인하고, 예측된 printed contour를 이용해 simulation-based printability risk를 분석하는 것을 목표로 한다.

## 핵심 연구 질문

### RQ1. AI surrogate model은 mask image로부터 printed resist gray image를 얼마나 정확히 예측할 수 있는가?

이 질문은 프로젝트의 1차 기반이다. Printed image prediction이 충분히 정확하지 않으면 이후 mismatch map이나 risk analysis도 신뢰하기 어렵다.

검증 방향:

- predicted printed image와 ground-truth printed image 비교
- 전체 image error뿐 아니라 edge-zone error 확인
- gray-scale printed image의 contour가 보존되는지 확인

### RQ2. 예측된 printed image에서 추출한 contour mismatch는 ground-truth printed image 기반 mismatch와 얼마나 일치하는가?

단순히 image prediction metric이 좋다고 해서 hotspot 분석에 쓸 수 있는 것은 아니다. Risk 분석은 contour와 edge 주변 mismatch에 민감하다.

검증 방향:

- predicted printed image에서 effective printed mask 생성
- ground-truth printed image에서도 effective printed mask 생성
- 두 mask에서 도출한 mismatch map과 hotspot candidate를 비교

### RQ3. Dose/focus corner 조건에서 mismatch risk가 증가하는 pattern을 선별할 수 있는가?

공정기술 관점에서 중요한 것은 nominal condition에서의 예측 정확도뿐 아니라, 공정조건이 흔들릴 때 취약해지는 pattern을 찾는 것이다.

검증 방향:

- nominal condition과 corner condition의 mismatch metric 비교
- process sensitivity score 정의
- corner condition에서 risk가 증가하는 sample을 시각화

### RQ4. LithoBench 기반 분석을 실제 공정 관점의 printability risk screening으로 해석할 수 있는가?

이 질문은 연구의 해석 범위를 다룬다. LithoBench는 실제 fab defect label이 아니라 simulator 기반 benchmark이므로, real wafer defect prediction이라고 주장해서는 안 된다.

검증 방향:

- 실제 defect label 없이 가능한 claim과 불가능한 claim 구분
- simulation-based hotspot candidate로 표현
- risk tag를 verified defect class가 아닌 screening indicator로 제한

## 연구 범위

포함 범위:

- LithoBench MetalSet 중심 분석
- Mask-to-printed resist image prediction
- Printed gray image의 contour threshold 기반 effective printed mask 생성
- Target/reference와 printed mask의 mismatch 분석
- Nominal/corner condition 비교
- Simulation-based printability hotspot candidate 정의

제외 범위:

- 실제 wafer defect prediction
- 실제 fab inspection data 기반 검증
- Bridge/open/necking defect type supervised classification
- Via/contact pattern full analysis
- 정밀 CD/EPE metrology
- 실제 장비 수준의 full focus-dose process window 재현

## 연구에서 사용하는 핵심 표현

사용할 표현:

- AI surrogate lithography simulator
- printed resist gray image
- effective printed mask
- contour mismatch
- simulation-based hotspot candidate
- printability risk
- process-corner-sensitive pattern

피할 표현:

- real wafer hotspot detection
- actual defect prediction
- bridge/open/necking classification
- full process window reproduction

## 연구의 최종 산출물

본 연구는 다음 산출물을 목표로 한다.

- AI printed image prediction 결과
- Prediction metric table
- Effective printed mask와 contour 시각화
- Mismatch map과 extra/missing print map
- Hotspot candidate 및 risk score 분포
- Nominal/corner condition별 process sensitivity 분석
- 연구 한계와 보완 계획

