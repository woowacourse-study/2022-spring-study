# 학습동기
스프링 책의 10장부터는 본격적으로 스프링 mvc에 대해 학습하게 된다. 
9장의 내용을 읽던 중 스프링의 동작방식 자체에 대해 잘 모른다는 사실을 인지했고 이에 따라 간단하게 스프링의 동작 플로우를 알아보기로 결정했다.  

# 학습내용
![](https://velog.velcdn.com/images/myspy/post/1218939d-4caf-49d4-b7ac-7c6c60c24f12/image.png)
위 그림은 스프링의 전체 실행 흐름도이다. 한번 쯤은 들어본 내용들이지만 정확하게는 모르겠다. 이제 차근 차근 알아보자.
우선 Tomcat(WAS)에 대해 알아보자

## WAS
우선 WAS에 대해 알아보자.
WAS는 Web Application Server의 줄임말이다.
> WAS는 웹 애플리케이션을 실행시켜 필요한 기능을 수행하고 그 결과를 웹 서버에게 전달하는 역할을 한다.

웹 서버가 정적인 자료만 제공할 수 있고 비즈니스 로직 처리, db 연동 문제를 해결하기 위해 등장한 것이 WAS이다.

특징은 다음과 같다.
- php, jsp 등의 언어들을 사용해 동적인 페이지를 만들어낼 수 있다.
- 프로그램 실행 환경과 데이터베이스 접속 기능을 제공한다.
- 비즈니스 로직 수행이 가능하다.
- 웹 서버 + 웹 컨테이너를 합친 형태이다.

특징 중 가장 중요한 계념이 컨테이너이다. 우리는 컨테이너를 통해 웹 서버와 통신해 비즈니스 로직으로 처리된 데이터를 동적으로 반환할 수 있다.

## Tomcat WAS
스프링은 WAS로 tomcat을 사용하고 있음을 위의 흐름도를 보면 알 수 있다. 그럼 tomcat은 무엇일까?

> 톰캣은 웹 서버와 연동하여 실행할 수 있는 자바 환경을 제공하여 자바서버 페이지와 자바 서블릿이 실행할 수 있는 환경을 제공하고 있다.
<위키백과>

쉽게 말하면 JSP(java server page), Servlet을 구동하기 위한 서블릿 컨테이너 역할을 하는 것이다. 톰켓은 JSP와 Servlet이 작동할 수 있는 환경을 제공한다.


위의 흐름도에서의미는 아파치 서버 + 톰캣을 합친 개념으로 Tomcat(Was)라고 명시하고 있다. 그럼 아파치는 무엇일까?

## Apache server
> 아파치 서버란 클라이언트에서 요청하는 HTTP요청을 처리하는 웹서버를 의미한다.

이는 정적 타입의 데이터만을 처리하기 때문에 보통 톰켓과 같이 사용해 WAS를 구성한다.

> Web Server + Web Container(Sevlet Container ) = Apache + Tomcat

이제 Servlet Container에 대해 알아봐야 하는데 그 전에 간단하게 Servlet에 대해 알아보자.

## Servlet
> Servlet은 클라이언트의 요청을 처리하고, 그 결과를 반환하는 자바 웹 프로그래밍 기술이다.

설명하자면 웹서버는 동적인 페이지를 제공하기 위해 다른 곳에 도움을 요청한다. 이를 도와주는 애플리케이션이 servlet이다. servlet을 통해 우리는 간단한 메서드 호출만으로 웹 요청과 응답을 동적으로 다룰 수 있다.

## Servlet Container
이제 Servlet Container에 대해 알아보자.
우선 container란 동적인 데이테들을 가공하여 정적인 파일로 만들어주는 모듈이다.

> 서블릿 컨테이너란 서버에 만들어진 서블릿을 관리해주는 역할을 하는 컨테이너다.

서블릿 컨테이너는 Client의 Request를 받아주고 Response할 수 있게 웹 서버와 소켓을 만들어 통신힌다.
서블릿 컨테이너의 가장 중요한 기능은 **요청을 올바른 서블릿에 처리되도록** 하고, JVM이 해당 요청을 처리한 후에는 **생성된 결과를 올바른 장소에 동적으로 반환**해 준다는 것이다.

## Filter
이제 Filter에 대해 알아보자.
> Fileter란 Dispatcher Servlet에 요청이 전달되기 전/후에 url 패턴에 맞는 모든 요청에 대해 부가작업을 처리할 수 있는 기능을 제공한다.

Filter는 스프링 컨테이너가 아닌 서블릿 컨테이너에 의해 관리 되는 것이고 디스패처 서블릿 전/후에 처리하는 것이다.

Filter를 사용하기 위해서는 javax.servlet의 Filter 인터페이스를 구현해야 한다.

```java
package javax.servlet;

import java.io.IOException;

public interface Filter {

   
    public default void init(FilterConfig filterConfig) throws ServletException {}

    
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    public default void destroy() {}
}
```

- init 메서드
  필터 객체를 초기화하고 서비스에 추가하기 위한 메서드이다. init메서드를 호출하여 필터 객체를 초기화하면 이후의 요청들은 doFilter를 통해 처리된다.

- doFilter 메서드
  Request, Response가 필터를 거칠 때 수행되는 메서드
  파라미터로 FilterChain이 있는데, FilterChain의 doFilter()를 통해 다음 대상으로 요청을 전달한다.

- destroy 메서드
  필터가 소멸될 때 수행되는 메서드


이제 예시를 들어 알아보자.
우선 Filter interface구현하는 class를 생성한다
```java
public class DumbheadFileter implements Filter{

	private final DumbheadExtractor dumheadExtractor;

	@Override
   	 public void init(FilterConfig filterConfig) throws ServletException {
        	System.out.println("Filter가 생성 됩니다.");
    	}

   	 @Override
    	public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException 	{
       	 	System.out.println("==========필터 시작!==========");
        
       		// 멍청이를 거르자 
        	String dumbhead = dumheadPExtractor.extractor((HttpServletRequest) request);
        	if (dumbhead == null) {
        		throw new DumbheadException("fuck off");
        	}
        	filterChain.doFilter(servletRequest, servletResponse);
        	System.out.println("==========필터 종료!==========");
    	}

    	@Override
    	public void destroy() {
        	log.info("Filter가 사라집니다.");
    	}

}

```

그리고 만든 filter를 bean으로 등록한다.

```java
@Configuration 
public class Config() {
	@Bean
   	 public DumbheadFilter dumheadFilter() {
    		FilterRegistrationBean registrationBean = new FilterRegistrationBean(new DumbheadFilter());
        	registrationBean.setOrder(1);
        	return registrationBean;
   	 }
}
```



## Dispatcher Servlet
이제 흐름도에서 Filter다음에 위치한 Dispatcher Servlet에 대해서 알아보자.

> 디스패처 서블릿은 HTTP 프로토콜로 들어오는 모든 요청을 가장 먼저 받아 적합한 컨틀롤러에 위임해주는 Front Controller이다.

1. 클라이언트로부터 어떠한 요청이 들어온다.
2. Tomcat(서블릿 컨테이너)이 요청을 받는다.
3. 프론트 컨트롤러인 디스패처 서블릿이 가장 먼저 요청을 받는다.
4. 공통적인 작업을 먼저 처리한 후에 해당 요청을 처리해야 하는 컨트롤러를 찾는다.
5. Handler Mapping을 통해 요청을 컨트롤러로 위임할 핸들러 어댑터를 찾아서 전달한다.
6. 핸들러 어댑터가 컨트롤러로 요청을 위임힌다.
7. 컨트롤러가 비즈니스로직을 통해 반환값을 반환한다.
8. HandlerAdapter가 반환값을 처리한다.
9. 서버의 응답을 클라이언트로 반환한다.

### Front Controller
> 프론트 컨트롤러는 주로 서블릿 컨테이너의 제일 앞에서 서버로 들어오는 클라이언트의 모든 요청을 받아서 처리해주는 컨트롤러이고 MVC 구조에서 함께 사용되는 디자인 패턴이다.

## Hander Mapping
> 핸들러 매핑이란 요청에 맞는 객체(컨트롤러)를 찾으면 핸들러 어댑터에 컨트롤러 실행을 위임하는 기능을 수행한다.

핸들러 매핑은 요청정보를 통해 컨트롤러를 찾아주는 기능 외에도 인터셉터를 적용해주는 기능도 제공한다.

## Hander Adapter
> 핸들러 어댑터는 핸들러 매핑을 통해 찾은 컨트롤러를 직접 실행하는 기능을 수행한다.

핸들러 어댑터는 컨트롤러(핸들러)가 처리한 결과값을 받아서 ModelAndView 형태로 바꿔 DispatcherServlet에 보내준다. 덕분에 어떤 형태의 객체를 결과값으로 받던지 간에 웹요청을 처리할 수 있게 된다.

## Intercepter
이제 Intercepter에 대해 알아보자.

> 인터셉터는 디스패처 서블릿이 컨트롤러를 호출하기 전과 후에 요청과 응답을 참조하거나 가공할 수 있는 기능을 제공한다.

웹 컨테이너에서 동작하는 필터와 달리 인터셉터는 스프링 컨텍스트에서 동작을 하는 것이다.

핸들러 매핑이 요청에 대한 컨트롤러를 찾을 때 등록된 인터셉터가 있따면 순차적으로 인터셉들을 거쳐 컨트롤러가 실행되도록 하고, 인터셉터가 없다면 바로 컨트롤러를 실행한다.

인터셉터말고 AOP를 사용할 수도 있지만 일반적으로 컨트롤러의 호출 과정에서 처리되는 작업들은 인터셉터를 통해 처리한다.

이유는 다음과 같다.
-  컨트롤러의 타입과 실행 메서드가 모두 다르기 때문에 포인트컷(적용할 범위 설정) 작성이 어렵다.
-  컨트롤러는 파라미터나 리턴 값이 일정하지 않다.


인터셉터를 사용하기 위해서org.springframework.web.servlet의 HandlerInterceptor 인터페이스를 구현(implements)해야 한다.

```java
package org.springframework.web.servlet;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.lang.Nullable;
import org.springframework.web.method.HandlerMethod;


public interface HandlerInterceptor {

default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
         throws Exception {
      return true;
   }

default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
         @Nullable ModelAndView modelAndView) throws Exception {
   }

default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
         @Nullable Exception ex) throws Exception {
   }

}
```

- preHandle 메서드
  preHandle 메서드는 컨트롤러가 호출되기 전에 실행된다.
  컨트롤러 이전에 처리해야 하는 전처리 작업이나 요청 정보를 가공하거나 추가하는 경우에 사용할 수 있다.

- postHandle 메서드
  postHandle 메서드는 컨트롤러를 호출한 후에 실행된다.
  컨트롤러 이후에 처리해야 하는 후처리 작업이 있을 때 사용할 수 있다.

- afterCompletion 메서드
  afterCompletion 메서드는 모든 작업이 완료된 후에 실행된다.

이제 예시를 통해 알아보자.
우선 HandlerIntercepter를 구현하는 class를 생성한다.
```java
public class DumbheadInterceptor implements HandlerInterceptor {
	
    private final DumbheadExtractor dumheadExtractor;

	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
    	System.out.println("전처리");
    
    	// 멍청이를 거르자 
        String dumbhead = dumheadPExtractor.extractor((HttpServletRequest) request);
        if (dumbhead == null) {
        	throw new DumbheadException("fuck off");
        }
    	return true;
	}

	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
                       ModelAndView modelAndView) {
    	System.out.println("후처리");
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
                            Exception ex) {
    	System.out.println("인터셉터 끝");
	}
}

```
그리고 만든 intercepter를 bean으로 등록한다.
```java
@Configuration
public class Config implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new DumbheadInterceptor())
        		.addPathPatterns("/NeoCommunity/*");      	
    }
}
```


이쯤에서 Filter와 Intercepter가 뭐가 다른지 궁금할 것이다. 두 개다 요청이 컨트롤러에 도달하기 전 중간에서 요청을 처리할 수 있다는 공통점이 있다. 차이점이 뭘까??
두개의 차이점을 알아보자.
## Filer vs Intercepter
### Filter
- 서블릿 컨테이너에 의해 관리된다.
- Request/Response 객체를 조작할 수 있다.
- 모든 요청에 대한 로깅또는 감사의 용도로 사용한다.
- 공통된 인증/인가 관련 작업에 사용한다.

### Intercepter
- 스프링 컨테이너에 의해 관리된다.
- Request/Response 객체를 조작할 수 없다.
- API 호출에 대한 로깅 또는 감사의 용도로 사용한다.
- 세부적인 보안 및 인증/인가 관련 작업에 사용한다.
- 컨트롤러로 넘겨주기 위한 정보를 가공하기에 적합하다. 
