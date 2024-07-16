---
description: Event Driven Architecture
---

# EDA ( 이벤트 기반 아키텍처 )

## 이벤트 기반 아키텍처란?

먼저 이벤트가 무엇인지 알고가자

간단히 얘기하면 상태의 변화를 이벤트라고 할 수 있다.

* 회원 정보 변경
* 상품의 가격 변경
* 재입고 알림 요청

위와 같이 데이터가 변경 , 등록 등을 이벤트라고 할 수 있는것이다.



### 이벤트 기반 구조의 개발을 왜 해야 하는지?

가장 큰 목적은 서비스간의 느슨한 결합을 위해 한다.

그래서 MSA 와의 궁합이 좋다고 할 수 있다.



### 이벤트 기반 구조를 사용한 이유

기존 레거시를 MSA로 옮기면서 서비스간의 결합도를 충분히 나누지 못했음

정산 시스템을 새로 개발 해야 하는 상황이고 기존 주문 로직과 연결하려 보니

서비스간의 문제가 발생할 가능성이 있어보임.

해서 주문 완료 이벤트를 기점으로 주문 알림과 정산 데이터를 쌓는 서비스를 하나씩 만들어 넣기위함.



### 이벤트 발행

그럼 주문완료 이벤트를 발행하여 전달해줄 Message Broker 를 어떤걸 사용할까

AWS 의 SQS를 이미 사용하고 있는 서비스가 있기 때문에

SNS / SQS 조합을 이용해서 이벤트를 발행/구독 서비스를 구성한다.

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

주문 서비스에서는 주문완료 이벤트가 발생하면 주문완료 토픽으로 SNS에 메세지를 발행한다.

SNS에서는 주문완료 토픽을 구독하고 있는 SQS 로 메세지를 전부 전달해준다.

SQS 에서는 메세지를 전달받아 이벤트를 처리한다.



### 이벤트 발행을 보장할 수 있는지?? <a href="#undefined" id="undefined"></a>

1. 이벤트 트랜잭션 안에서 메세지 발행

– 이벤트 발생과 발행이 함께하기 때문에 보장이 되지만 메세지 시스템의 장애가 서비스의 장애로 번질 수 있음.

2. 이벤트 트랜잭션 밖에서 메세지 발행

– 1번의 문제는 해결이 되지만 메세지 발행이 보장되지 않음.



### Transactional Outbox Pattern <a href="#transactional-outbox-pattern" id="transactional-outbox-pattern"></a>

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

Transactional Outbox Pattern 은 이벤트에 대한 상태를 OUTBOX 테이블에 저장하고

Relay 배치에서 새로 저장된 데이터를 발행하는 구현방식

&#x20;

하지만 이벤트 발행을 배치에 의존하고 싶진 않았기 때문에

이벤트 발행과 이벤트 저장을 동시에 진행하고 싶어서 찾아봄.

<figure><img src="../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

주문 완료 상태변경 transaction 과 이벤트 저장 transaction 을 묶어서 이벤트의 발행을 보장할 수 있다.

물론 이벤트 저장이 실패하면 롤백이 되어야 한다는 리스크가 존재하지만 내부시스템으로 인한 롤백이고

이벤트 재발행에 대한 문제도 해결할 수 있으니 장점이 더 많은 리스크 라고 할 수 있음.



<figure><img src="../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

\
이벤트 발행 시 저장소에 발행여부를 false 로 저장하고

이벤트 기록 전용 consumer 에서 true 처리를 하는것이 기본 프로세스.

```sql
create table event_record
(
    event_id bigint auto_increment not null primary key comment '이벤트 번호',
    orderNo varchar(32) not null comment '주문번호',
    topic varchar(20) not null comment '토픽',
    message text not null comment '이벤트 메세지',
    published char(1) not null comment '발행 여부',
    published_date datetime null comment '발행 확인 시간',
    created_date datetime not null comment '이벤트 생성 시간'
);
```



### 이벤트 발생과 발행 로직을 보자 <a href="#undefined" id="undefined"></a>

```java
@Transactional
public void updatePayDone(UpdatePayDone updatePayDone) {
    // 주문완료 이벤트 발생
    orderService.updatePayDone(updatePayDoneCommand.getOrderNum());

    // 주문완료 내부 이벤트 발행
    OrderEvent event = new OrderEvent(updatePayDone.getOrderNum(), SnsTopic.ORDER_PAY_DONE);
    applicationEventPublisher.publishEvent(event);
}
```

```java
@EventListener
@Transactional
// 주문 완료 이벤트 트랜잭션과 같이 묶여 있어야 한다.
public void saveOrderEvent(OrderEvent orderEvent) {
        EventRecordEntity eventRecordEntity = EventRecordEntity.of(orderEvent);
        orderEventRepository.save(eventRecordEntity);
}
```

```java
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
// 트랜잭션이 정상적으로 커밋이 되고 이벤트를 발행하도록 한다.
public void publish(OrderEvent orderEvent) {
    snsPublisher.convertAndSend(orderEvent.getTopic(), orderEvent);
}
```



### 이벤트 재발행 <a href="#undefined" id="undefined"></a>

그 후 Message Broker 의 재전송 주기에 맞춰서 실행되는 이벤트 발행 감지 배치를 하나 생성해서

이벤트 저장소에 있는 발행여부가 false 인 상태가 지속되는 이벤트를 발행 해주도록 한다.

SQS 는 기본적으로 5분정도의 재 발송 기능이 존재하기 때문에

배치 주기를 5분으로 설정해 놓았다.



