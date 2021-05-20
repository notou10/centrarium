---
layout: post
title:  "OpenCV-15 Face mask"
description: C++ Opencv, harr cascade를 통해 실시간으로 input에 마스크를 씌우는  내용입니다.
categories : [openCV]
date:   2021-01-26
comments: true
---

본 포스팅은 [여기](https://docs.opencv.org/4.1.0/d2/de8/group__core__array.html#ga84b2d8188ce506593dcc3f8cd00e8e2c)를 참고하여 작성하였습니다.


<blockquote align="center">  구현 방향  </blockquote>

1. Harr cascade 알고리즘을 통해 face 영역을 실시간으로 프레임을 받아 detect 한다.

2. face가 detect된 경우 각 face의 크기에 맞게 mask를 씌운다. 이 떄 mask는 입력을 받아 선택할 수 있다.

위와 같은 방향으로 프로젝트를 진행하였다.


<blockquote align="center">  cv::threshold()  </blockquote>

![1](/assets/img/OpenCV/210127/4.PNG)

각 인자로 

입력 Img

결과 Img

threshold

maxval

flag가 들어간다.

(flag가 default(THRESH_BINATY)인 경우) : threhold를 넘을 경우, 그 픽셀 값을 maxval로 바꾸어주고, 그렇지 않을시 0으로 바꾼다.


(flag가(THRESH_BINATY_INV)인 경우) : threhold를 넘을 경우, 그 픽셀 값을 0로 바꾸어주고, 그렇지 않을시 maxval으로 바꾼다.



<blockquote align="center">  cv::bitwise_xor()  </blockquote>

![1](/assets/img/OpenCV/210127/5.PNG)

src1, src2를 비교하여, 두 픽셀의 intensity 값이 같으면 0(intensity 0), 다르면 1(intensity 255)로 하여 두 이미지의 다른 부분을 가시화 하여 dst에 담아준다.





<blockquote align="center">  배경 제거  </blockquote>

배경 제거에는  cv::threshold,   cv::bitwise_xor 함수를 사용했다.

![1](/assets/img/OpenCV/210127/3.PNG)

앞선 포스팅에서 흰 배경이 rgb값이 0으로 이루는 PNG파일도 있기에, 위 과정을 수행했다.

mat을 gray scale로 바꾸어 주고, 

threshold를 통해 mask1에는 픽셀의 intensity가 240 이상이면 검은색, 240이하면 흰 색으로 mat 을 선언

그리고 mask2에는 10 이상이면 검은색, 10이하면 흰 색으로 선언해 준다.

선언해준 모습은 그림의 d, e를 통해 볼 수 있다.

그리고 d와 e를 xor 연산해주어 xor = 1인 부분만 흰 색(intensity = 255)으로 적용해준다.

마지막으로 마스크에서 intensity 값이 255인 부분만 copy 하여 ImageROI부분에 붙여넣어, 얼굴에 마스크가 씌이도록 해준다.


<blockquote align="center">  CODE  </blockquote>


```cpp


#include <iostream>
#include <opencv2/opencv.hpp>
#include <stdio.h>

using namespace std;
using namespace cv;

String face_cascade_name;
CascadeClassifier face_cascade;
Mat mask1 = cv::imread("C:\\opencv-4.1.0\\210122_facemask\\Project1\\Project1\\11.PNG");
Mat mask2 = cv::imread("C:\\opencv-4.1.0\\210122_facemask\\Project1\\Project1\\1.jpg");

void putMask(cv::Mat src, cv::Point center, cv::Size face_size, const cv::Mat& mask);

int main()
{
    cv::Mat mask = mask1;
    int is_applied = 0;
    face_cascade_name = "C:\\opencv-sources\\opencv-4.1.0\\data\\haarcascades\\haarcascade_frontalface_alt2.xml";
    if (!face_cascade.load(face_cascade_name))      //CascadeClassifier로 미리 trained 된 haarcascade_eye 선언
    {
        printf("해당 모델 없음\n");
        return -1;
    };


    VideoCapture cap1(1);        //opecncv에서 욎아 카메라에서 영상 받아놓는 역할
    Mat frame;

    while (1)
    {
        cap1.read(frame);
        
        if (is_applied == 1)
        {
            vector<Rect> faces;
           face_cascade.detectMultiScale(frame, faces, 1.1, 2, 0, Size(0, 0), Size(500, 500));       //얼굴 검출

            if (faces.size())
            {
                cout << faces.size() << "------------------" <<  faces[0].width << "--------------------------" << faces[0].height << "------------------" << faces[0].x <<"------------------" << faces[0].y << endl;

            }
            for (int i = 0; i < faces.size(); i++)
            {
                cv::Point center(faces[i].x + faces[i].width * 0.5, faces[i].y + faces[i].height * 0.5);           
                putMask(frame, center, cv::Size(faces[i].width, faces[i].height), mask);
                imshow("Video", frame);
            }

        }

        else if (is_applied == 0)
        {
            imshow("Video", frame);
        }


        int key = waitKey(10);

        if (key == 'c')
        {
            is_applied = 1;
        }

        else if (key == 'd')
        {
            is_applied = 0;
        }

        else if (key == 27)
        {
            break;
        }


        if (key == 'a')
        {
            mask = mask1;
        }

        if (key == 'b')
        {
            mask = mask2;
        }


    }

}

void putMask(cv::Mat src, cv::Point center, cv::Size face_size, const cv::Mat& mask)
{
   
    cv::Mat mask_temp, src1;
    cv::Mat mask_clone = mask.clone();
    cv::resize(mask_clone, mask_temp, face_size);

    cv::imshow("mask_clone", mask_clone);

    Mat imageROI(src, cv::Rect(center.x - face_size.width / 2, center.y - face_size.height / 2, face_size.width, face_size.height));

    Mat mask_gray;
    cvtColor(mask_temp, mask_gray, COLOR_BGR2GRAY);
    
    cv::Mat mask_image1, mask_image2;
    cv::threshold(mask_gray, mask_image1, 245 ,255, cv::THRESH_BINARY);
    cv::threshold(mask_gray, mask_image2, 10, 255, cv::THRESH_BINARY);
    
    cv::Mat mask_image;
    cv::bitwise_xor(mask_image1, mask_image2, mask_image);
    imshow("s", mask_image);
    imshow("d", mask_image1);
    imshow("e", mask_image2);


    mask_temp.copyTo(imageROI, mask_image);      //mask_gray에서 0이 아닌 값에만 mask_temp 값 복사

}

```

<blockquote align="center">  결과  </blockquote>

![1](/assets/img/OpenCV/210127/2.PNG)
