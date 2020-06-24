# Chapter.5 설계 원칙: SOLID

## 1. 단일 책임 원칙\(Single Responsibility Principle\)

* 클래스는 단 한개의 책임을 가져야 한다.

클래스가 여러 책임을 갖게 되면 그 클래스는 각 책임마다 변경되는 이유가 발생하기 때문에, 클래스가 한 개의 이유로만 변경되려면 클래스는 한 개의 책임만을 가져야 한다.

### 1.1 단일 책임 원칙 위반이 불러오는 문제

```java
public class DataViewer {

  public void display() {
    String data = loadHtml();
    updateGui(data);
  }

  public String loadHtml() {
    HttpClient client = new HttpClient();
    client.connect(url);
    return client.getResponse();
  }

  private void updateGui(String data) {
    GuiData guiModel = parseDataToGuiData(data);
    tableUI.changeData(guiModel);
  }

  private GuiData parseDataToGuiData(String data) {
    ... // 파싱 처리 코드
  }
  ... // 기타 필드 등 다른 코드
}
```

display\(\) 메서드는 loadHtml\(\)에서 읽어 온 HTML 응답 문자열을 updateGui\(\)에 보낸다. updateGui\(\) 메서드는 parseDataToGuiData\(\) 메서드를 이용해서 HTML 응답 메시지를 GUI에 보여주기 위한 GuiData 객체로 변환한 뒤에 실제 tableUI를 이용해서 데이터를 보여주고 있다.

DataViewer를 잘 사용하고 있는 도중에 데이터를 제공하는 서버가 HTTP 프로토콜에서 소켓 기반의 프로토콜로 변경되고 응답 데이터는 byte 배열을 제공한다.

![](../../.gitbook/assets/image%20%2816%29.png)

위의 그림과 같이, 연쇄적인 코드 수정은 두 개의 책임  
데이터 읽는 책임  
화면에 보여주는 책임  
이 한 객체에 아주 밀접하게 결합되어 있어서 발생한 증상이다.

데이터 읽기와 데이터를 화면에 보여주는 책임을 두 개의 객체로 분리하고 둘 간에 주고받을 데이터를 저수준의 String이 아닌 알맞게 추상화된 타입을 사용하 코드가 변경되는  상황을 막을 수 있다.

![](../../.gitbook/assets/image%20%2815%29.png)

단일 책임 원칙을 어길 때 발생하는 또 다른 문제점은 재사용을 어렵게 한다는 것이다.

HttpClient 패키지와 GuiComp 패키지가 각각 별도의 jar 파일로 제공된다고 가정한다.

이 상태에서 데이터를 읽어 오는 기능이 필요한 DataRequiredClient 클래스를 만들어야 한다면, 구현하기 위해 필요한 것은 DataViewer와 HttpClient jar 파일이다.  
하지만, 실제로는 DataViewer가 GuiComp를 필요로 하므로 GuiComp jar 파일까지 필요하다.  
사용하지 않는 기능이 의존하는 jar 파일까지 필요한 것이다.

![](../../.gitbook/assets/image%20%2812%29.png)

단일 책임 원칙에 따라 책임이 분리되었다면 DataRequiredClient 클래스를 구현할 때에는 데이터를 읽어 오는데 필요한 dataloader 패키지와 HttpClient 패키지만 필요하며, 데이터를 읽어 오는 것과 상관없는 GuiComp 패키지나 datadisplay 패키지는 포함시킬 필요가 없어진다.

![](../../.gitbook/assets/image%20%2813%29.png)

### 1.2 책임이란 변화에 대한 것

> 책임의 단위는 변화되는 부분과 관련되어 있다.

데이터를 읽어 오는 책임의 기능이 변경될 때 데이터를 보여주는 책임은 변경되지 않는다. 반대로 데이터를 테이블에서 그래프로 바꿔서 보여주더라도 데이터를 읽어 오는 기능은 변경되지 않는다.  
따라서 서로 다른 이유로 바뀌는 책임들이 한 클래스에 함께 포함되어 있다면 이 클래스는 단일 책임 원칙을 어기고 있다고 볼 수 있다.

그럼 어떻게 하면 단일 책임 원칙을 지킬수 있을까?  
바로 메서드를 실행하는 것이 누구인지 확인해 보는 것이다.

![](../../.gitbook/assets/image%20%2814%29.png)

GUIApplication 은 display\(\)를 사용하 DataProcessor는 loadData\(\)를 사용한다 해 보자.  
GUIApplication 이 화면에 표시되는 방식을 변경해야 할 경우, 변경되는 메서즈는 display\(\) 메서드다.  
반면에 DataProcessor 가 읽어 오는 데이터를 String이 아닌 다른 타입으로 변경해야 할 경우 String이 아닌 DataProcessor가 요구하는 타입으로 변경될 가능성이 높다.  
이렇게 클래스의 사용자들이 서로 다른 메서드들을 사용한다면 그들 메서드는 다른 택임에 속할 가능성이 높고, 책임 분리 후보가 될 수 있다.

## 2. 개방 폐쇄 원칙 \(Open - Closed Principle\)

> 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.

* 기능을 변경하거나 확장할 수 있으면서
* 그 기능을 사용하는 코드는 수정하지 않는다.

![](../../.gitbook/assets/image%20%2817%29.png)

메모리에서 byte를 읽어 오는 기능을 추가해야 할 경우, ByteSource 인터페이스를 상속받은 MemoryByteSource 클래스를 구현함으로써 기능 추가가 가능하다. 그리고 새로운 기능이 추가되었지만, 이 새로운 기능을 사용할 FlowController 클래스의 코드는 변경되지 않는다.  
즉, 기능을 확장 하면서도 기능을 사용하는 기존 코드는 변경되지 않는 것이다.

개방 폐쇄 원칙을 구현하는 또 다른 방법은 상속을 이용하는 것이다.  
상속은 상위 클래스의 기능을 그대로 사용하면서 하위 클래스에서 일부 구현을 오버라이딩 할 수 있는 방법을 제공한다.

예를 들어, 클라이언트의 요청이 왔을 때 데이터를 HTTP 응답 프로토콜에 맞춰 데이터를 전송해 주는 객체가 있다고 하자.

```java
public class ResponseSender {
  private Data data;
  public ResponseSender(Data data) {
    this.data = data;
  }

  public Data getData() {
    return data;
  }

  public void send() {
    sendHeader();
    sendBody();
  }

  protected void sendHeader() {
    // 헤더 데이터 전송
  }

  protected void sendBody() {
    // 텍스트로 데이터 전송
  }
}
```

ResponseSender 클래스의 send\(\) 메서드는 헤더와 몸체 내용을 전송하기 위해 sendHeader\(\) 메서드와 sendBody\(\) 메서드를 차례대로 호출하며, 이 두 메서드는 알맞게 HTTP 응답 데이터를 생성한다. sendHeader\(\) 메서드와 sendBody\(\) 메서드는 protected 공개 범위를 갖고 있기 때문에, 하위 클래스에서 이 두메서드를 오버라이딩 할 수 있다.

만약 압축해서 데이터를 전송하는 기능을 추가하고 싶다면, ResponseSender 클래스를 상속받은 클래스에서 sendHeader\(\) 메서드와 sendBody\(\) 메서드를 오버라이딩하면 된다.

```java
public class ZippedResponseSender extends ResponseSender {
  public ZippedResponseSender(Data data) {
    super(data);
  }

  @Override
  protected void sendBody() {
    // 데이터 압축 처리
  }
}
```

ZippedResponseSender 클래스는 기존 기능에 압축 기능을 추가해 주는데, 이 기능을 추가하기 위해 ResponseSender 클래스 코드는 바뀌지 않았다.  
즉, ResponseSender 클래스는 확장에는 열려 있으면서 변경에는 닫혀 있는 것이다.

ResponseSender 클래스 예제는 템플릿 패턴을 사용한 것이다. 템플릿 메서드 패턴은 상위 클래스에서 실행할 기본 코드를 만들고 하위 클래스에서 필요에 따라 확장해 나가는 패턴이다.

### 2.1 개방 폐쇄 원칙이 깨질 때의 주요 증상

추상화와 다형성을 이용해서 개방 폐쇄 원칙을 구현하기 때문에, 추상화와 다형성이 제대로 지켜지지 않은 코드는 개방 폐쇄 원칙을 어기게 된다. 개방 폐쇄 원칙을 어기는 코드의 전형적인 특징은 다음과 같다.

> **다운 캐스팅을 한다.**

예를 들어, 슈팅 게임을 개발하는 경우 플레이어, 적, 미사일 등을 그리기 위해 아래와 같은 상속 관계를 사용할 수 있다.

![](../../.gitbook/assets/image%20%2819%29.png)

그런데, 화면에 이들 캐릭터를 표시해 주는 코드가 다음과 같다면 어떨까?

```kotlin
public void drawCharacter(Character character) {
  if (character instanceof Missile) {  // 타입 확인
    Missile missile = (Missile) character; // 타입 다운 캐스팅
    missile.drawSpecific();
  } else {
    character.draw();
  }
}
```

위의 코드는 character 파라미터의 타입이 Missile인 경우 별도 처리를 하고 있다. 만약 위와 같이 특정 타입인 경우에 별도 처리를 하도록 drawCharacter\(\) 메서드를 구현한다면 drawCharacter\(\) 메서드는 Character 클래스가 확장될 때 함께 수정되어야 한다.  
즉, 변경에 닫혀 있지 않은 것이다.

instanceof와 같은 타입 확인 연산자가 사용된다면 해당 코드는 개방 폐쇄 원칙을 지키지 않을 가능성이 높다. 이런 경우에는 타입 캐스팅 후 실행하는 메서드가 변화 대상인지 확인해 봐야 한다.

개방 폐쇄 원칙을 깨뜨리는 코드의 또 다른 특징은 다음과 같다.

> **비슷한 if-else 블록이 존재한다.**

Enemy 캐릭터의 움직이는 경로를 몇 가지 패턴으로 정한다고 하자. 이 때, 정해진 패턴에 따라 경로를 이동하는 코드는 아래와 같이 작성할 수 있다.

```java
public class Enemy extends Character {
  private int pathPattern;
  public Enemy(int pathPattern) {
    this.pathPattern = pathPattern;
  }

  public void draw() {
    if (pathPattern == 1) {
      x += 4;
    } else if(pathPattern == 2) {
      y += 10;
    } else if(pathPattern == 4) {
      x += 4;
      y += 10;
    }
    ...;  // 그려 주는 코드
  }
}
```

Enemy 클래스에 새로운 경로 패턴을 추가해야 할 경우 Enemy 클래스의 draw\(\) 메서드에는 새로운 if 블록이 추가된다.  
즉, 경로를 추가하는데 Enemy 클래스가 닫혀 있지 않은 것이다.  
이를 개방 폐쇄 원칙을 따르도록 변경하면, 아래 그림과 같이 경로 패턴을 추상화하고 Enemy에서 추상화 타입을 사용하는 구조로 바뀐다.

![](../../.gitbook/assets/image%20%2818%29.png)

Enemy 코드는 PathPattern을 사용하도록 변경된다.

```java
public class Enemy extends Character {
  private PathPattern pathPattern;
  public Enemy(PathPattern pathPattern) {
    this.pathPattern = pathPattern;
  }

  public void draw() {
    int x = pathPattern.nextX();
    int y = pathPattern.nextY();
    ...; // 그려 주는 코드
  }
}
```

이제 새로운 이동 패턴이 생기더라도 Enemy 클래스의 draw\(\) 메서드는 변경되지 않으며, 새로운 타입의 PathPattern 구현 클래스를 추가해 주기만 하면 된다.

### 2.2 개방 폐쇄 원칙은 유연함에 대한 

개방 폐쇄 원칙은 변경의 유연함과 관련된 원칙이다. 만약 기존 기능을 확장하기 위해 기존 코드를 수정해 주어야 한다면, 새로운 기능을 추가하는 것이 점점 힘들어진다. 즉, 확장에는 당히고 변경에는 열리는 반대 상황이 발생하는 것이다.

개방 폐쇄 원칙은 변화가 예상되는 것을 추상화해서 변경의 유연함을 얻도록 해준다. 이 말은 변화되는 부분을 추상화하지 못하면 혹은 안하면 개방 폐쇄 원칙을 지킬 수 없게 되어 시간이 흐를수록 기능 변경이나 확장을 어렵게 만든다는 것을 뜻한다.  
따라서 코드에 대한 변화 요구가 발생하면, 변화와 관련된 구현을 추상화해서 개방 폐쇄 원칙에 맞게 수정할 수 있는지 확인하는 습관을 갖도록 하자.

## 3. 리스코프 치환 원칙\(Liskov Substitution Principle\)

리스코프 치환 원칙은 개방 폐쇄 원칙을 받쳐 주는 다형성에 관한 원칙을 제공한다.

> **상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.**

상위 타입 SuperClass와 하위 타입 SubClass가 있다고 하자. 특정 메서드는 상위 타입인 SuperClass를 이용할 것이다.

```java
public void someMethod(SuperClass sc) {
  sc.otherSomeMethod();
}
```

someMethod\(\)는 상위 타입인 SuperClass 타입의 객체를 사용하고 있는데, 이 메서드에 다음과 같이 하위 타입의 객체를 전달해도 someMethod\(\)가 정상적으로 동작해야 한다는 것이 리스코프 치환 원칙이다.

```java
someMethod( new SubClass() );
```

리스코프 치환 원칙이 제대로 지켜지지 않으면 다형성에 기반한 개방 폐쇄 원칙 역시 지켜지지 않기 때문에, 리스코프 치환 원칙을 지키는 것은 매우 중요하다.

### **3.1 리스코프 치환 원칙을 지키지 않을 때의 문제**

리스코프 치환 원칙을 설명할 때 자주 사용되는 대표적인 예가 직사각형-정사각형 문제이다. 직사각형을 표현하기 위한 Rectangle 클래스는 가로와 세로 두 개의 값을 구하거나 수정하는 기능을 제공할 것이다.

```java
public class Rectangle {
  private int width;
  private int height;

  public void setWidth(int width) {
    this.width = width;
  }

  public void setHeight(int height) {
    this.height = height;
  }

  public void getWidth() {
    return width;
  }

  public void getHeight() {
    return height;
  }
}
```

정사각형을 직사각형의 특수한 경우로 보고, 정사각형을 표현하기 위한 Square 클래스가 Rectangle 클래스를 상속받도록 구현을 했다고 하자. 정사각형은 가로와 세로가 모두 동일한 값을 가져야 하므로, Square 클래스는 Rectangle 클래스의 setWidth\(\) 메서드와 setHeight\(\) 메서드를 재정의해서 가로와 세로 값이 일치하도록 구현하였다.

```java
public class Square extends Rectangle {
  @Override
  public void setWidth(int width) {
    super.setWidth(widht);
    super.setHeight(widht);
  }

  @Override
  public void setHeight(int height) {
    super.setWidth(height);
    super.setHeight(height);
  }
}
```

이제 높이와 폭을 비교해서 높이를 더 길게 만들어 주는 기능을 제공한다고 해보자.

```java
public void increaseHeight(Rectangle rec) {
  if (rec.getHeight() <= rec.getWidth()) {
    rec.setHeight(rec.getWidth() + 10);
  }
}
```

increaseHeight\(\) 메서드를 사용하는 코드는 increaseHeight\(\) 메서드 실행 후에 width보다 height 값이 더 크다고 가정할 것이다. 그런데, increaseHeight\(\) 메서드의 rec 파라미터로 Square 객체가 전달되면 이 가정은 깨진다. Square의 setHeight\(\) 메서드는 높이와 폭을 모두 같은 값으로 만들기 때문에 increaseHeight\(\) 메서드를 실행하더라도 높이가 폭보다 길어지지 않게 된다.

이러한 문제를 해소하기 위해 increaseHeight\(\) 메서드에서 rec 파라미터의 실제 타입이 Square인 경우에는 이 기능을 실행하지 않도록 instanceof 연산자를 사용할 수 있을 것이다. 하지만, 앞서 봤듯이 __instanceof 연산자를 사용한다는 것 자체가 리스코프 치환 원칙 위반이 되고, 이는 increaseHeight\(\) 메서드가 Rectangle 확장에 열려 있지 않다는 것을 뜻한다.

```java
public void increaseHeight(Rectangle rec) {
  if (rec instranceof Square) {
    throw new CantSupportSquareException();
  }

  if (rec.getHeight() <= rec.getWidth()) {
    rec.setHeight(rec.getWidth() + 10);
  }
}
```

개념상 정사각형은 높이와 폭이 같은 직사각형이므로, Rectangle 클래스를 상속받아 Square 클래스를 구현하는 것이 합리적으로 보일 수 있으나, 실제 프로그램에서는 이 둘을 상속 관계로 묶을 수 없다. increaseHeight\(\) 와 같은 기능이 필요하다면, 실제 구현에서는 Square 클래스는 Rectangle 클래스를 상속받아 구현하기 보다는 별개의 타입으로 구현해 주어야 한다.

리스코프 치환 원칙을 어기는 또 다른 흔한 예는 상위 타입에서 지정한 리턴 값의 범위에 해당되지 않는 값을 하위 타입에서 리턴하는 것이다.

예를 들어, 입력 스트림으로부터 데이터를 읽어와 출력 스트림에 복사해 주는 복사 기능 다음과 같이 구현될 것이다.

```java
public class CopyUtil {
  public static void copy(InputStream is, OutputStream out) {
    byte[] data = new byte[512];
    int len = -1;

    // InputStream.read() 메서드는 스트림의 끝에 도달하면 -1을 리턴
    while ((len = is.read(data)) != -1) {
      out.write(data, 0, len);
    }
  }
}
```

InputStream의 read\(\) 메서드는 스트림의 끝에 도달해서 더 이상 데이터를 읽어올 수 없는 경우 -1을 리턴한다고 정의되어 있고, CopyUtil.copy\(\) 메서드는 이 규칙에 따라 is.read\(\)의 리턴 값이 -1이 아닐 때까지 반복해서 데이터를 읽어와 out에 쓴다.

그러나 만약 InputStream을 상속한 하위 타입에서 read\(\) 메서드를 아래와 같이 구현한다면 어떻게 될까?

```java
public class SatanInputStream implements InputStream {
  public int read(byte[] data) {
    ...
    return 0; // 데이터가 없을 때 0을 리턴하도록 구현
  }
}
```

SatanInputStream의 read\(\) 메서드는 데이터가 없을 때 0을 리턴하도록 구현하였다. SatanInputStream 객체의 사용자는 SatanInputStream 객체로 부터 데이터를 읽어 와서 파일에 저장하기 위해 다음과 같이 CopyUtil.copy\(\) 메서드를 사용할 것이다.

```java
InputStream is = new SatanInputStream(someData);
OutputStream out = new FileOutputStream(filePath);
CopyUtil.copy(is, out);
```

CopyUtil.copy\(\) 메서드는 InputStream의 read\(\) 메서드가 -1을 리턴할 때 반복문을 멈추도록 구현되어 있다. 그런데, SatanInputStream의 read\(\) 메서드는 데이터가 없더라도 -1을 리턴하지 않아, CopyUtil.copy\(\) 메서드는 무한루프를 돌면서 실행이 끝나지 않게 된다.

결과적으로, 위와 같은 문제가 발생하는 이유는 SatanInputStream 타입의 객체가 상위 타입인 InputStream을 올바르게 대체하지 않기 때문이다. 즉, 리스코프 치환 원칙을 지키지 않았기 때문에 발생한 것이다.

### **3.2 리스코프 치환 원칙은 계약과 확장에 대한 것**

리스코프 치환 원칙은 기능의 명세에 대한 내용이다.

앞서 직사각형-정사각형 문제 예에서 Rectangle 클래스의 setHeight\(\) 메서드는 사용자에게 다음과 같은 계약을 제공하고 있다.

* 높이 값을 파라미터로 전달받은 값으로 변경한다.
* 폭 값은 변경되지 않는다.

setHeight\(\) 메서드를 호출하는 코드는 높이 값만 변경될 뿐 폭은 바뀌지 않을 거라고 가정하는데, Square 클래스의 setHeight\(\) 메서드는 높이와 폭을 함께 변경한다. 따라서 setHeight\(\) 메서드를 사용하는 코드는 전혀 상상하지 못했던 결과로 인해 예상과 달리 비정상적으로 동작할 수 있다.

기능 실행의 계약과 관련해서 흔히 발생하는 위반 사례로는 다음과 같은 것들이 있다.

> * 명시된 명세에서 벗어난 값을 리턴한다.
> * 명시된 명세에서 벗어난 익셉션을 발생한다.
> * 명시된 명세에서 벗어난 기능을 수행한다.

하위 타입이 명세에서 벗어난 동작을 하게 되면, 이 명세에 기반해서 구현한 코드는 비정상적으로 동작할 수 있기 때문에, 하위 타입은 상위 타입에서 정의한 명세를 벗어나지 않는 범위에서 구현해야 한다.

**리스코프 치환 원칙은 확장에 대한 것이다. 리스코프 치환 원칙을 어기면 개방 폐쇄 원칙을 어길 가능성이 높아진다.**

간단하게 예를 살펴보자. 상품에 쿠폰을 적용해서 할인되는 액수를 구해 주는 기능을 구현할 경우, 다음 코드처럼 Coupon 클래스에서 Item 클래스의 값을 구한 뒤 할인되는 큼액을 계산할 수 있을 것이다.

```java
public class Coupon {
  public int calculateDiscountAmount(Item item) {
    return item.getPrice() * discountRate;
  }
}
```

그런데 특수 Item은 무조건 할인을 해주지 않는 정책이 추가되어, 이를 위해 Item 클래스를 상속받는 SpecialItem 클래스를 추가했다고 하자.

Coupon 클래스의 calculateDiscountAmount\(\) 메서드는 item 객체의 실제 타입이 SpecialItem인 경우 할인 액수를 0으로 처리해 주어야 하는데, 이를 반영하기 위해 Coupon 클래스를 다음과 같이 수정할 수 있을 것이다.

```java
public class Coupon {
  public int calculateDiscountAmount(Item item) {
    if (item instanceof SpecialItem) { // LSP 위반 발생
      return 0;
    }

    return item.getPrice() * discountRate;
  }
}
```

위 코드는 아주 흔한 리스코프 치환 원칙 위반 사례이다. Item 타입을 사용하는 코드는 SpecialItem 타입이 존재하는지 알 필요 없이 오직 Item 타입만 사용해야 한다.

그런데, 위 코드는 instanceof 연산자를 사용해서 SpecialItem 타입인지의 여부를 확인하고 있다. 즉, 하위 타입인 SpecialItem이 상위 타입인 Item을 완벽하게 대체하지 못하는 상황이 발생하고 있는 것이다.

타입을 확인하는 기능\(자바의 instanceof 연산자\)을 사용하는 것은 전형적인 리스코프 치환 원칙을 위반할 때 발생하는 증상이다. 클라이언트가\(Coupon 클래스\) instanceof 연산자를 사용한다는 것은 상위 타입\(Item 클래스\)만을 사용해서 프로그래밍 할 수 없다는 것을 의미한다.

리스코프 치환 원칙을 어기게 된 이유는 Item에 대한 추상화가 덜 되었기 때문이다.

상품의 가격 할인 가능 여부가 Item 및 하위 타입에서 변화되는 부분이 되며, 변화되는 부분을 Item 클래스에 추가함으로써 리스코프 치환 원칙을 지킬 수 있게 된다.

```java
public class Item {

  // 변화되는 기능을 상위 타입에 추가
  public boolean isDiscountAvailable() {
    return true;
  }
  ...
}

public class SpecialItem extends Item {
  // 하위 타입에서 알맞게 오버라이딩
  @Override
  public boolean isDiscountAvailable() {
    return false;
  }
}
```

위 코드에서 Item 클래스에 가격 할인 능 여부를 판단하는 기능을 추가하고, SpecialItem 클래스는 이 기능을 알맞게 재정의했다.

변화되는 부분을 상위 타입에 추가함으로써, instanceof 연산자를 사용하던 코드를 Item 클래스으로만 구현할 수 있게 되었다.

```java
public class Coupon {
  public int calculateDiscountAmount(Item item) {
    if(! item.isDiscountAvailable()) {  // instanceof 연산자 사용 제거
      return 0;
    }

    return item.getPrice() * discountRate;
  }
}
```

예를 통해서 봤듯이, 리스코프 치환 원칙이 위반되면 개방 폐쇄 원칙도 지킬 수 없게 된다.  
개방 폐쇄 원칙을 지키지 않으면 기능 확장을 위해 더 많은 부분을 수정해야 하므로, 리스코프 치환 원칙을 위반하면 기능 확장에 어려움을 겪게 된다.

