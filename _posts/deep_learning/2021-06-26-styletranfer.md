---
layout: post
title:  "Image Style Transfer Using Convolutional Neural Networks 논문 리뷰" 
description: Image Style Transfer Using Convolutional Neural Networks 논문을 읽고 요약한 내용입니다.
date:   2021-06-26
categories: [Deep Learning]
comments: true
author : Dongkyun kim
---

본 논문은 [여기](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Gatys_Image_Style_Transfer_CVPR_2016_paper.pdf)에서 확인하실 수 있습니다.

![1](/assets/img/Deep_learning/210626/1.PNG)

### Abstract
---

* 본 연구는 이미지 내에서 객체에 해당하는 content와, 전반적인 화풍을 담당하는 style을 분리하여, 이를 transfer 하도록 한 연구이다.

* 기존에는 semantic information을 잘 담은 representation을 찾지 못해 이미지의 style/content 분리가 힘들었다.

* CNN의 발달로, objection recognition을 통해 얻은 high level information을 잘 표현할 수 있게되어, style/content 분리를 잘 할수 있게 되었다.

### Intro
---
기존의 style transfer 연구는 texture transfer에 국한된 non-parametic problem이었다.

Target image의 semantic information까지 추출하여 transfer하는게 진정한 style transfer이라 말할 수 있지만, 기술력의 부족으로 low level image feature of target만 transfer하였다.

기술의 발달로, high level representation을 통해 semantic 정보까지 잘 담은 style을 content와 독립적으로 transfer할수 있게 되었다.


### 2.0 Deep learning representation
---

본 연구에서 feature map을 뽑을 때, VGG19의 normalized 버전을 사용하였다.

이 버전은 16 conv layer, 5 pooling layer, no fcn 으로 이루어진다.

Pooling layer로는 maxpooling 대신 average pooling을 사용하였는데, 이는 성능이 더 좋았기에 사용했다고 한다.

수치상으로 더 좋은게 아니고 good appealing 이었다고 함.

(mean activation의 뜻이 헤깔림)
![1](/assets/img/Deep_learning/210626/2.PNG)
각 conv layer, activation을 거친 픽셀당  평균이 1이 되도록 normalize 했다고 한다.

#### 2.0.1 hierarcy에 따른 style representation, content representation의 이해

![1](/assets/img/Deep_learning/210626/4.PNG)

Conv layer를 거쳐 나온 content에 대한 representation부터 살펴보자. 

밑에 그림 a, b, c, d, e 는 각 conv layer를 거쳐 나옹 representation들을 reconstruct하여 input image와 비슷한 모양으로 구성한 것이다.(conv 반대로 진행하고 평균내었다 생각함)

Higher layer로 갈수록 지엽적인 detail pixel information에 대해서는 부정확한 양상을 보이지만 그 이미지가 담고있는 content에 대해서는 잘 보존된(의미를 잘담은) 모습을 보인다.

따라서 higher layer을 거친 결과(loss에서 추후 사용)물을 수치로써 사용한다.

Style에 대한 representation을 살펴보자.

Style에 대한 information은 gamma matrix(texture, style을 잘 설명)을 통해 얻을 수 있다.

Content representation과 달리 모든 level의 conv layer의 output에서 gamma matrix를 구하여 loss를 구하는데 사용한다.

Higher level로 갈수록 이미지에 대한 style은 더 잘 담는 반면, 그 이미지의 전체적인 배열, 구조는 뭉게지는(부정확한) 모습을 보인다.

### 2.1 Content representation
---
앞서 말했듯이, hierarcy가 higher layer로 갈수록 object(content) representation이 명료해진다.

따라서 loss를 구할때는, 최상단에 있는 layer의 output만을 사용한다.






![1](/assets/img/Deep_learning/210626/4.PNG)
