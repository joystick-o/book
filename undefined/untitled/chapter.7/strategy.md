# 전략\(Strategy\) 패턴



한 과일 매장은 상황에 따라 다른 가격 할인 정책을 적용하고 있다. 매장은 첫 번째 손님을 위한 할인과 신선도가 떨어진 상품에 대한 할인 정책을 제공한다고 가정하면, 아래와 같이 if-else 블록으로 가격할인 정책을 적용할 것이다.

```java
public class Calculator {

    public int calculate(boolean firstGuest, List<Item> items) {
        int sum = 0;
        for(Item item : items) {
            if(firstGuest) {
                sum += (item.getPrice() * 0.9);     // 첫 손님 10% 할인
            } else if(item.isFresh()) {
                sum += (item.getPrice() * 0.8);     // 덜 신선한 상품은 20% 할인
            } else {
                sum += item.getPrice();
            }
        }
        return sum;
    }
}
```

위의 코드는 비교적 그럴듯 해보이지만, 다음과 같은 문제가 있다.

* 서로 다른 계산 정책이 한 코드에 섞여 있어, 정책이 추가될수록 코드 분석을 어렵게 만든다.
* 가격 정책이 추가될 때마다 calculate 메서드를 수정하는 것이 점점 어려워진다. 예를 들면, 마지막 손님 50% 할인같은 새로운 정책이 추가된다면, calculate 메서드에는 if 블록이 하나 더 추가되어야 한다.

위와 같은 문제를 해결하기 위한 방법 중 하나는 아래 그림처럼 가격 할인 정책을 별도 객체로 분리하는 것이다.

![](../../../.gitbook/assets/image%20%2838%29.png)

DiscountStrategy 인터페이스는 상품의 할인 정책을 추상화하였고, 각 콘크리트 객체는 상황에 맞는 할인 계산 알고리즘을 제공한다. Caculator 객체는 가격 합산 계산의 책임을 가진다. 가격 할인 알고리즘을 추상화하고 있는 DiscountStrategy를 **전략\(Strategy\)** 이라 부르고 가격 계산 기능의 책임을 갖고 있는 Calculator를 **콘텍스트\(Context\)** 라 부른다. 이렇게 특정 콘텍스트에서 각 알고리즘을 별도로 분리하는 설계 방법이 **전략 패턴** 이다.

전략 패턴에서 콘텍스트는 사용할 전략을 직접 선택하지 않고, Client가 사용할 전략을 콘텍스트에 전달해준다. 즉, DI\(의존 주입\)을 이용해서 콘텍스트에 전략을 전달해 준다.



```java
public class Calculator {

    private DiscountStrategy discountStrategy;

    public Calculator(DiscountStrategy discountStrategy) {
        this.discountStrategy = discountStrategy;
    }

    public int calculate(List<Item> items) {
        int sum = 0;
        for(Item item : items) {
            sum += discountStrategy.getDicountPrice(item);
        }
        return sum;
    }
}
```

```java
public interface DiscountStrategy {
    int getDicountPrice(Item item);
}
```

만약 전체 금액에 대한 할인 정책이 별도로 필요하다

```java
public interface DiscountStrategy {
    int getDicountPrice(Item item);
    int getDicountPrice(int totalPrice);

}
```

첫 번째 손님에 대해 할인을 해주는 FirstGuestDiscountStrategy 구현 클래스를 보자.

```java
public class FirstGuestDiscountStrategy implements DiscountStrategy {
    
    @Override
    public int getDiscountPrice(Item item) {
        return (int) (item.getPrice() * 0.9);
    }
}
```

```java
private DiscountStrategy strategy;

public void onFirstGuestButtonClick() {
    // 첫 손님 할인 버튼 누를 때 생성 
    strategy = new FirstGuestDiscountStrategy();
}

public void onCaluculationButtonClick() {
    // 계산 버튼 누를 때 실행 됨
    Calculator cal = new Caluculator(strategy);
    int price = cal.calculate(items);
    ...
}
```

위 코드를 보면 Calculator를 사용하는 코드에서 FirstGuestDiscountStrategy 클래스의 객체를 생성하는데 이는 클라이언트가 전략의 상세 구현에 의존이 발생한다는 것을 뜻한다.  
문제처럼 보일 수 있으나. 콘트리트 클래스와 클라이언트의 코드가 쌍을 이루기 때문에 유지 보수 문제가 발생할 가능성이 줄어든다.

전략 패턴을 사용할 때의 이점은 콘텍스트 코드의 변경 없이 새로운 전략을 추가 할 수 있따는 점이다.

