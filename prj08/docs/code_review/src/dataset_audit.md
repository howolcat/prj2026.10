# `src/dataset_audit.py` 코드 리뷰

> 2일차 · 216줄 · 가이드 15.1 · 예시 2 ("데이터를 학습하기 전에 건강검진하는 프로그램")

## 1. 한 줄 요약

이미지 · YOLO 라벨 · 교과 7 Manifest를 **읽기만 해서** 짝 · 형식 · Class · 좌표 문제를 목록으로 만들고, Class별 BBox 수를 세는 **기능 모음**입니다. 혼자 실행되지 않고 `scripts/03_audit_subject7_final.py`가 불러 씁니다.

## 2. 입력 · 출력 · 실행

| 구분 | 내용 |
|---|---|
| 입력 | `images_dir` · `labels_dir` · (선택) `manifest_path` |
| 출력 | `issues` (문제 목록) · `stats` (전체 통계) · `image_rows` (이미지별 통계) |
| 실행 | 직접 실행 안 함 · `from src.dataset_audit import audit_dataset, ...` |
| 데이터 수정 | **없음** (읽기 전용) |

## 3. 구조

| 줄 | 이름 | 역할 |
|--:|---|---|
| 17~20 | `VALID_CLASSES` · `UNUSED_CLASSES` · `IMAGE_EXTS` · `EPS` | 교과 7 기준 상수 |
| 25~38 | `SEVERITY` | 문제 종류 12개 → ERROR / WARN |
| 41 | `validate_yolo_row(row)` | TXT **한 줄** 검사 (순수 함수) |
| 81 | `_issue(...)` | 문제 한 건을 CSV 한 줄(dict)로 |
| 92 | `read_manifest(path)` | Manifest → `{파일명: 행}` |
| 98 | `audit_dataset(...)` | **폴더 전체** 검사 (①~⑤단계) |
| 199 | `compare_with_expected(stats, expected)` | 기대값 대조 → PASS / FAIL |
| 209 | `write_csv(rows, path, fields)` | utf-8-sig로 저장 |

### 흐름 그림

```mermaid
flowchart TD
    A["audit_dataset(images, labels, manifest)"] --> B["① stem으로 짝 맞추기 (집합 연산)<br/>images ∩ labels → pairs<br/>images − labels → MISSING_LABEL<br/>labels − images → MISSING_IMAGE"]
    B --> C["② 빈 TXT → EMPTY_LABEL"]
    C --> D["③ TXT 한 줄씩 반복"]
    D --> E["validate_yolo_row(row)"]
    E --> E1{"5칸?"}
    E1 -->|아니오| X1["COLUMN_COUNT"]
    E1 -->|예| E2{"숫자 변환?"}
    E2 -->|실패| X2["PARSE_ERROR"]
    E2 -->|성공| E3["Class 0~6 · Class 4 · 좌표 0~1 · 박스 경계"]
    E3 --> I["_issue() → ERROR / WARN 기록"]
    X1 --> I
    X2 --> I
    D --> J["형식이 정상인 줄만 Class 개수에 더함"]
    A --> K["④ read_manifest()<br/>NOT_IN_MANIFEST · MISSING_IN_DATASET"]
    I --> L[/"issues · stats · image_rows"/]
    J --> L
    K --> L
    L --> M["compare_with_expected(stats, EXPECTED)<br/>→ PASS / FAIL 표"]
    L --> W["write_csv(rows, path, fields)"]
```

두 갈래를 기억하면 됩니다. 문제는 `issues`로, 숫자는 `stats`로 모입니다. 형식이 깨진 줄은 Class 개수에 들어가지 않습니다.

## 4. 핵심 코드 읽기

### 4.1 상수 — 교과 7 기준을 코드에 고정

```python
VALID_CLASSES = range(0, 7)        # Class ID 0~6
UNUSED_CLASSES = {4}               # Class 4 고무장갑: 사용하지 않음
IMAGE_EXTS = {".jpg", ".jpeg"}     # 대소문자 상관없이 비교 (.JPG 포함)
EPS = 1e-6                         # 소수점 반올림 오차 허용 범위
```

`EPS`가 필요한 이유: `x = 0.95, w = 0.1`이면 오른쪽 끝은 `1.0`이어야 하지만, 컴퓨터 소수 계산에서는 `1.0000000000000002`처럼 아주 조금 넘을 수 있습니다. 백만분의 1까지는 정상으로 봐 줍니다.

### 4.2 `validate_yolo_row` — 한 줄 검사 순서

```python
parts = row.strip().split()
if len(parts) != 5:
    return ["COLUMN_COUNT"]          # ① 칸 수부터. 틀리면 더 볼 필요 없이 바로 반환
try:
    cls = int(parts[0])
    x, y, w, h = map(float, parts[1:])
except ValueError:
    return ["PARSE_ERROR"]           # ② 숫자로 못 읽으면 바로 반환
issues = []
if cls not in VALID_CLASSES:  issues.append("CLASS_RANGE")      # ③ Class 범위
elif cls in UNUSED_CLASSES:   issues.append("CLASS4_REVIEW")
if not (0.0 <= x <= 1.0 and 0.0 <= y <= 1.0): issues.append("CENTER_RANGE")   # ④ 중심 0~1
if not (0.0 < w <= 1.0 and 0.0 < h <= 1.0):   issues.append("SIZE_RANGE")     #    크기 0 초과 1 이하
if "CENTER_RANGE" not in issues and "SIZE_RANGE" not in issues:               # ⑤ 박스 경계
    left, right = x - w / 2, x + w / 2
    top, bottom = y - h / 2, y + h / 2
    if left < -EPS or top < -EPS or right > 1 + EPS or bottom > 1 + EPS:
        issues.append("BBOX_OUT_OF_IMAGE")
return issues
```

- **순서가 중요한 이유**: ①②에서 실패하면 숫자가 없으므로 ③~⑤를 할 수 없습니다. 그래서 일찍 `return`(조기 반환)합니다.
- **⑤를 조건부로 하는 이유**: 중심이 1.5인 박스는 당연히 경계도 넘습니다. 같은 원인으로 문제가 두 개 기록되지 않게 막습니다.
- **순수 함수**: 파일 · 전역 상태를 건드리지 않고 문자열만 받아 결과만 돌려줍니다 → 따로 테스트하기 쉽습니다.

### 4.3 `audit_dataset` ① ② — 집합으로 짝 맞추기

```python
images = {p.stem: p for p in images_dir.iterdir() if p.suffix.lower() in IMAGE_EXTS}
labels = {p.stem: p for p in labels_dir.iterdir() if p.suffix.lower() == ".txt"}

for stem in sorted(images.keys() - labels.keys()):     # 차집합: 이미지만 있음
    issues.append(_issue(images[stem].name, "MISSING_LABEL", ...))
for stem in sorted(labels.keys() - images.keys()):     # 차집합: TXT만 있음
    issues.append(_issue(labels[stem].name, "MISSING_IMAGE", ...))
pairs = sorted(images.keys() & labels.keys())          # 교집합: 짝이 맞음
```

`stem` = 확장자를 뺀 이름(`250410_144403911987`). 이름표만 모아 **집합 연산**으로 짝을 한 번에 찾습니다. `sorted()`를 쓰는 이유는 실행할 때마다 같은 순서로 결과를 남기기 위해서입니다(재현성).

### 4.4 `audit_dataset` ③ — 한 줄씩 검사하고 Class 세기

```python
lines = [ln for ln in txt.read_text(encoding="utf-8").splitlines() if ln.strip()]
if not lines:
    issues.append(_issue(txt.name, "EMPTY_LABEL", detail="BBox 0개"))   # WARN

for no, line in enumerate(lines, start=1):
    problems = validate_yolo_row(line)
    for p in problems:
        issues.append(_issue(txt.name, p, line=no, detail=line.strip()))
    if not ({"COLUMN_COUNT", "PARSE_ERROR", "CLASS_RANGE"} & set(problems)):
        cls = int(line.split()[0])
        class_counts[cls] += 1          # 형식이 정상인 줄만 센다
        per_image[cls] += 1
```

- 형식이 깨진 줄을 세지 않아서 **`bbox_total = class_0 + … + class_6`이 항상 성립**합니다. (처음 버전에서는 Class 9 줄이 합계에만 들어가는 버그가 있었고, Fixture 테스트로 찾아 고쳤습니다.)
- `CLASS4_REVIEW`(WARN)인 줄은 세는 쪽에 포함됩니다 → Class 4가 있으면 `class_4` 개수로도 보입니다.

### 4.5 `audit_dataset` ④ ⑤ — Manifest 대조와 통계

```python
if manifest_path is not None:                       # Manifest가 있을 때만
    manifest = read_manifest(Path(manifest_path))
    for name in sorted(image_names - manifest.keys()):  issues.append(_issue(name, "NOT_IN_MANIFEST"))
    for name in sorted(manifest.keys() - image_names):  issues.append(_issue(name, "MISSING_IN_DATASET"))
    for row in image_rows:                          # 이미지별 통계에 출처 · split · scene_type 붙이기
        m = manifest.get(row["file_name"], {})
        row["source_dataset"] = m.get("source_dataset", "")
        ...
stats["class4_files"] = len({i["file"] for i in issues if i["issue_type"] == "CLASS4_REVIEW"})
```

- `manifest_path=None` 기본값 덕분에 **Fixture 모드(Manifest 없음)와 실제 모드를 같은 함수**로 처리합니다.
- `image_rows`에 출처 · split · scene_type을 붙여 두었기 때문에, 이 결과(`audit_label_stats.csv`)가 **04 · 05 스크립트의 입력**이 됩니다.
- `class4_files`는 줄 수가 아니라 **파일 수**로 셉니다(기대값 메모 기준과 맞춤).

### 4.6 `compare_with_expected` · `write_csv`

```python
rows.append({"item": key, "expected": exp, "actual": act,
             "result": "PASS" if act == exp else "FAIL"})
...
with open(path, "w", encoding="utf-8-sig", newline="") as f:   # 엑셀 한글 깨짐 방지
```

## 5. 연결된 파일

| 관계 | 파일 |
|---|---|
| 이 파일을 쓰는 곳 | `scripts/03_audit_subject7_final.py` |
| 테스트 데이터 | `scripts/02_make_audit_fixture.py` → `data/audit_test/` |
| 결과가 쓰이는 곳 | `audit_label_stats.csv` → `04_split_analysis.py` · `05_make_split.py` |
| 기대값 | `reports/audit_expected.md` (→ `03`의 `EXPECTED`) |
