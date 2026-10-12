# 교과 8 · 코드 흐름도 (Mermaid)

> 기준: `main` · PR #73 (2026-10-12) · `prj08/`
> GitHub에서 이 파일을 열면 그림으로 보입니다. VS Code는 "Markdown Preview Mermaid Support" 확장을 설치하면 미리보기에서 보입니다.

**범례**: 원통 = 데이터(읽기 전용) · 육각형 = `src` 기능 · 사각형 = `scripts` 실행 파일 · 평행사변형 = 결과 파일 · 둥근 사각형 = 사람의 결정 · 점선 = import / 규칙 반영

---

## 1. 전체 파이프라인 (파일 단위)

```mermaid
flowchart LR
    subgraph IN["② 입력 · 교과 7 FINAL (Git 제외 · 읽기 전용)"]
        IMG[("data/subject7_final/images<br/>JPG 900장")]
        LBL[("data/subject7_final/labels<br/>TXT 900개 · 지문 5c2b4fc2c3e6")]
        MAN[("prj07/manifests/<br/>dataset_manifest.csv")]
    end

    EXP(["reports/audit_expected.md<br/>기대값 · Test Oracle"])
    YAML[/"configs/dataset.yaml<br/>Class 0~6 · Bridge 제외"/]

    subgraph D1["① 1일차 · 환경"]
        S01["scripts/01_check_environment.py"]
        ENV[/"README 팀원 환경 표"/]
    end

    subgraph AUD["③ 2일차 · Audit"]
        SRC1{{"src/dataset_audit.py"}}
        S02A["scripts/02a_make_audit_fixture.py"]
        FIX[("data/audit_test<br/>8쌍 + 버그 9종")]
        S02["scripts/02_audit_subject7_final.py"]
        O02[/"audit_issue.csv<br/>audit_check.csv<br/>audit_label_stats.csv"/]
    end

    subgraph ANA["④ 2일차 · 분포 · Leakage"]
        SRC2{{"src/capture_group.py"}}
        S03["scripts/03_split_analysis.py"]
        O03[/"class_scene_distribution.csv<br/>capture_group_compare.csv<br/>capture_groups.csv<br/>capture_group_summary.csv<br/>filename_time_check.csv"/]
    end

    subgraph DEC["⑤ 사람의 결정"]
        POL(["docs/split_policy.md<br/>5분 세션 · 600/75/75/150 · E1"])
        LOG(["reports/issue_log.md"])
    end

    subgraph SPL["⑥ 3일차 선행 · 배정"]
        S04["scripts/04_make_split.py"]
        O04[/"splits/kimchi_v1_split.csv<br/>split_check.csv · split_summary.csv<br/>Frozen Test 지문 4f9291849fe6"/]
    end

    NEXT["3일차 남은 일<br/>manifests/*.csv · data/yolo · Smoke Test"]

    S01 --> ENV

    IMG --> S02A
    LBL --> S02A
    S02A --> FIX
    FIX -->|"fixture 모드"| S02

    IMG --> S02
    LBL --> S02
    MAN --> S02
    SRC1 -.->|import| S02
    EXP -.->|EXPECTED 19항목| S02
    S02 --> O02
    O02 -.->|FAIL 5 원인| LOG

    O02 -->|audit_label_stats| S03
    IMG -->|EXIF 확인| S03
    SRC2 -.->|import| S03
    S03 --> O03

    O03 -.->|숫자 근거| POL
    POL -.->|정책 → 상수| S04
    O03 -->|capture_groups| S04
    O02 -->|audit_label_stats| S04
    S04 --> O04
    O04 -.-> NEXT
    YAML -.-> NEXT

    classDef data fill:#E9ECE8,stroke:#9AA3AE,color:#1B2430
    classDef src fill:#FFFFFF,stroke:#2a78d6,stroke-width:2px,color:#1B2430
    classDef script fill:#1B2430,stroke:#1B2430,color:#FFFFFF
    classDef out fill:#FFFFFF,stroke:#9AA3AE,stroke-dasharray:4 3,color:#1B2430
    classDef human fill:#FBF1E6,stroke:#C26A1E,stroke-width:2px,color:#1B2430
    classDef next fill:#F3F4F1,stroke:#B8C2CF,stroke-dasharray:4 3,color:#6B7280

    class IMG,LBL,MAN,FIX data
    class SRC1,SRC2 src
    class S01,S02A,S02,S03,S04 script
    class O02,O03,O04,ENV,YAML out
    class EXP,POL,LOG human
    class NEXT next
```

---

## 2. Audit — `02_audit_subject7_final.py` → `src/dataset_audit.py`

```mermaid
flowchart TD
    A["02 main()"] --> B{"--fixture ?"}
    B -->|예| C["data/audit_test<br/>Manifest 없음 · 기대값 대조 안 함"]
    B -->|아니오| D["data/subject7_final<br/>+ 교과 7 Manifest"]
    C --> E
    D --> E

    subgraph AUDIT["src/dataset_audit.py"]
        E["audit_dataset(images, labels, manifest)"]
        E --> F["stem으로 짝 맞추기<br/>images & labels → pairs<br/>images - labels → MISSING_LABEL<br/>labels - images → MISSING_IMAGE"]
        F --> G["TXT 한 줄씩 반복"]
        G --> H["validate_yolo_row(row)"]
        H --> H1{"5칸?"}
        H1 -->|아니오| X1["COLUMN_COUNT"]
        H1 -->|예| H2{"숫자 변환?"}
        H2 -->|실패| X2["PARSE_ERROR"]
        H2 -->|성공| H3["Class 0~6 · Class 4 · 좌표 0~1 · 박스 경계 검사"]
        H3 --> I["_issue() 로 문제 기록<br/>ERROR / WARN"]
        X1 --> I
        X2 --> I
        G --> J["형식이 정상인 줄만 Class 개수에 더함"]
        E --> K["read_manifest()<br/>NOT_IN_MANIFEST · MISSING_IN_DATASET"]
        I --> L[/"issues · stats · image_rows"/]
        J --> L
        K --> L
    end

    L --> M{"fixture?"}
    M -->|예| N[/"data/audit_test/audit_issue.csv<br/>9종 각 1건 확인"/]
    M -->|아니오| O["compare_with_expected(stats, EXPECTED)"]
    O --> P["write_csv() × 3"]
    P --> Q[/"audit_issue.csv · audit_check.csv · audit_label_stats.csv"/]
```

---

## 3. 세션 묶기 — `src/capture_group.py`

```mermaid
flowchart LR
    A["파일명<br/>250410_180800068444.jpg"] --> B["parse_capture_time()<br/>YYMMDD + HHMMSS + 마이크로초"]
    B --> C["촬영 시각 (datetime)"]
    C --> D["assign_capture_groups(times, gap)"]

    subgraph G["assign_capture_groups 내부"]
        D1["sort_values()<br/>시각 순 정렬"] --> D2["diff()<br/>앞 사진과의 시간 차"]
        D2 --> D3["gap > 5분 ?<br/>True = 새 세션 시작"]
        D3 --> D4["cumsum()<br/>True 누적합 = 세션 번호"]
        D4 --> D5["reindex()<br/>원래 행 순서로"]
    end

    D --> D1
    D5 --> E["세션 번호 1~18<br/>(5분 기준)"]
    C --> F["nearest_gap_seconds(val, train)<br/>가장 가까운 train까지 초"]
    F --> H["60초 이내 135장 · 최소 8초"]
```

---

## 4. 분석 — `scripts/03_split_analysis.py`

```mermaid
flowchart TD
    A[/"reports/audit_label_stats.csv"/] --> B["main()"]
    B --> C["parse_capture_time() 로 시각 · 날짜 컬럼 추가"]
    C --> D["class_scene_distribution()"]
    C --> E["filename_time_check()<br/>+ read_exif_time()"]
    C --> F["assign_capture_groups() × 1 · 5 · 10분"]
    F --> G["compare_gaps()"]
    F --> H["group_summary(5분)"]
    C --> I["nearest_gap_seconds()<br/>+ EXPECTED_LEAK 대조"]

    D --> D1[/"class_scene_distribution.csv"/]
    E --> E1[/"filename_time_check.csv<br/>EXIF 없음"/]
    G --> G1[/"capture_group_compare.csv<br/>106 · 18 · 13"/]
    F --> F1[/"capture_groups.csv"/]
    H --> H1[/"capture_group_summary.csv<br/>G01~G18"/]
    I --> I1["화면: PASS 4 / FAIL 1<br/>136 vs 135 경계값"]
```

---

## 5. 배정 — `scripts/04_make_split.py`

```mermaid
flowchart TD
    A[/"capture_groups.csv"/] --> L["load_images()<br/>두 CSV 합치기 · 900행 · 결측 확인"]
    B[/"audit_label_stats.csv"/] --> L
    L --> E1["split_e1()<br/>G01 → 2분 하위 세션 S1~S5"]
    E1 --> E2{"하위 세션 5개?<br/>(정책 표와 같나)"}
    E2 -->|아니오| STOP["ValueError로 멈춤<br/>문서와 코드 불일치"]
    E2 -->|예| U["배정 단위 표 (units)<br/>세션별 이미지 · 날짜 · Class 수"]

    U --> FX["고정 배정<br/>G18 → train<br/>G01-S1 · S2 → test · S3 → bridge<br/>S4 → val · S5 → train"]
    FX --> S["search(units, fixed)"]

    subgraph SEARCH["search() 전수 탐색"]
        S1["subsets_in_range()<br/>Test 후보 (75 ± 10)"] --> C1{"Class ≥ 20<br/>날짜 ≥ 2"}
        C1 -->|통과| S2["Val 후보 (75 ± 10)"]
        S2 --> C2{"Class ≥ 1<br/>날짜 ≥ 2"}
        C2 -->|통과| S3["Bridge 후보 (150 ± 10)"]
        S3 --> C3{"Class ≥ 20<br/>날짜 ≥ 2"}
        C3 -->|통과| S4["나머지 = Train"]
        S4 --> C4{"600 ± 10<br/>Class ≥ 100"}
        C4 -->|통과| OK["후보 목록에 추가"]
    end

    S --> S1
    OK --> SORT["정렬: 목표 차이 → 평가 Class 최소 → 이름 순<br/>5,606개 중 1순위"]
    SORT --> R["search() 한 번 더 → 같은 1순위인지<br/>(재현성)"]
    R --> V["verify(df)<br/>파일 900개 단위로 다시 검증 · 기준 8개"]
    V --> FP["fingerprint()<br/>Frozen Test 파일명 sha256"]
    FP --> O1[/"splits/kimchi_v1_split.csv"/]
    FP --> O2[/"reports/split_check.csv · 8 / 8 PASS"/]
    FP --> O3[/"reports/split_summary.csv · 600 / 74 / 80 / 146"/]
```

---

## 6. 모듈 의존 관계 (누가 누구를 import 하나)

```mermaid
flowchart LR
    subgraph SRC["src/ · 기능"]
        DA{{"dataset_audit.py"}}
        CG{{"capture_group.py"}}
    end
    subgraph SCR["scripts/ · 실행"]
        S01["01_check_environment.py"]
        S02A["02a_make_audit_fixture.py"]
        S02["02_audit_subject7_final.py"]
        S03["03_split_analysis.py"]
        S04["04_make_split.py"]
    end
    subgraph LIB["외부 라이브러리"]
        T["torch · ultralytics"]
        PD["pandas · numpy"]
        PIL["Pillow"]
        STD["pathlib · csv · shutil · hashlib · itertools"]
    end

    S01 --> T
    S02 -.->|import| DA
    S03 -.->|import| CG
    S03 --> PIL
    S03 --> PD
    S04 --> PD
    S04 --> STD
    S02A --> STD
    DA --> STD
    CG --> PD
```

> `04_make_split.py`는 `src`를 import하지 않고 **결과 CSV 두 개만** 읽습니다. 그래서 앞 단계 코드가 바뀌어도 CSV가 같으면 배정 결과가 같습니다.
