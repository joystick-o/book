# Chapter.4 재사용: 상속보단 조립

객체 지향의 주요 특징으로 재사용을 말하면서 그 예로 상속을 드는 경우가 있다.\
상속을 사용하면 상위 클래스에 구현된 기능을 그대로 재사용할 수 있기 때문에, 상속을 사용하면 재사용을 쉽게 할 수 있는 것은 분명하다.\
상속을 사용할 경우 몇 가지 문제점이 있는데, 상속을 통한 재사용 과정에서 발생할 수 있는 문제점을 살펴보고, 해소하는 방법을 알아보자.

## 1. 상속과 재사용

상속을 사용하면 쉽게 다른 클래스의 기능을 재사용하면서 추가 기능을 확장할 수 있다. 하지만, 상속은 변경의 유연함이라는 측면에서 치명적인 단점을 갖고 있다.

### 1.1 상속을 통한 재사용의 단점 : 상위 클래스 변경의 어려움

상속은 상위 클래스의 변경을 어렵게 만든다.\
어떤 클래스를 상속받는다는 것은 그 클래스에 의존한다는 뜻이다. 따라서 의존하는 클래스의 코드가 변경되면 영향을 받을 수 있다는 것이다.

최악의 경우 상위 클래스의 변화가 모든 하위 클래스에 영향을 줄 수 있다.\
이는 클래스 계층에 있는 클래스들을 한 개의 거대한 단일 구조처럼 만들어 주는 결과를 초래한다. 이런 이유 때문에, 클래스 계층도가 커질수록 상위 클래스를 변경하는 것은 점점 어려워 진다.

### 1.2 상속을 통한 재사용의 단점 : 클래스의 불필요한 증가

유사한 기능을 확장하는 과정에서 클래스의 개수가 불필요하게 증가할 수 있다.

다중 상속을 할 수 없는 자바에서는 한 개의 클래스만 상속받고 다른 기능은 별도로 구현해야 한다. 필요한 기능의 조합이 증가할수록(새로운 요구가 추가될수록), 상속을 통한 기능 재사용을 하게 되 클래스의 개수는 함께 증가하게 된다.

![](<../../.gitbook/assets/image (32).png>)

### 1.3 상속을 통한 재사용의 단점 : 상속의 오용

상속을 잘못 사용할 수도 있다.\
컨테이너 수화물 목록을 관리하는 클래스가 필요하다고 할 때 이 클래스는 세 가지 기능을 제공할 수 있을 것이다.

* 수화물을 넣는다.
* 수화물을 뺀다.
* 수화물을 넣을 수 있는지 확인한다.

담당 개발자는 목록 관 기능을 직접 구현하지 않고 ArrayList 클래스가 제공하는 기능을 상속 받아서 사용하기로 결정했다.

```java
public class Container extends ArrayList<Luggage> {
  private int maxSize;
  private int currentSize;

  public Container(int maxSize) {
    this.maxSize = maxSize;
  }

  public void put(Luggage lug) throws NotEnoughSpaceException {
    if(!canContain(lug)) {
      throw new NotEnoughSpaceException();
    }

    super.add(lug);
    currentSize += lug.size();
  }

  public void extract(Luggage lug) {
    super.remove(lug);
    this.currentSize -= lug.size();
  }

  public boolean canContain(Luggage lug) {
    return maxSize >= currentSize + lug.size();
  }

}
```

이 Container 클래스의 사용 방법은 다음과 같다.

```java
Container c = new Container(5);
if (c.canContain(size2Luggage)) {
    c.put(size2Luggage);
}
```

그런데 일부 개발자들이 잘못된 방법으로 이 클래스를 사용하고 있었다.\
IDE 에서 Container 클래스에 정의된 세 개의 메서드 뿐만 아니라 상위 클래스인 ArrayList 클래스에 등록된 메서드의 목록을 함께 보여준것이다.

```java
Luggage size3Lug = new Luggage(3);
Luggage size2Lug = new Luggage(2);
Luggage size1Lug = new Luggage(1);

Container c = new Container(5);
if (c.canContain(size3Lug)) {
  c.put(size3Luggage); // 정상 사용. Container 여분 5에서 2로 줄어듬
}

if (c.canContain(size2Lug)) {
  c.add(size2Luggage); // 비정상 사용. Container 여분 2에서 줄지 않음
}

if (c.canContain(size1Lug)) { // 통과됨! 원래는 통과되면 안됨!
  c.add(size1Luggage);
}
```

Container 클래스의 작성자는 put() 메서드를 사용해 추가하도록 안내 했지만,\
실제 개발자들은 이 사실을 망각하고 자동 완성 기능이 제시한 add() 메서드를 사용하기 시작했다.

그렇다면 이것은 add() 메서드를 사용한 개발자의 잘못일까?

위와 같은 문제가 발생하는 이유는 Container는 ArrayList가 아니기 때문이다.

상속은 IS-A 관계가 성립할 때에만 사용해야 하는데, "컨테이너는 ArrayList 이다.(Container is a ArrayList)"는 IS-A 관계가 아니다.

Container는 수화물을 보관하는 책임을 갖는 반면에, ArrayList는 목록을 관리하는 책임을 갖는다. 즉, 둘은 서로 다른 책임을 갖는 것이다.

같은 종류(IS-A 관계)가 아닌 클래스 간의 구현 재사용을 위해 상속받게 되면, 잘못된 사용으로 인한 문제가 발생하게 된다.

## 2. 조립을 이용한 재사용

객체 조립은 여러 객체를 묶어서 더 복잡한 기능을 제공하는 객체를 만들어 내는 것이다.\
객체 지향 언어에서 객체 조립은 보통 필드에서 다른 객체를 참조하는 방식으로 구현된다.

```java
public class FlowController {
    private Encryptor encryptor = new Encryptor(); // 필드로 조립
    
     public void process() {
      byte[] encryptedData = encryptor .encrypt(data);
     }
}
```

한 객체가 다른 객체를 조립해서 필드로 갖는다는 것은 다른 객체의 기능을 사용한다는 의미를 내포한다. 즉, Encryptor 클래스를 재사용한다는 것이다.

조립을 통한 재사용은 앞서 상속을 통한 재사용에서 발생했던 문제들을 해소해준다.

![](<../../.gitbook/assets/image (17).png>)

기능이 추가될 때마다 Storage 클래스를 상속받은 하위 클래스가 증가했던 방식과 달리 불필요한 클래스 증가를 방지할 수 있다.

또한, 조립을 사용하면 상속을 잘못 사용해서 발생했던 문제도 제거된다.\
Storage 클래스를 저장소 자체가 아닌 압축이나 암호화등을 목적으로 사용할 수 있게 되며, 경우에 따라 Storage 클래스의 내부 상태가 비정상적으로 변경돼서 저장 기능을 제대로 제공하지 못 할 수도 있다.

> **조립 방식의 또 다른 장점은 런타임에 조립 대상 객체를 교체할 수 있다는 것이다.**

예를 들어 다음의 코드를 살펴보자.

```java
public class Storage { ... }
public class CompressedStorage extends Storage { ... }
public class CompressedEncryptedStorage extends CompressedStorage { ... }

// 사용 코드
CompressedEncryptedStorage storage = new CompressedEncryptedStorage();
// ... storage의 압축 알고리즘을 변경하려면...?
```

위 코드에서 사용 코드 부분을 보면, 실제 코드를 실행하는 동안에는 CompressedEncryptedStorage 객체가 사용하는 압축 알고리즘을 변경할 방법이 없다. 알고리즘을 변경하려면 다음의 과정을 거쳐야 한다.

1. 소스 코드에서 CompressedEncryptedStorage 클래스가 다른 클래스를 상속받도록 변경한다.
2. 소스 코드를 컴파일한다.
3. 다시 배포한다.

반면에 조립하는 방법을 사용하면 얼마든지 런타임에 교체가 가능하다.

```java
public class Storage {
  private Compressor compressor = new Compressor();
  public void setCompressor(Compressor compressor) {
    this.compressor = compressor; // 조립 방식을 통해, 런타임에 압축 알고리즘 변경 가능
  }

  public void save(FileData fileData) {
    byte[] compressedByte = compressor.compress(fileData.getInputStream());
    ...
  }
  ...
}
```

Storage 클래스는 setCompressor() 메서드를 통해서 사용할 Compressor 객체를 전달받을 수 있도록 하였는데, 이를 통해 다음처럼 런타임에 사용할 Compressor 객체를 바꿀 수 있게 된다.

```java
Storage storage = new Storage();
storage.save(someFileData); // Compressor 객체로 압축
...
storage.setCompressor(new FastCompressor());
storage.save(anyFileData); // FastCompressor 객체로 압축
```

또한, Compressor 클래스나 Encryptor 클래스는 Storage 클래스에 의존하지 않기 때문에, Storage 클래스를 쉽게 변경할 수 있다. 따라서, 앞서 상속에서 발생했던 상위 클래스 변경이 어려워지는 문제가 발생하지 않는 것이다.

이렇듯 조립이 상속 기반의 재사용에서 발생했었던 여러 문제들을 해소해주기 때문에 아래와 같은 규칙이 만들어졌다.

> 상속보다는 객체 조립을 사용할 것

모든 상황에서 객체 조립을 사용해야 한다는 얘기는 아니며, 상속을 사용하다 보면 변경의 관점에서 유연함이 떨어질 가능성이 높으니 객체 조립을 먼저 고민하라.

상속대신 객체 조립을 사용할 경우, 상대적으로 런타임 구조가 복잡해지고 구현이 어렵다는 단점이 존재한다. 하지만, 장기적인 관점에서 구현/구조의 복잡함보다 변경의 유연함을 확보하는 데서 오는 장점이 더 크기 때문에, 기능을 재사용해야 할 경우 상속보다는 조립하는 방법을 먼저 고려해야 한다.

### 2.1 위임

위임은 내가 할 일을 다른 객체에게 넘긴다는 의미를 담고 있으며, 보통 조립 방식을 이용해서 위임을 구현한다.

예를 들어, 이미지 편집 툴을 만들 경우 마우스 포인터의 위치가 특정 도형이 차지하는 영역에 포함되어 있는지 확인하는 기능이 필요할 것이다. 그런데, 도형과 관련된 Bounds 클래스가 이 기능을 이미 제공하고 있다면, 도형을 표현하는 Figure 클래스의 contains() 메서드는 Bounds 객체에게 포함 여부 확인 여부를 대신 확인해 달라고 위임할 수 있다.

```java
public abstract class Figure {
  private Bounds bounds = new Bounds(); // 위임 대상을 조립 형태로 가짐
  ...
  private void changeSize() {
    // 크기 변경 코드 위치
    bounds.set(x, y, width, height);
  }
  
  public boolean contains(Point point) {
    // bounds 객체에 처리를 위임함
    return bounds.contains(point.getX(), point.getY());
  }
}
```

보통 위임은 조립과 마찬가지로 요청을 위임할 객체를 필드로 연결한다. 하지만, 꼭 필드로 정의해아 하는 것은 아니다. 위임의 의도는 다른 객체에게 내가 할 일을 넘긴다는데 있으므로, 객체를 새로 생성해서 요청을 전달한다 해도 위임이 의미에서 벗어나지 않는다.

```java
public abstract class Figure {
    public boolean contains(Point point) {
        Bounds bounds = new Bounds(x, y, width, height);
        return bounds.contains(point.getX(), point.getY());
    }
}
```

객체 지향은 책임에 따라 객체들이 세분화되는 특징을 갖는다.\
따라서 객체 지향적으로 구현을 하면 자연스럽게 많은 객체들이 만들어지고, 이 과정에서 조립과 위임을 통해 객체를 재사용하게 된다.

{% hint style="info" %}
위임을 사용하면 내가 바로 실행할 수 있는걸, 다른 객체에 한번 더 요청하게 된다. 이 과정에서 메서드 호출이 추가되기 때문에 실행 시간은 다소 증가한다. 연산 속도가 매우 중요한 시스템에서는 많은 위임 코드가 성능에 문제를 일으킬 수 있지만, 대부분의 경우에는 위임으로 인해 발생하는 미세한 성능 저하보다 위임을 통해서 얻을 수 있는 유연함/재사용의 장점이 크다.
{% endhint %}

### 2.2 상속은 언제 사용하나?

상속을 사용할 때에는, 재사용이라는 고나점이 아닌 기능의 확장이라는 관점에서 상속을 적용해야 한다. 또한 추가로 명확한 IS-A 관계가 성립되어야 한다.

![](<../../.gitbook/assets/image (19).png>)

안드로이드 UI 위젯에 관련된 클래스 계층도인데\
화면에 보이는 텍스트나 버튼등은 모두 UI 위젯이므로 예를들어 "버튼은 UI 위젯이다" 명확한 IS-A 관계이다.

이처럼 상속은 명확한 IS-A 관계에서 점진적으로 상위 클래스의 기능을 확장해 나갈 때 사용할 수 있다.\
단, 최초에는 명확한 IS-A 관계로 보여서 상속을 이용해서 기능을 확장했다고 하더라도 이후에 클래스의 개수가 불필요하게 증가하는 문제가 발생하거나 상위 클래스의 변경이 어려워지는 등 상위 클래스를 상속받을 때의 단점이 발생한다면, 조립으로 전환하는것을 고려해야 한다.
