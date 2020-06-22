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

