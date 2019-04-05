# spring-scalable-example

Application level에서 대규모 트래픽 감당 예제

## 사용기술
- Spring Cloud Gateway
- Spring WebFlux, Reactor
- RabbitMQ
- Redis

## 사전준비
RabbitMQ 동작을 위해 **Erlang** 필요.

RabbitMQ, Redis는 Gateway에 Embedded로 제공. 단, 이미 두 환경이 구축된 경우에는 해당 `Config`에서 `@Configuration`만 주석

## 요구사항
- 대규모 트래픽 가정
- 제한된 수의 상품을 팜
- 실패했던 성공했던 구매에 시도한 사람의 기록을 남겨야함

### 주의 사항
- Race condition (제한된 개수만큼만 판매)
- 대용량 트래픽에 대응하고, 정보를 남겨야함

ex) 맥북프로 최상위 옵션을 100만원에 100대만 판매. 들어올 트래픽은 100만으로 짐작됨.

## 핵심로직
```java
public Mono<Boolean> apply(Long userId) {
    return this.reactiveValueOperations.get(EVENT_APPLY_KEY)
            .doOnNext(size -> this.rabbitTemplate.convertAndSend(MicroserviceApplication.EVENT_TOPIC, "foo.bar.baz", userId))
            .filter(this::isPurchase)
            .flatMap(s -> this.reactiveValueOperations.increment(EVENT_APPLY_KEY))
            .flatMap(s -> this.reactiveListOperations.leftPush(EVENT_APPLY_LIST, userId.toString()))
            .map(add -> true)
            .defaultIfEmpty(false);
}
```