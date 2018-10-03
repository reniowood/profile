---
layout: post
title: "Spring Web MVC - 1. DispatcherServlet"
categories: Spring
---

다음 내용은 Spring Web MVC 5.1.0.RELEASE 버전의 공식 문서인 [Web On Servlet Stack](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/web.html)의 Spring Web MVC 부분 중 일부분을 발췌해 번역해 놓은 것입니다. 링크된 공식 문서의 내용과 다른 부분이나 생략된 부분이 있을 수 있습니다.

# Spring Web MVC

## DispatcherServlet

Spring MVC는 위임된 컴포넌트가 요청을 처리할 수 있는 공통된 알고리즘을 제공하는 ```Servlet```과 프론트 컨트롤러 패턴을 바탕으로 설계되었다.

```DispatcherServlet```은 여느 ```Servlet```과 마찬가지로 자바 configuration이나 ```web.xml```을 이용해 정의하고 설정한다. ```DispatcherServlet```은 Spring configurtaion을 이용해 요청 할당, 뷰 처리, 예외 처리에 필요한 위임된 컴포넌트를 찾는다.

### Context Hierarchy

```DispatcherServlet```은 ```ApplicationContext```의 확장인 ```WebApplicationContext```를 자신의 configurtaion으로 설정한다. ```WebApplicationContext```는 ```ServletContext```과 ```Servlet```에 대한 링크를 가지고 있다. ```WebApplicationContext```도 마찬가지로 ```ServletContext```에 연결되어 있다.

대부분의 어플리케이션은 하나의 ```WebApplicationContext```만 가지고 있어도 충분하다. 하나의 최상위 ```WebApplicationContext```를 여러 ```DispatcherServlet```이 공유하고, 각 ```DispatcherServlet```은 자식 ```WebApplicationContext```를 가질 수도 있다.

최상위 ```WebApplicationContext```는 여러 ```Servlet```이 공유하는 데이터 저장소나 비즈니스 서비스와 같은 기본적인 bean을 가진다. 이 bean들은 자식 ```WebApplicationContext```이 상속하고 재선언할 수도 있다.

### Special Bean Types

```DispatcherServlet```은 요청을 처리하고 응답을 만들기 위해 특별한 bean에 위임한다. 이 beans들은 Spring이 관리한다.

* HandlerMapping
  * 요청을 핸들러에 매핑한다.
* HandlerAdapter
  * 요청에 매핑된 핸들러를 실행한다.
* HandlerExceptionResolver
  * 예외를 확인하고 해당하는 핸들러나 HTML 에러 뷰에 넘겨준다.
* ViewResolver
  * 핸들러가 반환한 ```String``` 뷰 이름을 실제 ```View```로 변환한다.
* LocalResolver
  * 클라이언트가 사용한 ```Locale```을 확인해 국제화된 view를 제공한다.
* MultipartResolver
  * 파일 업로드와 같은 multipart 요청을 파싱한다.

### Web MVC Config

어플리케이션은 특별한 bean 목록에 있는 bean을 선언할 수 있다. ```DispatcherServlet```은 ```WebApplicationContext```를 확인하고, 필요한 bean이 없을 경우엔 ```DispatcherServlet.properties```에 나열된 기본 bean을 사용한다.

대부분의 경우엔 MVC Config(@EnableWebConfig를 사용한 Java configuration 등)에 필요한 bean을 정의한다.

### Servlet Configuration

```WebApplicationInitializer```는 Servlet 구현을 찾아내고 Servlet 컨테이너를 자동으로 초기화하는 데 사용하는 Spring MVC가 제공하는 인터페이스이다. ```AbstractDispatcherServletInitializer```는 ```DispatcherServlet```을 좀 더 쉽게 등록할 수 있도록 해주는 ```WebAplicationInitializer```의 추상 클래스이다.

```AbstractDispatcherServletInitializer```는 ```Filter``` 인스턴스를 추가할 수 있는 편리한 방법을 제공하고 자동으로 ```DispatcherServlet```에 추가해준다.

```DispatcherServlet```을 맞춤 설정하기 위해서는 ```createDispatcherServlet```을 오버라이드하면 된다.

### Processing

```DispatcherServlet```은 다음과 같은 방식으로 요청을 처리한다.

* ```WebApplicationContext```를 찾아 요청을 처리할 컨트롤러나 다른 요소의 속성으로 할당해준다.
* locale resolver가 필요한 경우엔 요청의 locale을 처리할 수 있도록 연결된다.
* theme resolver가 필요하면 요청에 연결된다.
* multipart file resolver를 정해놓았다면 요청에 multipart가 있는지 검사한다. multipart가 있다면 요청을 ```MultipartHttpServletRequest```로 감싼다.
* 요청을 처리할 핸들러를 찾는다. 핸들러를 찾으면 핸들러와 연관된 실행 체인(전처리기, 후처리기, 컨트롤러)을 실행한다.
* 모델이 반환되면, 뷰를 렌더링한다.

```WebApplicationContext```에 정의된 ```HandlerExceptionResolver``` bean이 처리 도중 발생한 예외를 처리한다. 예외를 처리하기 위한 로직을 따로 설정할 수도 있다.

Spring ```DispatcherServlet```은 Servlet API에 명시된 ```last-modification-date```를 반환할 수 있도록 지원한다. ```DispatcherServlet```은 ```LastModified``` 인터페이스를 구현한 핸들러를 찾아 해당 인터페이스의 ```log getLastModified(request)``` 메소드의 반환값을 전달한다.

```web.xml```의 Servlet 선언에 Servlet 초기화 파라미터를 추가함으로써 개별 ```DispatcherServlet``` 인스턴스를 설정할 수 있다.

* DispatcherServlet 초기화 파라미터
  * ```contextClass```
    * ```WebApplicationContext```를 구현한 클래스. 기본값은 ```XmlWebApplicationContext```
  * ```contextConfigLocation```
    * context를 찾을 위치를 나타내는 문자열. 콤마로 구분된 여러 문자열로 여러개의 context를 사용할 수 있다. 다른 위치에서 두 번 정의된 bean의 위치를 추가한 경우엔 뒤쪽에 있는 위치가 우선순위를 가진다.
  * ```namespace```
    * ```WebApplicationContext```의 namespace. 기본값은 ```[servlet-name]-servlet```
  * ```throwExceptionIfNoHandlerFound```
    * 요청을 처리할 핸들러를 찾지 못했을 대 ```NoHandlerFoundException```을 던질지 결정한다. 해당 예외는 ```HandlerExceptionResolver```가 처리한다.
    * 기본값은 ```false```이다. 이 때 ```DispatcherServlet```은 예외를 발생시키지 않고 404 상태 코드 응답을 돌려준다.