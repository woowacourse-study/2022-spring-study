# 서블릿 MVC vs 스프링 MVC

Servlet 환경 mvc를 살펴보고, 스프링에서 제공하는 기능을 알아보자!

## Servlet

- **Dynamic Web Page를 만들 때 사용되는 자바 기반의 웹 애플리케이션 프로그래밍 기술**
- Tomcat : **자바 서블릿 컨테이너 = 서블릿을 갖고 있고 서블릿을 적절히 관리하는 애로 생각하면 된다!**

![image](https://user-images.githubusercontent.com/61769743/171661612-0ff4442c-21ce-40b7-8981-45c50777d228.png)
<img width="1263" alt="images_jihoson94_post_32e7d1fa-4cfb-4e56-8e66-c1627f00d222_스크린샷 2021-05-16 오후 7 39 23" src="https://user-images.githubusercontent.com/61769743/171661640-93ee9b64-ae61-4d79-8aa3-9df33815df23.png">

## 주목해야 할 Servlet 주기

- init()
    - 톰캣이 서블릿이 생성되면 init() 메소드를 실행하여 서블릿을 초기화함
    - 서블릿 생성 시 최초 1번 실행
- **service()**
    - request를 처리하는 메소드
- destroy()
    - 서블릿을 제거할 때 실행되는 메소드
    - 보통 서버가 종료될 때 실행됨

## Servlet 주기 테스트

톰캣 설정파일에 Servlet 등록

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd" id="WebApp_ID" version="2.4">
	<display-name>newmadamequery</display-name>
	<welcome-file-list>
		<welcome-file>index.html</welcome-file>
	</welcome-file-list>

	<servlet>
		<servlet-name>CorinneServlet</servlet-name>
	<servlet-class>controller.CorinneServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>CorinneServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
</web-app>
```

HttpServlet 재정의

```java
package controller;

import java.io.IOException;
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class CorinneServlet extends HttpServlet {

    @Override
    public void init() {
        System.out.println("Init Servlet!");
        // 요청에 필요한 요소들을 초기화
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) 
    	throws ServletException, IOException {
        System.out.println("In service");
        // 요청에 맞는 Controller를 찾고 로직 실행
    }

    @Override
    public void destroy() {
				super.destroy();
        System.out.println("Destroy Servlet!");
    }
}
```
<img width="707" alt="스크린샷 2022-06-02 14 13 21" src="https://user-images.githubusercontent.com/61769743/171662123-0ddbdd36-ce70-4bc9-8028-6a9950d551a5.png">
<img width="760" alt="스크린샷 2022-06-02 14 13 45" src="https://user-images.githubusercontent.com/61769743/171662131-ba7db5e5-36ff-4291-8b6d-fa147761d37b.png">

## 이제 Servlet으로 MVC를 만들어 보자

```java
package controller;

import java.io.IOException;
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class CorinneServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    
    private ControllerMapping controllerMapping;

    @Override
    public void init() {
        // 요청 Uri, Method에 맞는 Controller 정보를 가지고 있는 RequestMapping을 초기화
        controllerMapping = new ControllerMapping();
        controllerMapping.initMapping();
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) 
    	throws ServletException, IOException {
	    	request.setCharacterEncoding("UTF-8");
				
        // 요청의 Method와 Uri를 구한다.
        String method = request.getMethod();
        String contextPath = request.getContextPath();
        String servletPath = request.getServletPath();
	    	
        // RequestMapping 에서 요청 Uri, Method에 해당하는 Controller를 불러온다.
        Controller controller = controllerMapping.findController(method, servletPath);
        try {
            // controller를 실행하고, 이동할 View uri를 불러온다.
            String uri = controller.execute(request, response);

	    // uri가 redirect라면 해당 uri로 redirect 시킨다.
            if (uri.startsWith("redirect:")) {	
            	String targetUri = contextPath + uri.substring("redirect:".length());
            	response.sendRedirect(targetUri);        
            }
            // uri가 view uri라면 해당 view로 이동시킨다.
            else {
            	RequestDispatcher requestDispatcher = request.getRequestDispatcher(uri);
                requestDispatcher.forward(request, response);
            }                   
        } catch (Exception e) {
            throw new ServletException(e.getMessage());
        }
    }

    @Override
    public void destroy() {
        super.destroy();
    }
}
```

```java
package controller;

import java.util.HashMap;
import java.util.Map;

public class ControllerMapping {

    private final Map<String, Controller> mappings = new HashMap<>();

    public void initMapping() {
        mappings.put("GET:/main", new ForwardController("/user/mainPage.jsp"));
        mappings.put("GET:/user", new ForwardController("/user/login.jsp"));
        mappings.put("POST:/user", new LoginController());
    }

    public Controller findController(String method, String uri) {
        final String key = method + ":" + uri;
        return mappings.get(key);
    }
}
```

### RequestDispatcher는 뭘까?

- 원하는 자원으로 이동해주는 역할을 함
- ServletContext 내에 포함되어 있기 때문에 특정 서블릿 내에 있는 자원으로만 이동할 수 있음


## Spring Container
![image](https://user-images.githubusercontent.com/61769743/171662745-3b487119-83c1-4fac-b4e7-5b279041d89a.png)
- Spring은 Servlet을 활용한 프레임워크
- 위에서 확인한 CorinneServlet 처럼 Spring은 **DispatcherServlet**을 가지고 있다.
- web.xml
    
    ```xml
    <servlet>
    	<servlet-name>dispatcher</servlet-name>
    	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    	//..	
    </servlet>
    <servlet-mapping>
    	<servlet-name>dispatcher</servlet-name>
    	<url-pattern>/</url-pattern>
    </servlet-mapping>
    ```
- SpringBoot에서는 DispatcherServletAutoConfiguration으로 자동 설정됨

![장바구니-3](https://user-images.githubusercontent.com/61769743/171662910-a7194c0e-430a-4439-920d-211b7632fc65.jpg)

## DispatcherServlet의 동작 과정

- 요청이 전송 되면 HandlerMapping으로 URL과 매칭되는 HandlerAdapter 검색
- 매칭된 HandlerAdapter로 컨트롤러를 실행하여 ModelAndView를 반환받음
- 이 응답을 처리할 ViewResolver를 찾아 View를 찾고 저장된 Model를 request에 저장함
- 해당 View 파일(jsp)로 request, response를 전달

|HttpServlet|DispatcherServlet|
|------|---|
|init()|onRefresh()|
|service()|doService()|
|destroy()|부모인 FrameworkServlet의 destroy() 사용|

## onRefresh() == init() 동작 과정

- resolver들과 **handlerMappings**를 초기화한다.

```java
@Override
protected void onRefresh(ApplicationContext context) {
    initStrategies(context);
}

/**
 * Initialize the strategy objects that this servlet uses.
 * <p>May be overridden in subclasses in order to initialize further strategy objects.
 */
protected void initStrategies(ApplicationContext context) {
    initMultipartResolver(context);
    initLocaleResolver(context);
    initThemeResolver(context);
    initHandlerMappings(context);
    initHandlerAdapters(context);
    initHandlerExceptionResolvers(context);
    initRequestToViewNameTranslator(context);
    initViewResolvers(context);
    initFlashMapManager(context);
}
```

## doService() == service() 의 동작 과정

```java
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
// ...
  try {
    doDispatch(request, response);
  }
  finally {
    // ...	
  }
}
```

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {				
  // handelrMappings 에서 request에 맞는 Handler를 찾아옴
  HandlerExecutionChain mappedHandler = getHandler(processedRequest);
  HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

  ModelAndView mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

  applyDefaultViewName(processedRequest, mv);
  mappedHandler.applyPostHandle(processedRequest, response, mv);

  processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}

@Nullable
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
  if (this.handlerMappings != null) {
    for (HandlerMapping mapping : this.handlerMappings) {
      HandlerExecutionChain handler = mapping.getHandler(request);
      if (handler != null) {
        return handler;
      }
    }
  }
  return null;
}
```

HandlerAdapter가 일종의 Controller라고 생각하면 된다. 

왜 Controller를 반환하지 않고 HandlerAdapter를 반환할까?

- `@Controller` 를 사용하지 않고 자신이 직접 만든 클래스를 이용해서 클라이언트의 요청을 처리할 수도 있기 때문
- DispatcherServlet 입장에서는 클라이언트 요청을 처리하는 객체의 타입이 반드시 `@Controller` 를 적용한 타입일 필요는 없음
- 이러한 이유로 스프링 MVC는 웹요청을 실제로 처리하는 객체를 핸들러 라고 표현함

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
			@Nullable Exception exception) throws Exception {

  // ...
  // Did the handler return a view to render?
  if (mv != null && !mv.wasCleared()) {
    render(mv, request, response);
    if (errorView) {
      WebUtils.clearErrorRequestAttributes(request);
    }
  }
  //...
}
```

```java
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
  // ...
  View view;
  String viewName = mv.getViewName();
  if (viewName != null) {
    view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
  }
  else {
    view = mv.getView();
  }

  view.render(mv.getModelInternal(), request, response);
}
```

### InternalResourceView

```java
@Override
protected void renderMergedOutputModel(
    Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
  exposeModelAsRequestAttributes(model, request);

  // ...
  RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);

  if (useInclude(request, response)) {
    rd.include(request, response);
  }

  else {
    rd.forward(request, response);
  }
}
```

```java
protected void exposeModelAsRequestAttributes(Map<String, Object> model,
    HttpServletRequest request) throws Exception {

  model.forEach((name, value) -> {
    if (value != null) {
      request.setAttribute(name, value);
    }
    else {
      request.removeAttribute(name);
    }
  });
}
```

## destroy()

```java
@Override
public void destroy() {
  getServletContext().log("Destroying Spring FrameworkServlet '" + getServletName() + "'");
  // Only call close() on WebApplicationContext if locally managed...
  if (this.webApplicationContext instanceof ConfigurableApplicationContext && !this.webApplicationContextInjected) {
    ((ConfigurableApplicationContext) this.webApplicationContext).close();
  }
}
```

### 간단 비교

|CorinneServlet|DispatcherServlet|
|------|---|
|ControllerMapping|HandlerMapping|
|Controller|HandlerAdapter|
|RequestDispatcher|View|
