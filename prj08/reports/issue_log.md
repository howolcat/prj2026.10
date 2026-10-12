# Issue Log

| 날짜 | 문제 | 원인 | 해결 |
|---|---|---|---|
| 10/12 | prj07 가상환경 activate 실패 | `.venv` 손상 (activate 없음) | `prj07/.venv` 새로 생성 |
| 10/12 | .gitignore 일부 규칙 미동작 | `echo` 명령이 파일에 그대로 기록됨 | 규칙 수정 + prj08 차단 구역 추가 |
| 10/12 | 터미널 `code` 명령 실패 (ENOENT .sock) | 복원된 옛 터미널의 연결 정보 | Developer: Reload Window |
| 10/12 | Ultralytics config not writable 경고 | `~/.config` 폴더 없음 | `mkdir -p ~/.config` |