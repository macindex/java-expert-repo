# Mensageria e observabilidade

## O que revisar

Kafka, filas, semântica at-least-once, consumidores idempotentes, correlação de logs, métricas, traces distribuídos e propagação de contexto.

## Sinais que o entrevistador procura

* Você sabe rastrear uma falha ponta a ponta.
* Você entende duplicidade de mensagens como risco de negócio, não apenas técnico.
* Você fala de observabilidade como requisito de operação, não como detalhe de implementação.

## Trechos de código úteis

```java
@KafkaListener(topics = "payments.created", groupId = "notification-service")
public void consume(ConsumerRecord<String, PaymentCreatedEvent> record) {
    var event = record.value();

    if (processedEventRepository.existsById(event.eventId())) {
        return;
    }

    notificationService.sendSms(event.customerPhone(), event.message());
    processedEventRepository.save(new ProcessedEvent(event.eventId()));
}
```

```java
public class CorrelationIdFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        var correlationId = Optional.ofNullable(request.getHeader("X-Correlation-Id"))
                .orElse(UUID.randomUUID().toString());

        MDC.put("correlationId", correlationId);
        response.setHeader("X-Correlation-Id", correlationId);

        try {
            filterChain.doFilter(request, response);
        } finally {
            MDC.remove("correlationId");
        }
    }
}
```

```java
log.info("payment processed orderId={} correlationId={}", orderId, correlationId);
```

## Como explicar na entrevista

Fale sobre o caminho de investigação: métrica, trace, log e estado do broker. Se houver duplicidade, explique a semântica de entrega e por que a proteção precisa existir na aplicação. Se houver falta de rastreabilidade, mostre como um correlation id e um trace context resolvem o problema.

## Perguntas para treinar

* Como você evita duplicidade em consumidores Kafka?
* O que você faz quando o time não consegue correlacionar logs e eventos?
* Quais métricas mínimas você exigiria em um serviço crítico?
