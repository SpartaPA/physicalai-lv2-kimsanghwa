# 모듈 3 — 컴퓨터 비전 + 카메라 모델

19·21강 과제: 설정값에 따른 Mask와 검출 결과 변화 분석. 보고서: [report.md](report.md)

## 1. 코드 출처

새 코드를 만들지 않고, 팀 프로젝트(Lv2 모듈 5)의 인지 노드를 재사용했다.

| 항목 | 내용 |
| --- | --- |
| 저장소 | [SpartaPA/Lv2_EYE4_Assignment](https://github.com/SpartaPA/Lv2_EYE4_Assignment) — 브랜치 `dev/sanghwa` |
| 커밋 | `1e03283` (2026-10-12, "RViz에서 마스크와 color_depth를 볼 수 있도록 노드 추가") |
| 패키지 | `lv2_module5/ros2_ws/src/realsense_tracker` |
| 인지 노드 | `realsense_tracker/perception_node.py` — 컬러 영상 구독 → 검출 → 토픽 발행 |
| 검출 함수 | `realsense_tracker/detector.py` — `make_mask()`, `detect()`, `draw()` |
| 설정 파일 | `config/tracker.yaml`(HSV·최소 면적 등), `config/camera.yaml`(카메라 토픽 이름·해상도) |
| 실행 파일 | `lv2_module5/tools/camera_view.launch.py`(카메라 + 인지 노드 + RViz), `rviz/tracking_view.rviz` |
| 실험 도구 | `lv2_module5/tools/hsv_tuning.py` — 트랙바로 HSV·최소 크기를 바꾸며 같은 `detect()` 결과를 보는 도구 |

`/perception/mask` 토픽은 원래 코드에 없어서, 기존 `make_mask()` 결과를 그대로 발행하는 부분만 `perception_node.py`에 추가했다(위 커밋 `1e03283`).
Mask를 만드는 방법(HSV 범위·블러·모폴로지)과 `/target` 이름·형식·QoS는 바꾸지 않았다.

## 2. 실행 환경

| 항목 | 버전·값 |
| --- | --- |
| OS | Ubuntu 24.04.5 LTS (노트북) |
| ROS 2 | Lyrical (`ros-lyrical-ros-base` 0.13.0) |
| Python | 3.12.3 |
| OpenCV | 4.6.0 (ROS 노드가 쓰는 시스템 Python) / 5.0.0 (`hsv_tuning.py`를 실행한 `lv2_module5/.venv`) |
| 카메라 | Intel RealSense D435 |
| 카메라 드라이버 | `realsense2_camera` 4.57.7 (ROS wrapper) / `pyrealsense2` 2.58.4 (`.venv`, 실험 도구용) |
| 입력 | **실시간 카메라** (rosbag 아님). 컬러 640×480 @30 fps, `rgb8` |
| 장면 | 카메라 앞 노트북 화면에 띄운 색 도형 이미지(파란 링 = 목표, 남색 별·청록 사각형·하늘색 육각형·보라 하트 등 = 비슷한 색 방해물) |

## 3. 실행 방법

ROS 2 환경과 EYE4 워크스페이스를 source한 터미널에서 실행한다.

```bash
source /opt/ros/lyrical/setup.bash
source ~/Lv2_EYE4_Assignment/lv2_module5/ros2_ws/install/setup.bash
```

**토픽 확인 (보고서 1~4절)**

```bash
# 터미널 1: 카메라 wrapper + 인지 노드(publish_debug_image: true) + RViz
ros2 launch ~/Lv2_EYE4_Assignment/lv2_module5/tools/camera_view.launch.py

# 터미널 2: 토픽 이름·타입, 발행·구독 노드와 QoS
ros2 topic list -t
ros2 topic info /camera/camera/color/image_raw --verbose
ros2 topic info /perception/mask --verbose
ros2 topic info /target --verbose

# 검출 결과 메시지 (발행 QoS가 best effort라 구독도 best effort로)
ros2 topic echo /target --qos-reliability best_effort
ros2 interface show geometry_msgs/msg/PointStamped
```

RViz 패널: `Camera color (raw)` = `/camera/camera/color/image_raw`, `Mask` = `/perception/mask`, `Detection` = `/perception/debug_image`.
Raw·Detection 영상(장당 약 0.9 MB)은 best effort 구독에서 거의 들어오지 않아 reliable로 구독하는 RViz로 확인했다(측정값은 `results/*_토픽정보.txt`).

**설정값 비교 실험 (보고서 5절)**

```bash
# camera_view.launch.py를 끈 상태에서 (카메라는 한 프로그램만 열 수 있음)
cd ~/Lv2_EYE4_Assignment/lv2_module5
.venv/bin/python tools/hsv_tuning.py
```

`hsv_tuning.py`는 ROS를 거치지 않고 `pyrealsense2`로 카메라를 직접 열어, `detector.py`의 같은 `detect()`·`draw()` 결과(왼쪽)와 Mask(오른쪽)를 한 창에 보여 준다.

## 4. 설정값 변경·적용 방법

| 설정 | 위치 | 코드에서 쓰는 곳 |
| --- | --- | --- |
| `hsv_lower`, `hsv_upper` (HSV 범위) | `config/tracker.yaml` | `detector.py` `make_mask()` 55행 `cv2.inRange(hsv, lower, upper)` |
| `blur_ksize`, `morph_ksize` | `config/tracker.yaml` | `make_mask()` 50행 GaussianBlur, 60~61행 열기·닫기 |
| `min_area_ratio` (최소 면적 / 화면 넓이) | `config/tracker.yaml` | `detect()` 106~107행 `contourArea >= min_area_ratio × W × H`인 컨투어만 후보 |
| 목표 선택 | (설정 없음) | `detect()` 114행 후보 중 면적이 가장 큰 1개 |

- **ROS 노드**: `perception_node`는 **시작할 때만** 파라미터를 읽는다(`declare_parameter`, 실행 중 변경 콜백 없음). `tracker.yaml`을 고친 뒤 launch를 다시 실행해야 적용된다.
- **실험 도구**: `hsv_tuning.py`의 트랙바로 바꾸면 다음 프레임부터 바로 적용된다. 최소 크기 트랙바 단위는 화면의 1/10000(20 = `min_area_ratio` 0.002, 80 = 0.008). `s` 키를 누르면 `tracker.yaml`의 `hsv_lower`·`hsv_upper`·`min_area_ratio`에 저장된다.

처리 흐름: `/camera/camera/color/image_raw` 수신(`on_color`, 172행) → `make_mask()`(블러 → HSV → `inRange` → 열기·닫기) → `detect()`(컨투어 → 최소 면적 필터 → 최대 면적 1개 → 중심·면적비) → `/target`(200행), `/perception/debug_image`(203행), `/perception/mask`(205행) 발행

## 5. 결과 파일 (`results/`)

| 파일 | 내용 | 보고서 |
| --- | --- | --- |
| `원본영상_토픽확인_rviz.png` | RViz `Camera color (raw)` 패널로 받은 원본 영상 | 2절 |
| `원본영상_토픽정보.txt` | Raw 토픽 이름·타입·발행 노드·QoS, 해상도 640×480, `rgb8` | 1·2절 |
| `마스크_토픽확인_rviz.png` | RViz `Mask`·`Detection` 패널 (같은 입력의 Mask와 검출 결과) | 3절 |
| `마스크_토픽정보.txt` | Mask 토픽 정보(`mono8`, 0 = 배경, 255 = 목표 색), 수신 fps, Mask 생성 과정 | 3절 |
| `검출결과_토픽정보.txt` | `/target`·`/perception/debug_image` 토픽 정보, `ros2 topic echo` 출력, 필드 의미 | 4절 |
| `실험1_밝기하한_최소.png` / `실험1_밝기하한_최대.png` | 실험 1: V 하한 0 / 252 (H 83~109, S 142~255, V 상한 255, 최소 크기 0) | 5절 |
| `실험2_최소면적_0.002.png` / `실험2_최소면적_0.008.png` | 실험 2: `min_area_ratio` 0.002 / 0.008 (HSV [93,120,35]~[130,255,255] 고정) | 5절 |

실험 캡처의 설정값은 각 캡처 아래쪽 트랙바 값과 같다.
