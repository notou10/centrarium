---
layout: post
title:  "OpenCV-2 Using Videos and webCam"
description: C++로 Opencv 를 이용해 웹캠과 비디오를 다루어봤습니다.
author: Dingkyun Kim
date:   2020-08-05
categories : [openCV]
comments: true
---

8월 5일자 openCV 세미나 발표 ppt 입니다.

cv::imshow 함수와 VideoCapture 클래스를 이용하여 구현했습니다.

카메라는 intel realsense D435 depth camera를 사용했습니다.

D435 depth camera의 특징은 좌우에 각각 렌즈 한 개 씩 달려있어 사람의 눈 처럼 depth 정보를 파악하고, 가운데의 projocter를 통해 한 세트의 패턴을 대상에 투사하여 결과를 얻어내는 역할을 해줍니다.

<blockquote align="center"> 발표자료 </blockquote>


![1](/assets/img/OpenCV/0805/1.PNG)
![1](/assets/img/OpenCV/0805/2.PNG)
![1](/assets/img/OpenCV/0805/3.PNG)
![1](/assets/img/OpenCV/0805/4.PNG)
![1](/assets/img/OpenCV/0805/5.PNG)
![1](/assets/img/OpenCV/0805/6.PNG)
![1](/assets/img/OpenCV/0805/7.PNG)
![1](/assets/img/OpenCV/0805/8.PNG)


