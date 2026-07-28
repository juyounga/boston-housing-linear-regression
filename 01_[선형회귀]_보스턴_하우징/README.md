# 저소득층 비율과 방 개수로 보스턴 주택가격을 설명할 수 있는가

**작성자:** OOO &nbsp;&nbsp;|&nbsp;&nbsp; **작성일:** 2026-07-28 

![Python](https://img.shields.io/badge/Python-3.13.9-3776AB?logo=python&logoColor=white) ![pandas](https://img.shields.io/badge/pandas-2.3.3-150458?logo=pandas&logoColor=white) ![NumPy](https://img.shields.io/badge/NumPy-2.3.5-013243?logo=numpy&logoColor=white) ![statsmodels](https://img.shields.io/badge/statsmodels-0.14.6-4051B5) ![scikit-learn](https://img.shields.io/badge/scikit--learn-1.8.0-F7931E?logo=scikitlearn&logoColor=white)


## 요약 (Executive Summary)

- **데이터:** Boston Housing Dataset / https://www.kaggle.com/datasets/altavish/boston-housing-dataset
- **규모:** 506 관측치 × 14 변수
- **문제 유형:** 예측(회귀)

> 보스턴 광역권 506개 인구조사 구역의 주택 중위가격(`MEDV`)을 13개 지역 특성 변수로 설명했다.
> 기계적 전처리(로그 변환 · 이상치 대체 · VIF · 후진소거)와 도메인 지식 기반 변수 제거를 각각 체크포인트로 나누어 6개 모형을 비교했고,
> **도메인 지식으로 중복 변수를 걷어낸 뒤 로그 변환한 모형(독립변수 7개)** 을 최종 채택했다.
> 조정 R² = 0.748(원본 척도 R² = 0.771), RMSE = 4.393천 달러로, 기준선 모형(독립변수 11개, RMSE 4.680) 대비 **변수는 4개 줄이고 오차는 6.1% 낮췄다.**
> 가격을 가장 크게 좌우하는 요인은 **저소득층 비율(`LSTAT`, β = −0.597)** 이었으며, 방 개수(`RM`, β = 0.133)는 6번째에 그쳤다.
> 즉 주택의 물리적 크기보다 **동네의 사회경제적 구성**이 가격을 더 강하게 설명한다.

## 핵심 결과

| 항목 | 내용 |
|---|---|
| 최종 모형 | `log(MEDV) ~ CHAS + NOX + RM + log(DIS) + TAX + PTRATIO + log(LSTAT)` |
| 설명력 | 조정 R² = 0.748 (원본 척도 R² = 0.771) &nbsp;/&nbsp; RMSE = 4.393 (천 달러), MAE = 3.058 |
| 유의 변수 | 7개 전부 유의 (HC3 기준 p < .005) — `LSTAT`·`DIS`·`NOX`·`TAX`·`PTRATIO`·`RM`·`CHAS` |
| 가정 검토 | 선형성 (위반) / 정규성 (위반) / 등분산성 (위반·HC3 보정) / 독립성 (위반·횡단면이라 해석 제외) |
| 이번에 연습한 기법 | 로그 변환 자동 판정, IQR 이상치 대체, VIF 기반 공선성 제거, 후진소거법, 4대 가정 검정, HC3 로버스트 표준오차 |

## 참고자료

### 데이터 출처

- Kaggle 데이터셋: Boston Housing Dataset (`altavish/boston-housing-dataset`)
- URL: https://www.kaggle.com/datasets/altavish/boston-housing-dataset
- 원출처: Harrison, D. & Rubinfeld, D.L. (1978). *Hedonic prices and the demand for clean air*, Journal of Environmental Economics & Management, 5(1), 81–102.
- 라이선스: 공개 데이터(Public Domain) — CMU StatLib 배포본
- 참고한 커널·문서:
  - scikit-learn 1.2 릴리스 노트 — `load_boston` 제거 사유(`B` 변수의 윤리적 문제)

## 회고

- **새로 익힌 기법:** 왜도·첨도로 로그 변환 대상을 **자동 판정**하는 방법(`log` / `log1p` / 반사 후 `log1p`의 구분), 등분산 위배 시 **HC3 로버스트 표준오차**로 유의성 판정을 보정하는 방법, 그리고 전처리를 하나씩 누적한 **체크포인트로 각 처리의 기여도를 분리 측정**하는 방식이다.
- **막혔던 지점과 해결 방법:**
  - ① 종속변수에 로그를 씌운 뒤 R²·AIC로 모형을 비교하려다 척도가 달라 비교가 성립하지 않는다는 것을 알았다 → 예측값을 원본 단위로 되돌린 **RMSE를 주 지표**로 삼아 해결했다.
  - ② 쌍별 상관이 −0.880인데도 VIF가 기준에 걸리지 않아 변수가 하나도 제거되지 않았다 → **도메인 지식으로 직접 제거한 버전을 따로 만들어 성능표에서 비교**하는 방식으로 풀었다.
  - ③ `DIS`의 계수 부호가 단변량 상관과 반대로 나와 오류를 의심했으나, 다변량 통제의 정상적인 결과임을 확인하고 그 자체를 인사이트로 정리했다.
- **다음 데이터셋에서 보완할 점:** 가정 위반을 **발견하는 것**까지는 했지만 **해소하는 것**은 등분산 하나뿐이었다. 선형성 위배에 대해 다항항·구간화 같은 대안을 준비해 두고, 처음부터 학습/검증 분할을 설계해 성능 수치를 일반화 가능한 값으로 보고하고 싶다.