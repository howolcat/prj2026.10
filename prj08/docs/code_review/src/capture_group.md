# `src/capture_group.py` 코드 리뷰

> 2일차 · 54줄 · 가이드 15.3 (Data Leakage) · 11.1 (capture_group)

## 1. 한 줄 요약

파일명에서 **촬영 시각**을 꺼내고, 시각이 가까운 사진끼리 **촬영 세션(capture_group)**으로 묶고, 두 사진 묶음 사이의 **가장 가까운 시간 차**를 재는 기능 모음입니다. Split에서 "같은 장면이 Train과 Test에 나뉘어 들어가는 것(Leakage)"을 막는 기반입니다.

## 2. 입력 · 출력 · 실행

| 함수 | 입력 | 출력 |
|---|---|---|
| `parse_capture_time` | 파일명 문자열 | `datetime` |
| `assign_capture_groups` | 촬영 시각 Series · 기준(분) | 세션 번호 Series (1부터) |
| `nearest_gap_seconds` | 시각 Series 2개 (target · reference) | target별 최소 시간 차(초) |

실행: 직접 실행 안 함 · `scripts/03_split_analysis.py`가 import

## 3. 구조

| 줄 | 이름 |
|--:|---|
| 20 | `parse_capture_time(file_name)` |
| 30 | `assign_capture_groups(times, gap_minutes)` |
| 46 | `nearest_gap_seconds(target, reference)` |

## 4. 핵심 코드 읽기

### 4.1 `parse_capture_time` — 파일명 → 시각

```python
stem = Path(file_name).stem                     # 250410_180800068444
date_part, _, time_part = stem.partition("_")   # '250410', '_', '180800068444'
if len(date_part) != 6 or len(time_part) != 12 or not time_part.isdigit():
    raise ValueError(f"예상과 다른 파일명 형식: {file_name}")
base = datetime.strptime(date_part + time_part[:6], "%y%m%d%H%M%S")   # 2025-04-10 18:08:00
return base + timedelta(microseconds=int(time_part[6:]))              # + 0.068444초
```

```
250410_180800068444
──┬─── ──┬─── ──┬───
YYMMDD HHMMSS  마이크로초(6자리)
```

- `partition("_")`은 `split`과 달리 항상 3개(앞 · 구분자 · 뒤)를 돌려줘서 안전합니다.
- 형식이 다르면 **조용히 틀린 시각을 만들지 않고 멈춥니다**. 틀린 시각으로 세션을 묶으면 Split 전체가 틀어지기 때문입니다.
- ⚠ "파일명 = 촬영 시각"은 **가정**입니다. EXIF가 없어서 연속 사진을 직접 보고 확인했습니다(`docs/split_policy.md` 3장).

### 4.2 `assign_capture_groups` — diff + cumsum 트릭

```python
order = times.sort_values()                                       # ① 시각 순 정렬
gap = order.diff()                                                # ② 바로 앞 사진과의 시간 차
is_new = gap.isna() | (gap > pd.Timedelta(minutes=gap_minutes))  # ③ 첫 장 또는 기준 초과 → True
group_no = is_new.cumsum()                                        # ④ True 누적합 = 세션 번호
return group_no.reindex(times.index)                              # ⑤ 원래 행 순서로 되돌림
```

| 사진 | 앞 사진과 차이 | is_new | cumsum = 세션 |
|:-:|---|:-:|:-:|
| 1 | (없음) | True | 1 |
| 2 | 45초 | False | 1 |
| 3 | 12분 | True | 2 |
| 4 | 7분 | True | 3 |
| 5 | 30초 | False | 3 |

- 반복문 없이 900장 세션 번호를 한 번에 매깁니다.
- ⑤ `reindex`가 없으면 결과가 **시각 순서**로 나와서, 원래 표에 붙일 때 행이 엇갈립니다.
- 경계: `gap > 5분`이므로 **정확히 5분(300.0초)이면 같은 세션**입니다. 정책 문서 "5분 **초과**이면 새 세션"과 일치합니다.

### 4.3 `nearest_gap_seconds` — 가장 가까운 사진까지의 시간

```python
ref_ns = np.asarray(reference.to_numpy(dtype="datetime64[ns]").astype("int64"))   # 시각 → 나노초 정수
out = []
for t in target:
    out.append(np.abs(ref_ns - t.value).min() / 1e9)   # 모든 reference와의 차이 중 최소 → 초
return pd.Series(out, index=target.index)
```

- 시각을 정수로 바꾸면 numpy가 **720개와의 뺄셈을 한 번에** 합니다.
- 이 함수로 "기존 validation 180장 중 135장이 60초 안에 train 사진이 있다 · 최소 8초"를 계산했습니다.

## 5. 연결된 파일

| 관계 | 파일 |
|---|---|
| 이 파일을 쓰는 곳 | `scripts/03_split_analysis.py` |
| 결과가 쓰이는 곳 | `reports/capture_groups.csv` → `scripts/04_make_split.py` |
| 같은 로직이 한 번 더 | `04_make_split.py`의 `split_e1()` (G01을 2분 기준으로 다시 나눔) |

## 6. 리뷰 포인트

- ✅ 형식 검증 후 실패하면 즉시 `ValueError` (조용한 오류 방지)
- ✅ diff · cumsum · reindex로 짧고 빠르게 구현 · 주석으로 원리 설명
- ✅ 시간 비교를 마이크로초까지 → 경계값(60.28초)을 정확히 판정
- 🔧 `04_make_split.py`의 `split_e1()`이 같은 diff + cumsum 로직을 다시 씀 → `assign_capture_groups()`를 재사용하면 한 곳만 관리
- 🔧 세션 이름(`G01`)을 만드는 코드가 `03`과 `04`에 따로 있음 → 여기에 `group_name(n) -> "G01"` 함수를 두면 항상 같은 이름
- 🔧 `nearest_gap_seconds`는 target × reference 전부 비교(180 × 720) → 지금은 충분히 빠르지만 데이터가 커지면 정렬 후 `np.searchsorted`로 개선 가능
- ❓ 날짜가 바뀌는 자정 근처(23:58 → 00:02) 촬영이 있다면 세션이 올바르게 나뉠까? (`datetime` 비교라 날짜까지 포함되므로 OK)
- ❓ 파일명 규칙이 다른 새 데이터(교과 14)가 들어오면 이 함수를 어떻게 바꿔야 할까?

## 7. 직접 확인해 보기

```bash
cd prj08 && source .venv/bin/activate
python - <<'EOF'
import pandas as pd
from src.capture_group import parse_capture_time, assign_capture_groups
names = ["250410_100000000000.jpg", "250410_100045000000.jpg", "250410_101300000000.jpg",
         "250410_102000000000.jpg", "250410_102030000000.jpg"]
t = pd.Series([parse_capture_time(n) for n in names])
print(assign_capture_groups(t, 5).tolist())   # 예상: [1, 1, 2, 3, 3]
EOF
```
