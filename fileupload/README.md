# FileUpload

## Intro

- 일반적으로 사용하는 HTML Form을 통한 파일 업로드의 이해
- 폼을 전송하는 두 가지 방식 차이

## HTML 폼 전송 방식

- application/x-www-form-urlencoded
- multipart/form-data

## 서블릿과 파일 업로드1

- 실행해보면 logging.level.org.apache.coyote.http11 옵션을 통한 로그에서 multipart/form- data 방식으로 전송된 것을 확인할 수 있다.

```http request
http://localhost:8080/servlet/v1/upload
```

- 파일 업로드 실행 로그

```shell
2021-06-27 18:29:26.207 DEBUG 8364 --- [nio-8080-exec-5] o.a.coyote.http11.Http11InputBuffer      : Received [------WebKitFormBoundaryOTkQLTAioL4Wt9fl
Content-Disposition: form-data; name="itemName"

mysql
------WebKitFormBoundaryOTkQLTAioL4Wt9fl
Content-Disposition: form-data; name="file"; filename="áá¨áá¦02-1 MySQL 5.1 áá¥áá¥á«áá­á¼ Makefile áá¢á¼áá¥á¼ áá³áá³ááµá¸áá³(64ááµáá³).txt"
Content-Type: text/plain

./configure \
    '--prefix=/usr/local/mysql'\
    '--localstatedir=/usr/local/mysql/data'\
    '--libexecdir=/usr/local/mysql/bin'\
    '--with-comment=Toto mysql standard 64bit'\
    '--with-server-suffix=-toto_standard'\
    '--enable-thread-safe-client'\
    '--enable-local-infile'\
    '--enable-assembler'\
    '--with-pic'\
    '--with-fast-mutexes'\
    '--with-client-ldflags=-static'\
    '--with-mysqld-ldflags=-static'\
    '--with-big-tables'\
    '--with-readline'\
    '--with-extra-charsets=complex'\
    '--with-plugins=partition,archive,blackhole,csv,federated,heap,myisam,myisammrg,innodb_plugin'\
    '--with-zlib-dir=bundled'\
    'CC=gcc' 'CXX=gcc' 'CFLAGS=-O2' 'CXXFLAGS=-O2'

------WebKitFormBoundaryOTkQLTAioL4Wt9fl--
]
```

- 멀티파트 사용 옵션
	- 업로드 사이즈 제한
	- 사이즈를 초과 하는 경우 SizeLimitExceededException 예외 발생

```yaml

spring:
  servlet:
    multipart:
      # 파일 하나의 최대 사이즈 (기본 1MB)
      max-file-size: 1MB
      # 멀티파트 요청 하나에 여러 파일을 업로드 할 수 있는데, 그 전체 합 (기본 10MB)
      max-request-size: 10MB
```

- 멀티파트 속성 켜기 (기본)
    - spring.servlet.multipart.enabled 옵션을 켜면 스프링의 DispatcherServlet 에서 멀티파트 리졸버(MultipartResolver)를 실행한다.
	- 멀티파트 리졸버는 멀티파트 요청인 경우 서블릿 컨테이너가 전달하는 일반적인 HttpServletRequest 를 MultipartHttpServletRequest 로 변환해서 반환한다.
	- MultipartHttpServletRequest 는 HttpServletRequest 의 자식 인터페이스이고, 멀티파트와 관련된 추가 기능을 제공한다.
	- 스프링이 제공하는 기본 멀티파트 리졸버는 MultipartHttpServletRequest 인터페이스를 구현한 StandardMultipartHttpServletRequest 를 반환한다.
	- 이제 컨트롤러에서 HttpServletRequest 대신에 MultipartHttpServletRequest 를 주입받을 수 있는데, 이것을 사용하면 멀티파트와 관련된 여러가지 처리를 편리하게 할 수 있다.
	- 추후 MultipartFile 이라는 것을 통해 편하게 사용할 수 있기 때문에 MultipartHttpServletRequest 를 잘 사용하지는 않는다.
	
  ```shell
  2021-06-27 18:38:37.074  INFO 8434 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
  2021-06-27 18:38:37.074  INFO 8434 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
  2021-06-27 18:38:37.075  INFO 8434 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
  2021-06-27 18:38:37.113  INFO 8434 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV1          : request=org.springframework.web.multipart.support.StandardMultipartHttpServletRequest@2e77fe
  2021-06-27 18:38:37.115  INFO 8434 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV1          : itemName=ㅇ
  2021-06-27 18:38:37.115  INFO 8434 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV1          : parts=[org.apache.catalina.core.ApplicationPart@435dab2d, org.apache.catalina.core.ApplicationPart@4b63e3b1]
  ```

- 멀티파트 속성 끄기
	- 옵션을 끄면 서블릿 컨테이너는 멀티파트와 관련된 처리를 하지 않는다.
	- 그래서 결과 로그를 확인해보면 `request.getParameter("itemName")` , `request.getParts()` 의 결과가 비어있다.

	```yaml
	spring:
	  servlet:
	    multipart:
	      enabled: false
	```

	- 로그를 보면 HttpServletRequest 객체가 RequestFacade StandardMultipartHttpServletRequest 로 변한 것을 확인할 수 있다.
		
	```shell
	2021-06-27 18:37:19.686  INFO 8424 --- [nio-8080-exec-2] c.e.c.ServletUploadControllerV1          : request=org.apache.catalina.connector.RequestFacade@612f85f2
	2021-06-27 18:37:19.686  INFO 8424 --- [nio-8080-exec-2] c.e.c.ServletUploadControllerV1          : itemName=null
	2021-06-27 18:37:19.686  INFO 8424 --- [nio-8080-exec-2] c.e.c.ServletUploadControllerV1          : parts=[]
	```

## 서블릿과 파일 업로드2

- 서블릿이 제공하는 Part를 사용하여 업로드 테스트


- Part 주요 메서드
	- part.getSubmittedFileName() : 클라이언트가 전달한 파일명 
	- part.getInputStream(): Part의 전송 데이터를 읽을 수 있다. 
	- part.write(...): Part를 통해 전송된 데이터를 저장할 수 있다.

- 큰 용량의 파일을 업로드를 테스트 할 때는 로그가 너무 많이 남아서 다음 옵션을 끄는 것이 좋다.

```yaml
logging.level.org.apache.coyote.http11=debug
```

- 다음 부분도 파일의 바이너리 데이터를 모두 출력하므로 끄는 것이 좋다.

```java
log.info("body={}", body);
```


```shell
2021-06-27 18:48:08.078 DEBUG 8483 --- [nio-8080-exec-1] o.a.coyote.http11.Http11InputBuffer      : Received [POST /servlet/v2/upload HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Content-Length: 1116
Pragma: no-cache
Cache-Control: no-cache
sec-ch-ua: " Not;A Brand";v="99", "Google Chrome";v="91", "Chromium";v="91"
sec-ch-ua-mobile: ?0
Upgrade-Insecure-Requests: 1
Origin: http://localhost:8080
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryCbSqHj45A6BJot7s
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://localhost:8080/servlet/v2/upload
Accept-Encoding: gzip, deflate, br
Accept-Language: ko,en;q=0.9,ko-KR;q=0.8,en-US;q=0.7
Cookie: Idea-b7b63a36=1eb6b06b-c44c-42cc-9f72-13d84592a7a7

------WebKitFormBoundaryCbSqHj45A6BJot7s
Content-Disposition: form-data; name="itemName"

d
------WebKitFormBoundaryCbSqHj45A6BJot7s
Content-Disposition: form-data; name="file"; filename="áá¨áá¦02-1 MySQL 5.1 áá¥áá¥á«áá­á¼ Makefile áá¢á¼áá¥á¼ áá³áá³ááµá¸áá³(64ááµáá³).txt"
Content-Type: text/plain

./configure \
    '--prefix=/usr/local/mysql'\
    '--localstatedir=/usr/local/mysql/data'\
    '--libexecdir=/usr/local/mysql/bin'\
    '--with-comment=Toto mysql standard 64bit'\
    '--with-server-suffix=-toto_standard'\
    '--enable-thread-safe-client'\
    '--enable-local-infile'\
    '--enable-assembler'\
    '--with-pic'\
    '--with-fast-mutexes'\
    '--with-client-ldflags=-static'\
    '--with-mysqld-ldflags=-static'\
    '--with-big-tables'\
    '--with-readline'\
    '--with-extra-charsets=complex'\
    '--with-plugins=partition,archive,blackhole,csv,federated,heap,myisam,myisammrg,innodb_plugin'\
    '--with-zlib-dir=bundled'\
    'CC=gcc' 'CXX=gcc' 'CFLAGS=-O2' 'CXXFLAGS=-O2'

------WebKitFormBoundaryCbSqHj45A6BJot7s--
]
2021-06-27 18:48:08.100  INFO 8483 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2021-06-27 18:48:08.100  INFO 8483 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2021-06-27 18:48:08.101  INFO 8483 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
2021-06-27 18:48:08.137  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : request=org.springframework.web.multipart.support.StandardMultipartHttpServletRequest@2767359d
2021-06-27 18:48:08.139  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : itemName=d
2021-06-27 18:48:08.139  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : parts=[org.apache.catalina.core.ApplicationPart@33493ad9, org.apache.catalina.core.ApplicationPart@7bf6a4bb]
2021-06-27 18:48:08.139  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : ==== PART ====
2021-06-27 18:48:08.139  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : name=itemName
2021-06-27 18:48:08.139  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : header content-disposition: form-data; name="itemName"
2021-06-27 18:48:08.140  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : submittedFileName=null
2021-06-27 18:48:08.140  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : size=1
2021-06-27 18:48:08.140  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : body=d
2021-06-27 18:48:08.140  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : ==== PART ====
2021-06-27 18:48:08.140  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : name=file
2021-06-27 18:48:08.140  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : header content-disposition: form-data; name="file"; filename="예제02-1 MySQL 5.1 버전용 Makefile 생성 스크립트(64비트).txt"
2021-06-27 18:48:08.141  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : header content-type: text/plain
2021-06-27 18:48:08.141  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : submittedFileName=예제02-1 MySQL 5.1 버전용 Makefile 생성 스크립트(64비트).txt
2021-06-27 18:48:08.141  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : size=719
2021-06-27 18:48:08.141  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : body=./configure \
    '--prefix=/usr/local/mysql'\
    '--localstatedir=/usr/local/mysql/data'\
    '--libexecdir=/usr/local/mysql/bin'\
    '--with-comment=Toto mysql standard 64bit'\
    '--with-server-suffix=-toto_standard'\
    '--enable-thread-safe-client'\
    '--enable-local-infile'\
    '--enable-assembler'\
    '--with-pic'\
    '--with-fast-mutexes'\
    '--with-client-ldflags=-static'\
    '--with-mysqld-ldflags=-static'\
    '--with-big-tables'\
    '--with-readline'\
    '--with-extra-charsets=complex'\
    '--with-plugins=partition,archive,blackhole,csv,federated,heap,myisam,myisammrg,innodb_plugin'\
    '--with-zlib-dir=bundled'\
    'CC=gcc' 'CXX=gcc' 'CFLAGS=-O2' 'CXXFLAGS=-O2'

2021-06-27 18:48:08.141  INFO 8483 --- [nio-8080-exec-1] c.e.c.ServletUploadControllerV2          : 파일 저장 fullPath=/Users/seok/IdeaProjects/spring-mvc2/fileupload/files/예제02-1 MySQL 5.1 버전용 Makefile 생성 스크립트(64비트).txt
```

## 스프링과 파일 업로드

- MultipartFile 주요 메서드 
	- file.getOriginalFilename() : 업로드 파일 명 
	- file.transferTo(...) : 파일 저장

```shell
2021-06-27 18:55:58.628 DEBUG 8553 --- [nio-8080-exec-2] o.a.coyote.http11.Http11InputBuffer      : Received [POST /spring/upload HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Content-Length: 1116
Pragma: no-cache
Cache-Control: no-cache
sec-ch-ua: " Not;A Brand";v="99", "Google Chrome";v="91", "Chromium";v="91"
sec-ch-ua-mobile: ?0
Upgrade-Insecure-Requests: 1
Origin: http://localhost:8080
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryEKurvdoaIsXoip8C
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://localhost:8080/spring/upload
Accept-Encoding: gzip, deflate, br
Accept-Language: ko,en;q=0.9,ko-KR;q=0.8,en-US;q=0.7
Cookie: Idea-b7b63a36=1eb6b06b-c44c-42cc-9f72-13d84592a7a7

------WebKitFormBoundaryEKurvdoaIsXoip8C
Content-Disposition: form-data; name="itemName"

d
------WebKitFormBoundaryEKurvdoaIsXoip8C
Content-Disposition: form-data; name="file"; filename="áá¨áá¦02-1 MySQL 5.1 áá¥áá¥á«áá­á¼ Makefile áá¢á¼áá¥á¼ áá³áá³ááµá¸áá³(64ááµáá³).txt"
Content-Type: text/plain

./configure \
    '--prefix=/usr/local/mysql'\
    '--localstatedir=/usr/local/mysql/data'\
    '--libexecdir=/usr/local/mysql/bin'\
    '--with-comment=Toto mysql standard 64bit'\
    '--with-server-suffix=-toto_standard'\
    '--enable-thread-safe-client'\
    '--enable-local-infile'\
    '--enable-assembler'\
    '--with-pic'\
    '--with-fast-mutexes'\
    '--with-client-ldflags=-static'\
    '--with-mysqld-ldflags=-static'\
    '--with-big-tables'\
    '--with-readline'\
    '--with-extra-charsets=complex'\
    '--with-plugins=partition,archive,blackhole,csv,federated,heap,myisam,myisammrg,innodb_plugin'\
    '--with-zlib-dir=bundled'\
    'CC=gcc' 'CXX=gcc' 'CFLAGS=-O2' 'CXXFLAGS=-O2'

------WebKitFormBoundaryEKurvdoaIsXoip8C--
]
2021-06-27 18:55:58.656  INFO 8553 --- [nio-8080-exec-2] c.e.controller.SpringUploadController    : request=org.springframework.web.multipart.support.StandardMultipartHttpServletRequest@6118ee50
2021-06-27 18:55:58.658  INFO 8553 --- [nio-8080-exec-2] c.e.controller.SpringUploadController    : itemName=d
2021-06-27 18:55:58.658  INFO 8553 --- [nio-8080-exec-2] c.e.controller.SpringUploadController    : multipartFile=org.springframework.web.multipart.support.StandardMultipartHttpServletRequest$StandardMultipartFile@5b36bb71
2021-06-27 18:55:58.659  INFO 8553 --- [nio-8080-exec-2] c.e.controller.SpringUploadController    : 파일 저장 fullPath=/Users/seok/IdeaProjects/spring-mvc2/fileupload/files/예제02-1 MySQL 5.1 버전용 Makefile 생성 스크립트(64비트).txt
```

## 예제로 구현하는 파일 업로드, 다운로드

- 요구사항
	- 상품을 관리 상품 이름
		- 첨부파일 하나
		- 이미지 파일 여러개
	- 첨부파일을 업로드 다운로드 할 수 있다.
	- 업로드한 이미지를 웹 브라우저에서 확인할 수 있다.

```text
고객이 업로드한 파일명으로 서버 내부에 파일을 저장하면 안된다. 
왜냐하면 서로 다른 고객이 같은 파일이름을 업로드 하는 경우 기존 파일 이름과 충돌이 날 수 있다. 
서버에서는 저장할 파일명이 겹치지 않도록 내부에서 관리하는 별도의 파일명이 필요하다.
```
