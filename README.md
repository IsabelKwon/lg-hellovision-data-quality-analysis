# LG헬로비전 고객 해지 데이터 품질 검증 및 해지 요인 분석

LG헬로비전 고객 해지 데이터(`cancel_yn`)의 품질과 정합성을 검증해 정제하고, 정제한 데이터로 해지 요인을 분석해 약정 단계별 대응 전략을 도출한 프로젝트입니다.

## 데이터

- 4개 분기 CSV 파일 (2023.02 ~ 2023.12, 11개월치 월별 스냅샷)
- 총 22,893,471행, 2,175,327명의 고객 (`sha2_hash`), 39개 컬럼
- 각 row는 "고객(`sha2_hash`) × 월(`p_mt`)" 단위의 스냅샷
- 39개 컬럼 = 식별자 2개(`sha2_hash`, `p_mt`) + 타겟 1개(`cancel_yn`) + 피처 36개
- 정제 후: 22,053,564행 / 2,135,261명 (해지 이력 보유 150,613명, 고객 단위 해지율 7.05%)

## 사용 기술

- **DuckDB, SQL** — CSV/Parquet 직접 쿼리, EDA, 정제, 해지 요인 분석 (CTE, `UNPIVOT`, 조건부 집계)
- **Python (pandas)** — 쿼리 결과 정리와 비교, 통계 검정(z-test, 신뢰구간) 계산

## 주요 결과

- **데이터 품질**: 같은 달에 해지와 유지가 함께 기록된 라벨 충돌 고객 40,063명을 원인 가설 검증 후 제외하고, 원본 = 정제 + 제외 행 수를 대조해 유실 없음을 확인
- **중복 집계 수정**: 해지 고객이 해지 후에도 매달 '해지'로 남아 같은 해지가 여러 번 집계되던 것을 고객별 첫 해지 달까지만 남겨 수정. 그 결과 약정 만료 직전 위험은 과대평가(4.38배 → 3.22배), 신규 가입 고객 위험은 과소평가(1.10배 → 1.72배)돼 있었음을 확인
- **해지 요인**: 약정 만료가 가까울수록 월 해지율이 단계적으로 상승. 약정 만료 1~6개월 전 고객은 전체의 6%이지만 해지의 15%가 발생
- **대응 전략**: 약정 만료 전 해지 방어(해지의 15%), 약정 만료 후 1년 넘은 고객의 재약정 유도(해지의 36%), 약정 기간 중 고객의 위험 고객군 기반 조기 경보(해지의 28%)

## 파일 구성

원본 데이터 탐색부터 정제, 재검증, 해지 요인 분석까지 아래 순서로 진행됩니다.

| 파일 | 내용 |
|---|---|
| `01_eda.ipynb` | 원본 데이터 최초 탐색 — 전체 프로파일링, 분기별 드리프트 체크, 이상치 검증, 타겟 밸런스 확인, `sha2_hash`+`p_mt` 중복/충돌 심층 조사(사업장 계정 가설의 고객 단위 재검증 포함), 해지 → 유지 번복 여부 원본 재검증 |
| `02_column_check.ipynb` | `sha2_hash`를 제외한 38개 컬럼 각각의 값 분포·척도·센티넬 값을 개별적으로 상세 확인 |
| `03_clean.ipynb` | EDA에서 발견한 문제를 반영한 정제 스크립트 실행, 정제된 데이터(`df_clean_part*.parquet`) 생성, 재검증 중 추가로 발견된 이슈 수정, 오염 행 33건이 동일한 행인지 검증, 전체 규칙을 단일 패스로 통합한 **재현 가능한 정제 스크립트**(원본 CSV → 최종 parquet, 행 수·전 컬럼 체크섬으로 등가성 검증 완료) |
| `04_column_check_clean.ipynb` | 정제된 데이터로 컬럼 체크를 재실행, 통계적 가설 검정(z-test, 신뢰구간)으로 일부 결정 검증 |
| `06_churn_factors.ipynb` | 해지 요인 분석 — Part 1: 범주형 전체를 `UNPIVOT`으로 일괄 스캔해 고객군별 해지율 비교, 해지 이후 행의 라벨 지속성·컬럼 동결 여부 검증 / Part 2: 고객별 첫 해지 달까지만 남겨 중복 집계를 제거한 뒤 재분석, 제거 전후 비교 / Part 3: 약정 만료 구간별 해지 비중과 해지 방어 대상 규모 추정 / Part 4: 약정 만료 시점을 동일하게 맞춘 상태에서 고객군을 비교해 겹치는 변수 확인 |

각 노트북 안에 발견 사항과 처리 근거가 마크다운 셀로 기록되어 있습니다.

---

# LG HelloVision Customer Churn Data: Quality Validation and Churn Factor Analysis

A project that validates the quality and integrity of LG HelloVision customer churn data (`cancel_yn`), cleans it, and analyzes churn factors on the cleaned data to derive response strategies by contract stage.

## Data

- 4 quarterly CSV files (2023.02 – 2023.12, 11 months of monthly snapshots)
- 22,893,471 rows in total, 2,175,327 customers (`sha2_hash`), 39 columns
- Each row is a snapshot at the "customer (`sha2_hash`) × month (`p_mt`)" level
- 39 columns = 2 identifiers (`sha2_hash`, `p_mt`) + 1 target (`cancel_yn`) + 36 features
- After cleaning: 22,053,564 rows / 2,135,261 customers (150,613 with a cancellation history, customer-level churn rate 7.05%)

## Tech Stack

- **DuckDB, SQL** — direct queries on CSV/Parquet, EDA, cleaning, churn factor analysis (CTE, `UNPIVOT`, conditional aggregation)
- **Python (pandas)** — organizing and comparing query results, computing statistical tests (z-test, confidence intervals)

## Key Results

- **Data quality**: Excluded 40,063 label-conflict customers, whose cancellation and retention were recorded in the same month, after testing hypotheses about the cause, and confirmed no rows were lost by reconciling original = cleaned + excluded row counts
- **Duplicate-count fix**: Cancelled customers remained as 'cancelled' every month after cancelling, so the same cancellation was counted multiple times; fixed by keeping each customer's rows only up to their first cancellation month. This showed that the risk right before contract expiry had been overestimated (4.38x → 3.22x) and the risk of newly joined customers underestimated (1.10x → 1.72x)
- **Churn factors**: The monthly churn rate rises step by step as contract expiry approaches. Customers 1–6 months before contract expiry are 6% of all customers but account for 15% of cancellations
- **Response strategy**: Retention before contract expiry (15% of cancellations), re-contract offers for customers more than one year past contract expiry (36% of cancellations), and early warning based on high-risk segments for customers still in contract (28% of cancellations)

## File Structure

The work proceeds in the order below, from exploring the original data through cleaning, re-validation, and churn factor analysis.

| File | Contents |
|---|---|
| `01_eda.ipynb` | First exploration of the original data — full profiling, quarterly drift checks, outlier validation, target balance check, in-depth investigation of `sha2_hash`+`p_mt` duplicates/conflicts (including a customer-level re-test of the business-account hypothesis), re-validation on the original data of whether cancellation → retention reversals occur |
| `02_column_check.ipynb` | Detailed, column-by-column check of the value distribution, scale, and sentinel values of each of the 38 columns excluding `sha2_hash` |
| `03_clean.ipynb` | Runs the cleaning script reflecting the issues found in EDA, creates the cleaned data (`df_clean_part*.parquet`), fixes additional issues found during re-validation, verifies whether the 33 corrupted rows are the same rows, and provides a **reproducible cleaning script** that combines all rules into a single pass (original CSV → final parquet, equivalence verified by row counts and checksums of all columns) |
| `04_column_check_clean.ipynb` | Re-runs the column checks on the cleaned data and validates some decisions with statistical hypothesis tests (z-test, confidence intervals) |
| `06_churn_factors.ipynb` | Churn factor analysis — Part 1: scans all categorical columns at once with `UNPIVOT` to compare churn rates by customer segment, and validates label persistence and column freezing in rows after cancellation / Part 2: re-analysis after removing duplicate counts by keeping each customer's rows only up to their first cancellation month, with a before/after comparison / Part 3: churn share by contract-expiry segment and sizing of the retention target group / Part 4: checks overlapping variables by comparing customer segments with contract-expiry timing held equal |

Findings and the rationale for each processing step are recorded as markdown cells inside each notebook.
