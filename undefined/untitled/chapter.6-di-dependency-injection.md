# Chapter.6 DI\(Dependency Injection\)와 서비스 로케이터

## 1. 어플리케이션 영역과 메인 영역

어플리케이션 영역은 고수준 정책 및 저수준 구현을 포함한 영역이고, 메인 영역은 어플리케이션이 동작하도록 각 객체들을 연결해주는 영역이다.

간단한 비디오 포맷 변환기를 살펴보자.  
핵심기능은 `변환 작업을 요청하면 순차적으로 변환을 처리한다`는 것이다.

변화가 발생하는 부분은 다음 두가지 이다.

* 변환 요청 정보 저장 : 파일에 보관한다. 또는 DB에 보관한다.
* 변환 처리 : ffmpeg을 사용한다. 또는 솔루션을 사용한다.

![](../../.gitbook/assets/image%20%2834%29.png)



Worker 클래스는 JobQueue에 저장된 객체로부터 JobData를 가져와 Transcoder를 이용해서 작업을 실행하는 책임이 있다.

```java
public class Worker {
    public void run() {
        JobQueue jobQueue = ...; // JobQueue를 구한다.
        Transcoder transcoder = ...; // Transcoder를 구한다.
        
        while (someRunningCondition) {
            JobData jabData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget());
        }
    }
}
```

Worker가 제대로 동작하려면 JobQueue나 Transcoder를 구현한 콘크리트 클래스의 객체가 필요하다.  
비슷하게 JobCLI 클래스도 JobQueue를 구현한 객체를 구해야 한다.

```java
public class JobCLI {
    public void interact() {
        printInputSourceMessage();
        String source = getSourceFromConsole();
        printInputTargetMessage();
        String target= getTargetFromConsole();
        
        JobQueue jobQueue = ...; // JobQueue를 구한다.
        JobQueue.addJob(new JobData(source, target);
        ...
    }
}
```

Locator라는 객체를 사용해서 객체를 제공해 보자.

```java
public class Locator {
    private static Locator instance;
    public static Locator getInstance() {
        return instance;
    }
    
    public static void init(Locator locator) {
        this.instance = locator;
    }
    
    private JobQueue jobQueue;
    private Transcoder transcoder;
    public Locator(JobQueue jobQueue, Transcoder transcoder) {
        this.jobQueue = jobQueue;
        this.transcoder = transcoder;
    }
    
    public JobQueue getJobQueue() { return jobQueue; }
    public Transcoder getTranscoder() { return transcoder; }
}
```



```java
public class Worker {
    public void run() {
        JobQueue jobQueue = Locator.getInstance().getJobQueue();
        Transcoder transcoder = Locator.getInstance().getTrancoder();
        
        while (someRunningCondition) {
            ...
        }
    }
}

public class JobCLI {
    public void interact() {
        ...
        JobQueue jobQueue = Locator.getInstance().getJobQueue();
        JobQueue.addJob(new JobData(source, target);
        ...
    }
}
```

![&#xD328;&#xD0A4;&#xC9C0; &#xAC04; &#xC21C;&#xD658; &#xC758;&#xC874;&#xC744; &#xBC1C;&#xC0DD;&#xC2DC;&#xD0A4;&#xC9C0; &#xC54A;&#xB3C4;&#xB85D; &#xD568;](../../.gitbook/assets/image%20%2836%29.png)

그렇다면 Locator를 초기화하고, JobCLI와 Worker 객체를 생성하고 실행해 주는 역할은 누가 하는가?  
바로 메인\(main\)영역에서 하는 일이다.

메인 영역은 다음 작업을 수행한다.

* 어플리케이션 영역에서 사용될 객체를 생성한다.
* 각 객체 간의 의존 관계를 설정한다.
* 어플리케이션을 실행한다.

```java
public class Main {
    public static void Main(String[] args) {
        // 상위 수준 모듈인 transcoder 패키지에서 사용할 
        // 하위 수준 모듈 객체 생성
        JobQueue jobQueue = new FileJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();
        
        // Locator 초기화
        Locator locator = new Locator(jobQueue, transcoder);
        Locator.init(locator);
        
        // 상위 수준 모듈 객체를 생성하고 실행
        final Worker worker = new Worker();
        Thread t = new Thread(new Runnable() {
            public void run() {
                worker.run();
            }
        });
        JobCLi cli = new JobCLI();
        cli.interact();
    }
}
```

![](../../.gitbook/assets/image%20%2833%29.png)

어플리케이션 영역에서 사용할 하위 수준의 모듈을 변경하고 싶다면 메인영역을 수정하면 된다. 예를 들어 FileJobQueue 객체 대신 DbJobQueue 객체를 사용하고 싶다면 FileJobQueue 대신 DbJobQueue 객체를 생성하고 조립해 주면 된다.

Worker객체와 JobCLI 객체는 Locator를 이용해서 필요한 객체를 가져온 뒤에 원하는 기능을 실행하였다. 이렇게 사용할 객체를 제공하는 책임을 갖는 객체를 서비스 로케이터\(Service Locator\)라고 부른다.

서비스 로케이터 방식은 몇가니 단점이 존재하는데, 그래서 외부에서 사용할 객체를 주입해주는 DI\(Dependency Injection\) 방식을 사용하는 것이 더 일반적이다.

## 2. DI\(Dependency Injection\)을 이용한 의존 객체 사용

사용할 객체를 직접 생성할 경우, 아래 코드처럼 콘크리트 클래스에 대한 의존이 발생하게 된다.

```java
public class Worker {
    public void run() {
        JobQueue jobQueue = new FileJobQueue();
        ...
    }
}
```

콘크리트 클래스를 직접 사용해서 객체를 생성하게 되면 의존 역전 원칙을 위반하게 되며,  
결과적으로 확장 폐쇄 원칙을 위반하게 된다.

이런 단점을 보완하기 위한 방법이 DI \( 의존주입 \) 이다.  
DI는 필요한 객체를 직접 생성하거나 찾지 않고 외부에서 넣어 주는 방식이다.

```java
public class Worker {
    private JobQueue jobQueue;
    private Transcoder transcoder;
    
    // 외부에서 사용할 객체를 전달받을 수 는 방법 제
    public Worker(JobQueue jobQueue, Transcoder transcoder) {
        this.jobQueue = jobQueue;
        this.transcoder = transcoder;
    }
    
    public void run() {
        while (someRunningCondition) {
            JobData jabData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget());
        }
    }
}
```

생성자를 이용해 의존 객체를 전달받도록 구현하면 main 클래도 변경할 수 있다.

```java
public class Main {
    public static void Main(String[] args) {
        // 상위 수준 모듈인 transcoder 패키지에서 사용할 
        // 하위 수준 모듈 객체 생성
        JobQueue jobQueue = new FileJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();
        
        // 상위 수준 모듈 객체를 생성하고 실행
        final Worker worker = new Worker(jobQueue, transcoder);
        Thread t = new Thread(new Runnable() {
            public void run() {
                worker.run();
            }
        });
        JobCLI cli = new JobCLI(jobQueue);
        cli.interact();
    }
}
```

여기서 중요한 점은 Worker나 JobCLI 객체가 스스로 의존하는 객체를 찾거나 생성하지 않고 main\(\)메서드에서 생성자를 통해 이들이 사용할 객체를 주입한다는 점이다. 때문에 이런 방식을 의존 주입이라고 부르는 것이다.

만약 main에서 하위 의존 모듈을 생성하고 주입하는 역할을 따로 조립기의 역할로 뺀다면 나중에 변경의 유연함을 얻을 수 있다.

```java
public class Assembler {
    public void createAndWire() {
        JobQueue jobQueue = new FileJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();
        this.worker = new Worker(jobQueue, transcoder);
        this.jobCLI = new JobCLI(jobQueue);
    }
    public Worker getWorker() {
        return this.worket;
    }
    public JobCLI getJobCLI() {
        return this.jobCLI;
    }
    ...
}
```

Assembler 클래스에서 객체를 생성하고 생성자를 이용해 의존 객체를 전달해 주고 있다.  
이제 Main 클래스는 Assembler 에게 객체 생성과 조립 책임을 위임한 뒤에 Assembler 가 생성한 Worker 객체와 JobCLI 객체를 구하는 방식으로 변경된다.

```java
public class Main {
    public static void main(String[] args) {
        Assembler assembler = new Assembler();
        assembler.createAndWire();
        final Worker worker = assembler.getWorker();
        JobCLI jobCli = assembler.getJobCli();
        ...
    }
}
```

### 2.1 생성자 방식과 설정 메서드 방식

생성자 방식은 생성자를 통해서 의존 객체를 전달받는 방식이다.  
그것이 생성자 방식이니까

설정 메서드 방

```java
public class Worker {
    private JobQueue jobQueue;
    private Transcoder transcoder;
    
    public void setJobQueue(JobQueue jobQueue) {
        this.jobQueue = jobQueue
    }
    public void setTransCoder(TransCoder transCoder) {
        this.transCoder= transCoder
    }
    
    public void run() {
        while (someRunningCondition) {
            JobData jabData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget());
        }
    }
}
```

#### 각 방식의 장단점

생성자 방식은 객체를 생성하는 시점에 필요한 모든 의존 객체를 준비할 수 있다.  
때문에, 객체를 생성하는 시점에서 의존 객체가 정상인지 확인할 수 있다.

```java
public class JobCLI { 
    private JobQueue jobQueue;
    
    public JobCLI(JobQueue jobQueue) {
        if (jobQueue == null) throw new IllegalArgumentException();
        this.jobQueue = jobQueue;
    }
}
```

```java
// 정상 생성
JobCLI jobCli = new JobCLI(jobQueue);
jobCli.interact(); // jobQueue 의존 객체가 존재함

// 비정상 생성인 경우
JobCLI jobCli = new JobCLI(null); // 익셉션 발생
jobCli.interact(); // 노실행
```

의존 객체를 먼저 생성할 수 없다면 생성자 방식을 사용할 수 없다.

설정 메서드 방식은 객체를 생성한 뒤에 의존 객체를 주입하게 된다.

```java
Worker worker = new Worker();
// 의존 객체를 설정하지 않음
// worker.setJobQueue(jobQueue);
// worker.setTranscoder(transcoder);

worker.run() // jobQueue, transcoder 이 null 이므로 익셉션 발생
```

객체를 생성한 이후에 설정할 수 있기 때문에, 어떤 이유로 인해 의존할 객체가 나중에 생성된다면 설정 메서드 방식을 사용해야 한다.  
또, 의존할 객체가 많을 경우, 이름을 통해서 어떤 객체가 설정되는 보다 쉽게 알수 있어 가독성을 높여 주는 효과를 얻을 수 있다.

## 3. 서비스 로케이터의 단점

서비스 로케이터의 가장 큰 단점은 동일 타입의 객체가 다수 필요할 경우, 각 개체 별로 제공 메서드를 만들어 주어야 한다는 점이다.

예를 들어, FileJobQueue 객체와 DbJobQueue 객체가 서로 다른 부분에 함께 사용되어야 한다면

```java
public class ServiceLocator {
    public JobQueue getJobQueue1() { ... }
    public JobQueue getJobQueue2() { ... }
}
```

이름부터 마음에 안드는데..?  
그렇다고 해서 이름을 File 이나 Db 같은 단어를 붙이게 되면 콘크리트 클래스에 직접 의존하는 것과 동일한 효과를 발생시킨다.

```java
public class Worker {
    private boolean someCondition;
    
    public void run() {
        JobQueue jobQueue = someCondition ?
            ServiceLocator.getFileJobQueue() : ServiceLocator.getDbJobQueue();
        ...
    }
}
```

만약 다른 JobQueue 구현 객체가 추가되면 위 코드도 함께 수정되어야 한다. OCP 원칙 파괴...

그렇다면 DI를 사용하면?

```java
Worker worker1 = new Worker();
worker1.setJobQueue(new FileJobQueue());

Worker worker2 = new Worker();
worker2.setJobQueue(new DbJobQueue());
```

이렇듯 서비스 로케이터는 변경의 유연함을 떨어뜨리는 문제를 갖고 있기 때문에,  
부득이란 상황이 아니라면 DI를 사용하도록 하자.

