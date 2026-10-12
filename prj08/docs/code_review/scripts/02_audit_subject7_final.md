# `scripts/02_audit_subject7_final.py` 코드 리뷰

> 2일차 · 118줄 · 가이드 15.1 · 15.6 Daily Gate

## 1. 한 줄 요약

`src/dataset_audit.py`를 불러 교과 7 FINAL 900장을 검사하고, **Audit 전에 고정해 둔 기대값 19항목**과 비교해 PASS / FAIL을 남깁니다. `--fixture` 옵션이면 테스트 데이터로 검사기 자체를 시험합니다.

## 2. 입력 · 출력 · 실행

| 모드 | 입력 | 출력 |
|---|---|---|
| `--fixture` | `data/audit_test/` | `data/audit_test/audit_issue.csv` (Git 제외) |
| 기본 | `data/subject7_final/` + `prj07/manifests/dataset_manifest.csv` + `EXPECTED` | `reports/audit_issue.csv` · `audit_check.csv` · `audit_label_stats.csv` |

```bash
python scripts/02_audit_subject7_final.py --fixture   # 검사기 시험
python scripts/02_audit_subject7_final.py             # 900장 · 정상 결과 PASS 14 / FAIL 5
```

## 3. 구조

| 줄 | 내용 |
|--:|---|
| 18~22 | `PROJECT_ROOT` · `from src.dataset_audit import ...` |
| 27~30 | 경로: `FINAL_DIR` · `FIXTURE_DIR` · `MANIFEST` · `REPORT_DIR` |
| 35~55 | `EXPECTED` 기대값 19항목 |
| 57 | `ISSUE_FIELDS` (CSV 컬럼) |
| 60~114 | `main()`: 옵션 → 검사 → 화면 → 저장 → 대조 |

## 4. 핵심 코드 읽기

### 4.1 교과 7 Manifest를 상대경로로

```python
MANIFEST = PROJECT_ROOT.parent / "prj07" / "manifests" / "dataset_manifest.csv"
```

`prj08/`의 한 단계 위(레포 루트)에서 `prj07/`로 들어갑니다. 레포 구조(prj07 · prj08 폴더)를 활용해 교과 7 결과를 **복사하지 않고 그대로** 읽습니다.

### 4.2 기대값 — 결과를 보고 고치지 않는다

```python
# 기대값: reports/audit_expected.md 1·2장에서 옮겨 적은 값 (실행 결과를 보고 고치지 않음)
EXPECTED = {
    "jpg_count": 900, "txt_count": 900, "pair_count": 900,
    "missing_label": 0, "missing_image": 0, "empty_label": 0,
    "class4_files": 0, "format_errors": 0,
    "bbox_total": 4602,
    "class_0": 868, "class_1": 1897, "class_2": 767, "class_3": 378,
    "class_4": 0, "class_5": 441, "class_6": 251,
    "manifest_rows": 900, "not_in_manifest": 0, "missing_in_dataset": 0,
}
```

이 숫자들은 교과 7 문서(10/07 QA)에서 왔습니다. 실제 라벨(10/08 수정본)은 4,601이라 **FAIL 5건이 정상**입니다. 이 FAIL을 숨기지 않고 추적해서 "10/08 수정이 문서에 반영되지 않았다"는 원인을 찾았고, 팀원 2명이 옛 버전을 쓰고 있다는 것도 발견했습니다.

### 4.3 한 스크립트, 두 모드

```python
parser.add_argument("--fixture", action="store_true", help="...")
args = parser.parse_args()

if args.fixture:
    data_dir, manifest, out_dir = FIXTURE_DIR, None, FIXTURE_DIR      # Manifest · 기대값 대조 안 함
else:
    data_dir, manifest, out_dir = FINAL_DIR, MANIFEST, REPORT_DIR
    if not MANIFEST.exists():
        raise FileNotFoundError(f"Manifest를 찾을 수 없습니다: {MANIFEST}")
```

- `action="store_true"` → 옵션을 쓰면 `True`, 안 쓰면 `False`
- Fixture 결과는 `data/audit_test/`에 저장 → **Git에 올라가지 않고**, 실제 결과(`reports/`)를 덮어쓰지도 않습니다.

### 4.4 검사 → 저장 → 대조

```python
issues, stats, image_rows = audit_dataset(data_dir / "images", data_dir / "labels", manifest)
write_csv(issues, out_dir / "audit_issue.csv", ISSUE_FIELDS)
if args.fixture:
    return                                         # 시험 모드는 여기까지
check_rows = compare_with_expected(stats, EXPECTED)
write_csv(check_rows, REPORT_DIR / "audit_check.csv", ["item", "expected", "actual", "result"])
write_csv(image_rows, REPORT_DIR / "audit_label_stats.csv", stat_fields)
```

`audit_label_stats.csv`는 단순 기록이 아니라 **다음 단계(03 · 04)의 입력 데이터**입니다.

## 5. 연결된 파일

| 관계 | 파일 |
|---|---|
| 불러 쓰는 기능 | `src/dataset_audit.py` |
| 기대값 출처 | `reports/audit_expected.md` |
| 교과 7 입력 | `prj07/manifests/dataset_manifest.csv` |
| 결과를 쓰는 곳 | `audit_label_stats.csv` → `03_split_analysis.py` · `04_make_split.py` |
| 원인 기록 | `reports/issue_log.md` |

## 6. 리뷰 포인트

- ✅ `--fixture` 하나로 테스트 / 실제 모드 · 테스트 결과는 Git 밖에 저장
- ✅ 기대값 주석에 Test Oracle 원칙 명시 → FAIL을 숨기지 않음
- ✅ 결과를 화면 + CSV로 남김 (가이드 9.4) · 화면에 FAIL 항목만 따로 출력
- ✅ Manifest가 없으면 즉시 멈춤
- 🔧 **기대값이 두 곳**(`audit_expected.md` · `EXPECTED`) → `configs/audit_expected.yaml` 하나에서 읽기 (Single Source of Truth)
- 🔧 **기대값의 데이터 버전 표기 없음** → 옛 버전 라벨이 "PASS 19"로 통과했던 근본 원인
  ```python
  EXPECTED_SOURCE = "교과 7 문서 (2026-10-07 QA · 4,602)"   # 화면 · CSV에 함께 출력
  ```
- 🔧 `"prj07"` 폴더 이름이 코드에 직접 들어 있음 → 폴더 이름을 바꾸면 여기가 깨짐
- 🔧 Fixture 결과를 자동 판정하지 않음 → `--fixture`일 때 9종 각 1건인지 `assert`
- ❓ kimchi_v1(4,601) 기준 기대값을 새로 만든다면, 기존 4,602 기대값은 지울까 남길까?

## 7. 직접 확인해 보기

```bash
cd prj08 && source .venv/bin/activate
python scripts/02_audit_subject7_final.py
git status --short reports/        # 아무것도 안 나오면 팀 기준 결과와 동일
```
