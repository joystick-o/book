# Chapter.3 다형성과 추상 타입

객체 지향이 주는 장점은 구현 변경의 유연함이다.  
유연함을 얻을 수 있도록 해주는 또 다른 방법은 추상화에 있는데  
추상화를 가능하게 해주는 다형성에 대한 내용부터 살펴보자.

## 1. 다형성과 상속

다형성은 한 객체가 여러 가지 모습을 갖는다는 것을 의미한다.  
즉, 다형성이란 한 객체가 여러 타입을 가질 수 있다는 것을 뜻한다.

```java
public class Plane {
    public void fly() {
        // 비행
    }
}
```

```java
public interface Turbo {
    public void boost();
}
```

```java
public class TurboPlane extends Plane implements Turbo {
    public void boost() {
        // 가
    }
}
```

TurboPlane 클래스는 Plane 클래스를 상속받고 있고, Turbo 인터페이스도 상속받고 있다.  
이런 관계를 갖는 경우 TurboPlane 객체에 Plane 타입이나 Turbo 타입에 정의된 메서드의 실행을 요청할 수 있다.

```java
TurboPlane tp = new TurboPlane();
tp.fly();
tp.boost();
```

또한, TurboPlane 타입의 객체를 Plane 타입이나 Turbo 타입에 할당하는 것도 가능하다.

```java
TurboPlane tp = new TurboPlane();

Plane p = tp;
p.fly();

Turbo t = tp;
t.boost();
```

### 1.1 인터페이스 상속과 구현 상속

다형성을 구현하기 위해 타입을 상속받는 방법을 사용하는데,  
타입 상속은 크게 인터페이스 상속과 구현 상속으로 구분해 볼 수 있다.

인터페이스 상속은 순전히 타입 정의만을 상속받는 것이다.

![](../../.gitbook/assets/image%20%285%29.png)

구현 상속은 클래스 상속을 통해서 이루어진다.  
구현 상속은 보통 상위 클래스에 정의된 기능을 재사용하기 위한 목적으로 사용된다.

TurboPlane 클래스의 객체는 Plane 타입도 되기 때문에,  
클래스 상속은 구현을 재사용하면서 다형성도 함께 제공해 준다.

TurboPlane 클래스는 Plane에 정의된 fly\(\) 메서드를 새롭게 구현하고 싶을 경우 자신만의 fly\(\) 메서드를 새롭게 구현할 수 있다.

```java
public class TurboPlane extends Plane {
    public void fly() {
        // TurboPlane의 구
    }
}
```

```java
Plane p = new TurboPlane();
p.fly();
```

p의 타입이 Plane 클래스이기 때문에 Plnae 클래스의 fly\(\) 메서드가 호출될 것이라고 생각할 수도 있지만, p가 가르키는 객체의 실제 타입이 TurboPlane 클래스이기 때문에 p.fly\(\) 는 TurboPlane 클래스의 fly\(\) 메서드가 호출되는 것이다.

## 2. 추상 타입과 유연함

추상화\(abstraction\)는 데이터나 프로세스 등을 의미가 비슷한 개념이나 표현으로 정의하는 과정이다.  
예를 통해 감을 잡아보자. 어떤 프로그램을 만드는 데 다음과 같은 세 개의 기능이 있다고 해보자.

* FTP에서 파일을 다운로드
* 소켓에서 데이터 읽기
* DB 테이블의 데이터를 조회

그런데 내용을 더 분석해 보니 위 세 가지 기능은 모두 로그를 수집하기 위한 기능이었다.  
로그를 수집하기 위해 FTP를 이용해 가져오거나, TCP 서버를 이용해 로그 데이터를 읽어 오거나, DB 테이블에 보관된 로그 데이터를 조회하는 것.  
이 세 기능을 추상화하면 '로그 수집' 이라는 개념으로 정의할 수 있는 것이다.

![](../../.gitbook/assets/image%20%286%29.png)

추상화된 타입은 오퍼레이션의 시그니처만 정의할 뿐 실제 구현을 제공하지는 못한다.

추상화를 한다고 해서 반드시 추상 타입을 만들어야 하는 것은 아니다.  
추상 타입을 이끌어 내는 과정에서 추상화가 사용된다는 것은 설명하기 위해 추상화를 언급했을 뿐.

sum += mark;

실제로 컴퓨터는 위 코드를 실행할 때, 메모리로부터 값을 읽어 오고 그 값을 변경하고 다시 메모리에 값을 할당하는 등의 작업을 실행한다.  
하지만, 위 코드 어디에도 이런 상세 구현에 대한 내용은 언급되지 않는다. 단지 위 코드는 컴퓨터가 수행하는 일련의 처리 과정을 개념적으로 추상화하고 있다.  
모델링 역시 추상화에 해당한다.  
어 회사의 직원에게 급여를 제공해야 한다고 하면

```java
public class Employee {
    public void pay() {
        ...
    }
}
```

위 코드에서 Employee 클래스는 개별 지원들을 추상화한 결과물이 된다.  
이 외에도 다양한 영역에서 다양한 수준의 추상화가 사용되기 때문에, 단순히 구현 클래스로부터 추상 타입을 이끌어 내는 것만이 추상화라고 오해하면 안된다.

### 2.1 추상 타입과 실제 구현의 연결

추상 타입과 실제 구현 클래스는 상속을 통해서 연결한다.  
즉, 구현 클래스가 추상 타입을 상속받는 방법으로 둘을 연결하는 것이다.

```java
// createLogCollector()는 알맞은 구현 클래스의 객체를 생성
LogCollector colletor = createLogCollector();
collector.collect();
```

createLogCollector\(\)가 SocketLogReader 클래스의 객체를 생성하면  
collector.collect\(\)는 SocketLogReader 타입의 collect\(\) 메서드를 실행하게 된다.

각 하위 타입들은 모두 상위 타입은 LogCollector 인터페이스에 정의된 기능을 실제로 구현하는데, 이들 클래스들은 실제 구현을 제공한다는 의미에서 `콘트리트 클래스(concrete class)` 라고 부른다.

### 2.2 추상 타입을 이용한 구현 교체의 유연함

콘크리트 클래스를 직접 사용해도 문제가 없는데, 왜 추상 타입을 사용하는 것일까???

```java
// 콘크리트 클래스를 직접 사용하면 문제가 되나?
SocketLogREader reader = new SocketLogReader();
reader.collect();
```

네. 전혀 문제가 없다. **처음에는**

