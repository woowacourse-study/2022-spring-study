## `@SpringBootApplication` 에 대해서

![스크린샷 2022-05-26 22.13.22.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5ec18a7e-b624-4041-a486-97f6ab0bcb48/스크린샷_2022-05-26_22.13.22.png)

`@SpringBootApplication` 은 아래 3개의 특성으로 대체할 수 있음.

### `@EnableAutoConfiguration`

- 사용자가 추가해야하는 jar 의존 패키지들을 자동으로 스프링 애플리케이션에 설정하도록 함
    - 예를 들면 HSQLDb가 클래스 경로에 존재하고 database 연결 bean들을 수동으로 설정한 바가 없다면 스프링부트는 database를 in-memory로 자동 설정함
- 사용자가 설정을 변경하고 싶다면 특정 부분에 대한 설정만 대체할 수 있음
- 현재 적용된 자동 설정들을 살펴보고 싶다면 애플리케이션을 실행 할 때 `--debug` 옵션을 주면 됨 (`java -jar [jar파일명] --debug`)
    - 자동설정과 일치
        
        ![스크린샷 2022-05-26 13.44.43.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3638e5a5-c706-48de-97ce-cd51159e1999/스크린샷_2022-05-26_13.44.43.png)
        
    - 자동설정과 불일치
        
        ![스크린샷 2022-05-26 13.46.02.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e6b08c27-68be-4c36-ad9b-deffb2c84c9e/스크린샷_2022-05-26_13.46.02.png)
        

### `@ComponentScan`

- application이 위치한 패키지에서 `@Component` 가 적용된 클래스를 스캔함
- `@Filter(type = FilterType.CUSTOM)` 개발자가 직접 만든 필터를 적용하는 것. 필터는 TypeFilter 인터페이스를 구현하여 만들 수 있음.
- TypeExcludeFilter, AutoConfigurationExcludeFIlter는 개발자가 해당 설정에서 제외하고싶은 Type과 자동설정을 정의할 수 있음.
    - 예시로는 WebMvcTypeExcludeFilter가 있음.(TypeExcludeFilter를 상속받음)

### `@Configuration`

- 다른 bean들을 등록하거나 다른 설정 클래스를 import할 수 있게끔 함.
- 스프링에서 CGLIB라는 바이트코드 조작 라이브러리를 사용해서 AppConfig를 상속받은 임의의 클래스를 만들고 그것을 스프링 빈으로 등록

![스크린샷 2022-05-25 15.46.36.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4e643d93-80de-4120-ad57-f0eaff921d2d/스크린샷_2022-05-25_15.46.36.png)

## `@Configuration` 과 `@SpringBootConfiguration` 의 차이

### `@SpringBootConfiguration`

- Application class에 붙는 annotation
- `@Configuration` 과 동일하게 `@Bean` 을 등록하는 메소드가 존재할 수 있음을 나타냄
    
    ![스크린샷 2022-05-26 21.56.40.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/89bfdb7c-9036-41e3-a6e5-6a9de46ea7df/스크린샷_2022-05-26_21.56.40.png)
    
- `@Configuration`과의 다른점은 `@SpringBootConfiguration` 을 사용하면 설정을 자동으로 찾아낼 수 있다는 것. 단위 혹은 통합테스트에 필요함.
    - `@SpringBootTest` 에는 해당 annotation이 필요함. `@SpringBootConfiguration` 을 `@Configuration` 으로 대체하면 아래의 메시지가 발생.
        
        ![스크린샷 2022-05-26 22.04.27.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5e67fa7-c1b1-467d-8a4d-ee78b466bba7/스크린샷_2022-05-26_22.04.27.png)
        
    - `@Autowired` 로 빈을 자동 주입 받고 싶다면 `@SpringBootConfiguration` 이 필요하다는 뜻!!
    - 자동주입이 필요없고 매번 ApplicationContext에서 Bean을 불러와도 된다면, 해당 어노테이션은 필요없음.
        
        ```java
        ApplicationContext context = new AnnotationConfigApplicationContext(HelloApplication.class);
        String[] beanDefinitionNames = context.getBeanDefinitionNames();
        
        AuthService authService = context.getBean(AuthService.class);
        ```
        
- 애플리케이션에 단 한 개만 존재해야 함. 즉, `@SpringBootApplication` 과 `@SpringBootConfiguration` 은 공존할 수 없음.
