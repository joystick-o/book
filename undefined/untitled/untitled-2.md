# Chapter.2 객체 지향

객체 지향을 지원하는 언어를 사용하더라도 실제 결과물은 객체 지향과 거리가 멀어질 수 있다.

이런 결과가 나오는 이유는 객체 자체와 객체 지향의 핵심인 캡슐화 및 추상화가 적용되지 않았기 때문이다.

## 1. 절차 지향과 객체 지향

### 1.1 절차 지향

프로시저로 프로그램을 구성하는 기법을 절차 지향 프로그래밍 이라고 부른다.

절차지향은 데이터를 중심으로 한 프로시저로 구분된다.  
각 프로시저는 데이터를 사용해서 기능을 구현하며, 필요에 따라 다른 프로시저를 사용하기도 하도, 여러 프로시저가 동일한 데이터를 공유한다.

시험 성적 관리 프로그램을 생각해보자.

> 평균 계산 프로시저는 각 과목의 점수가 보관된 데이터를 읽어서 합을 구한 뒤, 평균 값을 계산한다. 계산된 평균값은 다른 데이터로 생성된다.
>
> 화면 출력 프로시저는 평균 계산 프로시저가 생성한 평균 값 데이터와 과목 점수 데이터를 이용해서 화면에 성적을 출력한다.

다수의 프로시저들이 데이터를 공유하는 방식으로 만들어 지기 때문에, 절차 지향 프로그램은 자연스럽게 데이터를 중심으로 구현하게 된다.

그렇기때문에 최초에 절차 지향적으로 코드를 구현하는 것은 쉽지만 데이터 종류가 증가하고 사용하는 프로시저가 많아지면 다음과 같은 문제들이 발생한다.

> 데이터 타입이나 의미를 변경해야 할 때, 함께 수정해야 하는 프로시저가 증가한다.  
> 같은 데이터를 프로시저들이 서로 다른 의미로 사용하는 경우가 발생한다.

예를 들어, 장비의 꺼짐/켜짐 상태를 저장하기 위해 이름이 isOn 이고 타입이 boolean 인 데이터를 사용한다고 하자.

이 데이터를 사용하는 프로시저는 모두 isOn을 boolean 타입으로 처리하는데  
대기 상태의 값을 추가해야 된다고 생각해보자.

그럼 isOn의 타입을 변경해줘야 하고 이 데이터를 사용하는 프로시저를 함께 수정해줘야 한다.

이보다 더 안좋은 경우는 데이터를 서로 다른 의미로 사용하는 경우가 발생할 가능성이 높다는 것.  
최초에 만료일 데이터가 null 인 경우 이를 오류로 처리하도록 확인하는 프로시저를 만들었는데, 다른 프로시저에서 서비스를 무한정 사용한다는 의미로 만료일 데이터의 값을 null 설정할수도 있다.

절차 지향적 프로그램을 구성할 때 매우 흔하게 발생하는 문제들이며,  
이로 인해 한 곳을 수정하게 되면, 다른곳에서 문제가 발생하는 악순환이 발생하기도 한다.

### 1.2 객체 지향

데이터 및 데이터와 관련된 프로시저를 객체라고 불리는 단위로 묶는다.  
객체 지향 기법으로 작성된 프로그램은 객체들로 구성이 되고, 각 객체는 자신만의 데이터와 프로시저를 갖는다.

절차 지향과 달리 객체 지향은 객체 별로 데이터와 프로시저를 알맞게 정의해야 하고, 프로그램의 규모가 작을때는 절차 지향보다 복잡한 구조를 갖게 된다.

하지만 객체 지향적으로 만든 코드에서는 객체의 데이터를 변경하더라도 객체로만 변화가 집중되고 다른 객체에는 영향을 주지 않기 때문에, 요구 사항의 변화가 방생했을 때 절차 지향 방식보다 프로그램을 더 쉽게 변경할 수 있는 장점을 갖는다.

## 2. 객체

### 2.1 객체의 핵심은 기능을 제공하는 것

객체 지향의 가장 기본은 객체.  
객체는 데이터와 그 데이터를 조작하는 프로시저로 구성된다고 했는데, 이는 물리적 특징이고객체를 정의할 때 사용되는 것은 객체가 제공해야 할 기능이다.

예를 들어, 소리 크기 제어 객체를 정의해보자.  
이 객체가 제공하는 기능은 다음과 같을 것이다.

> 소리 크기 증가  
> 소리 크기 감소  
> 음소거

이 객체가 내부적으로 어떤 데이터 타입 값으로 보관하는지는 중요하지 않다.  
또한 실제로 객체가 어떻게 크기를 증가시키거나 감소시키는지 알 수 없다.  
단지, 소리 크기 증가, 소리 감소 증가, 음소거 라는 기능을 제공한다는 것이 중요할 뿐.

### 2.2 인터페이스와 클래스

객체는 객체가 제공하는 기능으로 정의된다고 했는데, 보통 객체가 제공하는 기능을 오퍼레이션 이라고 부른다. 즉, 객체가 제공하는 기능을 사용한다는 것은 결국 객체의 오퍼레이션을 사용한다는 의미가 된다. 그런데, 객체가 제공하는 오퍼레이션을 사용할 수 있으려면, 오퍼레이션의 사용법을 알아야 한다.

오퍼레이션의 사용법은 일반적으로 다음과 같이 세 개로 구성되며, 이 세 가지를 합쳐서 시그니쳐라고 부른다.

> 기능 식별 이름  
> 파라미터 및 파라미터 타입  
> 기능 실행 결과 값

객체가 제공하는 모든 오퍼레이션 집합을 객체의 인터페이스 라고 부르며,  
인터페이스는 객체를 사용하기 위한 일종의 명세서나 규칙이라고 생각하면 된다.

![](../../.gitbook/assets/image%20%282%29.png)

개념적으로 인터페이스와 클래스는 구분되어 있지만, 실제로 자바나 C\# 등의 언어는 인터페이스와 클래스가 각 언어에서 클래스 라고 부르는 것에 함께 정의되어 있다.  
뿐만 아니라 자바 언어는 오퍼레이션 정의만 있고 구현은 없는 인터페이스\(interface type\)를 제공하고 있다.

### 2.3 메세지

자바와 같은 언어에서는 메서드를 호출하는 것이 메세지를 보내는 과정에 해당된다.

```java
FileInputStream is = new FileInputStream(fileName);
byte[] data = new byte[512];
int readBytes = is.read(data);
```

is 변수는 FileInputStream 타입의 객체를 참조하는데, is.read\(data\) 코드는 is 가 참조하는 객체에 read\(\) 오퍼레이션을 실행해 달라는 메세지를 전송한다고 보면 된다.

## 3. 객체의 책임과 크기

객체는 객체가 제공하는 기능으로 정의된다고 했는데,  
이는 다시 말하면 객체마다 자신만의 책임\(responsibility\)이 있다는 의미를 갖는다.

![](../../.gitbook/assets/image%20%283%29.png)

한 객체가 갖는 책임을 정의한 것이 바로 타입/인터페이스라고 생각하면 된다.  
그럼, 객체가 갖는 책임은 어떻게 결정될까? 이결정을 하는 것이 바로 객체 지향 설계의 출발점이다.

처음에는 프로그램을 만들기 위해 필요한 기능 목록을 정리해야 한다.  
기능을 어떻게 객체들에게 분배하느냐에 따라서 객체의 구성이 달라진다.

객체 지향적으로 프로그래밍을 할 때, 가장 어려우면서 가장 중요한 것이 바로 객체마다 기능을 할당하는 과정이다.

객체가 얼마나 많은 기능을 제공할 것인가에 대한 확실한 규칙이 하나 존재하는데, 바로  객체가 갖는 책임 크기는 작을수록 좋다는 것이다.

객체가 갖는 책임이 커질수록 절차 지향적으로 구조로 변질되며, 절차 지향의 가장 큰 단점인 기능 변경의 어려움 문제가 발생하게 된다.

그래서 생긴 원칙이 있는데 바로 **단일 책임 원칙\(Single Responsibility Principle\)**이다.

## 4. 의존

```java
public class FlowController {
    public void process() {
        FileDataReader reader = new FileDataReader(fileName); // 객체생성
        byte[] plainBytes = reader.read() // 메서드 호출
        
        ByteEncryptor encrytor = new ByteEncryptor(); // 객체 생성
        byte[] encryptedBytes = encryptor.encrypt(plainBytes); // 메서드 호출
        
        FileDataWrite write = new FileDataWriter(); // 객체 생성
        writer.write(encryptedBytes); // 메서드 호
    }
}
```

이렇게 한 객체가 다른 객체를 생성하거나 다른 객체의 메서드를 호출할 때, 이를 그 객체에 의존\(dependency\) 한다고 표현한다.

위 코드에서 FlowController 가 FileDataReader 에 의존한다고 표현할 수 있다.

```java
public void process(ByteEncryptor encryptor) {
    // 내부에서 encryptor 를 사용할 가능성이 높
}
```

위 처럼 파라미터로 전달받는 경우에도 의존한다고 볼 수 있다.

의존을 한다는 것은 의존하는 타입에 변경이 발생할 때 나도 함께 변경될 가능성이 높다는 것은 뜻한다.  
의존의 영향은 꼬리에 꼬리를 문 것처럼 전파되는 특징을 갖는다.  
의존의 이런 특징 때문에 의존이 순환해서 발생할 경우 다른 방법이 없는디 고민해야 한다.

![](../../.gitbook/assets/image%20%284%29.png)

순환 의존이 발생할 경우 적극적으로 이를 해소하는 방법을 찾아야 한다.  
순환 의존이 발생하지 않도록 하는 칙 중의 하나로 의존 역전 원칙이\(Dependency inversion principle\) 있다.

### 4.1 의존의 양면성

```java
public class Authenticator {
    public boolean authenticate(String id, String password) {
        Member m = findMemberById(id);
        if (m == null) return false;

        return m.equalPassword(password); // password가 m의 암호와 동일하면 true
    }
    ...
}
```

Authenticator 클래스를 사용하는 코드는 다음과 같이 authenticate\(\) 메서드를 이용해서 사용자가 입력한 암호가 올바른지 여부를 판단할 것이다.

```java
public class AuthenticationHandler {

    public void handleRequest(String inputId, String inputPassword) {
        Authenticator auth = new Authenticator();
        if (auth.authenticate(inputId, inputPassword)) {
            // 아이디/암호 일치할 때의 처리
        } else {
            // 아이디/암호 일치하지 않을 때의 처리
        }
    }
}
```

위 코드에서 AuthentcationHandler 클래스는 Authenticator 클래스를 사용하고 있다.  
즉, AuthenticationHandler 클래스가 Authenticator 클래스에 의존하고 있고, Authenticator 클래스에 변화가 생기면 AuthenticationHandler 클래스도 영향을 받게 된다.

그런데, 잘못된 아이디를 입력한 것인지 아니면 암호가 틀린 것인지 여부를 확인해서 시스템상에 로그로 남겨달라는 요구가 추가되었다.

```java
public class AuthenticationHandler {
    public void handleRequest(String inputId, String inputPassword) {
        Authenticator auth = new Authenticator();
        try {
            auth.authenticate(inputId, inputPassword);
            // 아이디/암호가 일치하는 경우의 처리
        } catch(MemberNotFoundException ex) {
            // 아이디가 잘못된 경우의 처리
        } catch(InvalidPasswordException ex) {
            // 암호가 잘못된 경우의 처리
        }
    }
}
```

```java
public class Authenticator {
    public void authenticate(String id, String password) {
        Member m = findMemberById(id);
        if (m == null) throw new MemberNotFoundException();

        if (! m.equalPassword(password)) throw new InvalidPasswordException();
    }
    ...
}
```

AuthenticationHandler 클래스가 Authenticator 클래스에 의존하고 있는 상황에서, AuthenticationHandler 클래스의 변경 요구 때문에 Authenticator 클래스에 변화가 발생한 것이다.  
이는 의존이 다음과 같이 상호간에 영향을 준다는 것을 보여준다.

> 내가 변경되면 나에게 의존하고 있는 코드에 영향을 준다.  
> 나의 요구가 변경되면 내가 의존하고 있는 타입에 영향을 준다.

## 5. 캡슐화

캡슐화는 객체가 내부적으로 기능을 어떻게 구현하는지를 감추는 것이다.  
이를 통해 내부의 기능 구현이 변경되더라도 그 기능을 사용하는 코드는 영향을 받지 않도록 만들어 준다.

### 5.1 절차 지향 방식 코드

회원의 서비스 만료 날짜 여부에 따라 서비스를 제공하거나 안내 페이지를 보여줘야 한다고 하자.

```java
public class Member {
    private Date expiryDate;
    private boolean male;

    public Date getExpiryDate() {
        return expiryDate;
    }
    public boolean isMale() {
        return male;
    }
}
```

Member 객체를 이용해서 만료 여부를 확인하는 코드는 Member 가 제공하는 expiryDate 데이터의 값과 현재 시간을 비교하게 된다.

```java
if (member.getExpiryData() != null && 
    member.getExpiryDate().getDate() < System.currentTimeMillis()) {
    // 만료되었을 때의 처리
}
```

그런데 서비스를 잘 운영해 오던 중에, 여성 회원인 경우 만료 기간이 지났어도 30일간은 서비스를 사용할 수 있도록 정책이 변경되었다고 가정해 보자.

```java
long day30 = 1000 * 60 * 60 * 24 * 30; // 30일
if ((
        member.isMale() && member.getExpiryData() != null && 
        member.getExpiryDate().getDate() < System.currentTimeMillis()
    )
    ||
    (
        !member.isMale() && member.getExpiryData() != null && 
        member.getExpiryDate().getDate() < System.currentTimeMillis() - day30
    ))
{
    // 만료되었을 때의 처리
}
```

생각만해도 끔-찍

### 5.2 캡슐화 된 기능 구현

객체 지향적으로 코드를 재구성해 보자.

```java
public class Member {
    private Date expiryDate;
    private boolean male;

    public boolean isExpired() { // 만료 여부 확인 구현을 캡슐화
        if (male) {
            return expiryDate != null &&
                  expiryDate.getDate() < System.currentTimeMillis();
        }
         return expiryDate != null &&
                expiryDate.getDate() < System.currentTimeMillis() - DAY30;
    }
}
```

다른 클래스에서는 isExpired 메서드를 어떻게 구현했는지 알지 못한다.  
단지 서비스가 만료되었으면 true를 리턴한다는 것만 알고 있으며  
다음과 같이 사용만 하면 된다.

```java
if (member.isExpired()) {
    // 만료에 따른 처
}
```

### 5.3 캡슐화의 결과는 내부 구현 변경의 유연성 획득

기능 구현을 캡슐화하면 내부 구현이 변경되더라도, 기능을 사용하는 곳의 영향을 최소화할수 있다.  
캡슐화를 통해서 내부 기능 구현 변경의 유연함을 얻을 수 있다는 것을 의미한다.  
즉, 캡슐화를 잘 할수록 보다 쉽게 구현을 변경할 수 있게 된다.

### 5.4 캡슐화를 위한 두 개의 규칙

> Tell, Don't Ask
>
> 데미테르의 법칙 \(Law of Demeter\)

Tell, Don't Ask 규칙은 데이터를 물어보지 않고 기능을 실행해 달라고 말하는 규칙  
데이터를 읽는 것은 데이터를 중심으로 코드를 작성하게 만드는 원인이 되니  
데이터 대신 기능을 실행해 달라고 명령을 내려야 한다.

데미테르의 법칙은 Tell, Don't Ask 규칙을 따를 수 있도록 만들어 주는 또 다른 규칙

> 메서드에서 생성한 객체의 메서드만 호출  
> 파라미터로 받은 객체의 메서드만 호출  
> 필드로 참조하는 객체의 메서드만 호

신문배달부와 지갑

```java
// 고
public class Customer {
    private Wallet wallet;
    private Wallet getWallet() {
        return wallet;
    }
}
```

```java
// 지
public class Wallet {
    private int money;

    public int getTotalMoney() {
        return money;
    }
    public void substractMoney(int debit) {
        money -= debit;
    }
}
```

신문 배달부 클래스는 고객에게 요금을 받기 위해 아래와 같은 코드를 작성할 것이다.

```java
// Paperboy 클래스
int payment = 10000;
Wallet wallet = customer.getWallet();
if (wallet.getTotalMoney() >= payment) {
    wallet.substractMoney(payment);
} else {
    // 다음에 요금 받으러 오는 처리
}
```

이 코드는 동작에는 문제가 없다. 하지만 개념적으로 위 코드는 신문 배달부가 아래와 같은 방법으로 요금을 받아가는 것과 같다.

> 고객님 지갑 주세요 : customer.getWallet\(\)  
> 지갑에 돈이 있는지 확인 합니다 : wallet.getTotalMoney\(\) &gt;= payment  
> 지갑에서 돈을 빼가겠습니다 : wallet.substractMoney\(payment\)

실제로 신문 배달부 입장에서는 고객한테 요금을 받아 가기만 하면 된다.  
좀 더 현실적으로 코드를 바꿔 보자.

```java
public class Customer {
    private Wallet wallet;

    public int getPayment(int payment) {
        if (wallet == null) throw new NotEnoughMoneyException();
        if (wallet.getTotalMoney() >= payment) {
            wallet.substractMoney(payment);
            return payment;
        }
        throw new NotEnoughMoneyException();
    }
}
```

```java
int payment = 10000;
try {
    int paidAmount = customer.getPayment(payment);
    // ...
} catch(NotEnoughMoneyException ex) {
    // 다음에 요금 받으러 오는 처리
}
```

수정 전과 후를 비교해보면 이전 코드는 데미테르의 법칙을 어기고 있다.  
신문배달부 클래스에서는 customer 객체의 메서드만 사용해야 하는데, customer 객체를 통해서 가져온 wallet 객체의 메서드를 호출하고 있다.

데미테르의 법칙을 지키지 않는 전형적인 증상이 두 가지 있다.

> 연속된 get 메서드 호출  
> 임시 변수의 get 호출이 많음

연속된 get 메서드 호출의 모양은 다음과 같은 모양을 갖는다.

```java
value = somObject.getA().getB().getValue();
```

두 번째는 임시 변수에 할당된 객체의 get을 호출하는 코드다 많은 경우이다.

```java
A a = someObject.getA();
B b = a.getB();
value = b.getValue();
```

위 두가지 증상이 보인다면잘 확인해보고 적극적으로 캡슐화 하도록 노력해야 한다.

## 6. 객체 지향 설계 과정

지금까지 객체의 정의, 책임, 의존, 캡슐화에 대해서 살펴보았다.  
이들 내용을 종합적으로 정리해 보면 객체 지향 설계한 다음의 작업을 반족하는 과정이라고 볼 수 있다.

> 1. 제공해야 할 기능을 찾고 또는 세분화하고, 그 기능을 알맞은 객체에 할당한다.
>
>    A. 기능을 구현하는데 필요한 데이터를 객체에 추가한다. 객체에 데이터를 먼저 추가하고 그 데이터를 이용하는 기능을 넣을 수도 있다.
>
>    B. 기능은 최대한 캡슐화해서 구현한다.
>
> 2. 객체 간에 어떻게 메시지를 주고받을 지 결정한다.
> 3. 과정1과 과정2를 개발하는 동안 지속적으로 반복한다.

앞의 파일 데이터 암호화 예를 보면, 파일의 데이터를 읽어 와 암호화한 뒤에 파일에 저장하는 기능이 필요하다.  
하지만 암호화 객체가 실제로는 두 기능을 함께 제공하고 있다.

> 흐름제어 \( 데이터 읽고, 암호화하고, 데이터 쓰고\)  
> 데이터 암호

처음에는 이것이 불명확한 경우가 다. 구현을 진행하는 과정에서 암호화 알고리즘을 변경해랴 할 때, 데이터 암호화 기능과 흐름 제어가 한 객체에 섞여 있다는 것을 알게 될 수도 있다.

객체 설계는 한 번에 완성되지 않고 구현을 진행해 나가면서 점진적으로 완성된다.  
이는 최초에 만든 설계가 벽하지 않으며, 개발이 진행되면서 설계도 함께 변경된다는 것을 의미한다.  
따라서 설계를 할 때에는 변경되는 부분을 고려한 유연한 구조를 갖도록 노력해야 한다.

