# egov-standards
전자정보 프레임워크 표준 개발가이드

## 빌드도구
> 정자정보 프레임워크에서는 maven을 사용하도록 가이드하고 있다.   
> gradle이 성능 및 가독성에서 더 좋고 maven의 장점은 익숙함이다.  
> 기본적으로 gradle로 사용하고 maven도 사용할 수 있게 가이드 하겠다
- [gradle vs maven](https://hyojun123.github.io/2019/04/18/gradleAndMaven/)

## Lombok
> Lombok으로 인해서 편해지는 측면도 있지만 시스템을 이해하지 못한 상태에서 사용하는 경우 많은 문제가 생길 수 있습니다.    
> 권장하는 annotation : @Getter, @RequedArgsConstructor(Data 처리단위), @Slf4j(Logging)  
> 주의를 요하는 annotation : @Setter  
> 제한할 annotation : @Data, @ToString, @Equals, @HashCode

## dependency injection
> egov에서 가이드 하는 field injection을 사용하고 있는데 java, spring 진형에서 지양하는 방법이다. 특정 tool에서는 이 방법을 사용하는 경우 경고를 한다.
```java
public class OrderSerive {
  @Resource("orderRepository")
  // @Autowire
  private OrderRepository orderRepository;
}
```

- 최소한 가이드는 생성자 주입 방식을 사용하는 것으로 하자
- [관련 글](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)
- 생성자 주입의 장점
- 순환참조를 런타임시점이 아닌 컴파일 시점에서 찾을 수 있다. (제일 좋은 애러는 컴파일 애러)
- 주입되는 대상을 불변객체로 만들어 불필요한 위험 범위를 줄일수 있다.
- 테스트 코드 작성이 mock 객체로 변경해서 사용할 수 있다.
- NullPointerException 방지 (컴파일 시점에 애러 발생)

```java
public class OrderSerive {

  private final OrderRepository orderRepository;

  @Autowire // 생략가능(생성자가 하나인 경우) or @RequedArgsConstructor 로 대체 가능
  public OrderSerivce(OrderRepository orderRepository){
      this.orderRepository = orderRepository;
  }
}
```

- RequedArgsConstructor를 사용한 코드
```java
@RequiredArgsConstructor(access = AccessLevel.PROTECTED)
public class OrderSerive {
  private final OrderRepository orderRepository;
}
```

## VO, DTO 생성방식
> javabeans (get,set) 으로 가이드 하고 있다. 불완전 객체에 가변변수를 활용하는 방식으로 추천하지 않지만 많은 SI에서 사용하는
> 방식이고 익숙한 부분이 있어서 고치기 어려울 것 같다..

### 일반적인 javabeans
```java
public class IpAndPort {
  private String ip;
  private int port;

  public String getIp() { return ip; }
  public int getPort() { return port; }
  public void setIp(String ip) { this.ip = ip }
  public void setPort(int port) { this.port = port }
}
```

### Lombook 을 활용한 방식
```java
@Getter
@Setter
public class User {
  private String ip;
  private int port;
}
```

### 최근 방식 
```java
@Getter
public class User {
  private String ip;
  private int port; // 불변 객체로 생성

  // Builder 방식 (불변 객체가 추가되어어 기존 소스는 문제가 없다.)
  @Builder
  public User(String ip, int port){
      this.ip = ip;
      this.port = port;
  }

  // 정적 팩토리 방식
  public static User from(String url){
      String[] spliteData = url.split(":");
      this.ip = spliteData[0];
      this.port = Integer.parint(spliteData[1]);
  }
}
```

  
