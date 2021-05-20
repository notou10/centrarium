---
layout: post
title:  "OpenCV-16 Labeling"
description: C++ Opencv 를 통해 connectedComponentsWithStats() 함수를 이용하여 Labeling을 구현했습니다.
categories : [openCV]
date:   2021-02-09
comments: true
---

본 포스팅은 [여기](https://diyver.tistory.com/139?category=911404), [여기](https://thebook.io/006939/ch12/01/02/), [여기](https://btw-e-m.tistory.com/11)를 참고하여 작성하였습니다.


<blockquote align="center">  connectedComponentsWithStats() 함수 </blockquote>

![1](/assets/img/OpenCV/210209/1.PNG)
![1](/assets/img/OpenCV/210209/7.PNG)

레이블링을 수행 후, 각각의 객체가 어느 픽셀 을 중심으로, 어느 정도 크기를 가지며 존재하는지를 나타내주는 함수이다.

각 인자로는 

image : input Mat

labels : labeled된 이미지가 저장될 Mat

stats : 각 label에 대한 통계 정보를 담은 1차원 mat(?)

(좌상단 x, 좌상단 y, 너비, 높이, 면적(총 픽셀수)) (코드 캡쳐한 이미지 참고)

centroid : 각 객체의 중심 좌표들의 모음((x1, y1), (x2, y2)...)

connectivity : 4way or 8way

ltype : 출력 이미지의 label type (CV_32S, CV_16U)
(CV_32S : 32-bit signed integer: int ( -2147483648..2147483647 ),  CV_16U : 16-bit unsigned integer: ushort ( 0..65535 ), CV_16U의 경우 65535개 까지만 세 진다라고 생각했다.)

ccltype : component를 찾는데 사용할 알고리즘 (SAUF or BBDT).. 자세한 정보는 x

반환 값 : label의 총 개수 (N개가 반환된다면, label은 0~ N-1 까지 있다.)


##About labeling

인접한 같은 값들을 갖는 픽셀끼리 하나의 그룹으로 묶어주는 작업

그 그룹으로 묶인 객체들이 각각 1, 2 등의 이름을 가지며 각각 label이 된다.

![1](/assets/img/OpenCV/210209/3.PNG)

component를 찾아 그룹을 묶어주는데에는 4 way, 8way 방식이 있는데, 말 그대로 중심 픽셀 기준 상하좌우만 볼 것인인지, 상하좌우 + 대각선 까지 보며 그룹을 묶어 줄 것인지의 차이이다.


(4way search 기준)좌표 (1. 1)부터 시작하여, 중심 픽셀을 

1. 좌 -> 우 search

2. 1.이 한 행에 대해 종료되었으면 1행 밑으로 이동 즉(2,1)부터 다시  좌 -> 우 search 시작

하는 방식으로 진행된다.

이 떄 중심 픽셀을 이동하는 과정에서 중심 픽셀 기준 4 way에서 같은 값을 갖는 픽셀이 있을 때, 4way search를 종료될 때 까지 반복 한다.

![1](/assets/img/OpenCV/210209/4.PNG)

(입력 이미지)

![1](/assets/img/OpenCV/210209/5.PNG)

(4 way search 를 수행 한 이미지)

![1](/assets/img/OpenCV/210209/6.PNG)

(8 way search 를 수행 한 이미지)


<blockquote align="center">  Mat.at() 사용법 </blockquote>

Mat image;
image.at<DATA_TYPE>(row, col);   return type : DATA_TYPE

3가지 접근법 중 속도가 가장 느리지만, 유효성검사를 수행하기 때문에 안정적이다.
DATA_TYPE 에는 영상의 차원에 따라 적절한 타입을 넣어주면 된다. 

라고 한다.

출처: https://nextus.tistory.com/15 [ReStartAllKill]



<blockquote align="center">  putText() </blockquote>

![1](/assets/img/OpenCV/210209/8.PNG)

이미지에 글자를 써 주는 함수이다.

img : 입력 frame

text : 쓸 text

point : text의 시작 좌표, text 의 Bottem left 위치

fontFace : 폰트

fontScale : 폰트 크기

color : text 색

thickness : 굵기 

등을 인자로 갖는다.

<blockquote align="center"> CODE </blockquote>

```cpp

#include <iostream>
#include <opencv2/opencv.hpp>
#include <stdio.h>

using namespace cv;
using namespace std;

int main()
{
	VideoCapture cap1(1);        
	Mat frame;
	Mat frame_gray;
	int is_applied = 0;
	int count_isapplied = 0;

	while (1)
	{
		cap1.read(frame);

		if (is_applied == 0)
		{
			imshow("Video", frame);
		}

		else if (is_applied == 1)
		{

			cvtColor(frame, frame_gray, COLOR_BGR2GRAY);

			Mat img_threshold;
			threshold(frame_gray, img_threshold, 100, 255, THRESH_BINARY_INV);

			Mat img_labels, stats, centroids;
			int numOfLables = connectedComponentsWithStats(img_threshold, img_labels, stats, centroids, 8, CV_32S);

			for (int j = 1; j < numOfLables; j++) {
				int area = stats.at<int>(j, CC_STAT_AREA);		//면적
				int left = stats.at<int>(j, CC_STAT_LEFT);
				int top = stats.at<int>(j, CC_STAT_TOP);
				int width = stats.at<int>(j, CC_STAT_WIDTH);
				int height = stats.at<int>(j, CC_STAT_HEIGHT);


				rectangle(frame, Point(left, top), Point(left + width, top + height),
					Scalar(0, 0, 255), 1);

				putText(frame, to_string(j), Point(left, top), FONT_HERSHEY_SCRIPT_COMPLEX, 1, Scalar(0, 0, 0), 1);
				//int 값을 string으로 편환
			}

			imshow("Video", frame);

			cout << "numOfLables : " << numOfLables - 1 << endl;	// 최종 넘버링에서 1을 빼줘야 함
		}



		int key = waitKey(100);

		if (key == 'c')
		{
			if (count_isapplied % 2 == 0)
			{
				is_applied = 1;
				cout << "labeling on " << endl;

			}

			else if (count_isapplied % 2 != 0)
			{
				is_applied = 0;
				cout << "labeling off " << endl;


			}
			count_isapplied++;
		}

		else if (key == 27)
		{
			break;
		}
	}
	return 0;
}

```


<blockquote align="center"> 결과 </blockquote>

![1](/assets/img/OpenCV/210209/9.PNG)
