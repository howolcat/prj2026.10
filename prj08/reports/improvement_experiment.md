# 개선 실험 계획 — exp001 (7일차 · 산출물 4 · 가이드 20.3)

> **상태: 초안**  ← 회의 후 숫자를 확정하면 이 줄의 '초안'을 '확정'으로 바꾼다 (그래야 `10_train_experiment.py` 본 학습이 시작됨)
> 근거: `reports/failure_hypothesis.md` · 설정: `configs/exp001.yaml` · 비교 기준: `baseline_v1`

## 1. 무엇을 바꾸나 (변수 하나)

| 변수 | Baseline | exp001 |
|---|---|---|
| `imgsz` | 640 | ☐ 1280 (또는 960 · 회의 결정 · `configs/exp001.yaml`과 같은 값) |

## 2. 왜 바꾸나

- H1: 놓친 정답 159개 중 153개가 640 기준 32px 미만 작은 객체 · 학습량은 충분(40 epoch 천장)
- 기대: 작은 이물이 2배(1280) 또는 1.5배(960) 크게 보여 FN 감소

## 3. 무엇을 그대로 두나

Dataset Version `kimchi_v1` · Train 600 / Val 74 · YOLO11n 사전학습 · epochs 50 · patience 50 · batch 16 · AdamW 0.000909 · momentum 0.9 · seed 42 · deterministic · amp · workers 2 · 같은 학습 함수 · 같은 채점 코드

## 4. 1 epoch 시험 결과 (`--smoke`)

| 항목 | 값 |
|---|---|
| 1 epoch 시간 · 50 epoch 예상 | ☐ |
| GPU 메모리 최대 / 12GB | ☐ |
| 결정 | ☐ (batch 16 유지 가능 → 그대로 / 부족 → imgsz 960 으로 낮추기) |

## 5. 무엇을 보고 판단하나 (같은 Val 74장)

- 주 지표: **FN (conf 0.25) · 작은 객체 FN (/363) · Recall** — H1이 실제로 줄었는가
- 함께 볼 것: mAP50 · mAP50-95 · FP 증가 여부 · 갈변(H2) · branch FP(H3) 변화
- 판단 기준: 0.01~0.03 차이는 잡음 범위(Val 74장)로 보고, FN · 작은 객체 FN 의 뚜렷한 감소를 개선으로 본다
- 결과가 나빠도 기록한다 (`experiment_log.csv` · `baseline_vs_improved.md`) · Frozen Test 는 보지 않는다
