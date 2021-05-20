---
layout: post
title:  "OpenCV-8 disparity map using realsense d415"
description: C++ Opencv realsense d415 depth camera를 이용한 IR 영상 검출 및 disparity map 추출 관련 내용입니다.
categories : [openCV]
date:   2020-09-29
comments: true
---

본 포스팅은 [librealsense2 doc 문서](http://docs.ros.org/kinetic/api/librealsense2/html/classrs2_1_1pipeline.html#af515f94223b63b932d64f54d4649ec8a), [여기](https://m.blog.naver.com/PostView.nhn?blogId=dldlsrb45&logNo=220879295400&proxyReferer=https:%2F%2Fwww.google.com%2F)를 참고하여 만들었습니다.


<blockquote align="center"> Stereo matching </blockquote>

![1](/assets/img/OpenCV/0928/3.PNG)

Stereo matching이란, 기준 영상의 한 점에 대해 목표 영상에서 동일한 점을 찾는 과정이다.

즉 기준 영상과 목표 영상의 차이인 disparity(시차)를 계산 하는 과정이라 할 수 있다.

기준 영상을 좌측으로 놓든 우측으로 놓든, 시차가 계산되어 나오는 영역만 다를 뿐 시차 값은 동일하다고 한다.

시차 값이 클수록(밝을 수록) 3차원적 거리가 가깝고, 시차 값이 작을수록(어두울 수록) 3차원적 거리가 멀다.





<blockquote align="center"> CODE </blockquote>

본 과제의 목표는 realsense 2 SDK 라이브러리를 통해 webcam으로 좌우 IR camera 에 영상을 입력받아, 이를 stereoBM을 통해 계산하여 disparity map을 추출하는 것이다. 

코드의 순서는 

STEP 1. stereoBM 선언 

STEP 2. pipeline에 인자 값(활성화시킬 스트림 등) 주기 

STEP 3. While 문을 돌며 키 입력이 있을 떄 까지 프레임 추출

**while 문 안의 내용**

3-1  **pipe.wait_for_frames()** 를 통해 새 프레임 세트를 앞서 선언한 frames 객체에 담음 

3-2 각 frame 객체 (Mat객체와 비슷하다 생각)에 frames 객체의 IR 2개, BGR1 개의 frame을 담음

3-3 각 Mat 객체를 선언 후 방금 선언한 frame 객체의 주소값을 인자로 주어, 값 할당

3-4 stereoBM의 compute 함수를 통해 disparity map 추출

3-5 imshow



```cpp

#include <librealsense2/rs.hpp>
#include <opencv2/opencv.hpp>
//#include <opencv2/calib3d.hpp>

using namespace std;
using namespace cv;


int main()
{
    Ptr<StereoBM> left_matcher = StereoBM::create(16, 9);   //stereobm 선언
    rs2::pipeline pipe;     //비전 처리 모듈과 사용자 interaction 효과적으로 해주기 위한 pipeline 선언,  카메라 구성 및 스트리밍, 비전 모듈 트리거링 및 스레딩을 추상화

    //Create a configuration for configuring the pipeline with a non default profile
    rs2::config cfg;       //config를 통해 파이프라인 스트림, 장치 선택, 필터 요청, 기기의 파이프 라인 요구 사항과 충돌이 없는지 테스트 가능

    //Add desired streams to configuration
    cfg.enable_stream(RS2_STREAM_INFRARED, 1, 640, 480, RS2_FORMAT_Y8, 30);       //장치 스트림 활성화. 	RS2_STREAM_INFRARED : RealSense 장치에서 생성 된 IR, 깊이 데이터의 기본 스트림
    cfg.enable_stream(RS2_STREAM_INFRARED, 2, 640, 480, RS2_FORMAT_Y8, 30);        //장치 스트림 활성화. 	RS2_STREAM_INFRARED : RealSense 장치에서 생성 된 IR, 깊이 데이터의 기본 스트림
    cfg.enable_stream(RS2_STREAM_COLOR, 640, 480, RS2_FORMAT_BGR8, 30);           //RS2_FORMAT_Y8 : 8-bit per-pixel grayscale image(픽셀당 8비트)   , 마지막 30 : 초당 가져오는 스트림 프레임
    //cfg.enable_stream(RS2_STREAM_DEPTH, 640, 480, RS2_FORMAT_Z16, 30);          //RS2_format : 어떻게 binary data가 프레임 내에서 인코딩 되었나 보여줌. 

    //RS2_FORMAT_Z16 : 16 - bit linear depth values.The depth is meters depth scale * pixel value
    //Instruct pipeline to start streaming with the requested configuration

    pipe.start(cfg);        //방금 준 config 파일을 통해 파이프라인과 스트리밍 시작
    while (1)
    {

        // Camera warmup - dropping several first frames to let auto-exposure stabilize
        rs2::frameset frames;       //frameset 클래스로 frames이란 객체 생성 

        frames = pipe.wait_for_frames();        //새 프레임세트를 사용할 수 있을 떄 까지 기다리기. 읽지않은 최신 프레임세트를 가져옴, 함수가 호출되지 않은 동안 생성된 장치 프레임은 삭제

        //Get each frame
        rs2::frame ir_frame_left = frames.get_infrared_frame(1);        //IR 형식의 첫 번쨰 프레임 검색
        rs2::frame ir_frame_right = frames.get_infrared_frame(2);
        rs2::frame BGR_frame_1 = frames.get_color_frame();


        // Creating OpenCV matrix from IR image
        //Mat ir(Size(640, 480), CV_8UC1, (void*)ir_frame.get_data());        //(void*)ir_frame.get_data(): data의 시작점(헤드) 위치를 가+르킴. data가 카피되는게 아니라 포인터 개념


        Mat IR_left(Size(640, 480), CV_8UC1, (void*)ir_frame_left.get_data());
        Mat IR_right(Size(640, 480), CV_8UC1, (void*)ir_frame_right.get_data());
        //Mat BGR_1(Size(640, 480), CV_8UC3, (void*)BGR_frame_1.get_data());
        Mat disp;

        left_matcher->compute(IR_left, IR_right, disp);
        disp.convertTo(disp,CV_8U);     //32비트로 뱉기에 이를 16비트로 바꿔줌

        //applyColorMap(ir, ir, COLORMAP_JET);  //컬러로 visualize 해주기 위해서

        //imshow("VIDEO_PLAYER", BGR_1);
        imshow("VIDEO_PLAYER_L", IR_left);
        imshow("VIDEO_PLAYER_R", IR_right);
        imshow("VIDEO_PLAYER_d", disp);


        if (waitKey(10) == 27) break;       //10ms 내에 키 입력되면 break
        }
    return 0;
}

```

결과 이미지 : 

![1](/assets/img/OpenCV/0928/2.PNG)



<blockquote align="center"> 힘들었던 점  </blockquote>

1) stereoBM 객체 선언부에서 인자 값을 주는 부분에서 많이 해멨는데, 결과적으로 default 값을 사용하면 코드 두 줄로 결과 추출 가능

2)  compute의 인자로 들어가는 left, right 둘 다 8 bit(0~255) 싱글 이미지 이고, 이를 통해 만들어준 disp 객체는 32bit floating point disparity map이다. 

이를 convertTo()를 이용하여 행렬 원소 타입을 CV_8U 즉 8비트로 바꾸어 imshow() 를 통해 visualize 해주면 된다.

디버깅시 함수에 커서를 올려 output의 원소 타입을 읽어보고 이를 변환해주는 습관을 들여야겠다.

참고로, CV_8UC1 에서 C1은 채널 1개를 뜻하기에, 생략하여 CV_8U 로 해주어도 똑같은 값을 갖는다.