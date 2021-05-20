---
layout: post
title:  "OpenCV-7 Canny edge detection"
description: C++ Opencv webcam을 이용한 Canny edge detection 구현입니다.
categories : [openCV]
date:   2020-09-08
comments: true
---
본 포스팅은 [여기](https://m.blog.naver.com/samsjang/220507996391), [여기](https://www.youtube.com/watch?v=sRFM5IEqR2w&t=112s) 를 참고하여 포스팅했습니다.

<blockquote align="center"> Canny edge detection </blockquote>

Canny edge detection은 4개의 stage로 구성된다.

Stage 1. 가우시안 필터를 통한 노이즈 제거(blurring 효과를 통해 엣지를 찾는데 용이함)

Stage 2. Gradient 값 높은 부분 찾기 : x축, y축 gradient가 높은 부분을 찾기 위해 sobel 마스크를 convolution한다.

영상에서는 sobel마스크를 씌우는 것이, 미분의 효과를 낸다.

이를 통해 가장 빠르게 증가한 방향, 그리고 그 값을 알 수 있다.

그 값(norm)을 이용하여 증가량 비교, 방향을 찾을 수 있다.



![1](/assets/img/OpenCV/0909/3.PNG)
![1](/assets/img/OpenCV/0909/4.PNG)




Stage 3. Non maximum suppression : 가짜 Edge를 검출하여 제거하는 부분이다.

Local maximum 부분만을 에지 픽셀로 설정하는 기법인데, 같은 그래디언트의 방향중 가장 큰 값을 지닌 픽셀만 남겨두고 나머지는 0으로 만든다.

즉 그 픽셀 기준 제일 큰 값 제외 8개의 픽셀은 0값을 갖게 한다.

Stage 4. double threshholding : 임계치를 기준으로 파랑 이하 값은 버리고, 주황은 약한 엣지, 빨강은 강한 엣지로 둔다.

주황에 해당하는 약한 엣지 부분의 픽셀의 8방향에 대해 강한에지와 연결되어 있으면 그 정보를 살리고, 아니면 그 엣지를 0으로 둔다.



![1](/assets/img/OpenCV/0909/5.PNG)

![1](/assets/img/OpenCV/0909/8.PNG)

위 그림에서 왼쪽은 약한 에지와 강한 에지가 합쳐진 그림이고, 오른쪽은 연관성 있는(살아남은) 약한 에지와 강한 에지가 합쳐진 그림을 나타낸 것이다.











<blockquote align="center"> 코드 </blockquote>

```cpp
#include <opencv2/opencv.hpp>
using namespace cv;

//슬라이더가 위치를 변경할 때마다 호출되는 함수에 대한 포인터
//여기서 첫 번째 매개 변수는 트랙 바 위치이고 두 번째 매개 변수는 사용자 데이터
//콜백이 NULL 포인터이면 콜백이 호출되지 않고 업데이트만 됨.
static void on_trackbar(int, void*) {
}

void Show_canny(Mat frame);

int main()
{
	VideoCapture cam(0);        //opecncv에서 욎아 카메라에서 영상 받아놓는 역할
	Mat frame;
	cam.set(CAP_PROP_FRAME_WIDTH, 1280);       //프레임의 너비, 높이를 (1280, 720)으로 고정
	cam.set(CAP_PROP_FRAME_HEIGHT, 720);


	if (!cam.isOpened())
	{
		printf("카메라 없음\n");
		return -1;
	}
	namedWindow("Canny");

	//Track bar 만들기
	//인자 : (트랙바 이름, 윈도우 이름, 트랙바 최소값, 트랙바 최대값, 호출 함수)
	createTrackbar("low threshold", "Canny", 0, 1000, on_trackbar);
	createTrackbar("high threshold", "Canny", 0, 1000, on_trackbar);
	
	//트랙바 초기 값 지정
	setTrackbarPos("low threshold", "Canny", 50);
	setTrackbarPos("high threshold", "Canny", 150);

	Mat img_gray;
	img_gray = imread("1.jpg", IMREAD_GRAYSCALE);

	while (cam.read(frame))
	{
		if (frame.empty())
		{
			printf(" --(!) No camd frame -- Break!");
			break;
		}

		Show_canny(frame);
		
		if (waitKey(1) == 27)
			break;
	}
	destroyAllWindows();
	return 0;

}

void Show_canny(Mat frame) 
{
	int low = getTrackbarPos("low threshold", "Canny");
	int high = getTrackbarPos("high threshold", "Canny");

	Mat frame_gray;

	cvtColor(frame, frame_gray, COLOR_BGR2GRAY);

	Mat img_canny;
	Canny(frame_gray, img_canny, low, high);

	printf("low : %d             high :  %d\n", low, high);

	imshow("Canny", img_canny);
}
```

