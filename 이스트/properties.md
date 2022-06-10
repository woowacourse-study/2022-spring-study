### Profile

`@Profile` 을 지정해주어 환경에 따라 다른 설정 가능

```java
@Configuration
@Profile("local")
public class DumbHeadConfig {
		//DataSource 설정
}
```

1. 이를 컨테이너 초기화 전에 setActiveProfiles() 를 통해 프로필 선택 가능

```java
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        applicationContext.getEnvironment().setActiveProfiles("local");
        applicationContext.register(DumbHeadConfig.class);
        applicationContext.refresh();
```

1. 실행시 명령행의 -D 옵션으로 설정 가능

```java
java -jar -Dspring.profiles.active=dev 0.0.1-SNAPSHOT.jar &
```

### Properties

외부에서 값을 주입받아 설정 가능

```java
gift=for you
```

```java
@Configuration
public class DumbHeadConfig {

    @Value("${gift}")
    private String val;

		//DataSource를 외부에서 가져온 val 값들을 이용해 설정
}
```

### Spring Boot에서의 Properties

application-{profile}.properties 파일에서 {profile}을 하나의 Profile로 취급

spring.profiles.active 값으로 원하는 profile 지정 가능

**application.properties (기본 설정)**

```java
spring.profiles.active=local
```

**application-local.properties**

```java
gift=for you
```

```java
@Service
public class TTT {

    @Value("${gift}")
    private String fff;

    public String getFff() {
        return fff;
    }
}
```

```java
@SpringBootTest
public class fffTest {

    @Autowired
    private TTT ttt;

    @Test
    void fff() {
        //then
        assertThat(ttt.getFff()).isEqualTo("for you");
    }
}
```

![Screen Shot 2022-06-11 at 2 42 49 AM](https://user-images.githubusercontent.com/64204666/173130503-5beb90e4-af8c-4eb8-9aa9-5b11dfb5cb6c.png)
