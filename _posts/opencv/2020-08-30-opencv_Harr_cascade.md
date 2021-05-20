---
layout: post
title:  "OpenCV-6 Haar cascade"
description: C++ Opencv webcam을 이용한 Haar cascade 구현입니다.
categories : [openCV]
date:   2020-08-28
comments: true
---

본 포스트는 [여기](https://webnautes.tistory.com/1352), [여기](http://www.gisdeveloper.co.kr/?p=7208), [여기](https://www.youtube.com/watch?v=F5rysk51txQ) , [여기](https://blog.naver.com/dic1224/220989033563) 를 참고하여 만들었습니다.

<blockquote align="center"> Haar Feature Algorighm </blockquote>

CNN 이전 물체의 특징 기반 검출 알고리즘인 Haar cascade 를 이용한 얼굴 검출이다. 

Haar-feature 알고리즘은 Alfred Haar에 의해 1909년 제안되었는데, 오늘날의 컨볼루션 커널과 매우 유사하다.

 알고리즘을 학습시키기 위해, 검출할 물체가 있는 Positive image와 물체가 없는 Negative image를 input으로 넣고, 특징을 추출한다. 

특징은, 위와 같은 사각형들을 물체에 대해 컨볼루션하여 수치로 나타내서 판별한다.

![1](/assets/img/OpenCV/0902/2.PNG)
![1](/assets/img/OpenCV/0902/3.PNG)


사진상의 배경을 보면, 픽셀당 intensity를 0~255라고 가정할 떄, 그림의 1-a feature 을 씌워보면 feature 박스 안에 해당하는 값이 모두 124 정도의 값을 가질 것이다.

반면 눈썹 부분을 보면, 그림의 1-a feature를 씌워보면 눈썹 위 피부에 해당하는 명도 값은 상대적으로 밝은 50 정도의 값을 띌 것이고, 눈썹은 상대적으로 어두은 200정도의 값을 띌 것이다.

따라서 feature에 의해 눈썹이 검출된다.

코 부분도 마찬가지 원리로 코가 오똑 솟은 부분은 밝은 값, 팔자 주름이 있는 부분은 상대적으로 어두운 값은 갖기에 line feature을 대입해 보면 코로 검출됨을 알 수 있다.


![1](/assets/img/OpenCV/0902/4.PNG)

각 feature에 대해 Black pixel field - White pixel field의 값의 이상적인 값을 1이라고 할 때, 검출된 물체의 feature의 Black pixel field - White pixel field 값이 어느 정도 값(>0.7?) 을 넘으면 feature라고 판단하는 원리이다. (위사진 에서 명도 값은 0~1 범위로 normalized 된 형태)


<blockquote align="center"> Integral Image </blockquote>

![1](/assets/img/OpenCV/0902/5.PNG)

이러한 feature 들을 검출하기 위해 필요한 연산량은 상당히 크고, 연산량을 줄이기 위한 알고리즘인 Integral Image이다.

주어진 사진 크기에서 feature을 대입할 때, feature box의 크기의 변형이 무궁무진하기에 time complexity로 quadratic algorithm time complexity인 O(N^2)만큼의 값을 갖는다. 

Time complexity로 O(1) 상수의 값을 갖기 위해 Integral Image 기법이 사용되었다.

![1](/assets/img/OpenCV/0902/6.PNG)

우측을 빨간 박스에 해당하는 값은, 좌측의 노랑 박스의 명도 값을 모두 더한 값이다.

![1](/assets/img/OpenCV/0902/7.PNG)

좌측의 노란 박스 명도의 SUM을 구할때 더하기 연산을 여섯 번 해야하지만, Integral Image기법을 사용하면, 3번의 사칙연산을 하면 되기에 time complexity 가 O(1)만큼으로 줄어든다.

본 기법을 위의 Haar Feature Algorighm 알고리즘에 대입하여 적용하면, Integral Image 두 번의 연산으로 ```( ) - ( )``` 형태를 얻을 수 있을것이고 연산량 또한 줄 것이다. 



<blockquote align="center"> AdaBoost Algorithm </blockquote>

![1](/assets/img/OpenCV/0902/8.PNG)
![1](/assets/img/OpenCV/0902/9.PNG)

Classifier의 성능을 점진적으로 학습시키기 위한 알고리즘이다.

Adaptive Boost 알고리즘의 약자인데, (weak)classifier가 학습하는 과정에서, 틀린 data에 대해 가중치를 더 부여하여 다음 classify하는 과정에 영향을 미치게 하는 알고리즘이다.

위 사진의 상단 3개 박스를 좌측부터 보면, 처음 classifier인 선은 + 3개에 대해 틀렸다.

따라서 다음 과정에서 틀렸던 3 개는 가중치를 부여받고, 또 classify하면 이 때에는 - 3개에 대해 틀렸다.(두 번쨰 박스)

마지막 학습 과정에서 역시 틀렸던 - 가 가중치를 받는다.

학습 과정에서 연속적으로 틀리는 +들은 더 가중치를 받고, 반면 연속적으로 맞추는 +들은 가중치가 더욱 작아지는 방식이다.


마지막으로 위 세 가지 생성한 트리들을 합치면, 하단의 박스인 classifier를 얻을 수 있다.


이를 Haar 알고리즘에 대입하면,

 training data로서 객체가 있는/없는 영상은 Positive/Negative,

 feature은 classifier 역할(얼굴 윤곽을 찾는)을 한다.

Positive, Negative 이미지를 방금 그림의 +, -라 생각하자.

각 Positive, Negative 이미지의 수를 N, M 이라 할떄 이들의 초기 가중치는 1/N, 1/M 이다.


모든 feature에 대해 training data를 얼마 잘 분류하는지 수치화 하고, 그 중 수치가 가장 높은 feature를 weak classifier로 놓는다.

해당 weak classifier에 대해 weighted error를 구한다.(잘 맞추었으면 error rate가 낮고, 해당 classifier의 가중치가 커진다.)


위 과정을 T 번 반복하고, 해당 iteration에서의 weak classifier에 대해 가중치와 곱한뒤 합 연산을 하면 최종 classifier가 만들어진다.

 
![1](/assets/img/OpenCV/0902/10.PNG)


<blockquote align="center"> Code </blockquote>


```cpp

#include <opencv.hpp>
#include <iostream>
#include <stdio.h>

using namespace std;
using namespace cv;

void detectAndDisplay(Mat frame);

String face_cascade_name;
CascadeClassifier face_cascade;

int main()
{

    face_cascade_name = "C:\\opencv\\sources\\data\\haarcascades\\haarcascade_eye.xml";
    if (!face_cascade.load(face_cascade_name))      //CascadeClassifier로 미리 trained 된 haarcascade_eye 선언
    { 
        printf("해당 모델 없음\n"); 
        return -1; 
    };

    VideoCapture cam(1);        //opecncv에서 욎아 카메라에서 영상 받아놓는 역할
    Mat frame;

    cam.set(CAP_PROP_FRAME_WIDTH, 1280);       //프레임의 너비, 높이를 (1280, 720)으로 고정
    cam.set(CAP_PROP_FRAME_HEIGHT, 720);

    if (!cam.isOpened()) 
    { 
        printf("카메라 없음\n");
        return -1; 
    }


    while (cam.read(frame))         //frame 받아오기
    {
        if (frame.empty())
        {
            printf(" --(!) No camd frame -- Break!");
            break;
        }

        detectAndDisplay(frame);
        char c = (char)waitKey(3);     //해당 10ms 동안 esc 눌렸으면 break
        if (c == 27) { break; }
    }
    return 0;
}

//함수 정의
void detectAndDisplay(Mat frame)
{
    vector<Rect> faces;     //Rect 타입만큼 메모리 할당
    Mat frame_gray;

    cvtColor(frame, frame_gray, COLOR_BGR2GRAY);        //frame을 gray로 변환
    equalizeHist(frame_gray, frame_gray);       //해당 frame을 equalize (픽셀들의 화소값 범위가 좁다면(몰려있다면 넓혀준다.. contrast 중가)

    face_cascade.detectMultiScale(frame_gray, faces, 1.1, 2, 0, Size(0, 0), Size(50, 50));
    //얼굴을 검출해주는 함수
    //인자 7개 설명
    //(원본 이미지, 검출된 이미지, scalar factor, min neighbors, flags, 검출하려는 이미지 최소사이즈, 검출하려는 이미지 최대사이즈)

    //탐지한 객체 새기
    printf("%zd\n", faces.size());


    for (size_t i = 0; i < faces.size(); i++)       //객체개수 만큼 face 그리기
    {
        Point center(faces[i].x + faces[i].width / 2, faces[i].y + faces[i].height / 2);        //얼굴중 정 가운데 점 찎기

        ellipse(frame, center, Size(faces[i].width / 2, faces[i].height / 2),       //정 가운데 기준으로 타원 그리기
            0, 0, 360, Scalar(255, 0, 0), 4, 8, 0);
    }

    //ellipse 타원 인자 7개 설명
    //(그릴 대상 영상, 원 중심, 타원 크기, 타원의 각도, 호의 시작 각도, 호의 종료 각도, 선 색깔

    imshow("Face", frame);
}
```


<blockquote align="center"> 코드 설명 </blockquote>

 face_cascade.detectMultiScale의 인자 중 ```scalar factor, min neighbors, flags```
에서
 
  scalar factor :  이미지 피라미드에서 사용되는 scale factor

  min neighbors : 이미지 피라미드에 의한 여러 스케일의 크기에서 minNeighbors 횟수 이상 검출된 object는 valid 하게 하는 것
  
  flags : 1버전 cascade 쓸것인가

를 의미하는 것 같다.  (아래 사진 참고)

![1](/assets/img/OpenCV/0902/11.PNG)








