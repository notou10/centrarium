---
layout: post
title:  "OpenCV-10 feature matching - ORB, SIFT and BRISK"
description: C++ feature map 추출 관련 내용입니다.
categories : [openCV]
date:   2020-11-17
comments: true
---

<blockquote align="center"> feature란? </blockquote>

![1](/assets/img/OpenCV/1118/1.PNG)

인간이 이 특징들을 어떻게 찾는지는 이미 뇌에 내재된 상태로 살아간다. 

A, B는 평평한 표면이고, 그림 내 이들의 위치를 찾는 난이도는 비교해보자면 상 에 속한다.

그에 반해  C, D는 가장자리이고, 중 에 속한다.

마지막으로 E, F는 모서리이고, 하에 속한다.

따라서 모서리 부분은 좋은 특징이라 할 수 있다.


또한 이와 별개로 컴퓨터에 본 특성들 주변 영역에 대해 설명하는, 특징 설명을 해주어야 할 필요가 있다.

이를테면 하늘은 푸르고, 가운데에는 빌딩이 위치한다 등과 같은 설명이 필요하다.




<blockquote align="center"> feature matching </blockquote>

feature matching는 keypoint와 descriptor를 이용한다.

keypoint는 feature에 해당하는 이미지에서 쉽게 식별 가능하고 고유한 형태를 지니고 있는 영역을 말한다.

corner에 해당하는 단일 픽셀(점의 좌표)이 주로 해당된다.

descriptor는 keypoint에서 각각의 알고리즘이 주변의 정보까지 활용해 지역적 틍징을 뽑아낸 정보를 의미한다.

SIFT 알고리즘의 descriptor는 gradient 방향 크기의 vector 값의 그룹임이 그 예이다.

Descriptor는 keypoint와 동일한 개수만큼 생기고, 실제 유사도를 판별하기 위해 사용된다고 한다.


<blockquote align="center"> feature matching - Brute-Force Matcher </blockquote>

첫 번째 세트(이미지) 속 하나의 특성을 디스크립터로 취하고, 두 번쨰 세트의 다른 특성들과 거리 계산을 사용하여 매칭이 되고, 가까운 것을 반환한다.

BFMatcher 객체를 생성 하고, 인자의 첫 번째로 거리측정법을 준다.

SIFT의 경우 NORM_L2, ORB나 BRISK의 경우 NORM_HAMMING이 좋다고 한다.



<blockquote align="center"> ORB </blockquote>

ORB는 SIFT보다 나은 대안인 동시에 수많은 FAST 키포인트 탐지기와 BRIEF 디스크립터의 통합본이다.

FAST를 이용하여 키포인트를 찾고 Harris 코너 측정을 해서 상위 N개의 점을 찾는다고 한다.

SURF나 SIFT보다 훨씬 빠르고, ORB 디스크립터가 SURF 보다 잘 작동한다고 한다. 

파노라마 스티칭을 위한 저전력 장치에서 효율이 좋다고 한다.


<blockquote align="center"> BRISK </blockquote>

BRISK(Binary Robust Invariant Scalable Keypoints)는 FAST 또는 AGIST를 사용하여 스케일 공간에서 피라미드 기반으로 특징점을 검출하고, 디스크립터 계산은 특징점 근처에서 동심원 기반의 샘플링 패턴을 이용하여 이진 디스크립터를 계산한다고 한다.


Ptr<BRISK> 로 선언된 변수는 특징 검출(detector)과 특징 추출(descriptor)의 역할을 둘 다 하게 된다.


https://wjddyd66.github.io/opencv/OpenCV(8)/ 참고


<blockquote align="center"> SIFT </blockquote>

SIFT는 이미지의 크기나 회전에 불변하는 특징을 추출하는 알고리즘이다.
즉, 가장 중요한 것은 크기, 회전두가지에 대해서 불변하는 특징점을 추출하는 알고리즘이라는 것 이라고 한다.


<blockquote align="center"> CODE - 각 알고리즘 비교 </blockquote>

```cpp

#include <opencv2/opencv.hpp>
#include <iostream>
#include <opencv2/xfeatures2d/nonfree.hpp>

using namespace cv;
using namespace std;

int main()
{
	int a;

	cout << "ORB = 1, BRISK = 2, SIFT = 3 중 입력하세요 : " << endl;
	cin >> a;

	VideoCapture cap1(1);

	if (!cap1.isOpened())
	{
		cout << "카메라 없음" << endl;
	}

	Mat frame1, reference_frame, target_frame;
	namedWindow("Video", WINDOW_AUTOSIZE);

	while (1)
	{
		int key = waitKey(10);
		cap1 >> frame1;

		cv::imshow("Video", frame1);

		if (key == 97)		//reference
		{
			cap1 >> reference_frame;
			//cvtColor(reference_frame, reference_frame, COLOR_BGR2GRAY);
			cv::imshow("reference", reference_frame);
		}

		else if (key == 98)		//target
		{
			cap1 >> target_frame;
			//cvtColor(target_frame, target_frame, COLOR_BGR2GRAY);
			cv::imshow("target", target_frame);
		}
		
		if ((reference_frame.empty() != 1) && (target_frame.empty() != 1))
		{
			vector<KeyPoint> keypoint1, keypoint2;		//키포인트를 담을 벡터
			Mat descriptor1, descriptor2;		//descripter를 답을 벡터
			Mat img_matches;		//가시화 할 img

			if (a == 1)
			{
				Ptr<ORB> orbs = ORB::create(100);
				orbs->detectAndCompute(reference_frame, noArray(), keypoint1, descriptor1);		//keypoint1, descriptor에 값 적재
				orbs->detectAndCompute(target_frame, noArray(), keypoint2, descriptor2);

				vector<DMatch> matches;		//매치들을 담을 벡터
				BFMatcher matcher(NORM_HAMMING, 1);		//orb이므로 NORM_HAMMING를 사용하여 거리 측정, cross check 를 true로 주어 정확도 향상
				matcher.match(descriptor1, descriptor2, matches);		//두 이미지에서 최상의 일치를 얻음 (가장 일치하는것을 반환함), crossCheck

				cv::drawKeypoints(reference_frame, keypoint1, img_matches);
			/*	cv::drawMatches(reference_frame, keypoint1, target_frame, keypoint2, matches, img_matches, Scalar::all(-1),
					Scalar::all(-1), std::vector<char>(), DrawMatchesFlags::NOT_DRAW_SINGLE_POINTS);*/
				cv::imshow("Matched Image", img_matches);

				waitKey();
				break;
			}

			else if (a == 2)
			{					
				Ptr<BRISK> brisks = BRISK::create(40, 3, 1.0f);		//FAST 9-16 알고리즘에서 거르기 위해 사용하는 값을 threshold로 지정 가능

				brisks->detectAndCompute(reference_frame, noArray(), keypoint1, descriptor1);
				brisks->detectAndCompute(target_frame, noArray(), keypoint2, descriptor2);

				vector<DMatch> matches;
				BFMatcher matcher(NORM_HAMMING);		
				matcher.match(descriptor1, descriptor2, matches);



				cv::drawMatches(reference_frame, keypoint1, target_frame, keypoint2, matches, img_matches, Scalar::all(-1),
					Scalar::all(-1), std::vector<char>(), DrawMatchesFlags::NOT_DRAW_SINGLE_POINTS);
				cv::imshow("Matched Image", img_matches);

				if (waitKey() == 27)
				{
					break;
				}
			}

			else if (a == 3)
			{
				cv::Ptr<cv::xfeatures2d::SIFT> siftPtr = cv::xfeatures2d::SIFT::create();
				siftPtr->detectAndCompute(reference_frame, noArray(), keypoint1, descriptor1);
				siftPtr->detectAndCompute(target_frame, noArray(), keypoint2, descriptor2);


				vector<DMatch> matches;
				BFMatcher matcher(NORM_L2);
				matcher.match(descriptor1, descriptor2, matches);

				cv::drawMatches(reference_frame, keypoint1, target_frame, keypoint2, matches, img_matches, Scalar::all(-1),
					Scalar::all(-1), std::vector<char>(), DrawMatchesFlags::NOT_DRAW_SINGLE_POINTS);
				cv::imshow("Matched Image", img_matches);

				waitKey();
				break;
			}

		}		
	}
	return 0;
}
```

<blockquote align="center"> 결과 </blockquote>


![1](/assets/img/OpenCV/1118/5.PNG)

![1](/assets/img/OpenCV/1118/6.PNG)

![1](/assets/img/OpenCV/1118/7.PNG)


<blockquote align="center"> CODE - realtime </blockquote>

```cpp

#include <opencv2/opencv.hpp>
#include <iostream>
#include <opencv2/xfeatures2d/nonfree.hpp>

using namespace cv;
using namespace std;

int main()
{
	printf("1");
	VideoCapture cap1(1);
	
	if (!cap1.isOpened())
	{
		cout << "카메라 없음" << endl;
	}
	
	Mat frame1, reference_frame, target_frame;
	namedWindow("Video", WINDOW_AUTOSIZE);

	while (1)
	{
		int key = waitKey(10);
		cap1 >> target_frame;

		cv::imshow("Video", target_frame);

		if (key == 97)		//reference
		{
			cap1 >> reference_frame;
			//cvtColor(reference_frame, reference_frame, COLOR_BGR2GRAY);
			cv::imshow("reference", reference_frame);
		}

		if (target_frame.empty() || reference_frame.empty())
		{
			cv::waitKey(1);
			continue;
		}

		vector<KeyPoint> keypoint1, keypoint2;		//키포인트를 담을 벡터
		Mat descriptor1, descriptor2;		//descripter를 답을 벡터
		Mat img_matches;		//가시화 할 img


		Ptr<ORB> orbs = ORB::create(1000);
		orbs->detectAndCompute(reference_frame, noArray(), keypoint1, descriptor1);		//keypoint1, descriptor에 값 적재
		orbs->detectAndCompute(target_frame, noArray(), keypoint2, descriptor2);

		vector<DMatch> matches;		//매치들을 담을 벡터
		BFMatcher matcher(NORM_HAMMING, 1);		//orb이므로 NORM_HAMMING를 사용하여 거리 측정, cross check 를 true로 주어 정확도 향상
		matcher.match(descriptor1, descriptor2, matches);		//두 이미지에서 최상의 일치를 얻음 (가장 일치하는것을 반환함), crossCheck



		cv::drawMatches(reference_frame, keypoint1, target_frame, keypoint2, matches, img_matches, Scalar::all(-1),
			Scalar::all(-1), std::vector<char>(), DrawMatchesFlags::NOT_DRAW_SINGLE_POINTS);
		cv::imshow("Matched Image", img_matches);



		if (key == 27)
		{
			break;
		}

	}
	return 0;
}
```