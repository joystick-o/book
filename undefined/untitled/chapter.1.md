# Chapter.1 들어가기

> 처음에는 빠르게 요구 사항을 반영해 주었는데, 시간이 지날수록 간단한 요구 사항 조차도 제때 개발이 안되고 있어요

> 겉보기엔 간단하지만, 변경해야 할 곳이 너무 많습니다. 게다가 어딜 변경해야 할지도 다 알수 없어서, 변경 이후에 어떤 기능에 문제가 생길지 모르겠어요.

## 1. 지저분해지는 코드

![](../../.gitbook/assets/image%20%281%29.png)

그림과 같은 UI를 갖는 프로그램을 개발한다고 하자.  
메뉴1과 메뉴2를 누르면 화면 영역에 알맞은 내용이 출력된다.  
또한, 모든 화면은 버튼1을 공통적으로 사용하면 누를때마다 화면의 데이터가 변경된다.

```java
public class Application implements OnClickListener {

    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Button button1 = new Button("button1");

    private String currentMenu = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        button1.setOnClickListener(this);
    }

    public void clicked(Component eventSource) {
        if (eventSource.getId().equals("menu1")) {
            changeUIToMenu1();
        } else if (eventSource.getId().equals("menu2")) {
            changeUIToMenu2();
        } else if (eventSource.getId().equals("button1")) {
            if (currentMenu == null)
                return;
            if (currentMenu.equals("menu1"))
                processButton1WhenMenu1();
            else if (currentMenu.equals("menu2"))
                processButton1WhenMenu2();
        }
    }

    private void changeUIToMenu1() {
        currentMenu = "menu1";
        System.out.println("메뉴1 화면으로 전환");
    }
    private void changeUIToMenu2() {
        currentMenu = "menu2";
        System.out.println("메뉴2 화면으로 전환");
    }
    private void processButton1WhenMenu1() {
        System.out.println("메뉴1 화면의 버튼1 처리");
    }
    private void processButton1WhenMenu2() {
        System.out.println("메뉴2 화면의 버튼1 처리");
    }

}
```

위처럼 프로그램이 완성될때쯤 추가 요구 사항이 들어왔다.  
버튼2가 필요하다는 것이다.

```java
public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        button1.setOnClickListener(this);
        button2.setOnClickListener(this);
}

public void clicked(Component eventSource) {
        if (eventSource.getId().equals("menu1")) {
            changeUIToMenu1();
        } else if (eventSource.getId().equals("menu2")) {
            changeUIToMenu2();
        } else if (eventSource.getId().equals("button1")) {
            if (currentMenu == null)
                return;
            if (currentMenu.equals("menu1"))
                processButton1WhenMenu1();
            else if (currentMenu.equals("menu2"))
                processButton1WhenMenu2();
        }
        else if (eventSource.getId().equals("button2")) {
            if (currentMenu == null)
                return;
            if (currentMenu.equals("menu1"))
                processButton2WhenMenu1();
            else if (currentMenu.equals("menu2"))
                processButton2WhenMenu2();
        }
}
```

위와 같이 요구 사항이 계속해서 추가된다면....

## 2. 수정하기 좋은 구조를 가진 코드

이제 같은 상황을 객체 지향 방식으로 풀어보자.  
객체지향의 추상화와 다형성을 이용해서 변화되는 부분을 관리한다.  


1. **메뉴가 선택되면 해당 화면을 보여준다.**
2. **버튼1을 클릭하면 선택된 메뉴 화면에서 알맞은 처리를 한다.**

이 공통 동작을 표현하기 위해 ScreenUI 인터페이스를 정

```java
public interface ScreenUI {
    public void show();
    public void handleButton1Click();
}
```

각 메뉴별로 ScreenUI 인터페이스를 구현한 클래스를 작

```java
public class Menu1ScreenUI implements ScreenUI {
    public void show() {
        System.out.println("메뉴1 화면으로 전환");
    }

    public void handleButton1Click() {
        System.out.println("메뉴1 화면의 버튼1 처리");
    }

}
```

```java
public class Menu2ScreenUI implements ScreenUI {
    public void show() {
        System.out.println("메뉴2 화면으로 전환");
    }

    public void handleButton1Click() {
        System.out.println("메뉴2 화면의 버튼1 처리");
    }

}
```

이제 Application 클래스는 ScreenUI 인터페이스와 클래스를 이용해서 구현할 수 있다

```java
public class Application implements OnClickListener {

    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Button button1 = new Button("button1");

    private ScreenUI currentScreen = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        button1.setOnClickListener(this);
    }

    public void clicked(Component eventSource) {
        String sourceId = eventSource.getId();
        if (sourceId.equals("menu1")) {
            currentScreen = new Menu1ScreenUI();
            currentScreen.show();
        } else if (sourceId.equals("menu2")) {
            currentScreen = new Menu2ScreenUI();
            currentScreen.show();
        } else if (sourceId.equals("button1")) {
            if (currentScreen == null)
                return;
            currentScreen.handleButton1Click();
        }
    }

}
```



메뉴 클릭 처리 코드와 버튼 클릭 처리 코드는 목적이 다르며 서로 다른 이유로 변경이 된다.  
메뉴와 버튼이 동시에 바뀌는 경우가 있긴 하지만 그보다는 서로 다른 시점에 다른 이유로 변경될 가능성이 높을 것 같다.  
**이렇게 서로 다른 이유로 변경되는 코드가 한 메서드에 섞여 있으면 향후에 유지보수를 하기 어려워질 수 있다.**  
아래와 같이 코드를 분리하도록 하자

```java
public class Application {
    ...
    public Application() {
        menu1.setOnClickListener(menuListener);
        menu2.setOnClickListener(menuListener);
        button1.setOnClickListener(buttonListener);
    }
    
    private OnClickListener menuListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if (eventSource.getId().equals("menu1")) {
                currentScreen = new Menu1ScreenUI();
            } else if (eventSource.getId().equals("menu2")) {
                currentScreen = new Menu2ScreenUI();
            }
            currentScreen.show();
        }
    }
    
    private OnClickListener buttonListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if (currentScreen == null)
                return;
    
            if (eventSource.getId().equals("button1")) {
                currentScreen.handleButton1Click();
            }
        }
    }
}
```



버튼2를 추가해 달라는 요청이 왔다. 수정해보자.

```java
public interface ScreenUI {
    public void show();
    public void handleButton1Click();
    public void handleButton2Click();
}
```

```java
public class Menu1ScreenUI implements ScreenUI {
    public void show() {
        System.out.println("메뉴1 화면으로 전환");
    }

    public void handleButton1Click() {
        System.out.println("메뉴1 화면의 버튼1 처리");
    }

    public void handleButton2Click() {
        System.out.println("메뉴1 화면의 버튼2 처리");
    }
}
```

```java
public class Menu2ScreenUI implements ScreenUI {
    public void show() {
        System.out.println("메뉴2 화면으로 전환");
    }

    public void handleButton1Click() {
        System.out.println("메뉴2 화면의 버튼1 처리");
    }

    public void handleButton2Click() {
        System.out.println("메뉴2 화면의 버튼2 처리");
    }
}
```

```java
public class Application {
    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Button button1 = new Button("button1");
    private Button button2 = new Button("button2");

    private ScreenUI currentScreen = null;

    public Application() {
        menu1.setOnClickListener(menuListener);
        menu2.setOnClickListener(menuListener);
        button1.setOnClickListener(buttonListener);
        button2.setOnClickListener(buttonListener);
    }

    private OnClickListener menuListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if (eventSource.getId().equals("menu1")) {
                currentScreen = new Menu1ScreenUI();
            } else if (eventSource.getId().equals("menu2")) {
                currentScreen = new Menu2ScreenUI();
            }
            currentScreen.show();
        }
    }

    private OnClickListener buttonListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if (currentScreen == null)
                return;

            if (eventSource.getId().equals("button1")) {
                currentScreen.handleButton1Click();
            } else if (eventSource.getId().equals("button2")) {
                currentScreen.handleButton2Click();
            }
        }
    }

}
```

맨 처음 작성했던 코드와 수정된 코드를 비교해보면

Application 클래스에 모든 코드를 작성했었던 방식에서는 메뉴1 관련 코드와 메뉴2 관련 코드가 한 소스 코드에 섞여있다.  
따라서 메뉴1에 대한 관련 코드를 분석하거나 수정하려면, 메뉴1 관련 코드만 있는 경우와 비교해서 소스 코드릐 이곳저곳을 많이 이동해야 한다.  
메뉴의 개수가 증가할수록 위치 찾는 시간은 점점 길어져 불필요한 개발 시간이 증가되는 문제를 야기한다.

수정된 방식은 작성하는 클래스 개수가 다소 증가했으나, 메뉴 관련 코드들이 알맞게 분리되었다.   
메뉴1 화면과 관련된 코드는 Menu1ScreenUI 소스 코드에 위치해 있다.  
따라서 메뉴1 관련 코드를 분석하는 과정에서 메뉴2 관련 코드를 의식적으로 피할 필요가 없다. 또 버튼 클릭을 처리하는 코드가 단순화 되었다는 점.

Application 과 ScreenUI를 분리했을 때 생기는 장점은 메뉴3을 추가하면서 잘 드러난다.

```java
public class Menu3ScreenUI implements ScreenUI {

    public void show() {
        System.out.println("메뉴3 화면으로 전환");
    }

    public void handleButton1Click() {
        System.out.println("메뉴3 화면의 버튼1 처리");
    }

    public void handleButton2Click() {
        System.out.println("메뉴3 화면의 버튼2 처리");
    }
}
```

```java
public class Application {
    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Menu menu3 = new Menu("menu3");
    private Button button1 = new Button("button1");
    private Button button2 = new Button("button2");

    private ScreenUI currentScreen = null;

    public Application() {
        menu1.setOnClickListener(menuListener);
        menu2.setOnClickListener(menuListener);
        menu3.setOnClickListener(menuListener);
        button1.setOnClickListener(buttonListener);
        button2.setOnClickListener(buttonListener);
    }

    private OnClickListener menuListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if (eventSource.getId().equals("menu1")) {
                currentScreen = new Menu1ScreenUI();
            } else if (eventSource.getId().equals("menu2")) {
                currentScreen = new Menu2ScreenUI();
            } else if (eventSource.getId().equals("menu3")) {
                currentScreen = new Menu3ScreenUI();
            }
            currentScreen.show();
        }
    }

    private OnClickListener buttonListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if (currentScreen == null)
                return;

            if (eventSource.getId().equals("button1")) {
                currentScreen.handleButton1Click();
            } else if (eventSource.getId().equals("button2")) {
                currentScreen.handleButton2Click();
            }
        }
    }

}
```

처음 코드와 비교해보면 구조는 다소 복잡해 졌지만, 다음과 같은 장점을 얻을 수 있게 되었다.

{% hint style="warning" %}
새로운 메뉴 추가 시, 버튼 처리 코드가 영향을 받지 않음

한 메뉴 관련 코드가 한 개의 클래스로 모여서 코드 분석/수정이 용이함

서로 다른 메뉴에 대한 처리 코드가 섞여 있지 않아 수정이 용이
{% endhint %}

즉, 요구 사항이 바뀔 때, 그 변화를 좀 더 수월하게 적용할 수 있다는 장점을 얻었다.

이런 장점을 얻기 위해 사용된 것이 바로 **객체 지향 기법**이다.

객체 지향 기법을 적용하면 소프트웨어를 더 쉽게 변경할 수 있는 유연함을 얻을 수 있게 되고 이는 곧 요구 사항의 변화를 더 빠르게 수용할 수 있다는 것을 뜻한다.

## 3. 소프트웨어 가치

소프트웨어의 가치는 사용자가 요구하는 기능을 올바르게 제공하는데 있다.  
아무리 잘 만들었다고 해도 사용자가 요구하는 기능을 제공하지 않는 소프트웨어는 없는 것만 못하다.

하지만 요구사항은 언제든지 변할수 있기 때문에 요구사항에 맞게 빠르게 변화할 수 있어야한다. 이게 또 다른 소프트웨어의 가치 이다.

변화 가한 유연한 구조를 만들어 주는 핵심 기법 중의 하나가 바로 객체 지향이다.

