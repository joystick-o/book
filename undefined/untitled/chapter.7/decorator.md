# 데코레이터(Decorator) 패턴

상속은 기능을 확장하는 방법을 제공한다.\
데이터를 파일에 출력하는 FIleOut 클래스가 있을때, 버퍼 기능, 압축 기능을 추가하려면 상속을 받아서 구현할 수 있을 것이다.

![](<../../../.gitbook/assets/image (45).png>)

상속을 기용한 기능 확장 방법이 쉽긴 하지만 클래스가 불필요하게 증가하는 문제가 발생할 수 있다.

![](<../../../.gitbook/assets/image (5) (1).png>)

이러한 경우에 사용할 수 있는 패턴이 데코레이터 패턴이다. 데코레이터 패턴은 상속이 아닌 위임을 하는 방식으로 기능을 확장해 나간다.

![](<../../../.gitbook/assets/image (20).png>)

여기서 중요한 점은 기능 확장을 위해 FileOutImpl 클래스를 상속받지 않고 Decorator라 불리는 별도의 추상 클래스를 만들었다.

이 클래스는 모든 데코레이터를 위한 기븐 기능을 제공하는 추상 클래스이다.

```java
public abstract class Decorator implements FileOut {
    private FileOut delegate; // 위임 대상
    public Decorator(FileOut delegate) {
        this.delegate = delegate;
    }
    
    protected void doDelegate(byte[] data) {
        delegate.write(data); // delegate에 쓰기 위
    }
}
```

BufferedOut, EncryptionOut, ZipOut 클래스는 모두 데코레이터 클래스로서 Decorator 클래스를 상속받고 있다.

```java
public class EncryptionOut extends Decorator {
    public EncryptionOut (FileOut delegate) {
        super(delegate);
    }
    
    public void write(Byte[] data) {
        byte[] encryptedData = encrypt(data);
        super.doDelegate(encryptedData);   
    }
    
    private byte[] encrypt(byte[] data) {
        ...
    }
}
```

```java
FileOut delegate = new FileOutImpl;
FileOut fileOut = new EncryptionOut(delegate);
fileOut.write(data);
```

EncryptionOut의 write() 메서드를 실행하면 EncryptionOut의 write() 메서드에서 데이터를 암호화하고, FileOutImple 객체의 write로 암호화된 데이터를 전달한다. 여기서 EncrypteionOut 객체는 FileOutImpl 객체가 제공하는 파일쓰기 기능에 암호화 기능을 추가해 주는 역할을 수행하며, 기존 기능에 새로운 기능을 추가해 준다는 의미에서 데코레이터라고 부른다.

데코레이터 패턴의 장점은 데코레이터를 조합하는 방식으로 기능을 확장할 수 있다는 점이다.

예를 들어 데이터를 압축한 뒤 암호화를 해서 파일에 쓰고 싶다면, 다음과 같이 두개의 데코레이터 객체를 조합해서 쓰면된다.

```java
FileOut delegate = new FileOutImpl;
FileOut fileOut = new EncryptionOut(new ZipOut(delegate));
fileOut.write(data);
```

서로의 데코레이터가 서로 독립적으로 존재할 수 있고 암호화 구현이 변경되어도 다른 데코레이터는 변경의 영향이 없다. 이는 단일 책임 원칙을 지키도록 만들어 준다.

## 고려할점

데코레이터의 구현이 비교적 간단하지만, 정의 되어 있는 메서드가 증가하게 되면 그 만큼 데코레이터의 구현도 복잡해진다.\
사용자 입장에서 데코레이터 객체와 실제 구현 객체의 구분이 되지 않기 때문에 코드만으로는 기능이 어떻게 동작하는지 이해하기 어렵다는 점이다.
