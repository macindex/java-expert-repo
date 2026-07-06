# Spring, JPA e performance

## O que revisar

Entenda o ciclo de vida do Persistence Context, o efeito N+1, o uso de Join Fetch, EntityGraph, paginação e versionamento otimista.

## Sinais que o entrevistador procura

* Você diagnostica latência com foco em I/O, não só em CPU.
* Você evita resolver N+1 com `EAGER` indiscriminado.
* Você sabe reduzir o escopo transacional para não prender conexão à toa.

## Trechos de código úteis

```java
@Query("select distinct o from Order o join fetch o.items where o.id = :id")
Optional<Order> findByIdWithItems(@Param("id") Long id);
```

```java
@EntityGraph(attributePaths = {"items", "customer"})
List<Order> findByStatus(OrderStatus status);
```

```java
@Entity
public class Account {
    @Id
    private Long id;

    @Version
    private Long version;

    private BigDecimal balance;
}
```

```java
@Transactional
public void debit(Long accountId, BigDecimal amount) {
    var account = accountRepository.findByIdForUpdate(accountId)
            .orElseThrow();

    account.setBalance(account.getBalance().subtract(amount));
}
```

## Como explicar na entrevista

Se houver lentidão sob baixa CPU, descreva a hipótese de espera por banco ou lock. Depois mostre como você investigaria query plan, saturação do pool, tempo de execução por query e volume de consultas geradas pelo Hibernate.

## Perguntas para treinar

* Quando usar `Join Fetch` e quando preferir projeção?
* Em que caso lock pessimista faz sentido?
* Qual é o custo de deixar transações abertas por muito tempo?
