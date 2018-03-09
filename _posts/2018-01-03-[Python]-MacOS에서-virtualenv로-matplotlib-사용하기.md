---
layout: post
title: "MacOS에서 virtualenv로 matplotlib 사용하기"
categories: Python

---

[파이썬과 케라스로 배우는 강화학습] 실습을 하다보니 MacOS에서 TensorFlow와 Keras를 virtualenv를 사용해 설치하고 matplotlib을 사용하는 예제 코드를 실행하다 생기는 문제가 있어 이를 정리해 보았다.

## 문제

matplotlib를 pip로 설치해서 전역 환경에서 사용할 경우에는 문제가 없지만 virtualenv 환경에서 사용할 경우 다음과 같은

> RuntimeError: Python is not installed as a framework. The Mac OS X backend will not be able to function correctly if Python is not installed as a framework. (…) Please either reinstall Python as a framework, or try one of the other backends. (...) See 'Working with Matplotlib on OSX' in the Matplotlib FAQ for more information.

오류 메세지를 보여주며 실행되지 않는다.

## 해결

virtualenv와 같은 도구를 사용하려면 matplotlib의 backend를 MacOS의 기본 backend인 'MacOSX'가 아닌 다른 backend로 (e. g. TkAgg) 변경해주어야 한다. 변경하는 방법에는 크게 두 가지 방법이 있는데

1. 코드상에서 직접 backend를 지정해주는 방법이 있다. 다만 matplotlib의 pyplot 등을 사용하기 위해선 다른 import 전에 use 메서드를 사용해야만 한다.

   ```python
   import matplotlib
   matplotlib.use('TkAgg')
   import matplotlib.pyplot as plt
   ```

2. [Customizing matplotlib - The *matplotlibrc* file](https://matplotlib.org/users/customizing.html#the-matplotlibrc-file) 를 참조해 matplotlibrc에 설정을 추가하는 방법이 있다.

   ```yaml
   backend: TkAgg
   ```
