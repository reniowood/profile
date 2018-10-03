---
layout: post
title: "Spring Web MVC - 3. Annotated Controllers"
published: false
categories: Spring
---

다음 내용은 Spring Web MVC 5.1.0.RELEASE 버전의 공식 문서인 [Web On Servlet Stack](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/web.html)의 Spring Web MVC 부분 중 일부분을 발췌해 번역해 놓은 것입니다. 링크된 공식 문서의 내용과 다른 부분이나 생략된 부분이 있을 수 있습니다.

# Spring Web MVC

## Annotated Controllers

Spring MVC는 요청 할당과 예외 처리에 대한 어노테이션을 사용하는 ```@Controller```와 ```@RestController``` component를 기반으로 한 프로그래밍 모델을 제공한다. 어노테이션을 가진 컨트롤러는 기반 클래스를 상속하거나 특정 인터페이스를 구현할 필요 없이 유연한 메소드를 가질 수 있다.

```java
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String handle(Model model) {
        model.addAttribute("message", "Hello World!");
        return "index";
    }
}
```

위의 예제에서는 메소드가 ```Model```을 입력받아 문자열로 된 뷰 이름을 반환한다.

### Declaration

컨트롤러 bean은 Servlet의 ```WebApplicationContext```의 표준 Spring bean 정의를 사용해 정의할 수 있다. Spring은 ```@Component``` 어노테이션의 기능(classpath의 클래스 자동 감지 및 bean 정의 자동 등록)을 ```@Controller``` 어노테이션에 제공한다. 또한 ```@Controller```를 가진 클래스가 웹 component 역할임을 표시한다.

```@Controller``` bean을 자동 감지하기 위해서는 자바 configuration에 ```@ComponentScan```을 추가한다.

```@RestController```는 ```@Controller```와 ```@ResponseBody``` 어노테이션을 가진 복합 어노테이션으로, 컨트롤러의 모든 메소드가 ```@ResponseBody``` 어노테이션을 상속해 반환값을 HTML 템플릿에 쓰는 대신 응답 본문에 바로 쓴다.

#### AOP Proxies

컨트롤러를 런타임에 AOP 프록시를 이용해 호출할 때가 있다. 예를 들어 컨트롤러에 ```@Transactional``` 어노테이션을 추가하면 클래스 기반의 프록시를 사용한다. Spring Context 콜백이 아닌 인터페이스를 구현할 때에는 명시적으로 해당 클래스가 프록시의 대상임을 밝혀주어야 한다.

### Request Mapping

```@RequestMapping``` 어노테이션을 이용해 요청을 컨트롤러 메소드에 할당할 수 있다. URL이나 HTTP 메소드, 요청 파라미터, 헤더값, media type에 따라 할당이 가능하다. 클래스에 해당 어노테이션을 추가해 컨트롤러 메소드 전체에 공통된 할당을 할 수도 있다.

```@RequestMapping```의 HTTP 메소드를 특정한 단축 버전도 있다.

* ```@GetMapping```
* ```@PostMapping```
* ```@PutMapping```
* ```@DeleteMapping```
* ```@PatchMapping```

#### URI patterns

다음 패턴을 사용해 요청을 할당할 수 있다.

* ```?```는 하나의 문자에 대응된다.
* ```*```는 세부 경로에서 0개 이상의 문자에 대응된다.
* ```**```는 0개 이상의 세부 경로에 대응된다.

```@PathVariable```을 이용해 URI 변수값에 접근할 수 있다.

```java
@GetMapping("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
    // ...
}
```

클래스와 메소드 모두 URI 변수값에 접근할 수 있다.

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class OwnerController {

    @GetMapping("/pets/{petId}")
    public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```

URI 변수는 자동으로 적절한 타입으로 변환되거나 ```TypeMismatchException```을 던진다. 간단한 타입을 기본적으로 지원하며, 다른 타입을 추가로 지원할 수 있다.

```@PathVariable("customId")```와 같이 URI 변수의 이름을 명시할 수도 있다.

```{varName:regex}``` 문법은 URI 변수를 정규식과 함께 선언한다. 예를 들어, URL ```"/spring-web-3.0.5.jar"에 대해 다음 메소드는 이름, 버전, 확장자를 추출한다.

```java
@GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
public void handle(@PathVariable String version, @PathVariable String ext) {
    // ...
}
```

#### Pattern Comparison

두 개 이상의 패턴이 URL에 대응되는 경우, ```AntPathMatcher.getPatternComparator(String path)```를 이용해 더 구체적인 패턴을 찾아낸다.

```*```는 1점으로, ```**``` 2점으로, URI 변수를 각각 1점으로 매겼을 때, 더 높은 점수를 가진 쪽이 더 구체적인 패턴이다. 같은 점수일 때는 더 긴쪽이 선택된다. 같은 점수와 길이를 가졌을 때는 ```*```나 ```**```보다 URI 변수가 더 많은쪽이 선택된다.

```/**```는 점수를 셀 때 제외되며 항상 맨 마지막에 처리된다. 접두사 패턴(```/public/**```같은 것)은 ```**```을 가지지 않은 패턴보다 덜 구체적으로 간주된다.

#### Suffix Match

Spring MVC는 컨트롤러의 ```/person``` 패턴에 대해 ```/person.*```도 대응이 되도록 ```.*``` 접미사 패턴 매칭을 사용한다. 이 때의 확장자는 요청 content type을 해석하기 위해 사용된다.

브라우저가 해석하기 어려운 ```Accept``` 헤더를 보내던 시절에는 파일 확장자가 필요했지만 지금은 더 이상 필요하지 않다. 게다가 지금은 URI 변수, 경로 파라미터, URI 인코딩 등을 함께 사용할 때 모호하게 만든다. URL 기반의 인증과 보안도 더 힘들게 만든다.

파일 확장자 사용을 금지하려면 다음 두 가지를 모두 설정해야 한다.

* ```PathMatchConfigurer```의 ```useSuffixPatternMatching(false)```
* ```ContentNegotiationConfigurer```의 ```favorPathExtension(false)```

URL 기반의 content negotiation이 유용하기 때문에 파일 확장자를 사용하는 대신 쿼리 파라미터 기반의 전략을 추천한다. 만약 꼭 파일 확장자를 사용해야겠다면 ```ContentNegotiationConfigurer```의 ```mediaTypes```에 명시적으로 등록된 확장자만 사용하는 것을 고려한다.

#### Suffix Match and RFD

RFD(Reflected File Download) 공격은 XSS와 비슷하게 요청 입력값이 응답에 반영되도록 하지만, XSS와는 다르게 브라우저가 응답을 실행할 수 있는 파일로 다운받게 만든다.

Spring MVC의 ```@ResponseBody```와 ```ResponseEntity``` 메소드가 문제가 될 수 있다. 접두사 패턴 매칭을 사용하지 않고 경로 확장자를 사용하면 위험도를 낮출 수는 있지만 막을 수는 없다.

RFD 공격을 막기 위해서 Spring MVC는 응답 본문을 만들기 전에 ```Content-Disposition:inline;filename=f.txt```와 같은 헤더를 추가해 고정된 안전한 다운로드 파일을 제공한다. 이 기능은 URL 경로가 content negotiation을 위해 명시적으로 등록되거나 허용 목록에 있지 않으면 작동한다. 그러나 이 방법은 URL을 브라우저에 직접 입력할 때 부작용을 발생시킬 여지가 있다.

많은 공통 경로 확장자가 기본값으로 허용되어있다. 임의의 ```HttpMessageConverter``` 구현을 사용하게 되면 공통 경로 확장자들에 대해 ```Content-Disposition``` 헤더가 추가되지 않도록 파일 확장자를 등록할 수 있다.

#### Consumable Media Types

다음과 같이 요청의 ```Content-Type```을 기반으로 요청을 제한할 수 있다.

```java
@PostMapping(path = "/pets", consumes = "application/json") 
public void addPet(@RequestBody Pet pet) {
    // ...
}
```

```consumes```에 ```!text/plain```을 사용하면 ```text/plain```을 제외한 모든 content type이 사용 가능하다.

클래스에 ```consumes```를 추가하면 각 메소드의 ```consumes```는 클래스의 설정에 추가되지 않고 덮어 쓴다.

#### Producible Media Types

다음과 같이 요청의 ```Accept``` 헤더와 메소드가 만들어내는 content type을 기반으로 요청을 제한할 수 있다.

```java
@GetMapping(path = "/pets/{petId}", produces = "application/json;charset=UTF-8") 
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

media type은 character set도 지정할 수 있다. ```!```로 특정 content type을 제외하고 전부 받아들일 수도 있다.

클래스에 ```produces```를 추가하면 각 메소드의 ```produces```는 클래스의 설정에 추가되지 않고 덮어 쓴다.

#### Parameters, Headers

요청 파라미터 조건이나 헤더 값으로도 요청을 제한할 수 있다.

```java
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

```java
@GetMapping(path = "/pets", headers = "myHeader=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

#### HTTP HEAD, OPTIONS

```@GetMapping```(과 ```@RequestMapping(method=HttpMethod.GET)```)은 HTTP HEAD 요청 할당을 지원한다. 컨트롤러 메소드는 수정할 필요가 없다. ```javax.servlet.http.HttpServlet```에 적용된 응답 wrapper는 ```Content-Length``` 헤더가 응답 본문의 바이트수까지 포함하도록 보장한다.

```@GetMapping```(과 ```@RequestMapping(method=HttpMethod.GET)```)은 암묵적으로 HTTP HEAD 요청에 할당된다. HTTP HEAD 요청은 응답 본문을 쓰지는 않되, 응답 본문의 크기까지 포함한 전체 응답 바이트 수를 ```Content-Length``` 헤더에 담는것을 제외하면 HTTP GET 요청처럼 처리된다.

기본값으로 HTTP OPTIONS는 URL 패턴에 대응되며 허용하는 HTTP 메소드 목록에 ```Allow``` 응답 헤더가 설정되어있는 ```@RequestMapping``` 메소드에 의해 처리된다.

```@RequestMapping```에 HTTP 메소드 목록을 선언 하지 않으면 ```Allow``` 헤더는 ```GET,HEAD,POST,PUT,PATCH,DELETE,OPTIONS```로 정해진다. 컨트롤러 메소드는 반드시 지원하는 HTTP 메소드 목록을 선언해야 한다.

```@RequestMapping``` 메소드에 HTTP HEAD와 HTTP OPTIONS를 명시적으로 선언할 수 있지만, 대부분의 경우에 그럴 필요가 없다.

#### Custom Annotations

Spring MVC는 요청 할당을 위해 ```@RequestMapping``` 어노테이션을 가진 복합 어노테이션을 지원한다. ```@GetMapping```, ```@PostMapping```, ```@PutMapping```, ```@DeleteMapping```, ```@PatchMapping```이 그 예이다. 대부분의 컨트롤러 메소드를 특정 HTTP 메소드를 지원하도록 할당하기 때문에 이러한 복합 어노테이션을 제공한다.

임의의 요청 대응을 위한 요청 할당 속성을 추가하려면 ```RequestMappingHandlerMapping```를 상속하고 ```getCustomMethodCondition``` 메소드를 override 해야한다. 이 메소드를 통해 임의의 속성을 확인하고 직접 만든 ```RequestCondition```을 반환한다.

#### Explicit Registrations

다음과 같이 동적으로 등록하고 싶거나 특별한 용도를 위해 핸들러 메소드를 코드로 등록할 수 있다. 예를 들면 다른 URL을 같은 핸들러의 다른 인스턴스로 할당하는 경우에도 사용할 수 있다.

```java
@Configuration
public class MyConfig {

    @Autowired
    public void setHandlerMapping(RequestMappingHandlerMapping mapping, UserHandler handler) 
            throws NoSuchMethodException {

        RequestMappingInfo info = RequestMappingInfo
                .paths("/user/{id}").methods(RequestMethod.GET).build(); 

        Method method = UserHandler.class.getMethod("getUser", Long.class); 

        mapping.registerMapping(info, handler, method); 
    }

}
```

### Handler Methods

#### Method Arguments

다음은 컨트롤러 메소드 인자로 지원하는 타입 목록이다. Reactive 타입을 지원하지 않는다. JDK8의 ```java.util.Optional```은 ```required``` 속성을 가진 어노테이션과 함께 쓸 수 있다. 이 때 ```required```값은 ```false```가 된다.

* ```WebRequest```, ```NativeWebRequest```
  * 요청 파라미터와 요청 속성, 세션 속성 등에 접근할 수 있다.
* ```javax.servlet.ServletRequest```, ```javax.servlet.ServletResponse```
  * ```ServletRequest```나 ```MultipartRequest``` 등의 특정 타입을 선택한다.
* ```javax.servlet.http.HttpSession```
  * 세션을 강제로 만들어 ```null``값이 되지 않는다. 세션에 접근하는 것은 스레드 안전하지 않다.
* ```javax.servlet.http.PushBuilder```
  * HTTP/2 resource push를 위한 Servlet 4.0 push builder API. 클라이언트가 HTTP/2를 지원하지 않으면 값이 ```null```이다.
* ```java.security.Principal```
  * 현재 인증받은 사용자
* ```HttpMethod```
  * 요청의 HTTP 메소드
* ```java.util.Locale```
  * ```LocaleResolver```가 결정한 현재 요청의 locale
* ```java.util.TimeZone``` + ```java.time.ZoneId```
  * ```LocaleContextResolver```가 정한 현재 요청의 시간대
* ```java.io.InputStream```, ```java.io.Reader```
  * Servlet API가 제공하는 원본 요청 본문에 접근하기 위해 사용
* ```java.io.OutputStream```, ```java.io.Writer```
  * Servlet API가 제공하는 원본 응답 본문에 접근하기 위해 사용
* ```@PathVariable```
  * URI 템플릿 변수에 접근하기 위해 사용
* ```@MatrixVariable```
  * URI 세부 경로의 이름-값 쌍에 접근하기 위해 사용
* ```@RequestParam```
  * Servlet 요청 파라미터에 접근하기 위해 사용한다. 선언한 메소드 인자 타입에 따라 값이 변환된다.
* ```@RequestHeader```
  * 요청 헤더에 접근하기 위해 사용한다. 선언한 메소드 인자 타입에 따라 값이 변환된다.
* ```@CookieValue```
  * 쿠키에 접근하기 위해 사용한다. 선언한 메소드 인자 타입에 따라 값이 변환된다.
* ```@RequestBody```
  * HTTP 요청 본문에 접근하기 위해 사용한다. ```HttpMessageConverter``` 구현을 사용해 본문 내용을 선언한 메소드 인자 타입에 따라 변환한다.
* ```HttpEntity<B>```
  * 요청 헤더와 본문에 접근하기 위해 사용한다. ```HttpMessageConverter```가 변환한다.
* ```@RequestPart```
  * ```multipart/form-data```요청의 일부에 접근하기 위해 사용한다.
* ```java.util.Map```, ```org.springframework.ui.Model```, ```org.springframework.ui.ModelMap```
  * HTML 컨트롤러가 사용하고 뷰 렌더링을 위해 템플릿에 사용하는 모델에 접근하기 위해 사용한다.
* ```RedirectAttributes```
  * 리다이렉트에 사용되는 속성에 접근하기 위해 사용한다.
* ```@ModelAttribute```
  * 데이터 바인딩과 validation이 적용된 모델의 속성에 접근하기 위해 사용한다.
* ```Errors```, ```BindingResult```
  * ```@ModelAttribute``` 인자값의 validation과 데이터 바인딩의 에러 혹은 ```@RequestBody```나 ```@RequestPart```의 에러에 접근하기 위해 사용한다. 검증할 메소드 인자 바로 뒤에 ```Errors```나 ```BindingResult``` 인자를 선언해야 한다.
* ```SessionStatus``` + 클래스 단위 ```@SessionAttributes```
  * form processing을 완료해 클래스에 ```@SessionAttributes```로 선언한 세션 속성을 초기화한다.
* ```UriComponentBuilder```
  * 현재 요청과 관련된 URI를 만들기 위해 사용한다.
* ```@SessionAttribute```
  * 세션 속성에 접근하기 위해 사용한다.
* ```@RequestAttribute```
  * 요청 속성에 접근하기 위해 사용한다.
* 기타 다른 인자
  * 메소드 인자가 위에 설명한 값이 아닌 경우에 ```BeanUtils#isSimpleProperty```가 판단하기에 간단한 타입(원시 타입, ```String```, ```Date``` 등))이면 ```@RequestParam```으로, 아니면 ```@ModelAttribute```로 해석한다.

#### Return Values

다음 값들은 컨트롤러 메소드가 지원하는 반환값이다. Reactive 타입을 지원한다.

* ```@ResponseBody```
  * ```HttpMessageConverter``` 구현이 반환값을 변환해 응답에 쓴다.
* ```HttpEntity<B>```, ```ResponseEntity<B>```
  * ```HttpMessageConverter``` 구현이 반환값을 변환해 응답에 쓴다.
* ```HttpHeaders```
  * 헤더만 있고 본문이 없는 응답을 만든다.
* ```String```
  * ```ViewResolver``` 구현이 뷰 이름으로 해석한다. command object와 ```@ModelAttribute``` 메소드가 정한 암묵적인 모델과 함께 사용한다.
* ```View```
  * command object와 ```@ModelAttribute``` 메소드가 정한 암묵적인 모델과 함께 렌더링한다.
* ```java.util.Map```, ```org.springframework.ui.Model```
  * 반환값이 암묵적인 모델에 속성으로 추가되고, 뷰 이름은 ```RequestToViewNameTranslator```가 암묵적으로 정한다.
* ```@ModelAttribute```
  * 반환값이 모델에 속성으로 추가되고, 뷰 이름은 ```RequestToViewNameTranslator```가 암묵적으로 정한다.
* ```ModelAndView``` 객체
  * 사용할 뷰와 모델을 제공한다.
* ```void```
  * ```void```나 ```null```을 반환하는 함수가 ```ServletResponse``` 인자 또는 ```OutputStream``` 인자 또는 ```@ResponseStatus``` 어노테이션을 가진다면 응답을 완전히 작성한 것으로 간주한다. 양수값인 ```ETag```나 ```lastModified``` 타임스탬프 값을 만들었을 때도 똑같이 간주한다.
  
  만약 둘 다 아니라면 ```void``` 반환값 타입은 응답 본문이 없는 것을 의미하거나 뷰 이름을 기본값으로 사용한다는 것을 의미한다.
* ```DeferredResult<V>```
  * 나중에 다른 스레드가 비동기적으로 설정할 반환값을 미리 반환하기 위해 사용한다.
* ```Callable<V>```
  * Spring MVC가 관리하는 스레드에서 반환값을 비동기적으로 반환하기 위해 사용한다.
* ```ListenableFuture<V>```, ```java.util.concurrent.CompletionStage<V>```, ```java.util.concurrent.CompletableFuture<V>```
  * ```DeferredResult``` 대신 사용한다.
* ```ResponseBodyEmitter```, ```SseEmitter```
  * ```HttpMessageConverter``` 구현이 만드는 응답에 비동기적으로 사용할 객체의 스트림을 방출한다. ```ResponseEntity```의 본문으로도 사용 할 수 있다.
* ```StreamingResponseBody```
  * 응답의 ```OutputStream```에 비동기로 값을 쓴다. ```ResponseEntity```의 본문으로도 사용 할 수 있다.
* Reactive 타입 - Reactor, RxJava, ```ReactiveAdapterRegistry```를 통한 다른 값들
  * 여러 값을 가지는 스트림을 가진 ```DeferredResult``` 대신 사용한다.
  * streaming을 할 때(예를 들어, ```text/event-stream```, ```application/json+stream```)는 ```SseEmitter```와 ```ResponseBodyEmitter```가 대신 사용되며, Spring MVC가 관리하는 스레드가 ```ServletOutputStream```의 blocking I/O를 실행하며 쓰기를 다 마치면 back pressure가 적용된다.
* 다른 타입의 반환값
  * 반환값 타입이 위에 설명한 값이 아니고 ```String```이나 ```void```인 경우에 반환 값이 ```BeanUtils#isSimpleProperty```가 판단하기에 단순한 타입이 아니면 뷰 이름으로 정해진다. 단순한 타입이 아니면 해결되지 않는다.