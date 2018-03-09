---
layout: post
title: "Gradle 3.0의 dependencies 설정: api vs implementation"
categories: Gradle
---

Gradle이 버전 3.0으로 판올림하면서 dependencies의 설정값 중 compile대신 api나 implementation을 쓰도록 변경되었다.

## api vs implementation

그럼 어떤 상황에서 api를 쓰고, 어떤 상황에서는 implementation을 써야 할까?

### api

다른 라이브러리가 해당 라이브러리를 접근해야 하는 경우 사용한다.

라이브러리 A가 라이브러리 B를 api로 의존성 선언을 하였을 때, 라이브러리 A를 의존성으로 가지는 다른 라이브러리는 모두 라이브러리 B에 접근할 수 있다. 이는 기존 compile로 선언했을 때와 동일한 효과를 나타낸다.

### implementation

다른 라이브러리가 해당 라이브러리를 접근할 필요가 없을 때 사용한다.

api와는 다르게, implementation으로 의존성 선언을 하면 해당 라이브러리는 현재 라이브러리를 의존성으로 선언해도 다른 라이브러리가 접근할 수 없게된다.

## 왜?

이는 기존의 의존성 선언 방식은 어떤 라이브러리가 변경되었을 때 이를 의존성으로 선언한 다른 모든 라이브러리를 다시 컴파일하는 상황을 만들어내기 때문이다. 의존성의 그래프가 커지고 깊어질 수록, 특정 라이브러리가 변경되었을 때 다시 컴파일해야하는 상황이 빈번해질 것이다. 반면 implementation으로 의존성 선언을 하면 이런 상황이 확장되는 것을 막을 수 있다. 내가 사용하고 있는 라이브러리 중 내가 접근 가능한 라이브러리가 변경되었을 때만 다시 컴파일하면 되기 때문이다.

## 결론

기존 compile은 전부 implementation으로 바꾸고, 꼭 필요한 상황에서만 api를 사용하면 컴파일 시간이 짧아질 것이다.