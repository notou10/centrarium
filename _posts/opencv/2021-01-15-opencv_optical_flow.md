---
layout: post
title:  "OpenCV-13 Optical Flow"
description: C++ Optical Flow 관련 내용입니다.
categories : [openCV]
date:   2021-01-05
comments: true
---

본 포스팅은 [여기](http://blog.naver.com/PostView.nhn?blogId=samsjang&logNo=220662493920&parentCategoryNo=&categoryNo=&viewDate=&isShowPopularPosts=false&from=postView), [여기](https://paeton.tistory.com/entry/%EC%98%B5%ED%8B%B0%EC%B9%BC-%ED%94%8C%EB%A1%9C%EC%9A%B0-Optical-Flow), [여기](https://docs.opencv.org/3.4/d4/dee/tutorial_optical_flow.html) 를 참고하여 만들었습니다.

<blockquote align="center">  Optical Flow </blockquote>

Motion estimation을 하는 방식에는 크게 Block matching과 Optical flow 두 가지의 방법이 존재하는데, 후자에 관한 내용이다.

Optical Flow 를 한국말로 번역하면 광학 흐름으로, 두 개의 연속된 비디오 프레임 사이에 이미지 객체의 가시적인(카메라의 이동 등으로 생기는) 동작 패턴을 말한다.

주로 움직임을 통한 구조 분석, 블러 된 영상을 정제하는데 주로 사용하는 기술이다. 

<blockquote align="center">  Optical Flow의 개념 </blockquote>
2D 이미지 내에서 (x, y, t)(좌표, 시간)에서의 픽셀의 밝기를 Intensity I(x, y, t)라 하자.

극소량 t에 대해 픽셀의 Intensity 는 변하지 않는다고 가정 할 때,작은 움직임 δx와 δy가 각각 x, y의 축에서 일어났다고 하면, 그때의 intensity는 I(x + δx,y + δy,t + δt)라고 나타낼 수 있다.

이를 식(1)으로 표현하면 다음과 같다.

![1](/assets/img/OpenCV/0105/1.PNG)

그리고, 다음 테일러 급수 식을 살펴보면 다음과 같다.

![1](/assets/img/OpenCV/0105/3.PNG)

위 테일러 급수에서 2차 이상의 항은 무시하고 위  I(x + δx,y + δy,t + δt)를 표현해 보면(2), 다음과 같다.

![1](/assets/img/OpenCV/0105/2.PNG)

(변수 세 개에 대해 각 변수에 대해 편미분 한 꼴)

식(1)과 (2)가 같아야 하므로, 

![1](/assets/img/OpenCV/0105/4.PNG)

을 만족해야 하고, 이식을 δt로 나누어주면, 

![1](/assets/img/OpenCV/0105/5.PNG)

를 얻을 수 있다.

이 때 δI/δx, δI/δy ,δI/δt 는 각각 이미지 I 에 대한 x, y, t의 방향(편미분), 

δx/δt, δy/δt 는 각각 x, y 방향으로의 속도 임을 알 수 있다.


위 식을 정리하여 쓰면 최종적으로 위 식과 같이 정리된다.


![1](/assets/img/OpenCV/0105/6.PNG)

즉 영상을 x, y, t 방향으로 으로 미분 한 후(It구하기) 식에 넣으면 Vx, Vy를 얻을수 있다는 개념이다.

위 개념을 응용하여 Lucas-Kanade 방식을 사용하면 각 픽셀에 대한 모션 벡터 값을 얻을 수 있다고 한다.

Lucas-Kanade는 영상 내 이미지 사이의 시간적, 공간적 연속성과 이웃한 픽셀들의 움직임은 함께 간다는 것을 가정하여 각 픽셀에 대한 모션 벡터를 얻는 개념인데, 위 포스팅에선 다루지 않겠다.

<blockquote align="center"> goodFeaturesToTrack() </blockquote>

![1](/assets/img/OpenCV/0105/7.PNG)
![1](/assets/img/OpenCV/0105/8.PNG)


input 이미지

feature로 찾은 좌표들

corner로 리턴할 좌표의 최대 수

이미지 퀼리티의 thereshhold

코너들 사이의 최소 유클라디안 거리

interest한 영역에서만 찾을지 (Mat() 일 시 전체에 대해 코너 찾음)

편미분에 사용할 block의 사이즈

등이 인자로 들어간다.


<blockquote align="center"> calcOpticalFlowPyrLK() </blockquote>

![1](/assets/img/OpenCV/0105/9.PNG)
![1](/assets/img/OpenCV/0105/10.PNG)
![1](/assets/img/OpenCV/0105/11.PNG)

status :  다음 프레임에서 똑같은 feature가 찾아졌을 경우 각 벡터의 element들은 1, 아닐시  반환

err : 벡터의 element가 해당 feature에 대한 error로 설정됨, 오류 측정의 유형은 flag로 설정 가능 (이해 x)

window : search window의 사이즈

max pyramid : 0일시 피라미드 안 쓰임, 1 일 시 if pyramids are passed to input then algorithm will use as many levels as pyramids have but no more than maxLevel..?
피라미드를 사용한다는게 무슨 말인지 모름

(+ FPN에서 나온 피라미드와 똑같은 개념, 피라미드란 원근감에 따라 이미지의 물체의 형태가 바껴 보임을 인지하여 scale 변화에 강건하게 해주는 개념이다.)

<blockquote align="center"> CODE </blockquote>

```cpp
#include <opencv2/opencv.hpp>
#include <iostream>

using namespace cv;
using namespace std;

int main()
{
	VideoCapture cap1(1);

	int is_applied;

	if (!cap1.isOpened())
	{
		cout << "카메라 연결 불가" << endl;
	}

	Mat frame_pre;
	Mat frame_pre_gray;
	Mat frame_now;
	Mat frame_now_gray;
	Mat output;

	is_applied = 0;
	
	vector<Scalar> colors;		//Scalar = ( , , , ,) 인 자료형
	RNG rng;		//컬러 표시를 위한 난수 생성
	for (int i = 0; i < 100; i++)
	{
		int r = rng.uniform(0, 256);
		int g = rng.uniform(0, 256);
		int b = rng.uniform(0, 256);
		colors.push_back(Scalar(r, g, b));
	}
	vector<Point2f> p0, p1;		//( , ) 변위 보기 위해 벡터에 각각 좌표 담기

	while (1)
	{
		cap1.read(frame_pre);		//이전 프레임 받기

		if (is_applied == 1)
		{
			destroyWindow("Video_first");
			//cout << "circle detecting" << endl;
			cvtColor(frame_pre, frame_pre_gray, COLOR_BGR2GRAY);		//gray 상에서 찾기 위한 변환 
			goodFeaturesToTrack(frame_pre_gray, p0, 100, 0.3, 7, Mat(), 7, false, 0.04);		//previous frame에서 좋은 feature인 점 찾기

			Mat mask = Mat::zeros(frame_pre.size(), frame_pre.type());		//그림 그릴 빈 Mat 생성

			//현재 프레임
			cvtColor(frame_now, frame_now_gray, COLOR_BGR2GRAY);
			vector<uchar> status;
			vector<float> err;
			TermCriteria criteria = TermCriteria((TermCriteria::COUNT)+(TermCriteria::EPS), 10, 0.03);		//반복적인iterative 연산을하는 알고리즘의 종료 기준을 정의
			calcOpticalFlowPyrLK(frame_pre_gray, frame_now_gray, p0, p1, status, err, Size(500, 500), 2, criteria);

			vector<Point2f> good_new;
			for (uint i = 0; i < p0.size(); i++)		//코너의 사이즈만큼 찾음
			{
				if (status[i] == 1) {		//똑같은 feature를 찾은 경우
					good_new.push_back(p1[i]);		//good_new 에 해당 now의 좌표 집어넣음
					// draw the tracks
					line(mask, p1[i], p0[i], colors[i], 2, 2);		//pre, now 의 좌표 이음(빈 Mat에서)
					circle(frame_now, p1[i], 1, colors[i], -1);		//now의 좌표 동그랗게
				}
			}
			add(frame_now, mask, output);		//circle + line 합쳐서 보여주기 

			//imshow("Video_previous", frame_pre);		
			//imshow("Video_now", frame_now);
			//imshow("mask", mask);
			imshow("output", output);

			//p0 = good_new;
		}

		else if (is_applied == 0)
		{
			imshow("Video_first", frame_pre);


		}

		////////////////카메라 제어 부
		int key = cv::waitKey(2);

		if (key == 'c')
		{
			is_applied = 1;
		}
		if (key == 27)
		{
			break;
		}

		cap1.read(frame_now);		//이후 프레임 받기 
	}

	return 0;

}

```