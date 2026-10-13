# physicalai-lv2-kimsanghwa

김상화의 Physical AI Lv2 개인 과제 제출 저장소

| 항목 | 내용 |
| --- | --- |
| 이름 | 김상화 |
| GitHub | [SangHwaKim09](https://github.com/SangHwaKim09) |
| 저장소 | `SpartaPA/physicalai-lv2-kimsanghwa` |

## 모듈별 제출

| 모듈 | 주제 | 보고서 | 실행·재현 안내 |
| --- | --- | --- | --- |
| 1 | 임베디드 제어 기초 — OpenCR·DYNAMIXEL 위치 P 제어 | [lv2_module1/report.md](lv2_module1/report.md) | [lv2_module1/README.md](lv2_module1/README.md) |
| 3 | 컴퓨터 비전 + 카메라 모델 — 설정값에 따른 Mask와 검출 결과 변화 분석 (19·21강) | [lv2_module3/report.md](lv2_module3/report.md) | [lv2_module3/README.md](lv2_module3/README.md) |

## 모듈 3 보고서

**[lv2_module3/report.md](lv2_module3/report.md)**

- 내용: Raw Image·Mask·검출 결과를 ROS 2 토픽으로 받고, 설정값(HSV 경계값, 최소 면적)을 바꿨을 때 Mask와 검출 결과가 어떻게 달라지는지 비교
- 결과 파일: [lv2_module3/results/](lv2_module3/results/) — 토픽 정보, 실험별 A·B 수신 이미지·검출 로그, 목표 없음 확인 결과
- 코드 출처·실행 환경·설정값 변경 방법: [lv2_module3/README.md](lv2_module3/README.md)

## 저장소 구조

```text
physicalai-lv2-kimsanghwa/
├── README.md            ← 이 파일 (이름, 모듈별 보고서 링크)
├── lv2_module1/         ← 모듈 1: 임베디드 제어 기초
│   ├── README.md
│   ├── report.md
│   └── results/
└── lv2_module3/         ← 모듈 3: 컴퓨터 비전 + 카메라 모델
    ├── README.md
    ├── report.md
    └── results/
```
