# Chapter.3 다형성과 추상 타입

객체 지향이 주는 장점은 구현 변경의 유연함이다.\
유연함을 얻을 수 있도록 해주는 또 다른 방법은 추상화에 있는데\
추상화를 가능하게 해주는 다형성에 대한 내용부터 살펴보자.

## 1. 다형성과 상속

다형성은 한 객체가 여러 가지 모습을 갖는다는 것을 의미한다.\
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
        // 가속
    }
}
```

TurboPlane 클래스는 Plane 클래스를 상속받고 있고, Turbo 인터페이스도 상속받고 있다.\
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

다형성을 구현하기 위해 타입을 상속받는 방법을 사용하는데,\
타입 상속은 크게 인터페이스 상속과 구현 상속으로 구분해 볼 수 있다.

인터페이스 상속은 순전히 타입 정의만을 상속받는 것이다.

![](<../../.gitbook/assets/image (53).png>)

구현 상속은 클래스 상속을 통해서 이루어진다.\
구현 상속은 보통 상위 클래스에 정의된 기능을 재사용하기 위한 목적으로 사용된다.

TurboPlane 클래스의 객체는 Plane 타입도 되기 때문에,\
클래스 상속은 구현을 재사용하면서 다형성도 함께 제공해 준다.

TurboPlane 클래스는 Plane에 정의된 fly() 메서드를 새롭게 구현하고 싶을 경우 자신만의 fly() 메서드를 새롭게 구현할 수 있다.

```java
public class TurboPlane extends Plane {
    public void fly() {
        // TurboPlane의 구현
    }
}
```

```java
Plane p = new TurboPlane();
p.fly();
```

p의 타입이 Plane 클래스이기 때문에 Plnae 클래스의 fly() 메서드가 호출될 것이라고 생각할 수도 있지만, p가 가르키는 객체의 실제 타입이 TurboPlane 클래스이기 때문에 p.fly() 는 TurboPlane 클래스의 fly() 메서드가 호출되는 것이다.

## 2. 추상 타입과 유연함

추상화(abstraction)는 데이터나 프로세스 등을 의미가 비슷한 개념이나 표현으로 정의하는 과정이다.\
예를 통해 감을 잡아보자. 어떤 프로그램을 만드는 데 다음과 같은 세 개의 기능이 있다고 해보자.

* FTP에서 파일을 다운로드
* 소켓에서 데이터 읽기
* DB 테이블의 데이터를 조회

그런데 내용을 더 분석해 보니 위 세 가지 기능은 모두 로그를 수집하기 위한 기능이었다.\
로그를 수집하기 위해 FTP를 이용해 가져오거나, TCP 서버를 이용해 로그 데이터를 읽어 오거나, DB 테이블에 보관된 로그 데이터를 조회하는 것.\
이 세 기능을 추상화하면 '로그 수집' 이라는 개념으로 정의할 수 있는 것이다.

![](<../../.gitbook/assets/image (71).png>)

추상화된 타입은 오퍼레이션의 시그니처만 정의할 뿐 실제 구현을 제공하지는 못한다.

추상화를 한다고 해서 반드시 추상 타입을 만들어야 하는 것은 아니다.\
추상 타입을 이끌어 내는 과정에서 추상화가 사용된다는 것은 설명하기 위해 추상화를 언급했을 뿐.

sum += mark;

실제로 컴퓨터는 위 코드를 실행할 때, 메모리로부터 값을 읽어 오고 그 값을 변경하고 다시 메모리에 값을 할당하는 등의 작업을 실행한다.\
하지만, 위 코드 어디에도 이런 상세 구현에 대한 내용은 언급되지 않는다. 단지 위 코드는 컴퓨터가 수행하는 일련의 처리 과정을 개념적으로 추상화하고 있다.\
모델링 역시 추상화에 해당한다.\
어 회사의 직원에게 급여를 제공해야 한다고 하면

```java
public class Employee {
    public void pay() {
        ...
    }
}
```

위 코드에서 Employee 클래스는 개별 지원들을 추상화한 결과물이 된다.\
이 외에도 다양한 영역에서 다양한 수준의 추상화가 사용되기 때문에, 단순히 구현 클래스로부터 추상 타입을 이끌어 내는 것만이 추상화라고 오해하면 안된다.

### 2.1 추상 타입과 실제 구현의 연결

추상 타입과 실제 구현 클래스는 상속을 통해서 연결한다.\
즉, 구현 클래스가 추상 타입을 상속받는 방법으로 둘을 연결하는 것이다.

```java
// createLogCollector()는 알맞은 구현 클래스의 객체를 생성
LogCollector colletor = createLogCollector();
collector.collect();
```

createLogCollector()가 SocketLogReader 클래스의 객체를 생성하면\
collector.collect()는 SocketLogReader 타입의 collect() 메서드를 실행하게 된다.

각 하위 타입들은 모두 상위 타입은 LogCollector 인터페이스에 정의된 기능을 실제로 구현하는데, 이들 클래스들은 실제 구현을 제공한다는 의미에서 `콘트리트 클래스(concrete class)` 라고 부른다.

### 2.2 추상 타입을 이용한 구현 교체의 유연함

콘크리트 클래스를 직접 사용해도 문제가 없는데, 왜 추상 타입을 사용하는 것일까???

```java
// 콘크리트 클래스를 직접 사용하면 문제가 되나?
SocketLogReader reader = new SocketLogReader();
reader.collect();
```

네. 전혀 문제가 없다. **처음에는**

![](<../../.gitbook/assets/image (59).png>)

```java
public class FlowController {

    public void process() {
        FileDataReader reader = new FileDataReader();
        byte[] data = reader.read();

        Encryptor encryptor = new Encryptor();
        byte[] encryptedData = encryptor.encrypt(data);

        FileDataWriter writer = new FileDataWriter();
        writer.write(encryptedData);
    }
}
```

위와 같은 암호화 프로그램이 작성되어 있다.\
그런데 파일 뿐만 아니라 소켓을 통해 데이터를 읽어와 암호화 해달라는 요청이 왔다.\
그렇다면 소켓에서 데이터를 읽어 오는 SocketDataReader 를 만들어 사용할 수 있을것이다.

```java
public class FlowController {
    private boolean useFile;
    
    public FlowController(boolean useFile) {
        this.useFile = useFile;
    }

    public void process() {
        byte[] data = null;
        if (useFile) {
            FileDataReader fileReader= new FileDataReader();
            data = fileReader.read();
        } else {
            SocketDataReader socketReader = new SocketDataReader();
            data = socketReader.read();
        }

        Encryptor encryptor = new Encryptor();
        byte[] encryptedData = encryptor.encrypt(data);

        FileDataWriter writer = new FileDataWriter();
        writer.write(encryptedData);
    }
}
```

FlowController 자체는 파일이건 소켓이건 상관없이 데이터를 읽어 오고 이를 암호화해서 특정파일에 기록하는 책임을 진다. 그런데 본연의 책임(흐름 제어)과 상관없는 읽기 구현의 변경 때문에 함께 바뀌는 것이다.

기존 요구사항과, 추가 요구사항을 다시 살펴보도록하자.

* 기존 요구 사항 : 파일에서 데이터를 읽어와...
* 추가 요구 사항 : 소켓에서 데이터를 읽어와...

```java
//FileDataReader와 SocketDataReader를 추상화 한 ByteSource 인터페이
public interface ByteSource {
    public byte[] read();
}
```

추상 타입을 만들었으므로, 이제 FileDataReader와 SocketDataReader는 모두 ByteSource 타입을 상속받도록 바꿀 수 있다.

```java
public class FileDataReader implements ByteSource {
    public byte[] read() {
        ...
    }
}

public class SocketDataReader implements ByteSource {
    public byte[] read() {
        ...
    }
}
```

FileDataReader 클래스와 SocketDataReader 클래스가 ByteSorce 타입을 상속 받았으므로 이제 두클래스는 다형성을 통해 ByteSource 타입으로도 동작하 다음과 같이 수정할수 있겠다.

```java
ByteSource Sorce = null;
if (useFile) {
    source = new FileDataREader();
} else {
    source = new SocketDataReader();
}

byte[] data = source.read();
```

코드가 약간 단순해 지긴 했지만 여전히 if-else 블록이 남아있다.\
그리고 ByteSource 타입의 객체를 생성하는 부분이 동일하게 남아있다.\
이 부분을 처리하지 않으면 FlowController 는 ByteSource의 콘크리트 클래스가 변경될때마다 변경된다.

ByteSource의 종류가 변경되더라도 FlowController가 바뀌지 않도록 하는 방법에는 두 가지가 존재한다.

* ByteSource 타입의 객체를 생성하는 기능을 별도 객체로 분리한 뒤, 그 객체를 사용해서 ByteSource 생성
* 생성자를 이용해서 사용할 ByteSource를 전달받기 ( 6장에서 설명 )

ByteSource 타입의 객체를 생성해 주는 책임을 갖는 ByteSourceFactory 클래스를 구현

```java
public class ByteSourceFactory {

    public ByteSource create() { // 객체 생성 기능을 위한 오퍼레이션 정의
        if (useFile())
            return new FileDataReader();
        else
            return new SocketDataReader();
    }

    private boolean useFile() {
        String useFileVal = System.getProperty("useFile");
        return useFileVal != null && Boolean.valueOf(useFileVal);
    }

    // 싱글톤 패턴 적용
    private static ByteSourceFactory instance = new ByteSourceFactory();
    public static ByteSourceFactory getInstance() {
        return instance;
    }

    private ByteSourceFactory() { }
}
```

ByteSourceFactory 클래스의 create() 메서드는 ByteSource 타입의 객체를 생성하는 기능을 제공한다.\
ByteSourceFactory 클래스는 ByteSource 타입의 객체를 생성하는 과정을 추상화 했다고 볼 수 있다.

```java
public class FlowController {

    public void process() {
        ByteSource source = ByteSourceFactory.getInstance().create();
        byte[] data = source.read();

        Encryptor encryptor = new Encryptor();
        byte[] encryptedData = encryptor.encrypt(data);

        FileDataWriter writer = new FileDataWriter();
        writer.write(encryptedData);
    }
}
```

이제 HTTP를 이용해 암호화 할 데이터를 읽어 와야 하는 새로운 요구가 발생했다고 가정해보자.\
새로운 ByteSource 구현 클래스가 추가되겠지만, FlowController 클래스의 코드는 영향을 받지 않는다.\
변경되는 클래스는 ByteSourceFactory 이다.\
이로써 다음 두 가지의 유연함을 얻을 수 있게 되었다.

* ByteSource 의 종류가 변경되면, ByteSourceFactory 만 변경될뿐 FlowController 는 변경되지 않는다.
* FlowController의 제어 흐름을 변경할 때, ByteSource 객체를 생성하는 부분은 영향을 주지 않으면서 FlowController만 변경하면 된다.

객체는 책임을 작게 가질수록 변경에 대한 유연함을 가질 수 있다고 했는데, 최초 FlowController 코드는 데이터를 읽어 오는 객체를 생성하는 책임과 흐름을 제어하는 책임을 동시에 갖고 있었다.

그 책임을 분리할 수 있었던 것은 다음과 같은 두 번의 추상화를 진행한 덕분이다.

* 데이터 읽기 : ByteSource 인터페이스 도출
* ByteSource 객체를 생성하기 : ByteSourceFactory 도출

추상화는 공통된 개념을 도출해서 추상 타입을 정의해 주기도 하지만, 많은 책임을 가진 객체로부터 책임을 분리하는 촉매제가 되기도 한다.

{% hint style="info" %}
재사용

FileDataReader나 SocketDataReader는 데이터를 어떻게 읽어 오는지에 대한 상세한 구현을 다루고 있으며, 이들은 파일이나 소켓이라는 특정한 구현을 재사용하는 용도로 사용된다.\
반면에 FlowController는 데이터를 읽고 암호화하고 데이터를 쓰는 ㅎ개심 로직을 제공하고 있다. 여기서 FlowController는 상대적으로 상위 수준의 로직을 제공하며 FileDataReader는 파일에서 읽기라는 하위 수준의 구현을 제공한다.

암호화 알고리즘이 바뀌거나 데이터를 읽고 쓰는 대상이 바뀌더라도 FlowController가 제공하는 상위 수준의 로직은 바뀌지 않고 재사용되므로, 중요성으로 봤을 때 하위 수준의 상세 구현보다는 변하지 않는 상위 수준의 로직을 재사용할 수 있도록 설계하는 것이 더 중요하다.
{% endhint %}

### 2.3 변화되는 부분을 추상화하기

추상화를 잘 하려면 다양한 상황에서 코드를 작성하고 이 과정에서 유연한 설계를 만들어 보는 경험을 해봐야 한다. 하지만, 변화될 부분을 미리 예측해서 추상화하는 것은 쉽지 않다.

경험하지 않은 분야라 하더라도 추상화할 수 있는 방법이 있는데, 그것은 변화되는 부분을 추상화하는 것이다. 요구 사항이 바뀔 때 변화되는 부분은 이후에도 변경될 소지가 많다. 이런 부분을 추상화 한다면 향후 유연하게 대처할 수 있는 가능성이 높아진다.

추상화가 되어 있지 않은 코드는 주로 **`동일 구조를 갖는 if-else 블록`**으로 드러난다.\
앞서 봤던 FlowController 그리고 1장에서 봤던 소스를 예로 들수있다.

### 2.4 인터페이스에 대고 프로그래밍하기

객체 지향의 유명한 규칙 중 하나이다.\
인터페이스에 대고 프로그래밍하기 (program to interface)

실제 구현을 제공하는 콘크리트 클래스를 사용해서 프로그래밍하지 말고, 기능을 정의한 인터페이스를 사용해서 프로그래밍하라는 뜻이다.

그런데 인터페이스는 최초 설계에서 바로 도출되기 보다는, 요구사항의 변화와 함께 점진적으로 도출이 된다. 즉, 인터페이스는 새롭게 발견된 추상 개념을 통해서 도출되는 것이다.

추상 타입을 사용하면 기존 코드를 건드리지 않으면서 콘크리트 클래스를 교체할 수 있는 유연함을 얻을 수 있었는데 `인터페이스에 대고 프로그래밍하기` 규칙은 바로 추상화를 통한 유연함을 얻기 위한 규칙이다.

주의할 점은 모든곳에서 인터페이스를 사용할 경우 구조가 복잡해질 수 있기 때문에 변화 가능성이 높은 경우에 한해서 사용해야 한다.

### 2.5 인터페이스는 인터페이스 사용자 입장에서 만들기

FileDataReader 만 필요한 상황에서 유연함을 얻기 위해 FileDataReaderIF 라는 인터페이스를 만들고 사용하도록 변경했다.

그 후에, 소켓을 이용해 데이터를 읽어오는 기능이 필요하다면 어떻게 될까?

![](<../../.gitbook/assets/image (57).png>)

이름 때문에, 이 인터페이스를 상속 받아 구현한 클래스는 모두 파일로부터 데이터를 읽어 온다고 착각하기 쉽다.\
따라서 인터페이스를 사용하는 코드 입장에서 작성해야 한다.

### 2.6 인터페이스와 테스트

두 명의 서로 다른 개발자가 각각 FlowController 클래스와 FileDataReader 클래스를 개발한다고 가정해보자. FlowController 개발가 먼저 작업이 끝나 테스트해보려고 한다.

```java
public class FlowController {

    public void process() {
        FileDataReader reader = new FileDataReader();
        byte[] data = reader.read();
        ...
    }
}
```

```java
public void testProcess() {
    FlowController fc = new FlowController();
    fc.process();
    
    ...
}
```

FileDataReader 클래스가 구현되지 않아 테스트가 불가능하다.\
그렇다면 인터페이스를 사용해서 프로그래밍 되어 있다면?

```java
public class FlowController {
    private ByteSource byteSource;
    
    public FlowController(ByteSource byteSource) {
        this.byteSource = byteSource;
    }
    
    public void process() {
        byte[] data = byteSource.read();
        ...
    }
}
```

```java
public void testProcess() {
    ByteSource fileSource = new FileDataReader();
    FlowController fc = new FlowController(fileSource);
    fc.process();
    
    ...
}
```

물론 아직 FileDataReader 가 구현되지 않아 실행되지 않기에 아래와같이 테스트를 할 수 있다.

```java
public void testProcess() {
    ByteSource mockSource = new MockByteSource();
    FlowController fc = new FlowController(mockSource);
    fc.process();
    
    ...
}

class MockByteSource implements ByteSource {
    public byte[] read() {
        byte[] data = new byte[128];
        // data를 테스트 목적의 데이터로 초기화
        return data;
    }
}
```

fc.process() 코드를 실행하면, MockByteSource의 read() 메서드를 통해 byte 데이터를 읽어 오게 된다.\
즉, FileDataReader 클래스 없이 FlowController 클래스를 테스트할 수 있다.

사용할 대상을 인터페이스로 추상화하면 좀 더 쉽게 Mock 객체를 만들 수 있게 되며, 이는 사용할 코드의 완을 기다릴 필요 없이 내가 만든 코드를 먼저 빠르게 테스트 할 수 있다.
