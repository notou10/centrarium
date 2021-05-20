---
layout: post
title:  "OpenCV-14 Alpha Channel"
description: C++ Alpha channel 관련 내용입니다.
categories : [openCV]
date:   2021-01-26
comments: true
---

<blockquote align="center">  Alpha channel이란? </blockquote>


opencv에서 알파채널 값은 투명도이고, 0(완전 투명) ~ 255(완전 불투명)으로 값이 커진다.

(b, g, r, **a**) 에 알파채널 값이 위치한다.

알파채널 까지 포함되도록 하려면, Imread시 flag를 통해 가능하다.

![1](/assets/img/OpenCV/210126/1.PNG)


<blockquote align="center"> PNG  </blockquote>

기본적으로 PNG 파일의 경우 알파채널까지 포함된 RGBA 구조의 포멧이다.

반면 JPG의경우 알파채널이 없는 RGB 구조이다.

PNG를 imread를 통해 받아온 경우, 

![1](/assets/img/OpenCV/210126/2.PNG)

위와 같이 배경이 흰색인 부분의 픽셀 값을 보았을 때, (255, 255, 255, 255)인 경우가 있다.

이 경우는 r g b가 모두 255이므로 흰색, 완전 불투명으로 이루어져 배경이 흰색을 이룸을 알 수 있다.


![1](/assets/img/OpenCV/210126/3.PNG)


반면 이와 같이 실제 뷰어를 통해선 흰 색인 배경이, imshow를 통해 보았을 경우는 검은색으로 나온 경우가 있다.

이 경우는 r g b 가 모두 0으로 검은색을 이루지만 투명도를 완전 투명으로 주어 뷰어에선 흰 색을 보이게끔 된 구조이다.

허나 imshow에선 알파채널의 투명도 값이 적용되지 않은 상태로 보여주므로 배경 색이 검다.

이는 mask를 씌우는 연산이나, threshhold를 통해 배경을 흰 색으로 바꾸어주는 등의 연산을 통해 해결하면 될 것 같다.