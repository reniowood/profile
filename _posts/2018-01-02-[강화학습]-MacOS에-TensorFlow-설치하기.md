---
layout: post
title: "[강화학습] MacOS에 TensorFlow 설치하기"
categories: AI

---

TensorFlow는 Google에서 제공하는 인공지능을 위한 오픈소스 소프트웨어 라이브러리이다. MacOS에 TensorFlow를 설치하기 위해서는 virtualenv, pip, Docker 등을 이용해야 하나, Google에서 추천하는 방법은 독립된 환경을 만들어 사용할 수 있는 virtualenv를 추천한다.

[MacOS에 virtualenv 설치하기]({% post_url 2018-01-02-[Python]-MacOS에-virtualenv-설치하기 %})

## virtualenv로 설치하기

1. virtualenv 환경을 새로 만든다. python 3을 사용하기 위해 ```-p python3``` 옵션을 사용한다. 설치하기 원하는 디렉토리 위치를 ```targetDirectory```로 정해준다.

   ```shell
   $ virtualenv --system-site-packages -p python3 [targetDirectory]
   ```

2. virtualenv 환경을 활성화한다.

   ```shell
   $ source ~/tensorflow/bin/activate
   ```

3. 이제 프롬프트 앞에 virtualenv 환경을 나타내는  ```([targetDirectory])``` 가 나타난다.

   ```shell
   ([targetDirectory]) $
   ```

4. 8.1버전 이상의 pip를 설치한다.

   ```shell
   ([targetDirectory]) $ easy_install -U pip
   ```

5. tensorflow를 virtualenv 환경에 설치한다.

   ```shell
   ([targetDirectory]) $ pip3 install --upgrade tensorflow
   ```

## TensorFlow 사용할 준비하기

매번 TensorFlow를 사용하기 위해서는 virtualenv 환경을 활성화해야 한다.

```shell
$ source ~/[targetDirectory]/bin/activate
```

virtualenv 환경을 비활성화하려면 ```deactivate``` 명령어를 사용한다.

```shell
([targetDirectory]) $ deactivate
```

## 설치한 TensorFlow 검증하기

설치한 TensorFlow가 잘 설치되었는지 검증해야 한다. 우선 virtualenv 환경을 활성화한다.

```shell
$ source ~/[targetDirectory]/bin/activate
```

다음과 같은 간단한 TensorFlow 프로그램을 작성한다.

```python
# hello.py
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```

작성한 프로그램을 실행해본다. ```Hello, TensorFlow!``` 라는 문구를 표시해야한다.

```shell
([targetDirectory]) $ python3 hello.py
```



참조: TensorFlow 공식 문서 (https://www.tensorflow.org/install/install_mac)