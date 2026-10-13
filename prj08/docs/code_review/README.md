# 코드 리뷰 (파일별)

> **기준**: `main` · PR #73 (2026-10-12) · `prj08/`
> **구성**: 실제 코드와 **같은 폴더 구조**로 파일마다 리뷰 문서 1개
> **용도**: 10/13 리뷰 데이 · 코드 리뷰 / 프로젝트 리뷰

## 리뷰 문서 구조

```
docs/code_review/
├── README.md                          # 이 문서 · 목차 · 읽는 순서 · 표기법
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
    ├── 04_make_split.md               # scripts/04_make_split.py
    ├── 05_freeze_split.md             # scripts/05_freeze_split.py
    ├── 06_build_yolo_dataset.md       # scripts/06_build_yolo_dataset.py
    └── 07_verify_yolo_dataset.md      # scripts/07_verify_yolo_dataset.py
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
| `scripts/05_freeze_split.py` | 3 | 147 | 13항목 검사 후 `manifests/`로 Split Freeze | [05_freeze_split.md](scripts/05_freeze_split.md) |
| `scripts/06_build_yolo_dataset.py` | 3 | 179 | Split 목록으로 `data/yolo` 학습 폴더 구성 (Bridge 제외) | [06_build_yolo_dataset.md](scripts/06_build_yolo_dataset.md) |
| `scripts/07_verify_yolo_dataset.py` | 3 | 130 | `data/yolo`가 Freeze 본과 같은지 19항목 검증 | [07_verify_yolo_dataset.md](scripts/07_verify_yolo_dataset.md) |

## 그림은 어디에?

| 보고 싶은 것 | 위치 |
|---|---|
| 파일 **하나 안**의 흐름 (함수 순서 · 분기) | 각 리뷰 문서 3장 **흐름 그림** |
| 파일 **사이**의 흐름 (무엇을 읽고 무엇을 넘기나) · import 관계 | [`../code_flow.md`](../code_flow.md) |

같은 그림을 두 곳에 두지 않습니다. 코드를 고치면 **그 파일의 리뷰 문서 그림 하나만** 고치면 됩니다.

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
      ↓  splits/kimchi_v1_split.csv
05_freeze_split
      ↓  manifests/split_manifest.csv
06_build_yolo_dataset
      ↓  data/yolo
07_verify_yolo_dataset
```

## 표기법

모든 리뷰 문서는 같은 순서로 씁니다.

1. **한 줄 요약** — 이 파일이 왜 필요한가
2. **입력 · 출력 · 실행** — 무엇을 받아서 무엇을 만드나
3. **구조** — 함수 · 구간별 위치(줄 번호) + **흐름 그림**(Mermaid · 이 파일 안에서 어떤 순서로 도나)
4. **핵심 코드 읽기** — 중요한 코드 발췌와 설명
5. **연결된 파일** — 누가 이 파일을 쓰고, 이 파일은 누구를 쓰나
