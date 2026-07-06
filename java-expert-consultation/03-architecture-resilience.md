# Arquitetura, resiliência e idempotência

## O que revisar

Estude Clean Architecture, DDD, event-driven, retry com backoff, timeout, circuit breaker, bulkhead, idempotência, outbox e DLQ.

## Sinais que o entrevistador procura

* Você entende que resiliência não é só tratar exceção.
* Você sabe diferenciar falha transitória de falha definitiva.
* Você consegue justificar compensação, retry e fallback com base no domínio.

## Trechos de código úteis

```java
@CircuitBreaker(name = "fraudService", fallbackMethod = "fallbackAuthorize")
@Retry(name = "fraudService")
@TimeLimiter(name = "fraudService")
public CompletableFuture<AuthorizationResult> authorize(PaymentRequest request) {
    return CompletableFuture.supplyAsync(() -> fraudClient.authorize(request));
}

private CompletableFuture<AuthorizationResult> fallbackAuthorize(PaymentRequest request, Throwable throwable) {
    return CompletableFuture.completedFuture(AuthorizationResult.pending(request.orderId()));
}
```

```java
@Transactional
public void handleEvent(PaymentEvent event) {
    if (idempotencyRepository.existsByEventId(event.eventId())) {
        return;
    }

    paymentService.process(event);
    idempotencyRepository.save(new ProcessedEvent(event.eventId()));
}
```

```java
public interface OrderIntegrationService {
    void publish(OrderCreatedEvent event);
}

@Service
class KafkaOrderIntegrationService implements OrderIntegrationService {
    @Override
    public void publish(OrderCreatedEvent event) {
        kafkaTemplate.send("orders.created", event.orderId(), event);
    }
}
```

## Como explicar na entrevista

Mostre o trade-off entre disponibilidade, consistência e custo operacional. Em nível expert, a resposta boa não é só aplicar padrões; é explicar por que uma estratégia reduz impacto, onde ela falha e como você observou isso em produção.

## Perguntas para treinar

* Quando retry piora um incidente em vez de ajudar?
* Como você desenha idempotência em um fluxo assíncrono?
* Qual a diferença prática entre fallback e compensação?
