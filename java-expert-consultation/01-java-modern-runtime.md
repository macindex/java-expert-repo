# Java moderno e concorrência

## O que revisar

Foque em Java 17+ com Records, Sealed Classes, Pattern Matching, Streams com intenção clara e modelos de concorrência que você consiga defender em produção.

## Sinais que o entrevistador procura

* Você sabe explicar por que uma abstração reduz complexidade de domínio.
* Você entende quando concorrência melhora throughput e quando só aumenta contenção.
* Você não confunde novidade de linguagem com ganho automático de performance.

## Trechos de código úteis

```java
public record Money(BigDecimal amount, Currency currency) {
    public Money {
        Objects.requireNonNull(amount);
        Objects.requireNonNull(currency);
    }

    public Money add(Money other) {
        if (!currency.equals(other.currency())) {
            throw new IllegalArgumentException("Currencies must match");
        }
        return new Money(amount.add(other.amount()), currency);
    }
}
```

```java
public sealed interface PaymentCommand permits AuthorizePayment, CapturePayment {}

public record AuthorizePayment(String orderId, BigDecimal amount) implements PaymentCommand {}
public record CapturePayment(String paymentId) implements PaymentCommand {}
```

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    var futures = requests.stream()
            .map(request -> executor.submit(() -> paymentClient.authorize(request)))
            .toList();

    for (var future : futures) {
        future.get();
    }
}
```

## Como explicar na entrevista

Descreva o motivo da escolha. Por exemplo, Records ajudam em objetos de valor imutáveis; Sealed Classes limitam hierarquias inválidas; virtual threads são boas quando a aplicação é bloqueante, mas não eliminam gargalos de banco, fila ou rede.

## Perguntas para treinar

* Quando uma record é melhor que uma classe tradicional?
* Qual o risco de usar virtual threads sem revisar o acesso ao banco?
* Como você justificaria um tipo sealed em um domínio financeiro?
