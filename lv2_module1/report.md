## 문제 1

### 환경 및 설정
- Raspberrypi : Ubuntu Server 22.04.5 LTS, arm64
- 제어기 : OpenCR 보드
- 모터 : XM430-W210(모델명 1030), ID 1(기존 과제의 제공코드는 모델명 XM430-W350, ID 12번 기준이었지만 바퀴 제어를 위해 변경)
- 사용예제 : 'examples/opencr_position_p/opencr_position_p.ino'
- 입력 프롬프트 : s kp speed angle

### 수정 사항
- 모터 ID : 12 -> 1
- 모델명 : XM430-W210 -> 1030

### 증거
- 환경 확인 : [results/환경확인.txt]
- 업로드 로그 : [results/upload.log]
- 실행 A 로그 : [results/실행A.log]

### 값 구분 및 단위
| 항목 | 값 | 단위 |
|---|---|---|
| 목표값 (target_deg) | 90 | ° |
| 측정값 (position_deg, 최종) | 89.824 | ° |
| 제어 출력 (u_deg_s, 정지 직전) | 0.000 | °/s |
| 오차 (error_deg, 최종) | 0.176 | ° |

### 결과 설명
- 목표각 90°에 대해 약 5.2초 후 position_deg가 89.824°까지 도달했고 error_deg가 데드밴드(0.2°) 이내로 줄어들며 정지했다. 종료 시 xSTOP: user 로그가 출력되며 모터의 작동이 정지하는것을 확인했다.