---
layout: post
title:  "OpenCV-12 Hough detection"
description: C++ HoughCircles,HoughLinesP  관련 내용입니다.
categories : [openCV]
date:   2020-12-22
comments: true
---
본 포스팅은 [여기](https://diyver.tistory.com/102), [여기](https://diyver.tistory.com/100),  [여기](https://docs.opencv.org/4.1.0/dd/d1a/group__imgproc__feature.html#ga47849c3be0d0406ad3ca45db65a25d2d). [여기](https://opencv-python.readthedocs.io/en/latest/doc/26.imageHoughCircleTransform/imageHoughCircleTransform.html) 를 참고하여 만들었습니다.



<blockquote align="center">  HoughLine </blockquote>

![1](/assets/img/OpenCV/1222/4.PNG)

image : 입력 이미지

line : 라인들이 저장될 array

rho :  계산할 픽셀(매개 변수)의  acculumator의 해상도, 1일시 default(원본)

theta : 계산할 각도(라디안, 매개변수)의 해상도, 선 회전 각도 모든 방향에서 직선을 검출시 PI/180 사용

threshold : 변환된 그래프에서 라인을 검출하기 위한 최소 교차수




<blockquote align="center">  HoughLine에서의 좌표계 변환 </blockquote>


![1](/assets/img/OpenCV/1222/5.PNG)

(x, y) 좌표계 상에서 (1, 1)을 지나는 직선의 방정식은, *y = ax + b*에 (1, 1)을 대입한 식이고, 이는 곧 (a, b)좌표계에서 *1 = a + b*임과 같다.

이를 여러 점에 대해 수행 후 좌표계를 바꾸어 보면, 다음과 같은 좌표계 상의 값들을 얻을 수 있다.

![1](/assets/img/OpenCV/1222/6.PNG)
![1](/assets/img/OpenCV/1222/7.PNG)

(위는 xy 좌표계가 ab좌표계로 바뀐 모습이다.)


위 ab 좌표계에서, 선들이 세 번 이상(threshhold = 3인 경우) 접하는 경우일 떄의 (a, b)값을 구하여, 다시 xy좌표계로 변환하면, 다음과 같은 그림을 얻을 수 있다.

![1](/assets/img/OpenCV/1222/8.PNG)


허프 변환은, 이러한 방식으로 직선을 검출한다.




<blockquote align="center">  HoughCircles </blockquote>

![1](/assets/img/OpenCV/1222/2.PNG)


원을 찾기 위해선, 모든 점에 대해 x, y, r 세 가지 값을 모두 찾아주어야 하기에 상당히 비효율적이다.

비효율성을 줄이기 위해 openCV는 모든 픽셀에 대해 세 변수 값을 찾지 않고 가장자리의 기울기를 측정하여 원을 그리는 Hough Gradient 방법을 사용한다.


![1](/assets/img/OpenCV/1222/1.PNG)

들어가는 인자를 순서대로 살펴보면, 

image : 입력으로 들어가는 Mat(gray scale)

circles : 검출할 원의 중심 정보들이 저장될 array

method : 원 검출 방법(HOUGH_GRADIENT)

dp : (accumulator)/(input image의 해상도), 1이면 accumulator 와 image의 해상도가 같고, 2이면 accumulator의 해상도가 input image의 반 이다.(좀 더 너그럽게 찾는 듯) 

minDist : 원과 원과의 거리. 작을수록 원들간의 거리가 좁아도 검출된다.

parameter1 : 내부적으로 사용하는 canny edge 검출기에 전달되는 Paramter (경계를 찾는데 사용되는 부분이라 생각)

parameter2 : accumulator의 threshhold값. 높을수록 원을 찾는 기준이 엄격하다.

minRadius, MaxRadius : 원의 최소, 최대 반지름. 0일 시 반지름에 상관 없이 모두 찾는다.


Vec3i : (int , int , int)인 자료형

다음은 원의 자료형에 대해 간단히 살펴보겠다.

![1](/assets/img/OpenCV/1222/3.PNG)

img : 원이 그려질 Mat

center : center의 좌표(x, y)

Radius : 반지름

그 후 (선의 색, 굵기, 선 타입, shift 순)이다.

```cpp
#include <opencv2/opencv.hpp>
#include <iostream>

using namespace cv;
using namespace std;

int main()
{
	VideoCapture cap1(1);

	vector<Vec3f> circles;
	int is_applied;

	if (!cap1.isOpened())
	{
		cout << "카메라 연결 불가" << endl;
	}

	Mat frame1;
	Mat frame_gray;

	namedWindow("Video", WINDOW_AUTOSIZE);

	is_applied = 0;
	while (1)
	{
		cap1 >> frame1;

		if (is_applied == 1)
		{
			cout << "circle detecting" << endl;

			cvtColor(frame1, frame_gray, cv::COLOR_BGR2GRAY);
			HoughCircles(frame_gray, circles, HOUGH_GRADIENT, 1, 100, 50, 65, 0, 0);
			//해상도 그대로 사용, parameter2 에 65의 값 할당

			for (size_t i = 0; i < circles.size(); i++)
			{
				Vec3i c = circles[i];		//circle 배열의 원들 하나씩 가져옴
				Point center(c[0], c[1]);		//(x, y) 값 할당
				int radius = c[2];			//반지름 할당

				circle(frame_gray, center, radius, Scalar(0, 255, 0), 2);
				circle(frame_gray, center, 2, Scalar(0, 0, 255), 3);
				imshow("Video", frame_gray);
			}
		}

		else if (is_applied == 2)
		{
			Mat frame_canny;
			cout << "line detecting" << endl;
			cvtColor(frame1, frame_gray, cv::COLOR_BGR2GRAY);
			Canny(frame_gray, frame_canny, 150, 255);		//캐니 변환

			vector<Vec2f> lines;		//(float, float)인 벡터형
			HoughLines(frame_canny, lines, 1, CV_PI / 180, 150);		//threshhold로 150의 값 이용

			Mat img_hough;		//원본 이미지에 라인을 추가한 mat
			frame1.copyTo(img_hough);		

			Mat img_lane;		//canny알고리즘으로 라인을 그릴 mat
			threshold(frame_canny, img_lane, 150, 255, THRESH_MASK);

			for (size_t i = 0; i < lines.size(); i++)		//직선을 그리는 부분
			{
				float rho = lines[i][0], theta = lines[i][1];		// i 번째 라인의 rho, theta값 저장
				Point pt1, pt2;
				double a = cos(theta), b = sin(theta);
				double x0 = a * rho, y0 = b * rho;
				pt1.x = cvRound(x0 + 1000 * (-b));
				pt1.y = cvRound(y0 + 1000 * (a));
				pt2.x = cvRound(x0 - 1000 * (-b));
				pt2.y = cvRound(y0 - 1000 * (a));
				line(img_hough, pt1, pt2, Scalar(0, 0, 255), 2, 8);		//원본에 line 그리기
				line(img_lane, pt1, pt2, Scalar::all(255), 1, 8);		//canny mat에 라인 그리기
			}

			imshow("Video", img_hough);
			imshow("img_lane", img_lane);
		
		}

		if (is_applied == 0)
		{
		imshow("Video", frame1);

		}
		
		int key = cv::waitKey(5);

		if (key == 'c')
		{
			is_applied = 1;
		}

		else if (key == 'd')
		{
			is_applied = 2;
		}


		if (key == 27)
		{
			break;
		}

	}

	return 0;

}

```