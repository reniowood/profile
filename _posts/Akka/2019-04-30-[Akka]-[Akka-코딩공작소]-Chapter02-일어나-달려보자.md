---
layout: post
title: "[Akka 코딩공작소] Chatper 2. 일어나 달려보자"
categories: Akka
---

## 복제, 빌드, 인터페이스 테스트

예제 프로젝트 복제하기

```[bash]
git clone https://github.com/gilbutITbook/006877
```

Java 10은 0.13.17 이후의 sbt 버전을 사용해야 한다.

### sbt로 빌드하기

```[bash]
  sbt assembly # 코드를 컴파일하고 JAR로 묶는다
```

```[bash]
  sbt clean compile test # 타깃 빌드를 지우고 새로 컴파일한 다음 테스트를 실행한다
```

### [GoTicks.com](http://goticks.com) REST 서버 빨리 돌려보기

httpie 사용하면 편하다.

## 앱에 있는 액터 살펴보기

### 앱의 구조

ActorSystem을 먼저 만들고, akka-http의 Route를 이용해 HTTP 요청을 받아 액터로 메세지를 보내 요청을 처리한다.

- Route
  - akka-http가 제공하는 기능
  - HTTP 요청을 처리하는 방법을 편리한 DSL을 통해 정의한다.
- 액터 시스템의 컨텍스트가 아닌 자신의 컨텍스트로 만든 액터는 그 액터의 자식이 된다.

## 클라우드 속으로

헤로쿠 툴벨트([https://devcenter.heroku.com/articles/heroku-cli](https://devcenter.heroku.com/articles/heroku-cli)) 설치가 필요하다.

```[bash]
brew install heroku/brew/heroku
```

헤로쿠 계정도 만들어야 한다.

### 헤로쿠에 배포 및 실행하기

번역본 소스코드를 git에서 clone하여 실행하는 경우

```[bash]
git remote add heroku https://git.heroku.com/serene-beyond-12232.git
git subtree push --prefix chapter-up-and-running heroku master

http POST https://serene-beyond-12232.herokuapp.com/events/RHCP tickets:=250
http POST https://serene-beyond-12232.herokuapp.com/events/RHCP/tickets tickets:=4
```
