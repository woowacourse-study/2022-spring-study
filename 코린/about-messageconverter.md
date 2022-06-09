<img width="697" alt="스크린샷 2022-06-09 15 42 23" src="https://user-images.githubusercontent.com/61769743/172892303-48016be6-b58a-47c5-9e6f-70be60696f87.png">
<img width="955" alt="스크린샷 2022-06-09 15 43 13" src="https://user-images.githubusercontent.com/61769743/172892314-dd8b41d7-d6d6-455d-bcd1-55d0b6186f22.png">


## HttpMessageConverter

The `spring-web` module contains the `HttpMessageConverter` contract for reading and writing the body of HTTP requests and responses through `InputStream` and `OutputStream`.

`HttpMessageConverter` instances are used on the client side (for example, in the `RestTemplate`) and on the server side (for example, in Spring MVC REST controllers).

Concrete implementations for the main media (MIME) types are provided in the framework and are, by default, registered with the `RestTemplate` on the client side and with `RequestMappingHandlerAdapter` on the server side (see [Configuring Message Converters](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-config-message-converters)).

The implementations of `HttpMessageConverter` are described in the following sections. For all converters, a default media type is used, but you can override it by setting the `supportedMediaTypes` bean property. The following table describes each implementation:

spring-web 모듈은 HTTP 요청과 응답 바디를 읽고 쓰기 위해 InputStream과 OutputStream이 아닌 HttpMessageConverter를 가지고 있다.

HttpMessageConverter 객체들은 client side와 (RestTemplate) server side (Spring MVC REST controller들)에서 사용된다.

MIME 타입을 위한 구체적인 구현체들이 제공되고 있으며, client side의 RestTemplate 와 server side의 RequestMappingHandlerAdapter는 기본적으로 등록 되어 있다.

### 종류

`StringHttpMessageConverter`

An `HttpMessageConverter` implementation that can r§ead and write `String` instances from the HTTP request and response. By default, this converter supports all text media types (`text/*`) and writes with a `Content-Type` of `text/plain`.

`MappingJackson2HttpMessageConverter`

An `HttpMessageConverter` implementation that can read and write JSON by using Jackson’s `ObjectMapper`. You can customize JSON mapping as needed through the use of Jackson’s provided annotations. When you need further control (for cases where custom JSON serializers/deserializers need to be provided for specific types), you can inject a custom `ObjectMapper` through the `ObjectMapper` property. By default, this converter supports `application/json`.

`MappingJackson2XmlHttpMessageConverter`

An `HttpMessageConverter` implementation that can read and write XML by using [Jackson XML](https://github.com/FasterXML/jackson-dataformat-xml) extension’s `XmlMapper`. You can customize XML mapping as needed through the use of JAXB or Jackson’s provided annotations. When you need further control (for cases where custom XML serializers/deserializers need to be provided for specific types), you can inject a custom `XmlMapper` through the `ObjectMapper` property. By default, this converter supports `application/xml`.
