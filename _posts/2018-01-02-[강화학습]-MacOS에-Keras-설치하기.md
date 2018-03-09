---
layout: post
title: "MacOS에 Keras 설치하기"
categories: AI
---

Keras는 TenserFlow, CNTK, Theano 중 하나를 기반으로 돌아가는 Python으로 작성한 인공신경망 API이다.

케라스는

* 쉽고 빠른 프로토타이핑을 도와준다.
* CNN (Convolutional Neural Network) 과 RNN (recurrent network) 을 모두 지원한다.
* CPU와 GPU 모두에서 돌아간다.

는 장점을 가지고 있다.

케라스를 설치하기 위해서는 기반 프레임워크인 TenserFlow, Theano, CNTK 중 하나를 먼저 설치해야 한다.

[MacOS에 TensorFlow 설치하기]({% post_url 2018-01-02-[강화학습]-MacOS에-TensorFlow-설치하기 %})

## pip로 설치하기

Keras는 pip를 통해서나 소스를 직접 받아 설치할 수 있지만, 공식 문서는 pip로 설치하기를 권한다. 이미 TenserFlow를 virtualenv로 설치했다면 해당 virtualenv 환경에서 설치하면 된다.

```shell
([tenserFlowDirectory]) $ pip install keras
```

## 설치 확인하기

Keras의 [github repository](https://github.com/keras-team/keras) 의 examples 디렉토리를 살펴보면 여러 예제들을 확인할 수 있다. 그 중 하나인 [mnist_cnn.py](https://raw.githubusercontent.com/keras-team/keras/master/examples/mnist_cnn.py)를 다운받아 실행해보면 실제로 설치가 제대로 되었는지 확인할 수 있다.

```shell
([tenserFlowDirectory]) $ python mnist_cnn.py
```

참조: Keras Documentation (<https://keras.io/#installation>)
