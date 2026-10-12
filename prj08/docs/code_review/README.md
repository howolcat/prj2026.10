# 코드 리뷰 (파일별)

> **기준**: `main` · PR #73 (2026-10-12) · `prj08/`
> **구성**: 실제 코드와 **같은 폴더 구조**로 파일마다 리뷰 문서 1개
> **용도**: 10/13 리뷰 데이 · 코드 리뷰 / 프로젝트 리뷰

## 리뷰 문서 구조

```
docs/code_review/
├── README.md                          # 이 문서 · 목차 · 표기법 · 개선 후보 모음
├── configs/
│   └── dataset_yaml.md                # configs/dataset.yaml
├── src/
│   ├── dataset_audit.md               # src/dataset_audit.py
│   └── capture_group.md               # src/capture_group.py
└── scripts/
    ├── 01_check_environment.md        # scripts/01_check_environment.py
    ├── 02a_make_audit_fixture.md      # scripts/02a_make_audit_fixture.py
    ├── 02_audit_subject7_final.md     # scripts/02_audit_subject7_final.py
    ├── 03_split_analysis.md           # scripts/03_split_analysis.py
    └── 04_make_split.md               # scripts/04_make_split.py
```

`src/__init__.py`는 내용이 없는 파일(“src를 패키지로 인식시키는 표시”)이라 별도 문서를 만들지 않았습니다.

## 실제 코드 ↔ 리뷰 문서

| 코드 | 일차 | 줄 수 | 한 줄 역할 | 리뷰 문서 |
|---|:-:|--:|---|---|
| `configs/dataset.yaml` | 1 | 14 | YOLO가 읽는 데이터 경로 · Class 이름 | [dataset_yaml.md](configs/dataset_yaml.md) |
| `src/dataset_audit.py` | 2 | 216 | 데이터 "건강검진" 함수 모음 | [dataset_audit.md](src/dataset_audit.md) |
| `src/capture_group.py` | 2 | 54 | 파일명 → 촬영 시각 · 촬영 세션 묶기 | [capture_group.md](src/capture_group.md) |
| `scripts/01_check_environment.py` | 1 | 36 | 환경 · GPU · YOLO Load 확인 | [01_check_environment.md](scripts/01_check_environment.md) |
| `scripts/02a_make_audit_fixture.py` | 2 | 76 | 버그 9종을 심은 테스트 데이터 | [02a_make_audit_fixture.md](scripts/02a_make_audit_fixture.md) |
| `scripts/02_audit_subject7_final.py` | 2 | 118 | 900장 Audit + 기대값 대조 | [02_audit_subject7_final.md](scripts/02_audit_subject7_final.md) |
| `scripts/03_split_analysis.py` | 2 | 202 | 분포 · EXIF · 세션 비교 · Leakage | [03_split_analysis.md](scripts/03_split_analysis.md) |
| `scripts/04_make_split.py` | 3 선행 | 266 | 전수 탐색 배정 + 합격 기준 검증 | [04_make_split.md](scripts/04_make_split.md) |

## 읽는 순서 (추천)

데이터가 흐르는 순서대로 읽으면 이해가 쉽습니다.

```
01_check_environment → dataset_yaml
      ↓
dataset_audit (기능) → 02a_make_audit_fixture → 02_audit_subject7_final
      ↓  audit_label_stats.csv
capture_group (기능) → 03_split_analysis
      ↓  capture_groups.csv  +  docs/split_policy.md (사람의 결정)
04_make_split
```

## 표기법

모든 리뷰 문서는 같은 순서로 씁니다.

1. **한 줄 요약** — 이 파일이 왜 필요한가
2. **입력 · 출력 · 실행** — 무엇을 받아서 무엇을 만드나
3. **구조** — 함수 · 구간별 위치(줄 번호)
4. **핵심 코드 읽기** — 중요한 코드 발췌와 설명
5. **연결된 파일** — 누가 이 파일을 쓰고, 이 파일은 누구를 쓰나
6. **리뷰 포인트** — ✅ 잘한 점 · 🔧 개선 후보 · ❓ 토론 질문
7. **직접 확인해 보기** — 이해했는지 손으로 확인하는 실습

> 🔧는 "지금 당장 고쳐야 한다"가 아니라 "이해했는지 확인하는 항목"입니다. 고치기로 정하면 Issue Log에 옮기고 각자 브랜치에서 PR로 올립니다.

## 개선 후보 모음 (우선순위)

| 순위 | 파일 | 내용 | 언제 |
|:-:|---|---|---|
| 1 | `04_make_split.py` | 가이드 16.1 형식(`manifests/`) 출력 · `bridge_holdout` 이름 · 설명 줄 수정 | 3일차 |
| 2 | `scripts/` 번호 | 가이드 번호(`03_build_split` · `04_preview` · `05_smoke`)와 맞추기 | 3일차 시작 전 |
| 3 | `02a` · `02` | Fixture 결과 자동 판정 (`assert`) | 여유 있을 때 |
| 4 | `dataset_audit.py` | 줄 번호 어긋남 · 중복 stem · 인코딩 오류 처리 | 여유 있을 때 |
| 5 | `02_audit` | 기대값을 한 곳으로 · 기준 데이터 버전 표기 | kimchi_v2 전 |
| 6 | `requirements.txt` | 실제 설치 버전 기록 (`pip freeze`) | 4일차 Baseline 전 |
| 7 | `03` · `04` | 세션 이름(G01) 만드는 함수를 `src`로 통합 | 여유 있을 때 |
| 8 | `01` | 환경 확인 결과를 파일로 저장 | 여유 있을 때 |
