# LG헬로비전 고객 해지 데이터 품질 검증 및 해지 요인 분석

LG헬로비전 고객 해지 데이터(`cancel_yn`)의 품질을 검증해 정제하고, 해지 고객의 특성을 분석해 약정 단계별 대응 전략을 도출한 프로젝트입니다.

## 데이터

- 2023년 2월부터 12월까지 11개월의 월별 스냅샷, 분기별 CSV 4개
- 22,893,471행, 고객 2,175,327명, 39개 컬럼 (한 행 = 고객(`sha2_hash`) × 월(`p_mt`))
- 정제 후 22,053,564행, 고객 2,135,261명 (해지 고객 150,613명, 고객 단위 해지 고객 비율 7.05%)
- LG헬로비전 DATA SCHOOL 교육 과정에서 제공받은 데이터로 진행한 개인 프로젝트이며, 데이터는 저장소에 포함하지 않았습니다

## 사용 기술

- **DuckDB, SQL**: 대용량 CSV와 Parquet 직접 쿼리 (CTE, 윈도우 함수, `UNPIVOT`, 조건부 집계)
- **Python (pandas)**: 결과 정리와 비교, 통계 검정 (z-test, 95% 신뢰구간)

## 주요 결과

- **데이터 품질**: 같은 달에 해지와 유지가 함께 기록된 고객 40,063명을 원인 가설 검증 후 제외하고, 원본 = 정제 + 제외 행 수로 유실 없음을 확인
- **고객 단위 재집계**: 해지 고객이 처음 나타난 달부터 매달 '해지'로 기록돼 중복 집계되던 것을 고객 한 명당 한 행으로 수정 (약정 만료 1개월 전 4.38배 → 3.49배)
- **데이터 한계**: 해지 고객 전원이 처음부터 해지 상태이고, 유지에서 해지로 바뀐 고객은 0명. 해지 예측이 아닌 해지 고객과 유지 고객의 특성 비교로 분석
- **해지 고객 특성**: 약정 만료 1\~6개월 전 고객은 전체의 5.6%이지만 해지 고객의 15.0%. Top 5 고객군 중 약정 만료를 뺀 4개(20대, 유료 채널 2건 이상, 4주 이상 시청 없음, 디지털과 기가 결합)는 약정 시점이 동일한 고객끼리 비교해도 모든 구간에서 해지 고객 비율이 높고, 95% 신뢰구간도 겹치지 않음
- **대응 전략**
  - 약정 만료 6개월 전부터 해지 방어, 1\~3개월 전 고객 우선: 해지 고객의 15%를 2.7배 효율로 만남 (1\~3개월 전은 3.1배, 9\~12개월 전은 1.2배로 평균 수준)
  - 약정 만료 후 1년 넘은 고객(해지 고객의 36%): 재약정 유도 (재약정 고객은 평균의 0.61배)
  - 약정 만료까지 1년 넘게 남은 고객(해지 고객의 28%): 위험 신호 2개 이상부터 조기 경보 (이 구간 고객의 3%로 해지 고객의 11%, 신호 없는 고객의 4배 이상)

## 한계와 다음 단계

- 해지 고객의 값이 해지 이후 상태일 수 있어, 결과는 원인이 아닌 해지 고객의 특성
- 약정 시점만 통제했고, 위험 신호는 같은 무게로 셈
- 대응 전략의 실제 효과는 캠페인 실행 데이터가 없어 검증할 수 없었음
- 다음 단계: 해지 전 월별 기록으로 해지 예측 모델, 신호별 가중치 점수, 해지 방어 시점과 재약정 혜택의 A/B 테스트

## 파일 구성

| 파일 | 내용 |
|---|---|
| `01_eda.ipynb` | 원본 탐색: 프로파일링, 분기별 드리프트, 이상치, 고객 × 월 중복과 라벨 충돌의 원인 조사 |
| `02_column_check.ipynb` | `sha2_hash`를 제외한 38개 컬럼의 값 분포, 척도, 결측 표시 값 확인 |
| `03_clean.ipynb` | 정제 규칙 적용, 모든 규칙을 한 번에 처리하는 재현 가능한 정제 스크립트 (원본 CSV → parquet, 행 수와 체크섬으로 검증) |
| `04_column_check_clean.ipynb` | 정제 데이터 재점검, z-test와 신뢰구간으로 일부 결정 검증 |
| `05_churn_factors.ipynb` | Part 1: 행 단위 스캔, 해지 이후 행 검증<br>Part 2: 고객 단위 재집계, 변수별 최고값으로 Top 5 선정<br>Part 3: 약정 구간별 해지 고객 비중, 해지 방어 범위 비교<br>Part 4: 약정 시점을 맞춘 비교(95% 신뢰구간), 위험 신호 기준과 조기 경보<br>Part 5: 해지 시점이 관측되지 않는 데이터 한계 확인 |

각 노트북에 발견 사항과 처리 근거가 마크다운 셀로 기록되어 있습니다.

---

# LG HelloVision Customer Churn: Data Quality Validation and Churn Factor Analysis

A project that validates and cleans LG HelloVision customer churn data (`cancel_yn`), then analyzes the characteristics of churned customers to derive response strategies by contract stage.

## Data

- Monthly snapshots for 11 months (Feb to Dec 2023), 4 quarterly CSV files
- 22,893,471 rows, 2,175,327 customers, 39 columns (one row = customer (`sha2_hash`) × month (`p_mt`))
- After cleaning: 22,053,564 rows, 2,135,261 customers (150,613 churners, customer-level churner share 7.05%)
- A personal project using data provided in the LG HelloVision DATA SCHOOL program; the data is not included in this repository

## Tech Stack

- **DuckDB, SQL**: direct queries on large CSV and Parquet files (CTE, window functions, `UNPIVOT`, conditional aggregation)
- **Python (pandas)**: organizing and comparing results, statistical tests (z-test, 95% confidence intervals)

## Key Results

- **Data quality**: excluded 40,063 customers with cancellation and retention recorded in the same month after testing hypotheses about the cause, and confirmed no rows were lost (original = cleaned + excluded)
- **Customer-level recount**: churners are recorded as cancelled in every month from the first month they appear, so the same cancellation was counted many times; fixed by keeping one row per customer (1 month before contract expiry: 4.38x → 3.49x)
- **Data limitation**: every churner is cancelled from the first month seen and no customer switches from active to cancelled, so the analysis compares churners and active customers rather than predicting churn
- **Churner profile**: customers 1 to 6 months before contract expiry are 5.6% of customers but 15.0% of churners. The four Top 5 segments other than contract expiry (20s, 2+ paid channels, no viewing for 4+ weeks, digital + giga bundle) have higher churn in every contract stage when compared at the same contract timing, with non-overlapping 95% confidence intervals
- **Response strategy**
  - Retention from 6 months before contract expiry, 1 to 3 months first: reaches 15% of churners at 2.7x efficiency (3.1x for 1 to 3 months; 9 to 12 months is close to average at 1.2x)
  - More than a year past contract expiry (36% of churners): re-contract offers (re-contracted customers are 0.61x the average)
  - More than a year left on contract (28% of churners): early warning from 2+ risk signals (3% of these customers hold 11% of their churners, over 4x the rate of customers with no signal)

## Limitations and Next Steps

- Churners' values may reflect their state after cancellation, so the results describe churners rather than causes
- Only contract timing was controlled, and risk signals are counted with equal weight
- The actual effect of the response strategies could not be tested, as no campaign data was available
- Next steps: a churn prediction model from pre-cancellation monthly records, a weighted risk score, and A/B tests of retention timing and re-contract offers

## File Structure

| File | Contents |
|---|---|
| `01_eda.ipynb` | Original data exploration: profiling, quarterly drift, outliers, investigation of customer × month duplicates and label conflicts |
| `02_column_check.ipynb` | Value distribution, scale and missing-value markers of the 38 columns other than `sha2_hash` |
| `03_clean.ipynb` | Cleaning rules and a reproducible single-pass cleaning script (original CSV → parquet, verified by row counts and checksums) |
| `04_column_check_clean.ipynb` | Re-check of the cleaned data, some decisions validated with z-tests and confidence intervals |
| `05_churn_factors.ipynb` | Part 1: row-level scan, check of rows after cancellation<br>Part 2: customer-level recount, Top 5 segments (highest value per variable)<br>Part 3: churner share by contract stage, comparison of retention windows<br>Part 4: comparison at the same contract timing (95% confidence intervals), risk signals and early-warning rule<br>Part 5: data limitation that the moment of cancellation is never observed |

Findings and the rationale for each step are recorded as markdown cells in each notebook.
