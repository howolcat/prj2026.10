# `scripts/09b_failure_images.py` 코드 리뷰

> 6일차 · 184줄 · 가이드 19.A · 19.6 (대표 실패 사례 GT vs Prediction 비교 이미지)

## 1. 한 줄 요약

`reports/failure_analysis.csv`에 적힌 Val 사진마다 **정답(초록 · 놓친 정답은 노랑)과 Baseline 예측(빨강)을 한 장에 겹쳐 그려** `demo/failure_analysis/`에 저장하고, 어떤 사진을 그렸는지 숫자 목록(`reports/failure_image_list.csv`)을 남깁니다. 가중치 sha가 4일차 기록과 같은지 먼저 확인하고, 짝짓기는 09와 같은 `match_boxes()`를 씁니다. Val 사진만 그립니다.

## 2. 입력 · 출력 · 실행

| 구분 | 파일 |
|---|---|
| 입력 | `reports/failure_analysis.csv` (`image` · `failure_type` 칸) · `models/baseline/baseline_best.pt` · `reports/baseline_training_condition.md` (sha) · `manifests/val.csv` · `data/yolo/{images,labels}/val` |
| 출력 (Git) | `reports/failure_image_list.csv` (그린 사진 목록 · TP · FP · FN 숫자만) |
| 출력 (Git 제외 · NDA) | `demo/failure_analysis/<사진>_compare.jpg` (회사 사진 · Discord · Figma 금지) |

| 옵션 | 기본값 | 쓰이는 곳 |
|---|:--:|---|
| `--max` | 0 (전부) | 앞에서부터 n장만 그리기 |

```bash
python scripts/09b_failure_images.py            # failure_analysis.csv 의 사진 전부 (중복 제거)
python scripts/09b_failure_images.py --max 10   # 앞에서부터 10장만
```

그림 읽는 법은 이렇습니다.

| 색 | 의미 | 글자 위치 |
|---|---|---|
| 초록 굵은 선 | 정답(GT) 중 모델이 맞힌 것 | 박스 위 `GT Class` |
| 노랑 굵은 선 | 정답(GT) 중 모델이 놓친 것 (FN) | 박스 위 `MISSED Class` |
| 빨강 선 | 모델 예측 (conf ≥ 0.25) | 박스 아래 `Class 확신도` |
| 맨 위 띠 | 파일명 · 실패 유형 · TP / FP / FN | — |

## 3. 구조

| 줄 | 내용 |
|--:|---|
| 1~26 | 설명 문서(docstring): 사용법 · 입력 · 결과 · 그림 읽는 법 · 원칙 |
| 27~36 | `import` (`pathlib` · `argparse` · `csv` · `hashlib` · `re` · `sys`) · `PROJECT_ROOT` 경로 설정 |
| 38~40 | `from PIL import ...` · `from src.manifest import read_manifest` · `from src.evaluate_detector import match_boxes` |
| 42~55 | 경로 8개 · `NAMES` (Class 이름 7개) · `CONF` · `IMGSZ` · `IOU` · 색 3개 · `SAVE_LONG_SIDE` |
| 58~63 | `sha12()` (가중치 sha256 앞 12자리) |
| 66~76 | `font()` (DejaVu 글꼴 → 없으면 기본 글꼴) |
| 79~91 | `read_gt()` (정답 TXT → 박스 목록) · `to_px()` (YOLO 정규화 좌표 → 픽셀) |
| 94~118 | `draw()`: 정답 · 예측 박스 + 맨 위 띠 + 긴 변 1920으로 축소 |
| 121~134 | `main()` 검사: 가중치 있음 · sha 일치 · CSV 있음 |
| 136~151 | 사진 목록 정리: 중복 합치기 · Val만 남기기 · `--max` |
| 153~173 | 예측 → 짝짓기 → 그림 저장 (한 장씩 `stream=True`) |
| 175~180 | `failure_image_list.csv` 저장 · 6일차 Gate 안내 |
| 183~184 | `if __name__ == "__main__": main()` |

### 흐름 그림

```mermaid
flowchart TD
    A["main()"] --> B{"baseline_best.pt 있음?"}
    B -->|없음| X1["중단"]
    B -->|있음| C{"sha = 4일차 기록?"}
    C -->|다름| X2["중단: 다른 가중치"]
    C -->|같음| D{"failure_analysis.csv 있음?"}
    D -->|없음| X3["중단"]
    D -->|있음| E["사진별 실패 유형 모으기<br/>(중복 제거)"]
    E --> F["Val 사진만 남기기<br/>--max 적용"]
    F --> G["YOLO 예측<br/>conf 0.25 · imgsz 640"]
    G --> H["match_boxes()<br/>IoU 0.5 로 TP/FP/FN"]
    H --> I["draw()<br/>초록 · 노랑 · 빨강 + 띠"]
    I --> J[/"demo/failure_analysis/<br/>*_compare.jpg"/]
    H --> K[/"reports/failure_image_list.csv"/]
```

09와 같은 가중치 · conf · imgsz · 짝짓기를 쓰므로, 그림 위 TP / FP / FN은 `baseline_val_image_check.csv`의 같은 사진 숫자와 같아야 정상입니다.

## 4. 핵심 코드 읽기

### 4.1 가중치가 "그 Baseline"인지 sha로 확인

```python
text = CONDITION_MD.read_text(encoding="utf-8") if CONDITION_MD.exists() else ""
m = re.search(r"baseline_best\.pt` · sha `([0-9a-f]{12})`", text)
...
if not m or sha12(BEST_PT) != m.group(1):
    sys.exit("[중단] baseline_best.pt sha 가 4일차 기록과 다릅니다 (다른 가중치)")
```

4일차 08이 `baseline_training_condition.md`에 남긴 sha를 정규식으로 꺼내 지금 파일의 sha와 비교합니다. 파일 이름이 같아도 내용이 바뀌었으면 멈춥니다 → **5일차 평가 · 6일차 분석과 같은 모델로 그림을 그린다**는 보장입니다.

### 4.2 같은 사진이 여러 행이면 한 장으로 합치기

```python
order, types = [], {}
for r in rows:
    name = r["image"].strip()
    if name not in types:
        order.append(name)
        types[name] = []
    if r["failure_type"] not in types[name]:
        types[name].append(r["failure_type"])
...
order = [n for n in order if n in val_names]
```

- `order`는 처음 나온 순서를 지키고, `types`는 사진별 실패 유형을 모읍니다 (예: `FN + Class`).
- `val_names`에 없는 사진은 건너뛰고 따로 출력합니다 → **Failure Analysis는 Validation만** (가이드 19.2).

### 4.3 예측 결과를 정답과 같은 모양으로

```python
preds = [(int(c), float(s), *map(float, b))
         for c, s, b in zip(r.boxes.cls.tolist(), r.boxes.conf.tolist(), r.boxes.xywhn.tolist())]
gts = read_gt(Path(name).stem)
mb = match_boxes(gts, preds, IOU)
```

- 예측은 `(class, conf, x, y, w, h)`, 정답은 `(class, x, y, w, h)` — 둘 다 **0~1 정규화 좌표(`xywhn`)** 라서 바로 비교됩니다.
- `stream=True`로 한 장씩 받아 4K 사진도 메모리를 아낍니다.

### 4.4 정답 박스와 예측 박스가 겹쳐도 둘 다 보이게

```python
lw = max(3, w // 700)        # 선 굵기: 사진 크기에 맞춰
pad = lw * 2                 # 정답 박스를 바깥으로 조금 키움
d.rectangle([x1 - pad, y1 - pad, x2 + pad, y2 + pad], outline=color, width=lw + 2)
...
d.text((x1, y2 + 2), f"{NAMES.get(p[0], p[0])} {p[1]:.2f}", fill=PRED_COLOR, font=f)
```

- 정답은 `pad`만큼 크게 · 더 굵게, 예측은 원래 크기 · 가늘게 → 딱 맞힌 경우에도 초록 테두리 안에 빨강이 보입니다.
- 글자는 정답 = 박스 **위**, 예측 = 박스 **아래** → 서로 겹치지 않습니다.
- `MISSED`는 `match_boxes()`가 돌려준 `fn_idx`(짝을 못 찾은 정답 번호)로 판단합니다.

### 4.5 4K 사진을 가볍게 저장

```python
scale = SAVE_LONG_SIDE / max(canvas.size)
if scale < 1:
    canvas = canvas.resize((int(canvas.width * scale), int(canvas.height * scale)))
```

띠를 붙인 뒤 긴 변을 1920으로 줄입니다. 박스는 원본 크기에서 그렸으므로 위치가 어긋나지 않습니다.

> **참고**: 그릴 사진이 0장이면(`order`가 비면) 마지막 `csv.DictWriter(..., fieldnames=list(out_rows[0]))`에서 `IndexError`가 납니다. `if not out_rows: sys.exit(...)` 한 줄을 넣으면 안내 메시지로 끝낼 수 있습니다.

## 5. 연결된 파일

| 관계 | 파일 |
|---|---|
| 앞 단계 (가중치 · sha) | `scripts/08_train_baseline.py` → `baseline_best.pt` · `baseline_training_condition.md` |
| 같은 기준 | `scripts/09_evaluate_baseline.py` (conf 0.25 · imgsz 640 · IoU 0.5) |
| 입력 (사람이 고른 사례) | `reports/failure_analysis.csv` |
| 불러 쓰는 기능 | `src/evaluate_detector.py` (`match_boxes`) · `src/manifest.py` (`read_manifest`) |
| 결과를 쓰는 곳 | `reports/failure_image_list.csv` → 6일차 Gate ② · `reports/failure_hypothesis.md` |
| 뒤 단계 | `scripts/11_evaluate_experiment.py` (같은 대표 사진의 FN · FP를 Baseline vs exp001로 비교) |
