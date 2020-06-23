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
  if(character instanceof Missile) {  // 타입 확인
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
    if(pathPattern == 1) {
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

