<img width="697" alt="스크린샷 2022-06-09 15 42 23" src="https://user-images.githubusercontent.com/61769743/172892303-48016be6-b58a-47c5-9e6f-70be60696f87.png">
<img width="955" alt="스크린샷 2022-06-09 15 43 13" src="https://user-images.githubusercontent.com/61769743/172892314-dd8b41d7-d6d6-455d-bcd1-55d0b6186f22.png">


## HttpMessageConverter

spring-web 모듈은 HTTP 요청과 응답 바디를 읽고 쓰기 위해 InputStream과 OutputStream이 아닌 HttpMessageConverter를 가지고 있다.

HttpMessageConverter 객체들은 client side와 (RestTemplate) server side (Spring MVC REST controller들)에서 사용된다.

MIME(Main Media) 타입을 위한 구체적인 구현체들이 제공되고 있으며, client side의 RestTemplate 와 server side의 RequestMappingHandlerAdapter는 기본적으로 등록 되어 있다.

- RestTemplate
    - Spring 3부터 지원, REST API 호출이후 응답을 받을 때까지 기다리는 동기 방식
- RequestMappingHandlerAdapter
    - 특정 요청에 대한 Handler를 찾고 그에 맞는 응답 resolver를 찾아 처리함

### 종류

`StringHttpMessageConverter`

- HTTP 요청과 응답에서 `String` 을 읽고 쓸 수 있음
- 기본적으로 이 converter는 모든 text 미디어 타입(`text/*`)을 지원하고 `Content-Type`을 `text/plain` 으로 지정함
 
<img width="736" alt="스크린샷 2022-06-10 23 24 18" src="https://user-images.githubusercontent.com/61769743/173167470-192a6810-07be-4be3-bedf-3526071d332b.png">


`MappingJackson2HttpMessageConverter`
- Jackson의 ObjectMapper를 사용하여 Json을 읽고 쓸 수 있음
- Json이 제공하는 어노테이션을 사용함으로써 Json 맵핑을 커스텀화 할 수 있음
    - `@JsonIgnore`
- 더 많은 제어가 필요하면 커스텀한 ObjectMapper를 주입할 수 있음
- 기본적으로 application/json content type을 지원함
![스크린샷 2022-06-11 10 03 48](https://user-images.githubusercontent.com/61769743/173167483-bf7690db-d067-4a42-a9b9-a5c40dc1b5ff.png)
MappingJackson2HttpMessageConverter가 ObjectMapper를 가지는 모습
![스크린샷 2022-06-11 10 03 27](https://user-images.githubusercontent.com/61769743/173167501-22a42df3-9929-4411-80f6-707e5c246177.png)



`MappingJackson2XmlHttpMessageConverter`
- [Jackson XML](https://github.com/FasterXML/jackson-dataformat-xml)의 확장 버전인 `XmlMapper` 를 사용하여 XML을 쓰고 읽을 수 있음
- JAXB 혹은 Jackson이 제공하는 어노테이션을 사용하여 XML 맵핑을 커스텀화 할 수 있음
- 더 많은 제어가 필요하다면 ObjectMapper의 property를 통해 커스텀한 XmlMapper를 주입할 수 있음
- 기본적으로 해당 converter는 application/xml를 제공함

![스크린샷 2022-06-11 10 14 01](https://user-images.githubusercontent.com/61769743/173167535-77e61962-3919-433a-8fda-cc85fa54e60c.png)

스프링 부트에 XmlMapper 의존성이 존재하지 않음. 사용하려면 gradle 의존성을 추가해야함.

![스크린샷 2022-06-11 10 21 56](https://user-images.githubusercontent.com/61769743/173167524-50ebb262-3867-4722-86fa-e831541cda41.png)

