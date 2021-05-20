---
layout: post
title:  "OpenCV-4 split and merge"
description: C++ Opencv split, merge visualization 입니다.
categories : [openCV]
date:   2020-08-25
comments: true
---

<blockquote align="center"> 싱글 채널로 visualization  </blockquote>

```cpp 
#include <opencv.hpp>
#include <iostream>

using namespace std;
using namespace cv;



int main() {
    // load an image
    Mat my_image = cv::imread("1.jpg", cv::IMREAD_COLOR);

    // create a matrix for split
    Mat bgr[3];
    //Mat BLUE_multi, GREEN_multi, RED_multi;
    // my_image와 사이즈가 같은 모두 0으로 채워진 single channel Matrix 생성.
    //CV_8UC1 = 픽셀 하나에 0-255 값 가질 수 있다는 뜻 맨 마지막 숫자 1은, 채널값 1이라는 뜻

    //Mat Converted_Image = cv::Mat::zeros(my_image.size(), CV_8UC1);


    // split image into 3 single-channel matrices
    split(my_image, bgr);




    //// create 4 windows
    namedWindow("Original Image",WINDOW_NORMAL);
    resizeWindow("Original Image", 600, 600);
    namedWindow("Red", WINDOW_NORMAL);
    resizeWindow("Red", 300, 300);
    namedWindow("Green", WINDOW_NORMAL);
    resizeWindow("Green", 300, 300);
    namedWindow("Blue", WINDOW_NORMAL);
    resizeWindow("Blue", 300, 300);

    //// show 4 windows
    imshow("Original Image", my_image);
    imshow("Blue", bgr[0]);
    imshow("Green", bgr[1]);
    imshow("Red", bgr[2]);



    cv::waitKey(0);
    return 0;
}
```

결과 이미지
![1](/assets/img/OpenCV/0825/2.PNG)



<blockquote align="center"> 멀티 채널로 visualization  </blockquote>

```cpp
#include <opencv.hpp>
#include <iostream>

using namespace std;
using namespace cv;



int main() {
    // load an image
    Mat my_image = cv::imread("1.jpg", cv::IMREAD_COLOR);

    // create a matrix for split
    Mat bgr[3];
    Mat BLUE_multi, GREEN_multi, RED_multi;
    // my_image와 사이즈가 같은 모두 0으로 채워진 single channel Matrix 생성.
    //CV_8UC1 = 픽셀 하나에 0-255 값 가질 수 있다는 뜻 맨 마지막 숫자 1은, 채널값 1이라는 뜻

    Mat Converted_Image = cv::Mat::zeros(my_image.size(), CV_8UC1);


    // split image into 3 single-channel matrices
    split(my_image, bgr);


    // create 3 channel Color Matrix
    Mat R[] = { Converted_Image, Converted_Image, bgr[2] };
    Mat G[] = { Converted_Image, bgr[1], Converted_Image };
    Mat B[] = { bgr[0], Converted_Image, Converted_Image };



    merge(B, 3, BLUE_multi);
    merge(G, 3, GREEN_multi);
    merge(R, 3, RED_multi);
    //cv::split(BLUE_multi, bgr);

    //// create 4 windows
    namedWindow("Original Image",WINDOW_NORMAL);
    resizeWindow("Original Image", 600, 600);
    namedWindow("Red", WINDOW_NORMAL);
    resizeWindow("Red", 600, 600);
    namedWindow("Green", WINDOW_NORMAL);
    resizeWindow("Green", 600, 600);
    namedWindow("Blue", WINDOW_NORMAL);
    resizeWindow("Blue", 600, 600);

    //// show 4 windows
    imshow("Original Image", my_image);
    imshow("Blue", BLUE_multi);
    imshow("Green", GREEN_multi);
    imshow("Red", RED_multi);



    cv::waitKey(0);
    return 0;
}
```



결과 이미지

![1](/assets/img/OpenCV/0825/1.PNG)






<blockquote align="center"> 설명  </blockquote>

싱글 채널, 멀티 채널의 Mat를 각각 채널 별로 visualization 해준 모습이다. 

싱글 채널에서는, 각 B, G, R 정보에 대해서 싱글 채널로 imshow 했기에, gray scale로 나왔고, 

멀티 채널에서는 각 B, G, R 정보를 먼저 뽑아, 

Mat R[], G[], B[]에 그 채널을 제외한 나머지 채널은 0인 값을 갖는 3 - channel Mat를 구현하였다.

그리고 merge 함수를 통해 Blue_multi 와 같은 Mat에 각각 담아주었다.

(merge( ,  , ) 첫 번째 인자 : Mat 배열, 두 번쨰 인자 : 채널 수(배열 크기), 세 번째 인자 : output으로 받을 Mat)
