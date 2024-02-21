# 옵저버(Observer) 패턴

웹 사이트의 응답 속도가 느리거나 연결이 안되면 담당자에게 이메일로 통지해주는 시스템이 있다.

```java
public class StatusChecker {
    private EmailSender emailSender;

    public void check() {
        Status status = loadStatus();
        
        if (status.isNoNormal()) {
            emailSender.senderEmail(status);
        }
    }
}
```

SMS로 알려주는 기능 추가 요청이 왔다.

```java
public class StatusChecker {
    private EmailSender emailSender;
    private SmsSender smsSender;

    public void check() {
        Status status = loadStatus();
        
        if (status.isNoNormal()) {
            emailSender.sendEmail(status);
            smsSender.sendSms(status);
        }
    }
}
```

핵심은 상태가 변경될 때 정해지지 않은 임의의 객체에게 변경 사실을 알려준다는 점이다.\
이렇게 한 객체의 상태 변화를 정해지지 않은 여러 다른 객체에 통지하고 싶을 때 사용되는 패턴이 옵저버 패턴이다.

![](<../../../.gitbook/assets/image (38).png>)

옵저버 패턴에는 크게 주제 객체와 옵저버 객체가 등장하는데, 주제 객체는 다음의 두 가지 책임을 갖는다.

* 옵저버 목록을 관리하고, 옵저버를 등록하고 제거할 수 잇는 메서드를 제공한다.\
  add() 메서드와 remove() 메서드가 각각 옵저버를 목록에 등록하고 삭제하는 기능을 제공한다.
* 상태의 변경이 발생하면 등록된 옵저버에 변경 내역을 알린다. notifyStatus() 메서드가 등록된 옵저버 객체의 onAbnormalStatus() 메서드를 호출한다.

```java
public abstract class StatusSubject {
    private List<StatusObserver> observers = new ArrayList<StatusObserver>();
    
    public void add(StatusObserver observer) {
        observers.add(observer);
    }
    
    public void remove(StatusObserver observer) {
        observers.remove(observer);
    }
    
    public void notifyStatus(Status status) {
        for (StatusObserver observer : observers)
            observer.onAbnormalStatus(status); 
    }
}
```

```java
public class StatusChecker extends StatusSubject {
    
    public void check() {
        Status status = loadStatus();
        
        if (status.isNotNormal())
            super.notifyStatus(status);
    }
    
    private Status loadStatus() {
        ...
    }
}
```

```java
public interface StatusObserver {
    void onAbnormalStatus(Status status);
}
```

```java
public class StatusEmailSender implements StatusObserver {
    
    @Override
    public void onAbnormalStatus(Status status) {
        sendEmail(status);
    }
    
    private void sendEmail(Status status) {
        ...// 이메일 전송 코드
    }
}
```

옵저버 패턴을 적용할 때의 장점은 주제 클래스 변경 없이 상태 변경을 통지 받을 옵저버를 추가할 수 있다는 점이다.
