### AOP가 뭘까?

AOP는 Aspect Oriented Programming 의 약자

![](https://t1.daumcdn.net/cfile/tistory/994AA3335C1B8C9D28)

주황색 로직은 A,B,C 에서, 파란색 로직은 A,C 에서, 빨간색 로직은 A,B 에서 중복적으로 사용됨

→ Cross cutting concern 이라고 부름

→ 전체 어플리케이션에서 여기저기 사용되는 부가 기능들을 상속 등으로 처리하기에는 깔끔하지 못함

이러한 공통된 기능을 모듈화하여 분리하는 것을 AOP라고 함

→ 횡단하는 부분을 잘라낸다 하여 Cross cutting

→ 핵심적인 비즈니스 로직만 작성하고 공통적인 기능은 분리하여 재사용해 중복을 제거

![Screen Shot 2022-05-25 at 9 06 10 PM](https://user-images.githubusercontent.com/64204666/170512442-36fb96f4-27b7-487a-aea0-25b39858e756.png)

위의 코드는 비즈니스 로직을 수행하고 완료하면 트랜잭션을 커밋하고 예외가 발생하면 롤백되는 코드

트랜잭션 관련 코드가 매번 작성되기 때문에 코드 중복이 일어남

![Screen Shot 2022-05-25 at 9 10 18 PM](https://user-images.githubusercontent.com/64204666/170512565-aa2cf5c3-ef85-493e-a81d-ea686c268a9c.png)

공통적인 기능을 모듈로 분리해 재사용하여 코드 중복을 없앰

### 어떻게 가능한걸까?

프록시 패턴

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FEECrr%2FbtqFWZhqAhT%2Fl8kDltgwVpC7mAEC1uwKG1%2Fimg.png)

![Screen Shot 2022-05-26 at 1 29 14 AM](https://user-images.githubusercontent.com/64204666/170512776-9262af23-e644-4cec-9836-670eb1a4a2b7.png)

![Screen Shot 2022-05-26 at 1 27 28 AM](https://user-images.githubusercontent.com/64204666/170512857-7750b2bb-6061-488e-9d52-121c31ff7c80.png)

공통적인 기능을 프록시 클래스를 이용해 분리하여 핵심 기능 로직만 존재하도록 함

→ 이 역시도 메서드가 많아진다면 코드 중복이 많이 일어남

→ 이를 해결하기 위해 나온 것이 Spring AOP

### Spring AOP

![Screen Shot 2022-05-26 at 1 27 28 AM](https://user-images.githubusercontent.com/64204666/170512857-7750b2bb-6061-488e-9d52-121c31ff7c80.png)

이를 통해 공통 부가 기능을 사용하는 다수의 핵심 기능들을 지정하여 코드 중복을 줄일 수 있음

![Screen Shot 2022-05-26 at 2 06 15 AM](https://user-images.githubusercontent.com/64204666/170513181-5d779ad0-de01-4bee-9364-6d87a4281050.png)

![Screen Shot 2022-05-26 at 2 06 41 AM](https://user-images.githubusercontent.com/64204666/170513298-44005429-2d94-49db-8bcc-865d9ae1961b.png)

![Screen Shot 2022-05-26 at 2 06 57 AM](https://user-images.githubusercontent.com/64204666/170513419-77fdb040-7c57-4d3a-b2ee-2a82e90dca8b.png)

annotation으로 지정해 우리가 익숙한 형태로 사용 가능
