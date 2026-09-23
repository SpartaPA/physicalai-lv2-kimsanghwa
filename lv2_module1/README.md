# 모듈 1 — 임베디드 제어 기초

## 장비 및 환경
- 라즈베리파이 Ubuntu Server 22.04.5 LTS(arm64), OpenCR, 다이나믹셀 XM430-W210 (ID 1)
- PC는 SSH 접속 용도로만 사용, 업로드·시리얼 송수신·로그 저장은 라즈베리파이에서 직접 수행

## 실행 방법
1. `opencr_position_p.ino` (수정본)를 arduino-cli로 라즈베리파이에서 빌드
2. `opencr_ld` 업로더로 OpenCR에 업로드
3. `python3 -m serial.tools.miniterm /dev/ttyACM0 115200 --eol LF -e`로 접속
4. `s <Kp> <speed_deg_s> <angle_deg>` 입력하여 실행, `x`로 정지

## 결과 파일 위치
- `results/환경확인.txt`, `results/upload.log`, `results/실행A.log`
- 상세 해석: [`report.md`](report.md)