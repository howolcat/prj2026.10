# Audit 기대값 메모 (교과 7 → 교과 8)

> **용도**: 2일차 `src/dataset_audit.py` 실행 결과를 대조할 **기대값(Test Oracle)**
> **구분**: 진행용 보조기록 (가이드 7.1) · 최종 산출물 아님
> **작성**: 2026-10-12 · 최용훈 (교과 7 문서 정리)
> **원칙**: 이 파일의 숫자는 Audit 전에 고정한다. Audit 결과가 다르면 이 파일을 고치지 않고 `reports/audit_issue.csv`에 기록한다.

## 0. 출처 문서

| 약어 | 문서 | 내용 |
|---|---|---|
| H | `prj07/docs/subject08_handoff.md` | 교과 8 인계 · 주의사항 |
| Q | `prj07/reports/qa_summary.md` | QA · Validation · Class별 BBox |
| M | `prj07/manifests/dataset_manifest.csv` | 900장 출처 · split · scene_type · 검수 상태 |
| D1 | 2026-10-12 1일차 직접 확인 | `data/subject7_final` 기준 |

---

## 1. 파일 · 형식 검사 기대값

| 검사 항목 | 기대값 | 출처 | 1일차 확인 | 2일차 Audit |
|---|---:|---|---|---|
| JPG 수 | 900 | H, Q | ✅ 900 | |
| TXT 수 | 900 | H, Q | ✅ 900 | |
| JPG ↔ TXT Pair 일치 | 900 | H, Q | | |
| Pair 누락 (TXT 없음 / JPG 없음) | 0 / 0 | Q 5.4 | | |
| Empty TXT | 0 | H 8.1, Q 5.2 | ✅ 0 | |
| Class 4 포함 TXT | 0 | H 8.2, Q 5.4 | ✅ 0 | |
| Class ID 범위 오류 (0~6 밖) | 0 | Q 2 | | |
| 좌표 형식 오류 (칸 수 · 숫자 변환) | 0 | Q 2 | | |
| 좌표 범위 오류 (0~1 밖) | 0 | Q 2 | | |
| Manifest 행 수 | 900 | M | ✅ 900 | |
| QA 상태 | 900장 전체 PASS | M, Q | ✅ PASS 900 | |

## 2. Class별 BBox 기대값 (FINAL, 총 4,602개)

| Class | 이름 | BBox | 비율 |
|:-:|---|---:|---:|
| 0 | 나뭇잎·종이류 | 868 | 18.9% |
| 1 | 플라스틱류·돌·금속류 | 1,897 | 41.2% |
| 2 | 나뭇가지류 | 767 | 16.7% |
| 3 | 벌레류 | 378 | 8.2% |
| 4 | 고무장갑 (미사용) | 0 | 0.0% |
| 5 | 병해·갈변 | 441 | 9.6% |
| 6 | 파·고추 | 251 | 5.5% |
| | **합계** | **4,602** | 100% |

- 출처: H 8.1, Q 5.5 (두 문서 숫자 일치)
- 이미지당 평균 BBox: 4,602 / 900 = **약 5.1개**
- Class 불균형: Class 1 ÷ Class 6 = 1,897 ÷ 251 = **약 7.6배** (H 8.2-5)

## 3. 출처 · scene_type 기대값

| source_dataset / original_split | kimchi_with_target | object_only | normal_kimchi | 합계 |
|---|---:|---:|---:|---:|
| dataset1 / train | 500 | 0 | 0 | 500 |
| dataset2 / train | 100 | 120 | 0 | 220 |
| dataset2 / validation | 150 | 30 | 0 | 180 |
| **합계** | **750** | **150** | **0** | **900** |

- 출처: H 8.1, M (Manifest 재집계로 일치 확인 · 2026-10-12)
- other_review: 0장

## 4. 촬영 날짜 분포 (Manifest 파일명 앞 6자리 기준)

| 날짜 (YYMMDD) | dataset1 train | dataset2 train | dataset2 validation | scene_type |
|---|---:|---:|---:|---|
| 250410 | 115 | 0 | 35 | 전부 kimchi_with_target |
| 250411 | 203 | 0 | 47 | 전부 kimchi_with_target |
| 250424 | 182 | 100 | 68 | 전부 kimchi_with_target |
| 250513 | 0 | 120 | 30 | 전부 object_only |

- 출처: M 재집계 (2026-10-12)
- ⚠️ 파일명이 촬영 시각이라는 것은 **추정**이다 (H 8.2-3). 2일차에 이미지를 직접 보고 확인한다 (가이드 15.3).

## 5. Data Leakage 사전 확인 (Manifest 파일명 시간 기준)

| 확인 항목 | 값 | 출처 |
|---|---|---|
| 기존 validation 중 60초 이내에 촬영된 train 이미지가 있는 장 수 | **136 / 180 (75.6%)** | H 8.2-3, M 재계산 일치 |
| validation ↔ 가장 가까운 train 최소 시간차 | **8초** | H 8.2-3, M 재계산 일치 |
| 촬영 간격 5분 이상을 세션 경계로 볼 때 세션 수 | 18개 (참고) | M 재계산 |
| 그중 train과 validation이 섞인 세션 | 17개 (참고) | M 재계산 |

- 해석: 기존 train / validation은 **촬영 세션 단위로 나뉘지 않았다.** 기존 validation을 그대로 Validation으로 쓰면 점수가 부풀 가능성이 높다.
- 세션 수는 경계 기준(1분 · 5분 · 10분)에 따라 달라지므로 **2일차에 기준을 정한 뒤 확정**한다.

## 6. 이미 알려진 주의사항 → 2일차 Split 정책 안건

| # | 주의사항 | 근거 | Split 정책에서 정할 것 |
|---|---|---|---|
| 1 | 정상 김치(Negative) **0장** | H 8.2-1, Q 5.2 | 정상 김치 FP 평가 방법 · 알려진 한계로 기록 |
| 2 | object_only 150장이 **전부 dataset2 · 250513** | 3장, 4장 | 날짜 단위로 나누면 한 역할에 몰림 → 어떻게 배분할지 |
| 3 | 기존 train ↔ validation **유사 장면 75.6%** | 5장 | 기존 validation을 그대로 쓸지, 세션 단위로 다시 나눌지 |
| 4 | Class 4 **0건** | H 8.2-4 | 보고서에 "미사용 Class / Instance 0건" 기록 |
| 5 | Class 불균형 **7.6배** | 2장 | 각 역할에 모든 Class가 충분히 들어가는지 확인 |
| 6 | `too_small_to_identify` REVIEW **19장** | Q 5.3 | 작은 객체 FN 후보 → 6일차 Failure Analysis 참고 |

## 7. 문서 간 차이 메모 (QA 관점)

| 항목 | 차이 | 이 메모의 기준 |
|---|---|---|
| REVIEW 사유 수 | Q 3장 표 10 / 1 / 29 ↔ Q 5.3 · H 7 / 1 / 19 | 실제 이미지 수 **7 / 1 / 19 (27장)** (Q 3장은 이력 줄 수 기준, Q에 주석 있음) |
| Class 1 이름 | 수행가이드 "금속 등" ↔ 교과 7 "금속류" | 교과 7 `classes.yaml` 표기 **"금속류"** |
| FINAL 위치 | H 2장 `data/final/` ↔ 실제 `~/exe_01/master_data/final/` | 교과 8은 복사본 **`prj08/data/subject7_final/`** 사용 |

## 8. Audit 판정 규칙

1. Audit 결과가 이 메모와 **같으면** → PASS, 2일차 기록에 "기대값 일치"로 남긴다.
2. **다르면** → 데이터를 바로 고치지 않는다. `reports/audit_issue.csv`에 `항목 · 기대값 · 실제값 · 파일명 · 추정 원인`을 기록하고 팀이 원인을 확인한다.
3. 라벨 수정이 꼭 필요하면 수정 근거를 기록하고 **Dataset Version을 올린다** (예: `kimchi_v1 → kimchi_v2`, 가이드 11.5).
4. 교과 7 라벨 QA를 처음부터 다시 하지 않는다 (가이드 15.0).