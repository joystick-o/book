# 템플릿 메서드\(Template Method\) 패턴

프로그램을 구현하다 보면, 완전히 동일한 절차를 가진 코드를 작성하게 될 경우가 있다. 또한, 일부 구현만 다를 뿐 나머지의 구현은 똑같은 경우도 있다. 예를 들어 아래와 같이, DB 데이터와 LDAP을 이용해서 인증을 처리하는 클래스는 사용자 정보를 가져오는 구현만 다를 뿐 인증을 처리하는 과정은 완전히 동일할 수 있다.

![](../../../.gitbook/assets/image%20%2838%29.png)

이렇게 실행 과정/단계는 동일하지만 일부 구현이 다른 경우에 사용할 수 있는 패턴이 템플릿 메서드 패턴이다. 템플릿 메서드 패턴은 두 가지로 구성된다.

* 실행 과정을 구현한 상위 클래스
* 실행 과정의 일부 단계를 구현한 하위 클래

```java
public abstract class Authenticator {
    // 템플릿 메서
    public Auth authenticate(String id, String pw) {
        if ( ! doAuthenticate(id, pw) )
            throw createException();
        
        return createAuth(id);
    }

    protected abstract boolean doAuthenticate(String id, String pw);
    
    private RuntimeException createException() {
        throw new AuthException();
    }
    protected abstract Auth createAuth(String id);
}
```

authenticate\(\) 메서드는 DbAthenticator와 LdapAuthenticator에서 동일했던 실행 과정을 구현하고 있고, 두 객체에서 차이가 나는 부분은 별도의 추상 메서드로 분리하였다.

id/pw를 이용하여 인증 여부를 확인하는 단계는 doAuthenticate\(\) 추상 메서드로 분리하였고, Auth 객체를 생성하는 단계는 createAuth\(\) 추상 메서드로 분리하였다. authenticate\(\) 메서드는 **모든 하위 타입에 동일하게 적용되는 실행 과정을 제공**하기 때문에 **템플릿 메서드\(Template Method\)** 라고 부른다.

```java
public class LdapAthenticator extends Authenticator{
   
    @Override
    protected boolean doAuthenticate(String id, String pw) {
        return ldapClient.authenticate(id, pw);
    }

    @Override
    protected Auth createAuth(String id) {
        LdapContext ctx = ldapClient.find(id);
        return new Auth(id, ctx.getAttribute("name"));
    }

}
```

LdapAuthenticator 클래스는 이제 전체 실행 과정을 제공하지 않고, 일부 과정의 구현만을 제공한다. 전체 실행 과정은 상위 타입인 Authenticator의 authenticate\(\) 메서드에서 제공하게 된다.

템플릿 메서드 패턴을 사용하게 되면, 동일한 실행 과정의 구현을 제공하면서 동시에 하위 타입에서 일부 단계를 구현하도록 할 수 있다.

## 상위 클래스가 흐름 제어 주체

템플릿 메서드 패턴의 특징은 하위 클래스가 아니라 **상위 클래스에서 흐름 제어**를 한다는 것이다. 일반적으로 하위 타입이 상위 타입의 기능을 재사용 할지를 결정하기 때문에, 하위 타입서 흐름 제어를 하게 된다

