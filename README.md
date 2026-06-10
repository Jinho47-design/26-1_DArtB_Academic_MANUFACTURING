# 26-1_DArtB_Academic_MANUFACTURING

# 🏢 사랑의 밧데리
### : 2차 전지 셀(Cell) 수명 분석에 따른 출하 선별과 공정 성과 측정 통합 연구

> **DArt-B 26-1 학술제 제조업 팀** <br>

👥 **Team Members:** *권현하, 노동희, 신혜성, 양시연, 이상현, 최진호*

---

## 1. 📋 프로젝트 개요 (Overview)

### **배경 (Background)**
* **폭발적인 배터리 시장 성장:** EV와 ESS 수요 폭증으로 글로벌 배터리 수요는 2030년 약 4,200GWh까지 성장이 전망되며, 국내에서도 이차전지가 국가전략기술로 공식 지정되어 2029년까지 2,800억 원 규모의 집중 육성 정책이 추진 중입니다.
* **셀 제조사의 수익성 위기:** 리튬이온 배터리 셀 가격은 2013년 $827/TWh에서 2025년 $108/TWh까지 급락하였으며, 글로벌 오버 캐파 약 900GWh, 후공정 비용 편중(셀 Finishing 원가의 80%), 품질 검사 병목(500+ 사이클 충방전 필요) 등 구조적 페인 포인트가 심화되고 있습니다.
* **데이터 기반 공정 혁신의 필요성:** 배터리 산업의 승부처는 실험실에서 공장 데이터로 옮겨갔으며, 운영과 데이터가 마진을 결정하는 시대가 도래했습니다.

### **목표 (Goal)**
* 셀 제조 공정의 **극 초기(100사이클 이내) 데이터**만으로 배터리 수명을 사전 예측하고, 예측 결과를 기반으로 **출하 전 불량 선별 자동화**와 **충전 프로토콜(SOC) 최적화**를 통한 수명 연장 효과를 정량화합니다.
* 분석 정확도는 **PHM(Prognostics and Health Management)** 프레임워크로, 비즈니스 가치는 **BA-OEE(Battery-Adapted Overall Equipment Effectiveness)** 지표로 병렬 제시하여 전후 비교 가능한 수치를 도출합니다.

---

## 2. 📊 데이터 수집 (Data)

### **데이터셋 (Dataset)**
* **출처:** Severson et al. (2019), *Nature Energy* — Toyota Research Institute, Stanford, MIT 공동 연구
* **범위:** LFP(리튬인산철)/흑연 18650 셀, 72가지 충전 프로토콜
* **구성:**

| 항목 | 내용 |
|------|------|
| 행 / 열 | 114,314행 / 29열 |
| 셀 개수 | 134개 |
| 주요 변수 | Barcode, Protocol, Cycle_index, dc_internal_resistance, discharge/charge_Energy, temperature_min/max/avg, energy_efficiency, charge_duration |

---

## 3. 💡 핵심 기능 및 분석 방법론 (Core Features & Methodology)

단순 통계가 아닌, **물리·통계적 근거 기반 파생변수 설계**와 **다단계 피처 셀렉션**, **PHM × BA-OEE 통합 프레임워크**를 통해 수명 예측과 공정 성과를 정량화했습니다.

### **① 데이터 전처리 (Preprocessing)**
* **이상치 제거 기준 (물리·논리적 법칙 적용):**
    * `energy_efficiency > 1.0` → 열역학 법칙 위반 (방전에너지 > 충전에너지)
    * `dc_internal_resistance <= 0` → 완전도체는 물리적으로 불가능
    * `I_mean > -1.0` → 정상 방전 전류 기준(약 -4.4A) 이탈 시 측정 오류 판단
* **결측치 처리:**
    * `charge_duration` 칼럼 1,215개 결측치: 시스템 오류로 인한 475~521 사이클 구간 → **프로토콜 그룹별 중앙값 대체**
    * `paused = -128` 1행 및 비정상 측정 4개 행 삭제
    * 나머지 7개 칼럼 극소수 결측치 → 행 전체 삭제

### **② 가설 검정 (Hypothesis Testing)**
6개의 가설을 순차적으로 검증하여 열화 메커니즘을 규명하고, 통계적으로 유의한 피처만으로 모델을 구축했습니다.

| 가설 | 검정 변수 | Pearson r | p-value | 결과 |
|------|-----------|-----------|---------|------|
| 가설 0-1 | Knee Cycle ↔ Cycle Max | 0.83 | 1.35×10⁻³⁶ | **채택** |
| 가설 0-2 | Delta Q post_knee ↔ Cycle Max | 0.66 | 3.17×10⁻¹⁸ | **채택** |
| 가설 1 | Delta Q Min ↔ Cycle Max | 0.19 | 3.26×10⁻² | 채택 (메인 피처 부족) |
| 가설 2 | Delta Q Var ↔ Cycle Max | -0.11 | 0.197 | **기각** |
| 가설 3 | Delta Efficiency per Cycle ↔ Cycle Max | 0.79 | 2.31×10⁻³⁰ | **채택** |
| 가설 4 | Mean Charge Duration ↔ Cycle Max | 0.29 | 7.38×10⁻⁴ | **채택** |
| 가설 5 | Discharge Capacity Slope ↔ Cycle Max | 0.78 | 7.68×10⁻²⁹ | **채택** |
| 가설 6 | DC IR Slope ↔ Cycle Max | -0.55 | 4.19×10⁻¹² | **채택** |

### **③ 피처 셀렉션 (Feature Selection)**
* **1차 제거 (상관관계 기반):**
    * `charge_throughput_mean` → **Data Leakage** (사이클 누적량과 사실상 동일)
    * `temp_max_mean` → 수명과의 상관계수 -0.004로 예측력 부재
    * `charge_capacity_mean` → `discharge_capacity_mean`과 상관계수 1.00 (물리적 중복)
    * `charge_energy_mean` → `discharge_energy_mean`과 상관계수 0.94 (수학적 중복)
* **2~5차 제거 (VIF 다중공선성 기준):**
    * `discharge_energy_mean` / `discharge_capacity_mean` (VIF 185,000+)
    * `temp_avg_mean` → 온도-수명 상관관계 통계적 비유의
    * `dc_ir_mean` → 평균값보다 변화율(slope)이 더 많은 정보 포함
    * `energy_efficiency_mean` → `delta_efficiency_per_cycle`과 상관계수 0.90
    * `knee_cycle` / `delta_Q_post_knee` → **Data Leakage** (100사이클 내 확보 불가)

* **최종 선택 피처 6개:**

| 피처 | VIF | 의미 |
|------|-----|------|
| `discharge_capacity_slope` | 6.52 | 전주기 방전 용량 감소 기울기 |
| `delta_efficiency_per_cycle` | 38.94 | 사이클당 에너지 효율 저하 속도 |
| `deltaQ_mean` | - | 사이클당 평균 용량 변화량 |
| `deltaQ_min` | - | 최대 단일 용량 감소폭 |
| `charge_duration_mean` | 21.59 | 평균 충전 소요 시간 |
| `dc_ir_slope` | 7.65 | 직류 내부 저항 열화 기울기 |

### **④ Elastic Net 모델링**
* **모델 선정 이유:** 소표본(Train 45개) + 다중공선성 환경에서 L1+L2 페널티를 통한 과적합 억제, 자동 피처 선택, 계수 기반 해석 가능성 확보
* **데이터셋 분리:** Random Split이 아닌, **사이클 수 기준 고르게 할당**하여 분포 편향 방지

| 구분 | 셀 개수 | 최소 사이클 | 최대 사이클 |
|------|---------|------------|------------|
| Train | 45 | 171 | 2,189 |
| Primary Test | 45 | 327 | 2,237 |
| Secondary Test | 44 | 362 | 1,935 |

* **최적 하이퍼파라미터:** L1_ratio = 0.05 (Ridge 비중 95%), lambda = 0.013550
* **피처 중요도:** `discharge_capacity_slope` (0.2719) > `delta_efficiency_per_cycle` (0.1814)

* **모델 성능:**

| 데이터셋 | MAPE | RMSE |
|---------|------|------|
| Train | 9.8% | 188.1 |
| Primary Test | 11.7% | 182.6 |
| Secondary Test | 11.2% | 152.5 |

### **⑤ PHM × BA-OEE 통합 프레임워크**
* **PHM ① 조기 예측:** 100사이클 이내 데이터로 수명 사전 예측 (MAPE 11.9%, RMSE 184.6)
    * ±10% 오차 이내: 53.3% / ±15% 이내: 73.3% / ±20% 이내: 86.7%
* **PHM ② 출하 전 선별:** 임계값 900 설정 시 FP(불량 출하) = 0 달성
    * 즉시 출하(예측 ≥ 900): **38셀** / 재검사(500 ≤ 예측 < 900): **63셀** / 즉시 폐기(예측 < 500): **33셀**
    * 자동 판정률: 53% (59개 / 134개)
* **PHM ③ 공정 상관:** SOC 전환점이 수명에 유의미한 영향 (r = 0.42)
    * SOC 1%p 증가당 +4.56 cycle 연장 (배치 통제 후 회귀 계수)
    * SOC ≥ 75% 적용 시 113셀 평균 **+163.5 cycle 연장** (826 → 964 cycle, **+16.7%**)
* **BA-OEE 성과:**

| 지표 | 개선 전 | 개선 후 | 변화 |
|------|--------|--------|------|
| A (가용률) | 0.370 | 0.431 | **+16.7%** |
| P (성능률) | 0.937 | 0.937 | 변화 없음 |
| Q (품질률) | 0.855 | 0.855 | 변화 없음 |
| **BA-OEE** | **0.298** | **0.347** | **+16.5% 개선** |

---

## 4. 📈 주요 분석 결과 (Key Findings)

* **100사이클 이내 데이터만으로** 배터리 수명 예측 가능 (MAPE ~11%)
* **출하 전 선별 자동화:** 임계값 900 설정 시 불량 출하(FP) 0건 달성, 자동 판정률 53%
* **SOC 공정 최적화**를 통해 평균 수명 +16.7% 연장, BA-OEE +16.5% 개선
* **배치(Batch) 조건**이 수명 분산의 핵심 교란 변수로 작용 — 동일 충전 프로토콜도 배치 조건에 따라 수명이 크게 달라짐
* **Knee Point**는 수명의 약 71% 시점에 발생하는 사후 지표로, 100사이클 내 예측에는 Data Leakage를 유발하므로 모델에서 배제

---

## 5. 🛠 Tech Stack

* **Language:** Python 3.x
* **Data Analysis:** Pandas, NumPy, SciPy, Statsmodels, scikit-learn
* **Modeling:** Elastic Net (L1+L2 Regularization), VIF 다중공선성 분석
* **Visualization:** Matplotlib (NanumGothic), Seaborn
* **Framework:** PHM (Pecht 2008), BA-OEE (SEMI E10 표준 변형)
* **Environment:** Google Colab, Jupyter Notebook
* **Reference:** Severson et al. (2019), *Nature Energy* — Data-driven prediction of battery cycle life before capacity degradation
```