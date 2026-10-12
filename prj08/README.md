# 교과 8 · 조각김치 이물 객체검출 (YOLO11)

## 프로젝트 목적
교과 7에서 검수한 정답 데이터로 YOLO11 객체검출 모델을 학습·평가·개선한다.

## 교과 7 → 교과 8 연결
- 교과 7 FINAL: `data/subject7_final/` (원본: master_data/final, Git 미포함, 읽기 전용)
- JPG 900장 / YOLO TXT 900개 확인

- scene_type 분포: kimchi_with_target 750장 / object_only 150장 (normal_kimchi 0장, other_review 0장)
- Empty Label: 0개
- Class 4(고무장갑): 0건 (교과 7에서 enabled: false)
- QA 상태: 900장 전체 QA PASS
- 교과 7 QA 결과: `prj07/reports/qa_summary.md` 참고

## Dataset 설명
| source_dataset | original_split | 수량 |
|---|---|---|
| 이물검출_학습데이터1 | train | 500 |
| 이물검출_학습데이터2 | train | 220 |
| 이물검출_학습데이터2 | validation | 180 |
| **합계** | | **900** |

## Class
| ID | 이물 종류 | 영문 이름 | 사용 |
|---|---|---|---|
| 0 | 나뭇잎·종이류 | leaf_paper | O |
| 1 | 플라스틱류·돌·금속류 | plastic_stone_metal | O |
| 2 | 나뭇가지류 | branch | O |
| 3 | 벌레류 | insect | O |
| 4 | 고무장갑 | rubber_glove_unused | X (발견 시 REVIEW 확인) |
| 5 | 병해·갈변 | disease_browning | O |
| 6 | 파·고추 | green_onion_pepper | O |

## 실행환경
| 항목 | 값 |
|---|---|
| Python | 3.10.12 |
| PyTorch | 2.14.1+cu130 |
| Ultralytics | 8.4.175 |
| GPU | NVIDIA GeForce RTX 3060 (학교 PC) |
| YOLO11n Load | 성공 |

환경 확인: `cd prj08` → `python scripts/01_check_environment.py`

## 데이터 보안
- 기업 제공 데이터(NDA)는 Git에 올리지 않는다. (`data/`, `*.jpg`, `*.pt`, `runs/` 제외)

## Dataset Split
TBD
## Baseline
TBD
## Failure Analysis
TBD
## Improvement
TBD
## Final Model
TBD
## Frozen Test
TBD
## Detection / Auto Label
TBD
## 알려진 한계
TBD
## 교과 14 Handoff
TBD

## 알려진 한계
- 교과 7 FINAL에 normal_kimchi(정상 김치, Negative Sample)가 0장 → 정상 김치 FP 가능성, 5~6일차에 확인