### 설정의 역사

1. 원래는 Spring 프로젝트를 구축하기 위해 뷰 리졸버 등을 수동으로 설정해주어야 했다.
2. `@Enable~~` 의 등장으로 (ex: `@EnableWebMvc`) 설정을 자동화하기 시작
3. 이마저도 번거롭기 때문에 설정을 자동으로 해주는 SpringBoot 가 등장

스프링이 자동 설정해주는 것 외에 커스텀한 설정이 필요

→ `~Configurer` 인터페이스 구현을 통해 추가 설정 가능

`SpringBoot` 를 사용하면서 `WebMvcConfigurer` 를 구현한 설정 파일에 `@EnableWebMvc` 를 붙여주게 되면 기본 스프링 부트 설정이 무시됨!

### 추가 설정

- add~
    - 기본 설정이 없는 빈들에 대해 새로운 빈을 추가
- configure~
    - 수정자를 통해 기존의 설정을 대신하여 등록
- extend~
    - 기존의 설정을 이용하면서 추가로 설정을 확장
    

### 예시

- CORS 설정

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    public static final String ALLOWED_METHOD_NAMES = "GET,HEAD,POST,PUT,DELETE,TRACE,OPTIONS,PATCH";

    @Override
    public void addCorsMappings(final CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedMethods(ALLOWED_METHOD_NAMES.split(","))
                .exposedHeaders(HttpHeaders.LOCATION);
    }
}
```

- PathVariable 을 사용할 때 대응하는 값으로 받아 key value 쌍으로 받고 싶을때

```java
@RestController
public class pppController {

    @GetMapping("/{id}")
    public ResponseEntity dumbHeadController(@PathVariable Long id, @MatrixVariable String name) {
        System.out.println(id + name + "!!!");
        return ResponseEntity.ok().build();
    }
}
```

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        final UrlPathHelper urlPathHelper = new UrlPathHelper();
        urlPathHelper.setRemoveSemicolonContent(false);
        
        configurer.setUrlPathHelper(urlPathHelper);
    }
}
```

→ 현재 버전에서는 추가 설정없이 가능

- 특정 권한을 가진 사람만 접근 가능하게 하기
    
    ```java
    @RestController
    public class pppController {
    
        @GetMapping
        public ResponseEntity<String> getDumbHead() {
            return ResponseEntity.ok().body("you are dumb head");
        }
    }
    ```
    
    ```java
    public class DumbHeadInterceptor implements HandlerInterceptor {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            final String role = AuthorizationExtractor.extract(request);
            if (role.equals("dumbHead")) {
                return true;
            }
            return false;
        }
    }
    ```
    
    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
    
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(new DumbHeadInterceptor());
        }
    }
    ```
