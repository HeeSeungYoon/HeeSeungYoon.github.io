---
layout: post
title: 자율 주행 카메라 도로 인식 프로젝트
date: 2023-12-15 09:00:00 +0900
description: # Add post description (optional)
img: # Add image post (optional)
tags: [Autonomous Car, Rasberry Pi, OpenCV, Computer Vision] # add tag
---

라즈베리 파이 보드를 사용한 자율 주행 자동차 프로젝트를 하던 중에 카메라로 장애물과의 거리를 측정할 수 없을까라는 의문을 가져 라즈베리 파이용 카메라를 구해 트랙에서 장애물(벽)의 거리를 측정하는 프로그램을 구현해보려고 했다. 

결론적으로는 카메라 한 대 만으로는 거리를 측정할 수 없다는 것을 알아냈다. 대신에 뎁스 카메라(Depth Camera)를 사용하면 카메라와 사물의 거리를 측정할 수 있다. 

뎁스 카메라의 기술은 카메라가 발사하는 적외선이 물체에 부딪혀서 반사되어 돌아오는 시간을 측정해 사물의 실제 거리를 계산하는 ToF 방식을 사용한다. 하지만 빛과 노이즈 등 외부 간섭에 매우 취약하다는 단점이 있어 실내에서만 사용할 수 있다는 제약이 있다.

ToF 방식 뎁스 카메라의 단점을 개선한 스테레오 방식의 뎁스카메라는 하나의 카메라에 두 개의 렌즈로 촬영된 영상에서 픽셀의 차이를 인식하고 이를 계산하여 거리를 측정한다. 빛 등의 외부 간섭에 덜 취약해졌지만 측정 정보 계산량이 많아 실시간 처리가 힘들다.

아무튼, 라즈베리 파이용 카메라는 렌즈가 하나고 메우 작기 때문에 거리를 측정할 수 없다는 것이다. 그렇다면 카메라를 어떻게 사용할 것이냐라는 고민을 하면서 실제 자율 주행 자동차가 어떻게 카메라로 인식하고 처리하는지 찾아보았다.

다른 사람들이 수행한 자율 주행 자동차의 카메라는 실제 도로의 흰색과 노란색 선을 구분하여 다음 자동차의 예측 방향을 출력해주는 기능이였다. 하지만 내가 하려는 프로젝트는 아래의 트랙을 돌 수 있도록 하는 것이였다.

<div align='center'>
<img src='{{site.baseurl}}/assets/img/picamera/track1.jpg' width=400 height=400 /><img src='{{site.baseurl}}/assets/img/picamera/track2.jpg' width=400 height=400 />
<img src='{{site.baseurl}}/assets/img/picamera/track3.jpg' width=400 height=400 /><img src='{{site.baseurl}}/assets/img/picamera/track4.jpg' width=400 height=400 />
</div>

구현 순서는 다음과 같다.

# 1. 실시간으로 화면 읽어오기

---

# 2. 트랙의 벽 RGB 값을 지정하여 필터링
- HSV 색 공간으로 변환
    - Pi 카메라에서 촬영한 영상을 저장하는 기본 방식은 YUV420
    - YUV420에서 HSV로 바로 변환하지 못하기 때문에 BGR로 중간 변환

    우리들이 색을 판단하는 것과 가장 유사한 것이 HSV 색 공간 
    
    HSV: Hue(색상), Saturation(채도), Value(명도)
    
- 침식과 팽창 수행
    - 팽창 후 침식을 수행(CLOSE)하여 개체의 작은 구멍을 채움
    - 침식 후 팽창을 수행(OPEN)하여 노이즈 제거
    ```python
    kernel = np.ones((7,7), np.uint8)
    # 팽창 후 침식
    mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel)
    # 침식 후 팽창
    mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)
    ```
    - 트랙 벽 필터링
    - 원본 영상과 마스크에 bitwise_and 연산을 수행하여 필터링

## 2.1. 모폴로지 연산
침식(Erosion)

<div align='center'>
<img src='{{site.baseurl}}/assets/img/picamera/erosion.png' width=320 height=240/>
</div>

팽창(Dilatation)
<div align='center'>
<img src='{{site.baseurl}}/assets/img/picamera/dilatation.png' width=320 height=240/>
</div>

- 열림(Open) = 침식 + 팽창
- 닫힘(Close) = 팽창 + 침식
- Gradient = 팽창 - 침식 : 경계 추출
- TOPHAT = 원본 - 열림 : 밝은 부분 강조
- BLACKHAT = 원본 - 닫힘 : 어두운 부분 강조

---

# 3. 경계선 추출

- 가우시안 필터링으로 노이즈 제거
- 필터링한 영상을 GrayScale 영상으로 변환
- Canny Edge Detection 알고리즘 수행하여 경계선 추출

## 3.1. Canny Edge Detection

총 4단계의 과정을 거쳐 수행

1. Gaussian Filtering
2. Gradient 계산
    - sobel filter를 이용하여 gradient의 크기와 방향 성분을 구함

3. NMS(Non-maximum Suppression)
    - 하나의 edge가 여러 개의 픽셀로 표현되는 현상을 없애기 위해서 gradient의 크기가 local maximum인 픽셀만을 edge로 설정
    - edge는 gradient와 직교

4. Hysteresis edge tracking
    - low, high 두개의 threshold 값을 이용하여 strong edge, weak edge, non-edge 판별

---

# 4. 카메라가 보는 시야 지정

- Edge가 검출된 영상에서 트랙의 코스를 검출 하기 위해 카메라가 보는 시야 영역 지정(region of interest)
    ```python
    # 카메라 시야 모양 비율
    trap_bottom_width = 0.95
    trap_top_width = 0.8
    trap_height = 0.3

    height, width = img.shape[:2]
    
    # 검은 바탕의 빈 마스크 생성
    mask = np.zeros((height, width, 1), np.uint8)

    # 카메라 시야 영역 좌표 지정
    left_bottom = [width*(1-trap_bottom_width), height]
    right_bottom = [width*trap_bottom_width, height]
    left_top = [width*(1-trap_top_width), height*trap_height]
    right_top = [width*trap_top_width, height*trap_height]

    trap = np.array([left_bottom, left_top, right_top, right_bottom], np.int32)

    # 빈 마스크에 흰 바탕의 사다리꼴 모양 카메라 영역 생성
    mask = cv2.fillConvexPoly(mask, trap, 255)
    ```
---

# 5. Edge에서 직선 성분 추출

- Hough 변환을 사용하여 Edge 영상에서 직선 성분 추출
    ```python
    # HoughLinesP : 임의의 점을 이용하여 직선을 찾음
        # image - 8bit single-channel 이진 영상
        # rho - r 값의 범위 (0~1 실수)
        # theta - (0~180 정수)
        # threshold - 만나는 점 개수 임계값
        # minLineLength - 선의 최소 길이
        # maxLineGap - 선과 선 사이 최대 허용 간격

    lines = cv2.HoughLinesP(img, 1, np.pi/180, threshold, None, minLineLength, maxLineGap)
    ```
## 5.1. 추출한 직선 성분을 좌, 우 직선으로 분류

- 추출한 직선의 기울기를 계산
- 기울기의 방향과 직선의 x 좌표를 기준으로 영상 중앙으로부터 직선이 좌, 우 중 어느 방향에 있는지 판단

--- 

# 6. 가장 적합한 선 찾기

- 선형 회귀 방식으로 cv2.fitLine 함수를 이용하여 주어진 경계에 최적화된 직선을 추출
    ```python
    # cv2.fitLine(points, distType, param, reps, aeps)
        # points - 2D 점들의 입력 벡터
        # distType - 거리 계산 방식: DIST_L2 사용
        # param - distType에 전달할 인자 0: 최적값 선택
        # reps - 반지름 정확도: 선과 원본 좌표의 거리, 0.01 권장
        # aeps - 각도 정확도: 0.01 권장

    left_line = cv2.fitLine(left_points, cv2.DIST_L2, 0, 0.01, 0.01)
    ```
- 좌, 우 선 각각의 좌표 계산

# 7. 자동차 진행 방향 예측

- 두 직선의 교점 계산
- 교점의 x좌표가 영상 중심의 x 좌표에서 어느 방향으로 얼마나 떨어져 있는지 계산
- 교점이 치우쳐진 방향의 반대 방향에 장애물이 있다고 예측하여 교점 방향으로 자동차가 이동할 수 있도록 처리
- 원본 영상에 두 직선과 교점 출력

## 7.1. 두 직선의 교점 계산

<div align='center'>
<img src='{{site.baseurl}}/assets/img/picamera/crosspoint.png' width=800 height=400/>
</div>

```python
# m이 0이면 평행이거나 같은 직선
m = (x1-x2)*(y3-y4)-(y1-y2)*(x3-x4) 

    if m !=0:
        vx = ((x1*y2-y1*x2)*(x3-x4)-(x1-x2)*(x3*y4-y3*x4))/m
        vy = ((x1*y2-y1*x2)*(y3-y4)-(y1-y2)*(x3*y4-y3*x4))/m 
```