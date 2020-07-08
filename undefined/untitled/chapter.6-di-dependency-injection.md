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

![&#xD328;&#xD0A4;&#xC9C0; &#xAC04; &#xC21C;&#xD658; &#xC758;&#xC874;&#xC744; &#xBC1C;&#xC0DD;&#xC2DC;&#xD0A4;&#xC9C0; &#xC54A;&#xB3C4;&#xB85D; &#xD568;](../../.gitbook/assets/image%20%2835%29.png)

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

