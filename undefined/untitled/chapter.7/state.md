# 상태\(State\) 패턴

단일 상품을 판매하는 자판기에 들어갈 소프트웨어를 개발해 달라는 요구가 들어왔다. 이 자판기의 동작 방식은 다음과 같다.

![](../../../.gitbook/assets/image%20%2843%29.png)

조건에 따라 다른 코드를 실행해야 한다는 판단을 하고 작성한 코드를 보자.

```java
public class VendingMachine {
    public static enum State { NOCOIN, SELECTABLE }
    
    private State state = State.NOCOIN;
    
    public void insertCoin(int coin) {
        switch(state) {
        case NOCOIN:
            increaseCoin(coin);
            state = State.SELECTABLE;
            break;
        case SELECTABLE:
            increaseCoin(coin);
        }
    }
    
    public void select(int productId) {
        switch(state) {
        case NOCOIN:
            // 아무것도 하지 않음
            break;
        case SELECTABLE:
            provideProduct(producId);
            decreaseCoin();
            if (hasNoCoin())
                state = State.NOCOIN;
        }
    }
    ...
}
```

구현하는 도중 새로운 요구 사항이 들어왔다.

```java
public class VendingMachine {
    public static enum State { NOCOIN, SELECTABLE, SOLDOUT }
    
    private State state = State.NOCOIN;
    
    public void insertCoin(int coin) {
        switch(state) {
        case NOCOIN:
            increaseCoin(coin);
            state = State.SELECTABLE;
            break;
        case SELECTABLE:
            increaseCoin(coin);
        case SOLDOUT:
            returnCoin();
        }
    }
    
    public void select(int productId) {
        switch(state) {
        case NOCOIN:
            // 아무것도 하지 않음
            break;
        case SELECTABLE:
            provideProduct(producId);
            decreaseCoin();
            if (hasNoCoin())
                state = State.NOCOIN;
        case SOLDOUT:
            // 아무것도 하지 않음
        }
    }
    ...
}
```

VendingMachine 클래스의 코드를 다시 한 번 살펴보면, 조건문은 다음과 같은 의미를 내포하고 있다.

* 상태에 따라 동일한 기능 요청의 처리를 다르게 함.

기능의 상태에 따라 다르게 동작해야 할 때 사용할 수 있는 패턴이 상태 패턴이다.

![](../../../.gitbook/assets/image%20%2842%29.png)

상태 패턴에서 중요한 점은 상태 객체가 기능을 제공한다는 점이다. State 인터페이스는 동전 증가 처리와 제품 선택 처리를 할 수 있는 두 개의 메서드를 정의하고 있다. 이 두 메서드는 모든 상태에 동일하게 적용되는 기능이다.

콘텍스트는 클라이언트로부터 기능 실행 요청을 받으면, 상태 객체에 처리를 위임하는 방식으로 구현한다.

```java
public class VendingMachine {
    private State state;

    public VendingMachine() {
        state = new NoCoinState();
    }

    public void insertCoin(int coin) {
        state.increaseCoin(coin, this); // 상태 객체에 위임
    }

    public String select(int id) {
        return state.select(id, this); // 상태 객체에 위임
    }

    void changeState(State newState) {
        this.state = newState;
    }
    // ...
}
```

```java
public class NoCoinState implements State {

    @Override
    public void increaseCoin(int coin, VendingMachine vm) {
        vm.increaseCoin(coin);
        vm.changeState(new SelectableState());
    }

    @Override
    public String select(int productId, VendingMachine vm) {
        SoundUtil.beep();
    }

}
```

상태 패턴을 적용함으로써 VendingMachine 클래스에 구현되어 있는 상태 별 동작 구현 코드가 각 상태의 구현 클래스로 이동함을 알 수 있다.  


상태 패턴의 장점을 새로운 상태가 추가되더라도 콘텍스트 코드가 받는 영향은 최소화된다.

## 상태 변경은 누가?

앞서 예제에서는 상태에서 콘텍스트의 상태를 변경해 주었다.

상태 객체에서 콘텍스트의 상태를 변경하려면 콘텍스트의 다른 값에 접근해야 할 때도 있다.

```java
public class SelectableState implements State {

    @Override
    public String select(int productId, VendingMachine vm) {
        vm.provideProduct(productId);
        vm.decreaseCoin();
        
        if (vm.hasNoCoin()) // 상태 변경을 위해, vm 객체가 동전이 없는지 확인
            vm.changeState(new NoCoinState());
    }

}
```

콘텍스트에서 상태를 변경할 경우 콘텍스트의 코드가 다소 복잡해질 수 있다.



```java
public class VendingMachine {
    private State state;

    public VendingMachine() {
        state = new NoCoinState();
    }

    public void insertCoin(int coin) {
        state.increaseCoin(coin, this);     
        if(hasCoin()) 
            changeState(new SelectableState()); // 콘텍스트에서 상태 변경
    }

    public void select(int productId) {
        state.select(productId, this);    
        if(state.isSelecttable() && hasNoCoin)
            changeState(new NoCoinState()); // 콘텍스트에서 상태 변경
    }
    
    private void changeState(State newState) {
        this.state = newState;
    }

    private boolean hasCoin() {
        ...
    }

    private boolean hasNoCoin() {
        return ! hasCoin();
    }
    ...
}
```

이제 상태 객체는 자신이 수행해야 하는 작업만 처리하도록 바뀐다.

콘텍스의 상태 변경을 누가 할지는 주어진 상황에 알맞게 정해 주어야 한다.

{% hint style="info" %}
**콘텍스트 객체에서 변경하는 경우**

비교적으로 **상태 개수가 적고 변경 규칙이 거의 바뀌지 않는 경우에 유리**하다. 상태 종류가 지속적으로 변경되거나 상태 변경 규칙이 자주 바뀐다면, 콘텍스트의 상태 변경 처리 코드가 복잡해질 가능성이 높아지고 결국 상태 변경의 유연함이 떨어지게 된다.
{% endhint %}

{% hint style="success" %}
**상태 객체에서 콘텍스트의 상태를 변경할 경우**

**콘텍스트에 영향을 주지 않으면서 상태를 추가하거나 상태 변경 규칙을 바꿀 수 있게 된다.** 하지만, 상태 변경 규칙이 여러 클래스에 분산되어 있기 때문에, 상태 구현 클래스가 많아질수록 상태 변경 규칙을 파악하기 어려워진다는 단점이 있다. 또한, 상태 객체간에 의존도 발생한다.
{% endhint %}

