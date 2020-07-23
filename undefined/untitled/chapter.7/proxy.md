# 프록시\(Proxy\) 패턴

제품 목록을 구성할 때 관련된 모든 이미지를 로딩하도록 구현할 수 있는데, 이 경우 불필요하게 메모리를 사용하는 문제가 발생할 수 있다.

![](../../../.gitbook/assets/image%20%2846%29.png)

그렇다면 이미지가 실제로 화면에 보여질 때 데이터를 로딩한다면 문제가 해결된다.  
프록시 패턴은 실제 객체를 대신하는 프록시 객체를 사용해서 실제 객체의 생성이나 접근 등을 제어할 수 있도록 해주는 패턴이다.

![](../../../.gitbook/assets/image%20%2838%29.png)

```java
public class ProxyImage Implements Image {
    private String path;
    private RealImage image;
    
    public ProxyImage(String path) {
        this.path = path;
    }
    public void draw() {
        if (image == null) {
            image = new RealImage(path); // 최초 접근 시 객체 생성
        }
        image.draw(); // RealImage 객체에 위임
    }
}
```

ProxyImage 클래스는 draw\(\) 메서드가 호출되기 전까지 RealImage 객체를 생성하지 않고 최초로 draw\(\) 메서드를 실행할 때 RealImage 객체를 생성하고, 그 뒤에 생성된 객체의 draw\(\) 메서드를 호출한다.

```java
public void onScroll(int start, int end) {
    // 스크롤 시, 화면에 표시되는 이미지를 표시
    for (int i=start; i<=end; i++) {
        Image image = images.get(i);
        image.draw();
    }
}
```

ProxyImage 객체의 draw\(\) 메서드가 호출되기 전에는 RealImage 객체가 생성되지 않으므로 메모리에 이미지 데이터를 로딩하지 않는다. 불필요하게 메모리를 낭비하는 상황을 방지할 수 있다.

