---
description: ClassCastException
---

# JpaCursorItemReader ( jpql 쿼리)

spring batch job을 개발하던중

클래스 변환이 불가 하다면서 계속해서 에러가 발생했는데..

<figure><img src="../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

```java
@Bean
public Step eventIssueCheckStep() {
    return stepBuilderFactory.get(JOB_NAME + "Step").<EventEntity, Event>chunk(CHUNK_SIZE)
            .reader(eventIssueCheckItemReader(null)).processor(eventIssueCheckItemProcessor())
            .writer(eventIssueCheckItemWriter()).build();
}


@Bean
@StepScope
public JpaCursorItemReader<EventEntity> eventIssueCheckItemReader(
        @Value("#{jobParameters['requestDate']}") String requestDate) {
    LocalDateTime checkDateTime;
    checkDateTime =
            LocalDateTime.parse(requestDate, DateTimeFormatter.ofPattern("yyyyMMddHH:mm:ss")).minusMinutes(10);
    return new EventIssueCheckItemReader(entityManagerFactory).getItemReader(checkDateTime);
}

@Bean
public ItemProcessor<EventEntity, Event> eventIssueCheckItemProcessor() {
    // 에러 발생부분
    // items 에서 데이터를 꺼내지 못하고 에러가 발생해버린다.
    return items -> {
        return Event.of(items.getTopic(), items.getMessage());
    };
}
```

이유가 뭘까...

내가 어디까지 해줘야 하는데??



쿼리가 제대로 실행되지 않나?

데이터가 없나?

<div data-full-width="true">

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

</div>

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

쿼리도 잘 실행 됐는데..?

데이터도 있고...



디버그를 찍어보자..

<div align="left">

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

</div>

아... ???

객체가 왜 저렇게...???



jpql 쿼리를.. 이렇게 작성한게 문제였다

```java
public JpaCursorItemReader<EventEntity> getItemReader(LocalDateTime checkDateTime) {
    return new JpaCursorItemReaderBuilder<EventEntity>().entityManagerFactory(entityManagerFactory)
            .parameterValues(Map.of("checkDateTime", checkDateTime)).queryString(
                    "select e.topic, e.message from Entity e where e.createdDate >= :checkDateTime and e.published = false")
            .name(ClassUtils.getShortName(EventIssueCheckItemReader.class)).saveState(false).build();
}
```



나는 저 두개만 필요했을뿐인데 그냥 테이블 전체를 가져와야 하나?

일단은 이렇게 해결되었다...&#x20;

```java
public JpaCursorItemReader<EventEntity> getItemReader(LocalDateTime checkDateTime) {
    return new JpaCursorItemReaderBuilder<EventEntity>().entityManagerFactory(entityManagerFactory)
            .parameterValues(Map.of("checkDateTime", checkDateTime)).queryString(
                    "select e from Entity e where e.createdDate >= :checkDateTime and e.published = false")
            .name(ClassUtils.getShortName(EventIssueCheckItemReader.class)).saveState(false).build();
}
```
