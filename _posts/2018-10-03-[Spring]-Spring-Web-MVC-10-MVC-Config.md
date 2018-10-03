---
layout: post
title: "Spring Web MVC - 10. MVC Config"
categories: Spring
---

다음 내용은 Spring Web MVC 5.1.0.RELEASE 버전의 공식 문서인 [Web On Servlet Stack](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/web.html)의 Spring Web MVC 부분 중 일부분을 발췌해 번역해 놓은 것입니다. 링크된 공식 문서의 내용과 다른 부분이나 생략된 부분이 있을 수 있습니다.

# Spring Web MVC

## MVC Config

### Enable MVC Configuration

```@EnableWebMvc``` 어노테이션을 사용한다.

```java
@Configuration
@EnableWebMvc
public class WebConfig {
}
```

### The MVC Configuration API

```WebMvcConfigurer``` 인터페이스를 구현 할 수 있다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
}
```

### Type Conversion

```Number```와 ```Date``` 타입을 위한 formatter가 준비되어 있다. (```@NumberFormat```과 ```@DateTimeFormat``` 어노테이션도 지원한다). Joda-Time이 classpath에 있으면 Joda-Time formatting library도 지원한다.

다음과 같이 임의의 formatter와 converter도 추가 가능하다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // ...
    }
}
```

### Validation

JSR-303 Bean Validation provider가 classpath에 있으면 ```LocalValidatorFactoryBean```이 전역 validator로 등록되어 ```@Valid```와 ```Validated```를 처리한다.

임의의 전역 ```Validator``` 인스턴스를 사용하고 싶으면 다음과 같이 메소드를 override한다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public Validator getValidator(); {
        // ...
    }
}
```

임의의 ```Validator``` 구현을 추가할 수도 있다.

```java
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }

}
```

### Interceptors

다음과 같이 요청에 적용할 interceptor를 추가할 수 있다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleChangeInterceptor());
        registry.addInterceptor(new ThemeChangeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
        registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
    }
}
```

### Content Types

Spring MVC가 어떻게 요청의 media type을 결정할지 설정할 수 있다.

기본값은 URL 경로 확장자(```json```, ```xml```, ```rss``` 등)를 먼저 확인하도록 하고, 그 다음에 ```Accept``` 헤더를 확인하도록 되어있다.

```Accept``` 헤더만 확인하도록 하거나, URL을 기반으로 content type을 결정하고 싶으면 다음 링크를 참조한다. [Suffix Match](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-suffix-pattern-match)와 [Suffix Match and RFD](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-rfd)

다음과 같이 임의로 content type을 처리할 수 있다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.mediaType("json", MediaType.APPLICATION_JSON);
        configurer.mediaType("xml", MediaType.APPLICATION_XML);
    }
}
```

### Message Converters

```configureMessageConverters()```(Spring MVC가 제공하는 기본 converter를 대체한다.) 나 ```extendMessageConverters()```(기본 converter를 수정하거나 새로운 converter를 추가한다)를 override해서 ```HttpMessageConverter```를 설정할 수 있다.

```java
@Configuration
@EnableWebMvc
public class WebConfiguration implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
                .modulesToInstall(new ParameterNamesModule());
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
        converters.add(new MappingJackson2XmlHttpMessageConverter(builder.createXmlMapper(true).build()));
    }
}
```

위의 예제에선 [```Jackson2ObjectMapperBuilder```](https://docs.spring.io/spring-framework/docs/5.1.0.RELEASE/javadoc-api/org/springframework/http/converter/json/Jackson2ObjectMapperBuilder.html)을 이용해 ```MappingJackson2HttpMessageConverter```과 ```MappingJackson2XmlHttpMessageConverter```모두 설정한다.

다음 모듈이 classpath에 있으면 자동으로 등록한다.

* jackson-datatype-jdk7: java.nio.file.Path 등의 Java 7 타입을 지원한다.
* jackson-datatype-joda: Joda-Time 타입을 지원한다.
* jackson-datatype-jsr310: Java 8 시간 API 타입을 지원한다.
* jackson-datatype-jdk8: Optional 등의 Java 8 타입을 지원한다.

다음 Jackson 모듈도 지원한다.

* jackson-datatype-money: ```javax.money``` 타입을 지원한다 (비공식 모듈)
* jackson-datatype-hibernate: Hibernate 특화된 타입과 속성을 지원한다.

### View Controllers

다음 예제처럼 뷰를 만들어내는 로직이 필요없는 경우에 요청을 바로 처리할 수 있는 ```ParameterizableViewController```를 정의할 수 있다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

### View Resolvers

다음 예제는 JSP와 Jackson을 JSON 렌더링을 위한 기본 ```View```로 설정해 content negotation을 설정한다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.jsp();
    }
}
```

### Static Resources

다음 예제는 ```/resources```로 시작하는 요청에 대해 ```/public``` 혹은 classpath의 ```/static```에 대한 상대 경로에 존재하는 정적 리소스를 응답하도록 설정한다. 리소스는 브라우저 캐시가 1년 뒤에 만료되도록 설정한다. ```Last-Modified``` 헤더를 평가해 ```304``` 상태 코드가 반환된다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
            .addResourceLocations("/public", "classpath:/static/")
            .setCachePeriod(31556926);
    }
}
```

### Default Servlet

Spring MVC는 ```DispatcherServlet```을 ```/``` 경로에 연결하고, 정적 리소스는 기본 Servlet의 컨테이너가 처리할 수 있도록 해준다. 이를 위해 ```DefaultServletHttpRequestHandler```를 ```/**``` URL에 가장 낮은 우선순위로 연결하도록 한다. 기본적으로는 다른 모든 URL ```HandlerMapping```이 기본 Servlet으로 가는 요청보다 먼저 처리되도록 되어있지만, 임의로 설정한 ```HandlerMapping``` 인스턴스를 사용하기 위해서는 ```order``` 값을 ```DefaultServletHttpRequestHandler```의 값인 ```Integer.MAX_VALUE```보다 낮게 설정해아 한다.

다음과 같이 설정하면 ```DefaultServletHttpRequestHandler```를 이용할 수 있다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

기본 Servlet을 위한 ``RequestDispatcher```를 경로가 아닌 이름으로 찾으므로, Tomcat, Jetty 등과 같은 기본 이름이 아닌 다른 이름을 가진 임의의 기본 Servlet을 사용하기 위해서는 다음과 같이 명시적으로 기본 Servlet의 이름을 지정해주어야 한다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable("myCustomDefaultServlet");
    }

}
```

### Path Matching

다음과 같이 임의로 URL 경로 매칭에 대한 설정을 변경할 수 있다.

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer
            .setUseSuffixPatternMatch(true)
            .setUseTrailingSlashMatch(false)
            .setUseRegisteredSuffixPatternMatch(true)
            .setPathMatcher(antPathMatcher())
            .setUrlPathHelper(urlPathHelper())
            .addPathPrefix("/api",
                    HandlerTypePredicate.forAnnotation(RestController.class));
    }

    @Bean
    public UrlPathHelper urlPathHelper() {
        //...
    }

    @Bean
    public PathMatcher antPathMatcher() {
        //...
    }

}
```

### Advanced Java Configuration

```@EnableWebMvc```는 다음과 같은 역할을 하는 ```DelegatingWebMvcConfiguration```를 불러온다.

* Spring MVC 어플리케이션을 위한 기본 Spring configuration을 제공한다.
* ```WebMvcConfigurer```를 찾아 configuration을 변경할 수 있도록 위임한다.

고급 설정이 필요하면 ```@EnableWebMvc```를 제거하고 ```DelegatingWebMvcConfiguration```를 직접 상속하면 된다.

```java
@Configuration
public class WebConfig extends DelegatingWebMvcConfiguration {

    // ...

}

```