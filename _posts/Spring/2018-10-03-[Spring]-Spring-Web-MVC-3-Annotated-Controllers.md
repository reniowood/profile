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
  * data binding과 validation이 적용된 모델의 속성에 접근하기 위해 사용한다.
* ```Errors```, ```BindingResult```
  * ```@ModelAttribute``` 인자값의 validation과 data binding의 에러 혹은 ```@RequestBody```나 ```@RequestPart```의 에러에 접근하기 위해 사용한다. 검증할 메소드 인자 바로 뒤에 ```Errors```나 ```BindingResult``` 인자를 선언해야 한다.
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

#### Type Conversion

컨트롤러 메소드의 인자가 ```String``` 기반의 요청 입력값 어노테이션을 (예를 들어 ```@RequestParam```, ```@RequestHeader```, ```@PathVariable```, ```@MatrixVariable```, ```@CookieValue```)가지고  타입이 ```String``` 타입이 아니라면 타입 변환이 필요할 수도 있다.

이런 경우에는 설정한 converter에 따라 자동으로 타입을 변환한다. 기본값으로 ```int```, ```long```, ```Date``` 등의 간단한 타입을 지원한다. ```WebDataBinder```을 통해서 혹은 ```FormattingConversionService```에 ```Formatters```를 등록하면 타입 변환을 설정할 수 있다. [Spring Field Formatting](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#format) 참조.

#### Matrix Variables

[RFC 3986](https://tools.ietf.org/html/rfc3986#section-3.3)은 세부 경로에서의 이름-값 쌍에 대해 이야기한다. Spring MVC는 이 이름-값 쌍을 Tim Bernes-Lee의 [오래된 글](https://www.w3.org/DesignIssues/MatrixURIs.html) 을 기반으로 "matrix variable"이라고 부르지만, URI 경로 파라미터라고 부르기도 한다.

Matrix variable은 어느 세부 경로에도 나타날 수 있으며, 각 변수는 세미콜론으로 구분하고 두 개 이상의 값은 콤마로 구분한다. (예를 들어 ```/cars;color=red,green;year=2012```). 두 개 이상의 값은 변수 이름을 반복해서도 나타낼 수 있다. (예를 들어 ```color=red;color=green;color=blue```)

URL이 matrix variable을 가지고 있다면, 컨트롤러 메소드에 요청을 할당할 때 변수의 값에 대응하는 URI 변수를 사용해야 하며 요청을 matrix variable의 존재와 순서와 무관하게 대응해야 한다. 다음 예제에선 matrix variable을 사용한다.

```java
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11
}
```

모든 세부 경로가 matrix variable을 포함할 수 있으며 어떤 경로 변수에 어떤 matrix variable이 대응되는지 모호하지 않게 만들어야 한다. 다음 예제에선 모호하지 않게 경로 변수를 적어 놓았다.

```java
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22
}
```

다음과 같이 matrix variable은 기본값이 있는 선택 변수로 정의할 수 있다.

```java
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1
}
```

다음 예제와 같이, 경로의 모든 matrix variable을 얻기 위해서 ```MultiValueMap```을 사용할 수 있다.

```java
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]
}
```

Spring MVC에서 matrix variable을 사용하기 위해서는 ```removeSemicolonContent=false```로 ```UrlPathHelper```을 설정해야 한다. (XML 설정으로는 ```<mvc:annotation-driven enable-matrix-variables="true"/>```을 사용한다)

#### Using ```@RequestParam```

```@RequestParam``` 어노테이션으로 Servlet 요청 파라미터를 메소드 인자에 할당할 수 있다.

```java
@Controller
@RequestMapping("/pets")
public class EditPetForm {

    // ...

    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, Model model) { 
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...

}
```

기본적으로 ```@RequestParam``` 어노테이션을 가진 파라미터는 필수값이 되지만, ```required``` 설정값을 ```false```로 하거나 인자의 타입을 ```java.util.Optional```로 설정할 수도 있다.

파라미터 타입이 ```String```이 아니면 자동으로 타입을 변환한다. [Type Conversion](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-typeconversion) 참고

```@RequestParam``` 어노테이션을 가진 인자가 ```Map<String, String>``` 타입이거나 ```MultiValueMap<String, String>```타입이면 모든 요청 파라미터를 해당 인자에 집어넣는다.

간단한 타입의 인자에 대해서는 ```@RequestParam```을 사용하지 않아도 사용한 것으로 간주하고 요청 파라미터를 할당한다.

#### Using ```@RequestHeader```

요청 헤더를 메소드 인자에 할당하기 위해 ```@RequestHeader``` 어노테이션을 사용한다.

다음 헤더를 가진 요청이 있다고하자.

```
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300
```

다음 예제는 ```Accept-Encoding```과 ```Keep-Alive``` 헤더값을 가져온다.

```java
@GetMapping("/demo")
public void handle(
        @RequestHeader("Accept-Encoding") String encoding, 
        @RequestHeader("Keep-Alive") long keepAlive) { 
    //...
}
```

```String``` 타입이 아닌 파라미터에 대해선 자동으로 타입을 변환한다.

```@RequestHeader``` 어노테이션을 가진 ```Map<String, String>```, ```MultiValueMap<String, String>```, ```HttpHeader``` 인자에는 자동으로 전체 헤더 값이 할당된다.

* 콤마로 구분된 문자열을 배열이나 문자열 collection 혹은 다른 변환 가능한 타입으로 변환할 수 있다. 예를 들면 ```@RequestHeader("Accept")``` 어노테이션을 가진 메소드 파라미터의 타입은 ```String```뿐만 아니라 ```String[]```이나 ```List<String>```도 될 수 있다.

### Using ```@CookieValue```

HTTP 쿠키값을 가져오기 위해 ```@CookieValue``` 어노테이션을 사용한다.

다음 쿠키값을 가져오기 위해서는

```
JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
```

다음 예제와 같이 사용한다.

```java
@GetMapping("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) { 
    //...
}
```

```String``` 타입이 아닌 파라미터에 대해선 자동으로 타입을 변환한다.

#### Using ```@ModelAttribute```

```@ModelAttribute``` 어노테이션을 사용해 모델의 속성에 접근하거나 값을 초기화 할 수 있다. 모델의 속성 필드의 이름과 HTTP Servlet 요청 파라미터의 이름이 같으면 따로 일일이 설정하지 않고도 요청 파라미터로부터 모델 속성에 값을 바로 할당할 수 있다.

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) { } 
```

위 예제의 ```Pet``` 인스턴스는 다음과 같이 값을 알아낸다.

* ```Model```에 이미 추가되어있는 모델로부터 가져온다.
* ```@SessionAttributes```를 이용해 HTTP 세션으로부터 가져온다.
* ```Converter```를 통해 전달된 URI 경로 변수로부터 만든다.
* 기본 생성자를 이용해 만든다.
* Servlet 요청 파라미터와 일치하는 인자를 가진 생성자를 이용해 만든다. 인자 이름은 JavaBeans의 ```@constructorProperties```나 바이트코드에 런타임에 가지고 있는 파라미터 이름을 통해 정해진다.

보통 ```Model```을 이용해 모델과 속성을 초기화하지만, URI 경로 변수를 ```Converter<String, T>```를 이용해 변환해 초기화하기도 한다. 다음 예제에서 모델 속성의 이름 ```acount```가 URI 경로 변수의 이름 ```account```와 일치하기 때문에, 경로 변수의 ```String``` 값을 등록된 ```Converter<String, Account>```을 이용해 변환해 ```Account``` 인스턴스를 만든다.

```java
@PutMapping("/accounts/{account}")
public String save(@ModelAttribute("account") Account account) {
    // ...
}
```

모델 속성 인스턴스를 얻어온 이후에 data binding을 적용한다. ```WebDataBinder``` 클래스가 Servlet 요청 파라미터 이름(쿼리 파라미터와 폼 필드)과 대상 ```Object```의 필드 이름을 맞춰본다. 맞는 값을 찾은 대상 필드에 타입 변환한 값을 적용한다. data binding(과 validation)에 대해 더 알고 싶다면 [Validation](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#validation)을 참조한다. data binding을 설정하기 위해선 [Using ```DataBinder```](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-initbinder)을 참조한다.

Data binding은 에러를 만들기도 한다. 이 때, 기본값으로 ```BindException``` 발생한다. 컨트롤러 메소드에서 이러한 에러가 발생했는지 확인하려면 다음 예제처럼 ```@ModelAttribute``` 인자 바로 옆에 ```BindingResult``` 인자를 추가한다.

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

data binding없이 모델 속성을 사용하고 싶을 때는 ```Model```을 컨트롤러에 직접 주입하거나 다음 예제와 같이 ```@ModelAttribute(binding=false)``` 어노테이션을 사용한다.

```java
@ModelAttribute
public AccountForm setUpForm() {
    return new AccountForm();
}

@ModelAttribute
public Account findAccount(@PathVariable String accountId) {
    return accountRepository.findOne(accountId);
}

@PostMapping("update")
public String update(@Valid AccountUpdateForm form, BindingResult result,
        @ModelAttribute(binding=false) Account account) { 
    // ...
}
```

```javax.validation.Valid``` 어노테이션 혹은 Spring의 ```@Validated``` 어노테이션을 추가하면 data binding 후에 자동으로 validation을 할 수 있다.

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

다른 어떤 인자로도 해석되지 않는 간단한 타입이 아닌 인자는 ```@ModelAttribute``` 가 없어도 있는 것 처럼 처리한다.

#### Using ```@SessionAttributes```

```@SessionAttributes```를 사용하면 요청간의 HTTP Servlet 세션에 모델을 저장할 수 있다. 이 어노테이션은 세션 속성 클래스에 사용한다. 세션에 저장할 모델 속성이나 모델 속성의 타입 이름을 나열한다.

다음 예제에서 ```@SessionAttributes``` 어노테이션을 사용한다.

```java
@Controller
@SessionAttributes("pet")
public class EditPetForm {
    // ...
}
```

첫 요청을 처리할 때, ```pet```이란 이름의 모델 속성이 모델에 추가되고 HTTP Servlet 세션에도 자동으로 추가된다. 이 모델 속성은 다른 컨트롤러 메소드가 ```SessionStatus``` 메소드 인자를 이용해 저장공간을 지울 때까지 남아있게 된다.

```java
@Controller
@SessionAttributes("pet") 
public class EditPetForm {

    // ...

    @PostMapping("/pets/{id}")
    public String handle(Pet pet, BindingResult errors, SessionStatus status) {
        if (errors.hasErrors) {
            // ...
        }
            status.setComplete();
            // ...
        }
    }
}
```

#### Using ```@SessionAttribute```

이미 존재하는 전역 세션 속성에 접근하기 위해 메소드 파라미터에 ```@SessionAttribute```를 사용할 수 있다.

```java
@RequestMapping("/")
public String handle(@SessionAttribute User user) { 
    // ...
}
```

세션 속성을 추가하거나 삭제하기 위해서는 컨트롤러 메소드에 ```org.springframework.web.context.request.WebRequest```나 ```javax.servlet.http.HttpSession```를 주입한다.

컨트롤러 실행 과정의 일부로 세션에 모델 속성을 임시로 저장하기 위해서는 ```@SessionAttributes```를 사용한다.

#### Using ```@RequestAttribute```

```@SessionAttribute```와 유사하게, ```@RequestAttribute```를 사용해 이미 존재하는 요청 속성 (Servlet ```Filter```나 ```HandlerInterceptor```가 미리 만들어놓은)에 접근한다.

```java
@GetMapping("/")
public String handle(@RequestAttribute Client client) {
    // ...
}
```

#### Redirect Attributes

기본적으로 모든 모델 속성은 리다이렉트 URL의 URI 템플릿 변수로 노출된다고 봐도 좋다. 원시 타입이나 원시 타입의 컬렉션 혹은 배열은 쿼리 파라미터로 추가된다.

리다이렉트를 위해 특별히 준비한 모델 인스턴스가 있다면 원시 타입 속성값을 쿼리 파라미터로 추가할 수 있다. 그러나 모델에는 렌더링을 위해 추가로 존재하는 속성값도 있다. 이러한 속성값이 URL에 노출되는 것을 막으려면 ```@RequestMapping``` 메소드는 ```RedirectAttributes``` 타입의 인자를 선언하고 어떤 속성값을 ```RedirectView```에 나타낼지 정할 수 있다. 메소드가 리다이렉트를 하게 되면 ```RedirectAttributes```의 내용을 사용하고 그렇지 않으면 모델의 내용을 사용한다.

```RequestMappingHandlerAdapter```는 컨트롤러 메소드가 리다이렉트를 할 때 기본 ```Model```의 내용을 사용하지 않도록 설정할 수 있는 ```ignoreDefaultModelOnRedirect``` 설정값을 제공한다. 대신 메소드가 ```RedirectView```에 넘겨줄 속성값을 ```RedirectAttributes``` 타입의 속성값에 반드시 명시해야 한다. Spring MVC는 이전 버전에 대한 호환성을 위해 ```ignoreDefaultModelOnRedirect```의 값을 ```false```로 둔다. 하지만 새로운 어플리케이션에는 ```true```로 설정하기를 권장한다.

현재 요청의 URI 템플릿 변수는 리다이렉트 URL을 만들 때 자동으로 사용 가능해지고 해당 변수들을 ```Model```이나 ```RedirectAttributes```에 명시적으로 추가해야 한다. 다음 예제는 리다이렉트를 정의하는 법을 보여준다.

```java
@PostMapping("/files/{path}")
public String upload(...) {
    // ...
    return "redirect:files/{path}";
}
```

데이터를 리다이렉트 목적지에 전달하기 위해선 플래시 속성값을 사용할 수도 있다. 다른 리다이렉트 속성값과는 다르게 플래시 속성값은 HTTP 세션에 저장한다.

#### Flash Attributes

플래시 속성값은 이용해 요청의 속성값을 다른 곳에 쓰기 위해 저장할 수 있다. 예를 들면 POST-리다이렉트-GET 패턴과 같은 리다이렉트에 쓰기 위한 값을 저장할 수 있다. 리다이렉트 하기 전에 플래시 속성값을 임시로 저장해 리다이렉트 이후의 요청에 사용할 수 있게 해주며 그 뒤에 즉시 삭제한다.

Spring MVC는 플래시 속성값을 지원하기 위해 ```FlashMap```과 ```FlashMapManager```를 제공한다. ```FlashMap```을 이용해 플래시 속성값을 가지고 있으며, ```FlashMapManager```를 이용해 ```FlashMap``` 인스턴스를 저장하고 읽고 관리한다.

플래시 속성값은 항상 사용 가능하며 명시적으로 설정할 필요가 없다. 그러나 사용하지 않으면 HTTP 세션을 절대 만들지 않는다. 요청 마다 이전 요청에서 넘어온 ```FlashMap``` 이 있으며, 이후 요청을 위한 출력 ```FlashMap```이 있다. 두 ```FlashMap``` 인스턴스 모두 ```RequestContextUtils```의 정적 메소드를 통해 Spring MVC의 어디서든 접근이 가능하다.

컨트롤러가 직접 ```FlashMap```을 가지고 사용할 일은 많지 않다. 대신, ```@RequestMapping``` 메소드가 ```RedirectAttributes``` 타입의 인자를 사용해 리다이렉트를 위해 플래시 속성값을 추가할 수 있다. 이 인자에 추가한 속성값은 자동으로 출력 ```FlashMap```에 추가된다. 마찬가지로, 입력 ```FlashMap```의 속성값도 자동으로 리다이렉트 대상 URL을 처리하는 컨트롤러의 ```Model```에 추가된다.

#### Multipart

```MultipartResolver```가 활성화되면, ```multipart/form-data``` 형식을 가진 POST 요청의 컨텐츠를 파싱해 일반 요청 파라미터처럼 사용할 수 있도록 해준다. 다음 예제에서 하나의 일반 폼 필드와 하나의 업로드 파일을 접근할 수 있다.

```java
@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(@RequestParam("name") String name,
            @RequestParam("file") MultipartFile file) {

        if (!file.isEmpty()) {
            byte[] bytes = file.getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }

        return "redirect:uploadFailure";
    }

}
```

* Servlet 3.0 multipart 파싱을 사용하면, Spring의 ```MultipartFile``` 대신 ```javax.servlet.http.Part```를 사용할 수 있다.

multipart 데이터를 모델 객체의 data binding의 일부로도 사용할 수 있다. 예를 들어 이전 예제의 폼 필드와 파일을 가진 폼 객체를 사용할 수 있다.

```java
class MyForm {

    private String name;

    private MultipartFile file;

    // ...

}

@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(MyForm form, BindingResult errors) {

        if (!form.getFile().isEmpty()) {
            byte[] bytes = form.getFile().getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }

        return "redirect:uploadFailure";
    }

}

```

RESTful 서비스를 사용할 때에는 브라우저가 아닌 클라이언트가 multipart 요청을 보낼 수 있다. 다음 예제는 JSON과 함께 파일이 있는 요청 내용을 보여준다.

```
POST /someUrl
Content-Type: multipart/mixed

--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="meta-data"
Content-Type: application/json; charset=UTF-8
Content-Transfer-Encoding: 8bit

{
    "name": "value"
}
--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
Content-Disposition: form-data; name="file-data"; filename="file.properties"
Content-Type: text/xml
Content-Transfer-Encoding: 8bit
... File Data ...
```

```@RequestParam```을 이용해 ```String```으로 메타데이터 부분을 가져올 수도 있지만, JSON을 역직렬화하고 싶다면 ```@RequestPart``` 어노테이션을 이용해 ```HttpMessageConverter```로 변환한 multipart 데이터에 접근할 수 있다.

```java
@PostMapping("/")
public String handle(@RequestPart("meta-data") MetaData metadata,
        @RequestPart("file-data") MultipartFile file) {
    // ...
}
```

```@RequestPart```와 ```javax.validation.Valid```를 함께 사용하거나 Spring의 ```@Validated``` 어노테이션을 함께 사용할 수 있다. 기본값으로 validation 에러는 400(BAD_REQUEST)응답을 만드는  ```MethodArgumentNotValidException```을 던진다. 아니면 다음 예제와 같이 ```Errors```나 ```BindingResult``` 인자를 이용해 validation 에러를 컨트롤러 내부에서 처리할 수도 있다.

```java
@PostMapping("/")
public String handle(@Valid @RequestPart("meta-data") MetaData metadata,
        BindingResult result) {
    // ...
}
```

#### Using ```@RequestBody```

```@RequestBody``` 어노테이션을 사용해 요청 본문을 읽거나 ```HttpMessageConverter```를 통해 ```Object```로 역직렬화할 수 있다.

```java
@PostMapping("/accounts")
public void handle(@RequestBody Account account) {
    // ...
}
```

```@RequestPart```와 ```javax.validation.Valid```를 함께 사용하거나 Spring의 ```@Validated``` 어노테이션을 함께 사용할 수 있다. 기본값으로 validation 에러는 400(BAD_REQUEST)응답을 만드는  ```MethodArgumentNotValidException```을 던진다. 아니면 다음 예제와 같이 ```Errors```나 ```BindingResult``` 인자를 이용해 validation 에러를 컨트롤러 내부에서 처리할 수도 있다.

```java
@PostMapping("/accounts")
public void handle(@Valid @RequestBody Account account, BindingResult result) {
    // ...
}
```

#### HttpEntity

```HttpEntity```는 ```@RequestBody```를 사용하는 것과 비슷하지만, 요청 헤더와 본문 모두를 가지고 있다.

```java
@PostMapping("/accounts")
public void handle(HttpEntity<Account> entity) {
    // ...
}
```

#### Using ```@ResponseBody```

```@ResponseBody```를 이용해 ```HttpMessageConverter```로 직렬화한 응답 본문을 반환할 수 있다.

```java
@GetMapping("/accounts/{id}")
@ResponseBody
public Account handle() {
    // ...
}
```

클래스에 ```@ResponseBody```를 사용하면 모든 컨트롤러 메소드가 해당 어노테이션을 상속받게 된다. ```@RestController```가 같은 역할을 하며, 이 어노테이션은 ```@Controller```와 ```@ResponseBody```를 합쳐놓은 것과 같다.

```@ResponseBody```를 reactive 타입과 함께 사용할 수 있다. [Asynchronous Requests](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-async)와 [Reactive Types](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-async-reactive-types)를 참조하라.

```@ResponseBody``` 메소드를 JSON 직렬화 뷰와 함께 사용할 수 있다.

#### ResponseEntity

```ResponseEntity```는 ```@ResponseBody```를 사용하는 것과 비슷하지만, 응답 헤더와 본문 모두를 가지고 있다.

```java
@PostMapping("/something")
public ResponseEntity<String> handle() {
    // ...
    URI location = ... ;
    return ResponseEntity.created(location).build();
}
```

#### Jackson JSON

Spring은 Jackson JSON 라이브러리를 지원한다.

##### Jackson Serialization Views

Spring MVC는 객체의 일부 필드만을 렌더링하도록 도와주는 Jackson의 직렬화 뷰에 대한 지원을 내장했다. ```@ResponseBody```나 ```ResponseEntity```와 함께 이 기능을 사용하려면 다음과 같이 Jackson의 ```@JsonView``` 어노테이션을 직렬화 뷰 클래스에 추가한다.

```java
@RestController
public class UserController {

    @GetMapping("/user")
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}

public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
```

* ```@JsonView```는 뷰 클래스의 배열도 지원하지만 하나의 컨트롤러 메소드에는 하나만 명시할 수 있다. 여러 뷰를 동시에 사용하려면 인터페이스를 합성해야 한다.

뷰를 채우는 컨트롤러를 위해서는 다음 예제와 같이 직렬화 뷰 클래스를 모델에 추가할 수 있다.

```java
@Controller
public class UserController extends AbstractController {

    @GetMapping("/user")
    public String getUser(Model model) {
        model.addAttribute("user", new User("eric", "7!jd#h23"));
        model.addAttribute(JsonView.class.getName(), User.WithoutPasswordView.class);
        return "userView";
    }
}
```

### Model

```@ModelAttribute```를

* ```@RequestMapping``` 메소드의 인자에 추가해 모델의 객체를 생성하거나 객체에 접근하고 ```WebDataBinder```를 이용해 요청에 연결할 수 있다.
* ```@Controller```나 ```@ControllerAdvice``` 클래스의 메소드에 추가해 ```@Requestmapping``` 메소드 어노테이션 이전에 모델을 초기화하도록 도와준다.
* ```@RequestMapping``` 메소드에 추가해 해당 메소드의 반환값이 모델 속성값임을 표시한다.

이번 절에서는 두 번째 사용법인 ```@ModelAttribute``` 메소드에 대해 이야기한다. 컨트롤러는 ```@ModelAttribute``` 메소드를 몇개든 가질 수 있다. 모든 ```@ModelAttribute``` 메소드는 ```@ControllerAdvice```를 통해 컨트롤러 간에도 공유할 수 있다.

```@ModelAttribute``` 메소드는 유연한 메소드 서명을 가진다. ```@RequestMapping``` 메소드가 지원하는 인자 중 ```@ModelAttribute``` 자체나 요청 본문과 관련되지 않은 대부분을 지원한다.

다음 예제는 ```@ModelAttribute``` 메소드를 보여준다.

```java
@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountRepository.findAccount(number));
    // add more ...
}
```

다음 예제에선 ```@ModelAttribute``` 메소드가 하나의 속성값만을 더한다.

```java
@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountRepository.findAccount(number);
}
```

```@RequestMapping``` 메소드의 반환값을 모델 속성값으로 사용할 수 있을 때 ```@ModelAttribute```를 ```@RequestMapping```메소드에 덧붙일 수 있다. 하지만 해당 메소드의 반환값을 뷰 이름으로 해석하지 못하게 하는 경우가 아니면, HTML 컨트롤러의 기본 동작이므로 ```@ModelAttribute```를 필수적으로 추가해주지 않아도 된다.

```@ModelAttribute```는 다음 예제와 같이 모델 속성값 이름을 임의로 변경할 수 있도록 도와준다.

```java
@GetMapping("/accounts/{id}")
@ModelAttribute("myAccount")
public Account handle() {
    // ...
    return account;
}
```

### Using ```DataBinder```

```@Controller```나 ```@ControllerAdvice``` 클래스는 ```WebDataBinder```의 인스턴스를 초기화하는 ```@InitBinder``` 메소드를 가질 수 있다. 해당 메소드는

* 요청 파라미터(폼 데이터나 쿼리 데이터)를 모델 객체에 할당하거나
* 문자열 값인 요청 값(요청 파라미터, 경로 변수, 헤더, 쿠키 등)을 컨트롤러 메소드 인자 타입으로 변환하거나
* HTML 폼 렌더링에 필요한 모델 객체의 형식을 정한다.

```@InitBinder``` 메소드는 컨트롤러의 특정한 ```java.bean.PropertyEditor```나 Spring ```Converter```와 ```Formatter``` 컴포넌트를 등록할 수 있다. 추가로 MVC 설정을 이용해 전역으로 공유하는 ```FormattingConvensionService```에 ```Converter```와 ```Formatter```를 등록할 수 있다.

```@InitBinder``` 메소드는 ```@RequestMapping``` 메소드가 지원하는 인자 중 ```@ModelAttribute``` 인자를 제외하곤 대부분 지원한다. 인자는 보통 ```WebDataBinder``` 타입으로 선언되며 반환값이 없다 (```void```). 다음 예제는 ```@InitBinder```의 사용 예제를 보여준다.

```java
@Controller
public class FormController {

    @InitBinder 
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...
}
```

```FormattingConversionService```를 통해 ```Formatter``` 기반의 설정을 사용한다면, 위와 비슷한 방식으로 다음 예제처럼 컨트롤러별로 ```Formatter``` 구현을 등록할 수 있다.

```java
@Controller
public class FormController {

    @InitBinder 
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
    }

    // ...
}
```

### Exceptions

### Controller Advice

보통 ```@ExceptionHandler```, ```@InitBinder```, ```@ModelAttribute``` 메소드는 정의된 ```@Controller``` 클래스 안에서 적용된다. 이런 메소드를 컨트롤러간에도 공유하고 싶다면 메소드를 가진 클래스에 ```@ControllerAdvice```나 ```@RestControllerAdvice```를 추가한다.

```@ControllerAdvice```는 ```@Component```가 붙어 있어 해당 어노테이션이 붙어있는 클래스는 Spring bean으로 등록된다. ```@RestControllerAdvice```도 ```@ControllerAdvice```와 ```@ResponseBody```가 붙어있어, ```@ExceptionHandler``` 메소드가 (뷰 해석이나 템플릿 렌더링이 아닌) 메세지 변환을 통해 응답 본문으로 렌더링된다는 것을 의미한다.

Spring이 시작할 때, ```@RequestMapping```과 ```@ExceptionHandler``` 메소드를 위한 내부 클래스가 ```@ControllerAdvice``` 타입의 Spring bean을 찾아 런타임에 해당 메소드들에 적용한다. ```@ControllerAdvice```의 ```@ExceptionHandler``` 메소드는 ```@Controller```의 ```@ExceptionHandler``` 메소드 뒤에 적용된다. 이와는 반대로, ```@ModelAttribute```나 ```@InitBinder``` 메소드는 컨트롤러 각각의 메소드 이전에 적용된다.

기본값으로 ```@ControllerAdvice``` 메소드는 모든 요청에 적용되나, 다음 예제와 같이 어노테이션의 속성값을 통해 적용할 컨트롤러의 집합을 줄여나갈 수 있다.

```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```

위의 예제의 값들은 런타임에 평가하고 많이 사용하면 성능에 부정적인 영향을 미친다. 자세한 사항은 [```@ControllerAdvice```](https://docs.spring.io/spring-framework/docs/5.1.0.RELEASE/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html) Javadoc을 참고하라.