---
title: 스프링 프레임워크 핵심 정리 1 (XML설정)
titleEn: 
author: BabyK
date: 2026-02-07
category: Web
layout: post
tags: [Web, Spring]
published: true
---
<br>
  
흔히 스프링의 강점으로 DI 나 객체의 라이프사이클 관리라는 프레임워크 자체의 제어권 역전을 강조하지만  
프레임워크는 사용자의 의도에 따라 설계되고 이런 특성들은 단순히 코드 패턴으로도 구현가능하다.  
오히려 번거롭고 쓸데없이 헤비한 프레임워크에 대해 왜? 라는 생각을 해볼 수 있다.  
  
그 답은 퍼포먼스가 아니라 프레임워크가 가지고 있는 구조적 특성 (작업 용이성) 때문이다.  
개발자는 디자인 패턴이나 코드로 작성된 구조에 신경쓸 필요없이 비즈니스 로직에만 집중 할 수 있다.  
화면과 서비스, 리포지토리를 분리해 서로의 역할과 충돌지점을 적절히 분리한 구조적 특징들은 점차     
REST 서비스의 유행과 함께 프론트/백엔드의 역할을 더욱 명확히 구분하는 트렌드로 발전되었고   
가벼운 마이크로 서비스의 사용으로 이어졌다.  
이러한 구조적 분리는 대형 프로젝트의 관점에서 보면 협업, 유닛 테스트, 전처리 나 실행 로직의  
손쉬운 편집 및 재구성과 같은 장점으로 작용할 수 있다.  
  
스프링의 라이브러리와 내부 구조를 이해하는 것은 중요하고 필요한 일이지만 개인적인 생각으로는 경험적인 측면에서 단순히 해봤는가 라는 관점이면 충분하다고 생각한다. 
스프링은 개발 편의를 위해 사용되는 도구일 뿐이다.   
각각의 역할들을 이해하고 수정, 디버깅하는 것은 중요한 일이지만 우리에게는 더 큰 목적이 있다.   
이 도구를 사용해 빠르고 효율적으로 웹서비스를 완성 시키는 것이다.  
  
코드 내부를 들여다볼 필요가 없도록 레이어를 설계해 작업 효율을 높인 추상화적 특성을 기억하자.  
특히나 AI 로 인해 작업시간은 줄어들었고 개발자 한명이 돌봐야할 서비스나 툴의 개수가 예전보다 더 많아졌다.  
과거와 달리 이제는 라이브러리의 호환문제를 해결하기 위해 하루종일 씨름하는 일도 없다.  
  
두꺼운 책이나 메뉴얼을 두고 클래스를 하나하나 분석하기 보다는 주요 세팅값의 의미와 사용법, 경로설정들을 기억하고 사용법에 익숙해지면 그것으로 OK 라고 생각한다. 남는 시간에는 코드에 사용된 패턴이나 인프라, 혹은 웹개발의 핵심인 DB 와 관련된 지식에 집중하자.  
요즘은 더 쉽게 사용할 수 있는 스프링 부트나 Node.js 등 더 쉽고 편리한 도구들이 많다.  

스프링이 변화해 온 과정을 살펴보면 크게 3단계로 구분할 수 있는데  
-> XML 을 사용하던 초기 설정방식  
-> @Configuration, @Bean 어노테이션을 사용해 빈을 명시하는 방식  
-> Spring Boot 현재. 자동 설정되어 있어 세팅할 부분이 별로 없음

이중에서 XML 을 사용하는 초기의 레거시 방식에 대해 알아보자.  
<br>

### 설치 라이브러리
대부분의 오류는 설치된 라이브러리 사이의 충돌로 인해 발생하는데 pom.xml 메이븐 의존성 설정 부분에서 각각의 스프링 버젼에 맞는 라이브러리들의 호환 버젼을 잘 확인하여 작성해야 한다.  

**스프링 핵심 모듈**  
아래 핵심 모듈중 스프링이 관리하는 인스턴스인 Bean 들의 관계를 형성하고 대부분의 설정을 담당하는 주요 모듈은 spring-context 이다.   

- spring-core
- spring-context
- spring-webmvc
- spring-aops
- spring-jdbc
- spring-tx
- spring-test

**추가 라이브러리**
- jakarta.servlet.jsp.jstl &nbsp;&nbsp; (라이브러리 버젼 주의)  
- jackson-databind  
- lombok - 클래스 생성자, getter, setter 생성 &nbsp;&nbsp; (IDE 에서 오류 메시지를 없애려면 추가 설정 필요함)
- log4j - 로거
- jdbc driver - Oracle, MySQL, MariaDB 등등
- HikariCP - jdbc pool API
- junit - 유닛 테스트
- mybatis - DB 데이터 교환용 객체의 매핑을 도와주는 ORM 도구
- aspectjweaver, aspectjrt - 스프링 AOP를 구현하는 라이브러리
<br>
<br>

### 설정
스프링이 시작될때 아래 세 파일에 명시된 내용들이 모두 취합되어 서비스가 생성된다.  
웹 영역이 없어도 서비스 영역은 사용가능하기 때문에  
보통 root-context.xml (어떤 이름이든 상관없음) 과 servlet-context.xml 둘을 나눠서 설정해준다.  
- pom.xml - Maven 라이브러리 리스트
- root-context.xml - 비즈니스/DB 영역
- servlet-context.xml - 웹 (화면) 요청 처리 영역
  
스프링의 처리과정을 보면  

-> 브라우저  
-> DispatcherServlet (요청 주소의 매핑)  
-> @Controller 실행   
-> @Service (DB 처리)  
-> @Controller 실행 반환  
-> ViewResolver &nbsp;&nbsp; (출력될 화면 JSP/HTML 선택)  
-> 브라우저 화면 출력  

`DispatcherServlet` - 클라이언트가 브라우저에서 요청한 주소를 가지고 @Controller 에 작성된 매핑 주소 ( @GetMapping("/board/list") ) 와 연결한다.  
실제 DispatcherServlet 은 요청 연결부터 ViewResolver 의 과정까지 모든 웹 처리의 모든 흐름을 관리한다.  

조금더 정확하게 설명하면 스프링 개발에 사용되는 톰캣은 서버의 기능이 추가된 Servlet 이고    
HTTP 요청은 브라우져에서 톰캣(서블릿) 에 의해 전달될 서블릿이 결정된다.  
<a onclick="showHtmlInNewTab(3)" style="color: #dc690bff; font-style: italic;">Web.xml</a> 파일을 열어보면 servlet-class 에 스프링의 DispatcherServlet 이 명시되어 있다.  

  
`<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>`  
  
웹은 HTTP 프로토콜로 통신되고 서블릿의 역할은 서버의 맨 앞에서 HTTP 요청을 받아 자바 코드로 변환해 DispatcherServlet 으로 넘겨주고 서버에서 처리된 요청을 HTTP 응답으로 만들어 내보내는 역할이라고 볼 수 있다.  

프론트와 백엔드가 완전히 분리된 구조에서는 톰캣은 REST 백엔드 서버의 입구 역할을 하고 프론트에서 화면단의 파일들을 전송해주는 Nginx 와 같은 프론트 서버를 별도로 구성하게 되는 경우가 많다.  
<br>
<br>

#### 파일 경로 예시
<div class="row" align="center">
    <img src="/img/2026-01-28-spring_framework_quickReview1_01.png">
</div>

  * src/main/java  - 자바 소스코드  
  * src/main/java/mapper - mybatis 매핑용 .java 인터페이스 파일
  * src/main/resource  - 기타 설정파일( mybatis 설정 및 매퍼, * log4j2 설정 등)  
  * src/main/webapp/resources - 정적파일 ( css, js )  
  * src/main/webapp/WEB-INF - 주요설정파일 ( web.xml, * root-context.xml, servlet-context.xml )  
  * src/main/webapp/views - JSP 파일  
  * src/test - 유닛테스트 소스코드  
  * src/target - 컴파일하고 패키징한 결과 ( .class .war .jar )   ( target 폴더는 .gitignore 에 포함되어 깃에 업로드 안됨 )
<br>


#### 서버기동시 로딩 순서

-> Tomcat 시작    
-> web.xml 읽음  
-> ContextLoaderListener  
-> root-context 로딩 (Service, DB, MyBatis, Transaction)  
->  DispatcherServlet 생성  
-> servlet-context 로딩  (Controller, ViewResolver, MVC 설정)  
-> 요청 대기  


#### *pom.xml* &nbsp;&nbsp; <a onclick="showHtmlInNewTab(0)" style="color: #dc690bff; font-style: italic;">예시보기</a>
이클립스의 경우 프로젝트를 선택하고 마우스 우측클릭 하고 `Configure` -> `Convert to Maven Project` 를 선택해서  
프로젝트를 Maven 프로젝트로 지정하면 위의 라이브러리들을 리스트업 할 수 있는 pom.xml 파일이 추가된다.  
<br>

#### *root-context.xml* &nbsp;&nbsp; <a onclick="showHtmlInNewTab(1)" style="color: #dc690bff; font-style: italic;">예시보기</a>    
스프링이 자동으로 컴포넌트를 찾아 Bean 으로 등록하고 어노테이션 기반 설정을 활성화하게 해줌  
이 파일은 톰캣 설정파일 web.xml 에 등록해서 톰캣이 기동될 때 읽어들인다

```xml
// pom.xml 에 추가한 라이브러리의 bean 들을 설정해준다
<beans xmlns="..."
	xmlns:xsi="..."
	xmlns:util="..."
	xmlns:context="..."
	xmlns:mybatis-spring="..."
	xmlns:aop="..."
	xmlns:tx="..."
	xsi:schemaLocation="...">
```

* `<context:component-scan base-package="com.baby.service"/>`  
패키지 경로 com.baby.service 에 생성한 클래스에서 Service Bean 을 찾아 자동등록하고 클래스에 추가한 어노테이션
@Service @component @Repository 을 사용
  
* `<bean class="org.mybatis.spring.SqlSessionFactoryBean">`
mybatis 실행 엔진 생성문으로 각 설정은 사용할 DB연결 (예시에서는 히카리풀을 사용한 오라클 JDBC), 매퍼(SQL 작성파일) 의 위치와 설정파일 경로 등을 선언

* `<bean id="transactionManager" class="DataSourceTransactionManager">`  
@Transactional 이 실제로 동작하게 만드는 핵심 객체

* `<tx:annotation-driven/>`  
@Transactional 어노테이션을 인식하게 만드는 설정  

* `<mybatis-spring:scan base-package="com.baby.mapper" />`  
@Mapper 어노테이션이 사용된 interface를 찾아 자동 등록  
설정하지 않으면 '<bean class="BoardMapper"/>' 와 같이 별도로 매퍼들을 하나하나 설정해줘야 함  

* `<context:component-scan base-package="com.baby.aop"/>`  
AOP 클래스 등록  
`<aop:aspectj-autoproxy/>`  
AOP 활성화. @Aspect
<br>
<br>

#### *servlet-context.xml* &nbsp;&nbsp; <a onclick="showHtmlInNewTab(2)" style="color: #dc690bff; font-style: italic;">예시보기</a>  
Spring MVC 의 웹요청 처리 부분을 담당하는 DispatcherServlet 설정  

* `<context:component-scan base-package="com.baby.controller"/>`   
컨트롤러 자동등록. @Controller

* `<bean class="InternalResourceViewResolver">`  
  `prefix = /WEB-INF/views/`  
  `suffix = .jsp`  
  ViewResolver 설정. 컨트롤러가 변환한 문자열을 실제 JSP 경로로 변환  
  WEB-INF 에 JSP 파일을 모아두는 이유는 브라우저가 직접 전근할 수 없고 항상 Conroller를 거치게 하기 위함으로 
  반대로 resources 경로에 모아둔 css/js 같은 정적 파일들은 Spring Controller 를 거치지 않고 바로 파일이 반환됨  

  ```java
  // 실제 JSP 파일 경로 '/WEB-INF/views/board/list.jsp'  
  @GetMapping("read")
  	public String read(
      return "board/list" // JSP 화면 렌더링
    )
  ```



* `<mvc:resources mapping="/resources/**" location="/resources/"/>`  
  정적 파일(css/js) 처리    

* `<mvc:annotation-driven/>` MVC 기능 활성화  


* `<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver"/>`  
파일 업로드 기능 활성화. 없으면 MultipartFile 처리불가  
```java
@PostMapping("/upload")
public String upload(MultipartFile file) {}
```
<br>

#### *web.xml* &nbsp;&nbsp; <a onclick="showHtmlInNewTab(3)" style="color: #dc690bff; font-style: italic;">예시보기</a>  
서버(톰캣) 가 애플리케이션을 시작할 때 가장 먼저 읽는 설정  

* ContextLoaderListener  
비즈니스 로직과 DB 가 설정된 **root-context.xml** 를 로딩해서 Bean 들을 생성  

  ```xml
  <listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
  </listener>

  <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>WEB-INF/spring/root-context.xml</param-value>
  </context-param>
  ```

* `<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>`   
실제 DispatcherServlet 클래스경로. 스프링 MVC 의 핵심으로 모든 웹 요청의 입구가 된다  

* **servlet-context.xml** 를 등록
  ```xml
    <init-param>
      <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>WEB-INF/spring/servlet-context.xml</param-value>
    </init-param>
  ```

* `<load-on-startup>`  
  서버 시작시 DispatcherServlet 바로 생성  
* `<multipart-config>`  
  파일 업로드 경로, 파일 하나의 최대크기, 요청 전체 최대크기, 메모리 저장한계 설정   

* `<servlet-mapping>`  
  가장 중요한 부분으로 요청 매핑 주소를 설정한다. 여기 설정한 이름이 주소의 맨 앞부분에 추가되어 url이 구성된다

<br>
<br>
<br>
<br>

참조  
<a href="https://docs.spring.io/spring-framework/reference/core/appendix.html" target="_blank" style="color: rgb(68, 25, 240); font-style: italic;">https://docs.spring.io/spring-framework/reference/core/appendix.html</a>

<a href="https://docs.spring.io/spring-framework/reference/core/beans.html" target="_blank" style="color: rgb(68, 25, 240); font-style: italic;">https://docs.spring.io/spring-framework/reference/core/beans.html</a>

<script>
    function showHtmlInNewTab(num) {
      let fetchAddr = '';
      switch(num) {
        case 0:
          fetchAddr = '/source/pom.xml';
          break;
        case 1:
          fetchAddr = '/source/root-context.xml';
          break;
        case 2:
          fetchAddr = '/source/servlet-context.xml';
          break;
        case 3:
          fetchAddr = '/source/web.xml';
          break;
      }
      fetch(fetchAddr)
        .then(res => res.text())
        .then(data => {
          const encodedHtml = data
            .replace(/</g, '&lt;')  
            .replace(/>/g, '&gt;');

          const newWindow = window.open();
          newWindow.document.write(`
          <body style='background:black; color:white;'>
          <pre style='white-space:pre-wrap; word-wrap:break-word;'>${encodedHtml}</pre>
          </body>
          `);
          newWindow.document.close();
        });
    }
</script>