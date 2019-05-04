---
layout: post
title: "[Real World HTTP] Chapter 2. HTTP/1.0의 시맨틱스: 브라우저 기본 기능의 이면"
categories: HTTP
---

## 단순한 폼 전송(x-www-form-urlencoded)

* 다음과 같은 폼을 전송하려면

  ```html
  <form method="POST">
    <input name="title">
    <input name="author">
    <input type="submit">
  </form>
  ```

* 다음 curl 커맨드를 사용한다.

  ```bash
  $ curl --http1.0 -d title="The Art of Community" -d author="Jono Bacon" http://localhost:18888
  ```

* 위 요청의 바디는 다음과 같은 형식이 된다.

  ```
  title=The Art of Community&author=Jono Bacon
  ```

* 브라우저에서는 RFC 1866에서 정한 변환 포맷에 따라 변환이 필요한 문자를 변환한다.

  ```
  title=The+Art+of+Community&author=Jono+Bacon
  ```

* curl 커맨드의 ```-data-urlencode``` 를 이용하면 RFC 3986의 방법으로 변환한다.

  ```
  title=The%20Art%20of%20Community&author=Jono%20Bacon
  ```

## 폼을 이용한 파일 전송

* 다음처럼 하면 폼에서 파일을 보낼 수 있다.

```html
<form action="POST" enctype="multipart/form-data">
</form>
```

* 한 번에 여러 파일의 내용을 보내기 위해 브라우저에서 요청의 바디에 다음과 같은 경계 문자열을 추가한다. (크롬 브라우저의 예)

```
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryyOYfbccgoID172j7
```

* 경계 문자열을 사용해 여러 파일의 내용을 블록으로 구분한 바디는 다음과 같은 형식을 가진다.

```
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryyOYfbccgoID172j7
Content-Disposition: form-data; name="title"

xxx
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryyOYfbccgoID172j7
Content-Disposition: form-data; name="author"

yyy
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryyOYfbccgoID172j7--
```

* 각 블록은 헤더와 컨텐츠로 구분된다. ```Content-Disposition``` 헤더를 이용해 각 블록의 내용을 표현한다.
* ```x-www-form-urlencoded``` 와는 달리 ```multipart/form-data``` 를 사용하면 항목마다 메타 정보를 추가할 수 있다.
* curl 커맨드에서는 ```-F``` 옵션을 사용하면 ```enctype="multipart/form-data"``` 가 설정된 폼과 같은 형식으로 데이터를 전송한다.

## 폼을 이용한 리디렉트

* 300번대 스테이터스 코드를 사용한 리디렉트는 다음과 같은 한계를 가진다.
  * GET의 쿼리로 보낼 수 있는 데이터양에 한계가 있다.
  * 전송되는 내용이 URL로 노출된다.
* HTML의 폼을 이용해 리디렉트를 할 수도 있다.
  * 서버에서 응답 HTML에 보내고 싶은 값을 ```<input type="hidden">``` 태그의 값으로 저장한다.
  * 폼이 제출되는 곳을 리디렉트로 보내고 싶은 곳으로 정한다.
  * HTML이 브라우저에서 로딩될 때 자동으로 폼이 제출되게 하면, 서버에서 원하는 데이터를 응답에 담아 리디렉트하는 효과가 나타난다.
  * 이 방법은 SAML 인증 프로토콜과 OpenID Connect 등의 사양에서 이용한다.

## 콘텐트 니고시에이션

* 클라이언트와 서버가 응답 형식에 대해 협상한다.

### 파일 종류 결정

* 요청 헤더 예제

```
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
```

* q는 품질 계수(```0<=q<=1```)로 우선순위를 나타낸다.
* 서버는 요청한 파일 종류 중 반환 가능한 형식중 우선순위가 가장 높은 형식으로 반환한다. 반환할 수 있는 형식이 없으면 ```406 Not Acceptable``` 오류를 반환한다.

### 표시 언어 결정

* 요청 헤더 예제

```
Accept-Language: en-US,en;q=0.8,ko;q=0.6
```

### 문자셋 결정

* 요청 헤더 예제

```
Accept-Charset: windows-949,utf-8;q=0.7,*;q=0.3
```

* 모던 브라우저 중 ```Accept-Charset``` 을 요청하는 브라우저는 없다.

* 응답에는 MIME 타입과 함께 ```Content-Type``` 헤더에 문자셋이 담긴다.

* HTML 문서 안에 쓸 수도 있다.

* ```html
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8>
  ```

  또는

  ```html
  <meta charset="UTF-8">
  ```

* 사용할 수 있는 문자셋은 IANA에서 관리한다.

### 압축을 이용한 통신 속도 향상

* 압축을 하면 처리 시간을 크게 향상시킬 수 있다.
* 클라이언트는 요청 시에 해제 가능한 압축 방식을 헤더에 지정한다.

```
Accept-Encoding: deflate, gzip
```

* curl 커맨드에서는 ```—compressed``` 옵션을 지정하면 된다.
* 서버는 전송받은 압축 방식 목록 중 지원하는 방식으로 응답의 바디를 압축해 전송한다.
* 서버 응답시  ```Content-Encoding``` 헤더를 추가해 압축 방식을 알려주고, ```Content-Length``` 헤더에는 압축된 파일 크기를 넣는다.
* 구글은 gzip보다 효율이 좋은 brotli를 공개했다.
* 클라이언트에서 서버로 업로드할 때도 압축을 이용할 수 있도록 논의중이다.
* 공유 사전 압축 방식
  * 서버와 클라이언트가 공유 사전을 함께 사용해 더 압축률이 높은 압축을 한다.
  * 크롬에서 사용하는 [sdch](https://en.wikipedia.org/wiki/SDCH)라는 압축 방식이 이러한 방식을 사용한다.
  * HTTP/2의 헤더 부분 압축에도 사용된다.

## 쿠키

* 웹사이트의 정보를 브라우저 쪽에 저장하는 작은 파일

* 서버에서는 ```Set-Cookie``` 헤더를 이용해 쿠키값을 보내면, 나중에 클라이언트가 저장해둔 값을 ```Cookie``` 헤더에 넣어 요청을 전송한다.

* 브라우저에서도 자바스크립트를 이용해 쿠키를 읽을 수 있다.

  ```javascript
  document.cookie;
  ```

* curl 커맨드에선 ```-b``` 옵션 혹은 ```—cookie``` 옵션을 사용한다.

###  쿠키의 잘못된 사용법

* 쿠키의 제약
  * 브라우저나 설정에 따라 세션이 끝나면 초기화되거나 삭제된다.
  * 최대 4킬로바이트까지만 보낼 수 있다.
  * HTTP 통신에선 쿠키가 평문을 노출된다.
* 인증 정보나 사라져도 문제가 없는 정보만 쿠키에 넣도록 한다.

### 쿠키에 제약을 주다

* ```Set-Cookie``` 헤더에 옵션을 추가할 수 있다.
* 예

```
Set-Cookie: SID=31d4d96e407aad42; Path=/; Secure; HttpOnly
Set-Cookie: lang=en-US; Path=/; Domain=example.com
```

* 옵션
  * ```Expires```, ```Max-Age``` : 쿠키의 수명을 결정
  * ```Domain``` : 전송할 대상 서버
  * ```Path``` : 쿠키를 전송할 대상 서버의 경로 
  * ```Secure```: https를 사용할 때만 서버로 쿠키를 전송한다.
  * ```HttpOnly```: 자바스크립트 엔진으로부터 쿠키를 감춘다.
  * ```SameSite```: 크롬 브라우저에 추가된 속성으로 같은 출처의 도메인에 전송한다.

## 인증과 세션

### BASIC 인증과 Digest 인증

* BASIC 인증은 유저명과 패스워드를 BASE64로 인코딩한 것

  * ```Authorization: "Basic [인코딩한 정보]"```

  * 가역변환이므로 SSL/TLS 통신을 하지 않으면 로그인 정보가 유출될 수 있다.

  * curl 커맨드에는 ```-u``` 옵션이나 ```—user``` 옵션으로 보낸다.

  * ```bash
    $ curl --http1.0 --base -u username:password http://localhost:18888
    ```

* Digest 인증은 해시 함수를 이용해 더 강력한 인증 방식이다.

  * 브라우저가 보호된 영역으로 접속하려고 하면 ```401 Unauthorized``` 스테이터스 코드와 함께 다음과 같은 헤더를 담아 응답이 돌아온다.

    * ```
      WWW-Authenticate: Digest realm="영역명", nonce="2482509423", algorithm=MD5, qop="auth"
      ```

  * 위의 헤더에 주어진 값과 무작위로 생성한 ```cnonce``` 를 바탕으로 ```response```를 구한다.

    * ```
      A1 = 유저명 ":" realm ":" 패스워드
      A2 = HTTP 메서드 ":" 콘텐츠 URI
      response = MD5(MD5(A1) ":" nonce ":" nc ":" cnonce ":" qop ":" MD5(A2)) 
      ```

    * nc는 특정 ```nonce``` 값을 사용해 전송한 횟수

    * qop가 없을 때는 생략한다.

    * 같은 nc값이 다시 사용되면 리플레이 공격을 탐지 할 수 있다.

  * 클라이언트에서는 ```cnonce``` 와 ```response``` 값으로 헤더를 만들어 재요청을 보낸다.

    * ```
      Authorization: Digest username="유저명", realm="영역명", nonce="2482509423", uri="/secret.html", algorithm=MD5, qop=auth, nc=0000001, cnonce="12312343", response="2394024324"
      ```

  * 서버는 같은 계산을 해 재요청과 동일한 ```response```가 계산되면 사용자가 인증되었음을 알 수 있다.

  * 유저명과 패스워드를 모두 요청에 포함하지 않고도 서버에서 사용자를 인증 할 수 있게 된다.

  * curl 커맨드에서는 ```—digest``` 옵션과 ```-u```/```—user``` 옵션을 이용해 Digest 인증을 사용할 수 있다.

### 쿠키를 사용한 세션 관리

* 최근에는 BASIC 인증과 Digest 인증 모두 많이 사용하지 않는다.
  * 최상단 페이지에 접속하자마자 인증을 요구해야 하므로 사용자 친화적이지 않다.
  * 요청할 때마다 인증이 필요하므로 번거롭다.
  * 로그인 화면을 사용자화할 수 없다.
  * 명시적인 로그오프를 할 수 없다.
  * 로그인한 단말을 식별할 수 없다.
* 최근에는 폼을 이용한 로그인과 쿠키를 이용한 세션 관리를 많이 사용한다.
  * 클라이언트는 폼으로 id/password 전송 (SSL/TLS 필수)
  * 서버에서는 인증 후 세션 토큰을 쿠키를 통해 발행 (데이터베이스에 저장해 둔다)
  * 클라이언트는 두 번째 접속 이후부터 쿠키에 저장된 세션 토큰으로 로그인을 인증받는다.
  * 웹 서비스에 따라서는 CSRF 대책으로 랜덤 키를 보내는 경우도 있다.

### 서명된 쿠키를 이용한 세션 데이터 저장

* 통신 속도가 빨라지면서 쿠키를 사용한 데이터 관리 시스템도 많이 사용한다.
* RoR의 기본 세션 스토리지는 쿠키를 이용한다. 서버는 전자서명한 쿠키를 확인해 인증한다.
  * 서명도 확인도 서버에서 하므로 클라이언트는 열쇠를 갖지 않는다.
* 그러나 다른 브라우저로 접근한 사용자를 같은 사용자로 인식하지 못한다.

## 프록시

* 프록시는 HTTP 등의 통신을 중계한다.

* 캐시 기능을 붙이거나 네트워크를 보호하는 방화벽 역할도 할 수 있다.

* GET 등의 메서드 다음에 오는 경로명 형식만 바뀐다.

* 프록시 서버에 인증이 필요한 경우 ```Proxy-Authenticate``` 헤더를 사용한다.

* 중계되는 프록시는 중간의 호스트 IP 주소를 ```X-Forwarded-For``` 헤더 혹은 RFC 7239의 ```Forwarded``` 헤더에 저장한다.

* curl 커맨드에서 프록시를 사용하려면 ```-x```/```—proxy``` 옵션을 사용한다. 인증용 유저명과 패스워드는 ```-U```/```—proxy-user``` 옵션을 사용한다.

  ```bash
  $ curl --http1.0 -x http://localhost:18888 -U user:pass http://example.com/helloworld
  ```

  

* HTTPS 통신의 프록시 지원은 HTTP/1.1의 ```CONNECT``` 메서드를 이용한다.

## 캐시

* GET과 HEAD 메서드 이외는 기본적으로 캐시되지 않는다.

### 갱신 일자에 따른 캐시

* 웹 서버는 RFC 1123에 기술된 형식으로 날짜를 ```Last-Modified``` 헤더에 담아 응답에 보낸다.
* 웹 브라우저가 캐시된 URL을 다시 읽을 때는 서버에서 반환된 일시 그대로 ```If-Modified-Since``` 헤더에 넣어 요청한다.
* 웹 서버는 요청의 일시와 서버의 콘텐츠 일시를 비교해 변경됐으면 ```200 OK``` 과 변경된 콘텐츠, 그렇지 않으면 ```304 Not Modified``` 응답만을 보낸다.

### Expires

* ```Expires``` 헤더에는 날짜와 시간이 들어간다.
* 클라이언트는 지정한 기한 내라면 요청을 아예 전송하지 않고 강제로 캐시를 이용한다.
* ```Expires``` 를 메인 페이지 등에 사용할 때는 주의해야 한다.
* RFC 2068에서는 최대 1년의 캐시 수명을 설정하자는 가이드라인이 추가됐다.

### Pragma: no-cache

* 클라이언트가 요청 헤더에 ```Pragma: no-cache``` 를 넣으면 프록시에게 '캐시가 있어도 원래 서버에서 가지고 와라'라고 지시한다.
  * HTTP/1.1에서는 ```Cache-Control``` 로 통합되었지만 하위 호환성을 위해 남아있다.
* 이런 헤더처럼 클라이언트가 지시하는 헤더는 부자연스럽다. 이제는 보안 통신이 보편적이기 때문에 더욱 더 프록시의 캐시를 외부에서 관리하는 것이 의미가 없어졌다.

### ETag 추가

* HTTP/1.1에서 추가된 ```Etag``` 는 파일의 해시 값으로 캐시 여부를 판별한다.
* 서버가 응답에 ```ETag``` 헤더를 보내면, 클라이언트는 추후 요청에 ```If-None-Match``` 헤더에 그 값을 넣어 보낸다.
* ```ETag``` 값은 서버에서 자유롭게 만들 수 있다.

### Cache-Control (1)

* 서버는 ```Cache-Control``` 로 더 유연한 캐시 제어를 지시할 수 있다.
* ```Expires``` 보다 우선해서 처리된다.
* ```no-cache``` 는 캐시하지 않는 것이 아니라, 매번 접속을 시켜 캐시 유효성을 검증하게 한다. ```no-store``` 가 캐시하지 않는다.
* 보통 ```(private || public) && (max-age || s-maxage || no-cache || no-store)``` 으로 설정한다.

### Cache-Control (2)

* 클라이언트에서 ```Cache-Control``` 로 프록시에 지시할 수 있다.
  * ```Cache-Control: no-cache``` : ```Pragma: no-cache``` 와 같다.

* ```Cache-Control``` 로 서버에서 프록시로 지시를 할 수도 있다.

### Vary

* 같은 URL이라도 클라이언트에 따라 반환 결과가 다름을 나타내는 헤더
* 어떤 헤더 때문에 반환 결과가 다른지 나타낸다.
* 그러나 유저 에이전트 이름은 관례적인 것이라 추측이 틀릴 수 있다.

## 리퍼러

* ```Referer``` 는 사용자가 어느 경로로 웹사이트에 도달했는지 서버가 파악할 수 있도록 서버에 보내는 헤더다.

* 웹 서비스는 리퍼러 정보를 수집함으로써 어떤 페이지가 자신의 서비스에 링크를 걸었는지 알 수 있다.
* 액세스 출발지 및 액세스 목적지의 스키마 조합에 따라 리퍼러를 전송하는지 아닌지가 달라진다.
  * HTTPS에서 HTTP로 갈 때만 전송하지 않는다.
* ```Referrer-Policy``` 헤더나 메타 태그 등으로 정책을 설정하거나 ```Content-Security-Policy``` 헤더로 보안 설정을 지정할 수 있다.

## 검색 엔진용 콘텐츠 접근 제어

* 크롤러의 접근을 제어하는 두 가지 방법
  * robots.txt
  * 사이트맵

### robots.txt

* 서버 콘텐츠 제공자가 크롤러에 접근 허가 여부를 전하기 위한 프로토콜

* 다음과 같은 형식으로 읽기를 금지할 이름과 장소를 지정한다.

  ```
  User-agent: *
  Disallow: /cgi-bin/
  Disallow: /tmp/
  ```

* 메타 태그로도 지정할 수 있다.

  ```html
  <meta name="robots" content="noindex" />
  ```

### robots.txt와 재판 결과

* 정식 표준은 아니지만 법적으로 효력이 있는 판례가 나오고 있다.

### 사이트맵

* 웹사이트에 포함된 페이지 목록과 메타데이터를 제공하는 XML 파일
* robots.txt가 블랙리스트라면 사이트맵은 화이트리스트처럼 사용된다.