---
layout: post
title: "MacOS에 virtualenv 설치하기"
categories: Python
---

virtualenv는 분리된 Python 환경을 만들어준다. 분리된 Python 환경이란 전역 site-packages를 함께 사용하는 환경이 아닌 다른 분리된 Python 환경과 라이브러리를 공유하지 않는 환경을 말한다.

## pip로 설치하기

pip를 이용해 설치할 때는 SSL 지원 문제로 인해 1.3버전 이상을 사용하여야 한다.

```
# pip install virtualenv
```

참조: virtualenv documentation (<https://virtualenv.pypa.io/en/stable/installation/>)
