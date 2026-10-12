# 교과 8 · 조각김치 이물 객체검출 (YOLO11)

> 교과 7에서 검수한 정답 데이터로 YOLO11 객체검출 모델을 학습 · 평가 · 개선한다.
>
> **Dataset Version** `kimchi_v1` · **라벨 지문** `5c2b4fc2c3e6` · **진행 상황** 2일차 완료 (Split 정책 확정) · 3일차 Split Freeze 예정

---

## 1. 프로젝트 목적

- 교과 7 FINAL(JPG 900장 + YOLO TXT 900개)을 학습용 Dataset으로 구성한다.
- YOLO11n Baseline을 학습하고 Validation에서 Precision · Recall · mAP · FP · FN을 분석한다.
- 실패 원인 하나만 바꾼 Improved Model을 같은 Validation에서 비교해 Final Model을 고르고, Frozen Test로 최종 성능과 한계를 확인한다.
- Final Model로 Detection · Auto Label 후보를 만들고 교과 7 라벨링 도구로 Human Review까지 연결한다.

## 2. 교과 7 → 교과 8 연결

- 교과 7 FINAL: `data/subject7_final/` (원본: `master_data/final` · Git 미포함 · 읽기 전용)
- JPG 900장 / YOLO TXT 900개 · Pair 900 · 동일 이미지(md5) 0쌍
- scene_type: kimchi_with_target 750장 / object_only 150장 (normal_kimchi 0장 · other_review 0장)
- Empty Label 0개 · Class 4(고무장갑) 0건 (교과 7에서 enabled: false)
- BBox 4,601개 (`kimchi_v1` · 아래 Class 표)
- QA 상태: 900장 전체 QA PASS · 교과 7 QA 결과: `prj07/reports/qa_summary.md`

## 3. Dataset 설명

| source_dataset | original_split | 수량 |
|---|---|---:|
| 이물검출_학습데이터1 | train | 500 |
| 이물검출_학습데이터2 | train | 220 |
| 이물검출_학습데이터2 | validation | 180 |
| **합계** | | **900** |

## 4. Class

| ID | 이물 종류 | 영문 이름 | 사용 | BBox | 비율 |
|---|---|---|---|---:|---:|
| 0 | 나뭇잎·종이류 | leaf_paper | O | 924 | 20.1% |
| 1 | 플라스틱류·돌·금속류 | plastic_stone_metal | O | 1,897 | 41.2% |
| 2 | 나뭇가지류 | branch | O | 711 | 15.5% |
| 3 | 벌레류 | insect | O | 378 | 8.2% |
| 4 | 고무장갑 | rubber_glove_unused | X (발견 시 REVIEW 확인) | 0 | 0.0% |
| 5 | 병해·갈변 | disease_browning | O | 439 | 9.5% |
| 6 | 파·고추 | green_onion_pepper | O | 252 | 5.5% |
| | **합계** | | | **4,601** | 100% |

> **Dataset Version: `kimchi_v1`** · 라벨 지문(sha256 앞 12자리): `5c2b4fc2c3e6` · 5명 일치 확인 (2026-10-12)
> 교과 7 문서(qa_summary · handoff)는 4,602개로 적혀 있지만, 10/08 라벨 추가 수정(Class 2→0 56개 등)을 반영한 4,601개를 기준으로 한다. 상세: `reports/issue_log.md`, `docs/split_policy.md`

## 5. 실행환경

| 항목 | 값 |
|---|---|
| OS | Windows + WSL Ubuntu 22.04 · VS Code |
| Python | 3.10.12 |
| PyTorch | 2.14.1+cu130 |
| Ultralytics | 8.4.175 |
| GPU | NVIDIA GeForce RTX 3060 (학교 PC) |
| YOLO11n Load | 성공 |

### 팀원 환경 확인 (2026-10-12)

| 팀원 | Ultralytics | CUDA | GPU | YOLO Load | 라벨 지문 |
|---|---|---|---|---|---|
| 최용훈 | 8.4.175 | True | RTX 3060 | 성공 | `5c2b4fc2c3e6` |
| 박건 | 8.4.175 | True | RTX 3060 | 성공 | `5c2b4fc2c3e6` |
| 이승훈 | 8.4.175 | True | RTX 3060 | 성공 | `5c2b4fc2c3e6` |
| 김하민 | 8.4.175 | True | RTX 3060 | 성공 | `5c2b4fc2c3e6` |
| 심준형 | 8.4.175 | True | RTX 3060 | 성공 | `5c2b4fc2c3e6` |

## 6. 설치 · 실행 방법

### 6.1 처음 한 번 (환경 · 데이터 준비)

```bash
cd ~/exe_01/prj07/prj07_team01/prj08        # 각자 레포 위치

# 가상환경
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 교과 7 FINAL 복사 (Git에 없음 · FINAL 위치는 각자 PC 기준으로 바꾸기)
FINAL=~/exe_01/master_data/final
mkdir -p data/subject7_final
cp -r "$FINAL/images" "$FINAL/labels" data/subject7_final/
chmod -R a-w data/subject7_final             # 읽기 전용 잠금

# 데이터 버전 확인 → 5c2b4fc2c3e6 이어야 kimchi_v1
LC_ALL=C cat data/subject7_final/labels/*.txt | sha256sum | cut -c1-12
```

### 6.2 스크립트 실행 (모두 `prj08/`에서)

| 순서 | 명령 | 정상 결과 |
|:-:|---|---|
| 1 | `python scripts/01_check_environment.py` | CUDA True · YOLO Load 성공 (Class 80개) |
| 2 | `python scripts/02a_make_audit_fixture.py` | `data/audit_test/` 생성 · 버그 9종 목록 |
| 3 | `python scripts/02_audit_subject7_final.py --fixture` | 9종 각 1건 검출 |
| 4 | `python scripts/02_audit_subject7_final.py` | PASS 14 / FAIL 5 (FAIL = 교과 7 문서 4,602 기준이라 정상) |
| 5 | `python scripts/03_split_analysis.py` | Leakage PASS 4 / FAIL 1 (136 vs 135 경계값 · 정상) |
| 6 | `python scripts/04_make_split.py` | 합격 기준 8 / 8 PASS · Frozen Test 지문 `4f9291849fe6` |
| 7 | `git status --short` | **아무것도 안 나오면** 내 결과 = 팀 기준 결과 (확인 후 `git restore reports/ splits/`) |

## 7. 폴더 구조

```
prj08/
├── README.md                          # 산출물 7 · 이 문서
├── requirements.txt                   # pillow · pyyaml · ultralytics==8.4.175 · pandas
├── .venv/                             # ⚠ Git 제외 · prj08 전용 가상환경
├── yolo11n.pt                         # ⚠ Git 제외 (*.pt) · YOLO Load 때 자동 다운로드된 COCO 사전학습 모델
│
├── configs/
│   └── dataset.yaml                   # 산출물 1 · path: data/yolo · Class ID 0~6 영문 이름 · Bridge 미포함
│
├── src/                               # 재사용 "기능" (혼자 실행하지 않음)
│   ├── __init__.py                    # src를 패키지로 인식시키는 빈 파일
│   ├── dataset_audit.py               # Pair · YOLO 형식 · Class · 좌표 · Manifest 검사 함수
│   └── capture_group.py               # 파일명 → 촬영 시각 · 촬영 세션(capture_group) 묶기 · 시간 차 계산
│
├── scripts/                           # 실제 "실행" 파일 (번호 = 실행 순서)
│   ├── 01_check_environment.py        # 1일차 · Python / PyTorch / Ultralytics / GPU · YOLO11n Load
│   ├── 02a_make_audit_fixture.py      # 2일차 · 버그 9종을 심은 테스트 데이터 생성 (검사기 검증용)
│   ├── 02_audit_subject7_final.py     # 2일차 · 900장 Audit + 기대값 19항목 대조 (--fixture: 테스트 모드)
│   ├── 03_split_analysis.py           # 2일차 · Class 분포 · EXIF · 1/5/10분 세션 비교 · Leakage 재확인
│   └── 04_make_split.py               # 3일차 선행 · 정책대로 전수 탐색 배정 + 합격 기준 8개 검증
│
├── docs/
│   ├── split_policy.md                # 산출물 1 · Split 정책 확정본 (5분 세션 · 600/75/75/150 · 예외 E1)
│   ├── code_flow.md                   # 코드 흐름 Mermaid 6개
│   └── code_review/                   # 코드와 같은 구조의 파일별 리뷰
│       ├── README.md                  # 목차 · 읽는 순서 · 우선순위
│       ├── configs/                   # dataset_yaml.md
│       ├── src/                       # dataset_audit.md · capture_group.md
│       └── scripts/                   # 01 · 02a · 02 · 03 · 04 리뷰
│
├── reports/                           # 결과 · 기록 (파일명 · 숫자만 · Git에 올림)
│   ├── figures/                       # (예정) 학습 그래프 · Confusion Matrix 등 산출물 5
│   ├── audit_expected.md              # Audit 전에 고정한 기대값 (Test Oracle · 교과 7 문서 기준)
│   ├── audit_issue.csv                # 900장 문제 목록 (0건 → 제목 줄만)
│   ├── audit_check.csv                # 기대값 vs 실제 19항목 (PASS 14 / FAIL 5)
│   ├── audit_label_stats.csv          # 이미지별 BBox · Class 수 · 출처 · split · scene_type → 03 · 04의 입력
│   ├── class_scene_distribution.csv   # 출처 × split × scene_type별 이미지 · Class 수
│   ├── filename_time_check.csv        # 파일명 시각 vs EXIF (EXIF 없음)
│   ├── capture_group_compare.csv      # 1 · 5 · 10분 기준 세션 수 비교 (106 · 18 · 13)
│   ├── capture_groups.csv             # 이미지별 촬영 시각 + 1/5/10분 세션 번호 → 04의 입력
│   ├── capture_group_summary.csv      # 5분 세션 G01~G18 상세 (시각 · 이미지 · Class 수)
│   ├── split_check.csv                # 배정 합격 기준 8개 PASS / FAIL
│   ├── split_summary.csv              # 역할별 이미지 · BBox · Class · 날짜 · 세션 (600 / 74 / 80 / 146)
│   └── issue_log.md                   # 문제 · 원인 · 해결 기록
│
├── splits/
│   └── kimchi_v1_split.csv            # 파일별 역할 (file_name · capture_group · role) · 3일차에 manifests/로 변환 예정
│
├── daily_csv/
│   └── 10_12.csv                      # 일일 작업 기록
│
├── manifests/                         # (3일차) split_manifest.csv · train · val · test · bridge_holdout.csv
├── demo/                              # (6 · 8일차) 실패 사례 · Prediction · Auto Label 결과 (일부 Git 제외)
├── presentation/                      # (9일차) 발표 PDF · Q&A
├── models/                            # ⚠ Git 제외 · baseline/ · improved/ · final/
├── runs/                              # ⚠ Git 제외 · Ultralytics 학습 결과
│
└── data/                              # ⚠ Git 제외 (NDA)
    ├── subject7_final/                # 교과 7 FINAL 복사본 · 읽기 전용 · 라벨 지문 5c2b4fc2c3e6
    │   ├── images/*.jpg               # 900장 (파일명 = 촬영 시각 YYMMDD_HHMMSS + 마이크로초)
    │   └── labels/*.txt               # 900개 (class x y w h)
    └── audit_test/                    # 02a가 만든 Fixture (8쌍 + 버그 9종) · 언제든 다시 생성 가능
```

> Git은 빈 폴더를 저장하지 않습니다. `manifests/`, `demo/`, `presentation/`, `models/` 하위 폴더는 각자 PC에만 있고, 파일이 생기는 날 Git에 올라갑니다.

## 8. 실행 순서와 데이터 흐름

```
[1일차]  01_check_environment.py ──→ 화면 출력 → 5장 팀원 환경 표

[2일차]  02a_make_audit_fixture.py ──→ data/audit_test/ (8쌍 + 버그 9종)
              │
              ▼
         02_audit_subject7_final.py --fixture ──→ 9종 각 1건 검출 확인   (검사기부터 검사)
         02_audit_subject7_final.py           ──→ reports/audit_issue · audit_check · audit_label_stats.csv
              │  (import src.dataset_audit)            │
              │                                         ▼
         03_split_analysis.py ←──────────── audit_label_stats.csv
              │  (import src.capture_group)
              ▼
         reports/class_scene_distribution · capture_group_* · capture_groups · filename_time_check.csv
              │
              ▼
         [사람] 팀 회의 → docs/split_policy.md (숫자 근거 + 결정)

[3일차 선행]
         04_make_split.py ←── capture_groups.csv + audit_label_stats.csv + 정책 상수
              ▼
         splits/kimchi_v1_split.csv · reports/split_check.csv · split_summary.csv · Frozen Test 지문 4f9291849fe6
```

| 파일 | 입력 | 출력 | 의존 |
|---|---|---|---|
| `01_check_environment.py` | 현재 Python 환경 | 화면 | torch · ultralytics |
| `02a_make_audit_fixture.py` | `data/subject7_final` 앞 8쌍 | `data/audit_test/` | shutil |
| `02_audit_subject7_final.py` | images · labels · 교과 7 Manifest · `EXPECTED` | `reports/audit_*.csv` | `src.dataset_audit` |
| `03_split_analysis.py` | `audit_label_stats.csv` · images(EXIF) | `reports/` CSV 5개 | `src.capture_group` · PIL |
| `04_make_split.py` | `capture_groups.csv` · `audit_label_stats.csv` | `splits/` · `reports/split_*.csv` | pandas만 (src 의존 없음) |

> 함수 단위 설명과 리뷰는 [`docs/code_review/README.md`](docs/code_review/README.md), 그림은 [`docs/code_flow.md`](docs/code_flow.md)를 참고합니다.

## 9. Dataset Split

**상태**: 정책 확정 (2026-10-12 · 팀 회의) · 배정 선행 완료 · **3일차에 Freeze** (`manifests/` 생성)

| 역할 | 목표 | 현재 배정 | 용도 |
|---|---:|---:|---|
| Train | 600 | 600 | 학습 (Baseline · Improved) |
| Val | 75 | 74 | 모델 비교 · 개선 실험 |
| Frozen Test | 75 | 80 | Final Model 선택 후 **마지막에 한 번** |
| Bridge Holdout | 150 | 146 | 교과 14 연결용 보존 · 교과 8 사용 금지 · `dataset.yaml` 미포함 |

- **배정 단위**: 5분 촬영 세션(capture_group) 18개 · 한 세션은 한 역할에만 (기존 validation 180장은 전부 train과 같은 세션 → 그대로 사용하지 않음)
- **object_only 150장(G18)**: 한 세션이라 전부 Train
- **예외 E1**: 파·고추 252개 중 251개가 G01에만 있어 G01만 2분 하위 세션으로 분할 배정 (Class 6 점수는 참고값)
- **검증**: 합격 기준 8 / 8 PASS (`reports/split_check.csv`) · Frozen Test 지문 `4f9291849fe6`
- 근거와 결정 기록: `docs/split_policy.md`

## 10. Baseline

TBD (4일차)

## 11. Failure Analysis

TBD (6일차)

## 12. Improvement

TBD (7일차)

## 13. Final Model

TBD (8일차)

## 14. Frozen Test

TBD (8일차 · Final Model 선택 후 한 번만)

## 15. Detection / Auto Label

TBD (8일차)

## 16. 알려진 한계

- **정상 김치(Negative) 0장** → 정상 김치를 이물로 잘못 찾는(FP) 경향을 평가할 수 없음 · 5~6일차 Prediction에서 확인
- **object_only 150장은 전부 Train** → "이물만 있는 장면"의 성능은 평가하지 않음
- **Class 6(파·고추)은 G01 한 세션에만 존재** → 예외 E1로 분할 · Class 6 점수는 참고값
- **Val이 Class 1 중심** (BBox 389개 중 351개) → 모델 비교 때 전체 mAP와 함께 Class별 AP 확인
- **Frozen Test 80장** → 작아서 점수 변동이 클 수 있음
- **촬영 조건이 단조로움** (4일 · 같은 트레이 · 같은 조명) → 다른 환경에서의 성능은 이 데이터로 알 수 없음

## 17. 데이터 보안

- 기업 제공 데이터(NDA)는 Git · 디스코드 · 외부 클라우드에 올리지 않는다. (`data/`, `*.jpg`, `*.pt`, `runs/`, `models/` 제외)
- **보안정책이 `.gitignore`보다 우선**한다 (가이드 22.2) · Public 레포 사용 가능 여부는 선생님 확인 중
- 커밋 전 확인 (아무것도 안 나와야 정상):

```bash
git diff --cached --name-only | grep -iE '\.(jpe?g|png|pt)$|/data/'
```

## 18. 교과 14 Handoff

TBD (9일차 · `docs/handoff_subject14.md`)

## 참고 문서

| 문서 | 내용 |
|---|---|
| [`docs/split_policy.md`](docs/split_policy.md) | Split 정책 · 근거 · 결정 기록 |
| [`docs/code_review/README.md`](docs/code_review/README.md) | 파일별 코드 리뷰 (코드와 같은 폴더 구조) |
| [`docs/code_flow.md`](docs/code_flow.md) | Mermaid 코드 흐름도 |
| `reports/audit_expected.md` | Audit 기대값 (Test Oracle) |
| `reports/issue_log.md` | 문제 · 원인 · 해결 기록 |
