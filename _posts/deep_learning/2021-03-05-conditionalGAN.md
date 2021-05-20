---
layout: post
title:  "Conditional GAN"
description: 공부한 내용입니다.
date:   2021-03-05
categories: [Deep Learning]
comments: true
---

본 논문은 [여기](https://arxiv.org/abs/1411.1784)에서 볼 수 있습니다.

[https://greeksharifa.github.io/generative%20model/2019/03/19/CGAN/](https://greeksharifa.github.io/generative%20model/2019/03/19/CGAN/)를 참고해서 만들었습니다.

# cGAN

Conditional GAN은,다음과 같은 특징을 갖는다.

- y 라는 condition을 feed해서, G, D를 조절한다.

- 생성될 데이터의 라벨을 제어할 수 있다.

- c가 추가된다는 조건 외에는 전체적인 구조가 GAN과 동일하다.



# condition y란?

먼저 기존 GAN의 목적함수는 다음과 같다.

![설명](/assets/img/Deep_learning/210305/1.PNG)


그리고 cGAN의 목적함수는 다음과 같다.


![설명](/assets/img/Deep_learning/210305/2.PNG)

수식은 다음과 같고, 실제 구현시에는 y 가 각각 input(G : pz(z), D : input image x)에 합쳐진 형태로 구현된다.


# condition GAN의 구조


![설명](/assets/img/Deep_learning/210305/3.PNG)


G 먼저 살펴보자.

1) latent vector z, y는 각각 ReLU 를 통해 200, 1000의 노드로 hidden layer를 통해 매핑된다.

2) 그 두 노드가 합쳐져서 1200개의 노드로 합쳐진다.

3) 1200개의 노드가 784의 노드로 sigmoid layer 를 통해 매핑된다.


D는 maxout activation 을 통해 진행되었다.

x의 경우 240 units with 5 piece, y 는 50units with 5 pieces에서, maxout layer를 거쳐 240 units with 4 pieces로 매핑된 후 sigmoid 를 통해 단일 값으로 매핑되었다 한다.

maxout activation은 요즘 쓰이지 않는 개념인데, dense layer와 비슷한 개념이라고 생각했다.

코드상으론 다음과 같다.

input에 y가 concat된 형태로 네트워크에 들어가는게 핵심이다.

torch에선 nn.embeding()를 사용하여 구현한 코드들이 많은데, 자세히 살펴보지는 않았다.

![설명](/assets/img/Deep_learning/210305/4.PNG)



# Experiment


![설명](/assets/img/Deep_learning/210305/6.PNG)

G, D에 각각 원하는 라벨(ex. 0 : 0 : [1 0 0 0 0 0 0 0 0 0])로 one hot 인코딩 한 형태로 y를 집어넣어준 모습이다.

