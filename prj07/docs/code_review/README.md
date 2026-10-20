# 코드 리뷰 (파일별)

> **기준**: `main` · `cbf3d10` (2026-10-20) · `prj07/` · 라벨링 프로그램 v4.1
> **구성**: 실제 코드와 **같은 폴더 구조**로 파일마다 리뷰 문서 1개 ([PRJ08 코드 리뷰](../../../prj08/docs/code_review/README.md)와 같은 방식)
> **진행**: 1차로 저장 · 교차검수 흐름의 **핵심 5개**를 먼저 작성했고, 나머지 13개는 같은 형식으로 이어서 작성합니다.

## 리뷰 문서 구조

```
docs/code_review/
├── README.md                     # 이 문서 · 목차 · 읽는 순서 · 표기법
├── main.md                       # main.py                      (예정)
├── configs/
│   └── classes_yaml.md           # configs/classes.yaml         (예정)
├── src/
│   ├── yolo/
│   │   ├── yolo_loader.md        # src/yolo/yolo_loader.py      (예정)
│   │   └── yolo_writer.md        # src/yolo/yolo_writer.py      ✔
│   ├── bbox/
│   │   ├── bbox_manager.md       # src/bbox/bbox_manager.py     (예정)
│   │   └── bbox_editor.md        # src/bbox/bbox_editor.py      (예정)
│   ├── ui/
│   │   ├── main_window.md        # src/ui/main_window.py        ✔
│   │   ├── canvas.md             # src/ui/canvas.py             (예정)
│   │   └── validation_window.md  # src/ui/validation_window.py  (예정)
│   ├── records/
│   │   ├── manifest.md           # src/records/manifest.py      ✔
│   │   └── history.md            # src/records/history.py       (예정)
│   ├── validation/
│   │   └── validator.md          # src/validation/validator.py  ✔
│   └── qa/
│       ├── self_test.md          # src/qa/self_test.py          (예정)
│       └── reports.md            # src/qa/reports.py            (예정)
└── tools/
    ├── label_stats.md            # tools/label_stats.py         (예정)
    ├── make_reports.md           # tools/make_reports.py        (예정)
    └── merge_results.md          # tools/merge_results.py       ✔
```

`__init__.py` 파일들은 내용이 없는 파일("이 폴더를 패키지로 인식시키는 표시")이라 별도 문서를 만들지 않았습니다.

## 실제 코드 ↔ 리뷰 문서

| 코드 | 줄 수 | 한 줄 역할 | 리뷰 문서 |
|---|--:|---|---|
| `main.py` | 94 | 프로그램 시작 · `classes.yaml` 읽고 검사 → 화면 띄우기 | 예정 |
| `configs/classes.yaml` | 75 | Class · 팀원 · scene_type · review_reason · 팀 규칙(`allow_self_review` 등) | 예정 |
| `src/yolo/yolo_loader.py` | 279 | data 폴더 → 900장 작업 목록 · 단계별 라벨 경로 · TXT 읽기 · 원본 지문 | 예정 |
| `src/yolo/yolo_writer.py` | 129 | TXT 저장 · work / final 이동 · raw 쓰기 금지 · raw → final 직행 금지 | [yolo_writer.md](src/yolo/yolo_writer.md) |
| `src/bbox/bbox_manager.py` | 110 | BBox 데이터 (생성 · 수정 · 삭제 · 되돌리기) · 화면 없이 테스트 가능 | 예정 |
| `src/bbox/bbox_editor.py` | 163 | 마우스로 BBox 그리기 · 이동 · 크기 조절 | 예정 |
| `src/ui/main_window.py` | 1,310 | 화면 · 메뉴 · 단축키 · **저장 버튼 하나로 작업자 저장 / 반려 / QA PASS** | [main_window.md](src/ui/main_window.md) |
| `src/ui/canvas.py` | 402 | 이미지 그리기 · Zoom · Pan · 화면 좌표 ↔ 원본 좌표 | 예정 |
| `src/ui/validation_window.py` | 96 | 전체 Validation 결과 창 · 더블클릭으로 이동 | 예정 |
| `src/records/manifest.py` | 109 | 작업대장 `dataset_manifest.csv` (이미지 1장 = 1줄) | [manifest.md](src/records/manifest.md) |
| `src/records/history.py` | 49 | 작업 이력 `label_history.csv` (누가 · 언제 · 무엇을, 지우지 않음) | 예정 |
| `src/validation/validator.py` | 330 | 설정 검사 · QA PASS 직전 검사 · 900장 Validation (오류 8종) | [validator.md](src/validation/validator.md) |
| `src/qa/self_test.py` | 190 | Golden 20 · Pilot 50 자동 테스트 (임시 폴더 · 원본 그대로) | 예정 |
| `src/qa/reports.py` | 501 | 산출물 자동 생성 (7 · 8 · 9 · 11번) · Handoff 패키지 | 예정 |
| `tools/label_stats.py` | 112 | RAW vs FINAL 라벨 통계 (읽기만) | 예정 |
| `tools/make_reports.py` | 71 | 화면 없이 산출물 다시 만들기 | 예정 |
| `tools/merge_results.py` | 257 | 팀원 결과 묶기(pack) · 합치기(merge) · 충돌 표시 | [merge_results.md](tools/merge_results.md) |

## 그림은 어디에?

| 보고 싶은 것 | 위치 |
|---|---|
| 파일 **하나 안**의 흐름 (함수 순서 · 분기) | 각 리뷰 문서 3장 **흐름 그림** |
| 데이터가 **폴더 사이**로 흐르는 모습 (raw → work → final) | [`../../README.md`](../../README.md) 11장 데이터 위치와 흐름 |

같은 그림을 두 곳에 두지 않습니다. 코드를 고치면 **그 파일의 리뷰 문서 그림 하나만** 고치면 됩니다.

## 읽는 순서 (추천)

데이터가 흐르는 순서대로 읽으면 이해가 쉽습니다. ✔ = 작성 완료

```
main → classes_yaml                          (시작: 설정 읽기 · 검사)
      ↓
yolo_loader                                  (data 폴더 → 900장 목록 · 라벨 읽기 · 원본 지문)
      ↓
canvas → bbox_manager → bbox_editor          (화면에서 박스 보기 · 고치기)
      ↓
main_window ✔                                 (저장 버튼: 작업자 / 반려 / QA PASS)
      ↓  ↘
yolo_writer ✔        manifest ✔ → history     (TXT 를 work · final 로 / 작업대장 · 이력 기록)
      ↓
validator ✔ → validation_window              (F7 · 900장 검사 · 결과 창)
      ↓
self_test → reports                          (Golden · Pilot → 산출물 · Handoff)
      ↓
merge_results ✔ → make_reports → label_stats  (5명 결과 합치기 → 최종 숫자 · 통계)
```

처음 읽는다면 **main_window의 흐름 그림**부터 보세요. 저장 버튼 하나에서 나머지 4개 문서(yolo_writer · manifest · validator · merge_results)로 이어집니다.

## 표기법

모든 리뷰 문서는 PRJ08과 같은 순서로 씁니다.

1. **한 줄 요약**: 이 파일이 왜 필요한가
2. **입력 · 출력 · 실행**: 무엇을 받아서 무엇을 만드나 · 누가 가져다 쓰나
3. **구조**: 함수 · 구간별 위치(줄 번호) + **흐름 그림**(Mermaid · 이 파일 **안에서만** 어떤 순서로 도나)
4. **핵심 코드 읽기**: 중요한 코드 발췌와 짧은 설명 · 알아둘 한계

줄 번호와 코드 발췌는 위 **기준** 커밋의 코드와 같습니다. 코드를 고치면 리뷰 문서의 줄 번호도 함께 고쳐 주세요.
