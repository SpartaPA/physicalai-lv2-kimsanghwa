## 1. 기존 비전 노드 실행과 토픽 구조 확인

| 자료        | 토픽 이름                          | 메시지 타입                                                | 발행 노드                                                                           | 수신 도구                                                       |
| :---------: | :------------------------------: | :-----------------------------------------------------: | :-------------------------------------------------------------------------------: | :-----------------------------------------------------------: |
| Raw image | /camera/camera/color/image_raw | sensor_msgs/msg/Image(rgb8, 640 X 480, 30fps)         | realsense2_camera(RealSense wrapper)                                            | RViz tracking_view.rviz 실행 후 "Camera color (raw) 패널         |
| Mask      | 발행하지 않음                        | ROS 메시지가 아닌 NumPy 배열 uint8 (H×W, 목표 색 = 255, 나머지 = 0) | [detector.py](http://detector.py)의 detect() → make_mask()가 매 프레임의 계산만 수행(발행은 X) | tools/image_[capture.py](http://capture.py)가 \_mask.png로 저장 |
| 검출 결과     | /target                        | geometry_msgs/msg/PointStamped                        | perception_node                                                                 | ros2 topic echo /target                                     |


## 2. Raw Image 토픽 받아보기

[실행방법]
터미널 1 : 카메라와 인지 노드 실행
```bash
export ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST
ros2 launch ~/Lv2_EYE4_Assignment/lv2_module5/tools/camera_view.launch.py rviz:=false
```

터미널 2 : 토픽 확인과 이미지 뷰어
```bash
export ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST
ros2 run rqt_image_view rqt_image_view /camera/camera/color/image_raw/compressed
```


## 3. Mask 토픽 받아보기

[실행방법]

- RViz2로 실행 및 확인
```bash
ros2 launch ~/Lv2_EYE4_Assignment/lv2_module5/tools/camera_view.launch.py
```

- rqt_image_view로 실행 및 확인
```bash
ros2 run rqt_image_view rqt_image_view /perception/mask
```
- 원본에서의 파란색이 검출된 부분은 흰색으로 남았고, 그 외의 부분은 모두 검정색으로 나타났다.
흰색으로 남은 부분은 "목표하는 색으로 판정되 픽셀"을 의미한다. 또한 배경으로 나타난 검정색 픽셀들은 검출되지 않은 HSV값이거나, 검출되었다 하더라도 최소면적값보다 작아 무시된 픽셀값들이다.

- 목표외에 작게 남은 영역이나 누락된 부분은 파라미터 튜닝을 통해 검출되지 않도록 했다.

## 4. 검출 결과 토픽 받아보기

- 검출 결과 토픽을 구독했을때 수신되는 메시지

```bash
$ ros2 topic echo /target --qos-reliability best_effort
header:
  stamp:
    sec: 1791776423
    nanosec: 281854980
  frame_id: camera_color_optical_frame
point:
  x: -0.25272503352717113
  y: 0.5666501065606818
  z: 0.144423828125
---
header:
  stamp:
    sec: 1791776423
    nanosec: 315212158
  ...
point:
  x: -0.25261894969708587
  y: 0.5666307664186512
  z: 0.1444482421875
```

## 5. 설정값을 바꿔 변화 검증하기

- 실험 1 : V의 최소값, 즉 밝기의 하한을 변경하여 진행한다. 얼마나 어둡거나 밝은 파란색까지 목표로 인정할지를 다르게 한다.
- 밝기 하한 최소로 실험했을 경우 : 채도가 낮던 높던 거의 모든 파란색을 인식했다.
- 밝기 하한 최대로 실험했을 경우 : 가장 채도가 낮은 파란색만을 인식했다.

- 선택 이유의 근거 : V 하한을 최소로 두면 화면 아래쪽의 어두운 영역이 HS조건을 우연히 만족해 Mask에 남아 candidates가 13개로 잡혔다. 반대로 최대로 두면 목표인 파란 링이 모두 0이 되어 사라졌고, 더 밝은 객체가 목표로 설정되었다. 변화가 가장 명확하게 나타났기 때문에 이 항목을 실험조건으로 설정하였다.

- 실험 2 : 최소 면적 0.002 -> 0.008
- 최소면적이 0.002일 때는 목표물을 정상적으로 검출하지만, 최소면적을 0.008로 변경하면 목표물보다 최소면적이 커지게 되어 목표물을 검출하지 못하는것을 확인할 수 있다.