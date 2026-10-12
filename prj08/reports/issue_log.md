# Issue Log

| 날짜 | 문제 | 원인 | 해결 |
|---|---|---|---|
| 10/12 | prj07 가상환경 activate 실패 | `.venv` 손상 (activate 없음) | `prj07/.venv` 새로 생성 |
| 10/12 | .gitignore 일부 규칙 미동작 | `echo` 명령이 파일에 그대로 기록됨 | 규칙 수정 + prj08 차단 구역 추가 |
| 10/12 | 터미널 `code` 명령 실패 (ENOENT .sock) | 복원된 옛 터미널의 연결 정보 | Developer: Reload Window |
| 10/12 | Ultralytics config not writable 경고 | `~/.config` 폴더 없음 | `mkdir -p ~/.config` |
| 10/12 | Audit: 교과 7 문서 Class 수(4,602)와 FINAL 라벨(4,601) 불일치 | 10/07 QA 이후 10/08 라벨 추가 수정(Class 2→0 56개 등)이 교과 7 문서에 반영 안 됨. 이승훈 10/08 zip = master = subject7_final (diff 0) | 데이터 수정 없음 · 현재 FINAL(4,601)을 kimchi_v1 후보로 팀 확정 필요 · 근거 reports/audit_check.csv |
| 10/12 | 팀원 데이터 버전 불일치 해결 | 10/07본(4,602)과 10/08 수정본(4,601) 혼재 | kimchi_v1 = 10/08 수정본(4,601) 확정 · 5명 라벨 sha256 지문 일치 확인 |
| 10/12 | Leakage 60초 이내 validation 기대 136 / 실제 135 | 1장이 가장 가까운 train과 60.28초 차이 · 교과 7은 초 단위로 잘라 계산(포함) · 교과 8 코드는 마이크로초까지 계산(제외) | 기준 정밀도 차이 · 결론(Leakage 높음)에는 영향 없음 · 기대값 유지 |