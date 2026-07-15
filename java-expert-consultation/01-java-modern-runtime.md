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

-----------------------------------------------------------------------
1. Por que uma abstração reduz a complexidade de domínio?
Uma abstração reduz a complexidade porque ela separa o "o quê" do "como". No design de software, o domínio de negócios deve ser protegido de detalhes de infraestrutura e mecanismos de entrega.


Isolamento do Core Business: Ao criar interfaces e modelos de domínio ricos, expressamos as regras de negócio em uma linguagem ubíqua (DDD), totalmente agnóstica a bancos de dados ou protocolos de comunicação.


Redução da Carga Cognitiva: Um desenvolvedor que precisa implementar uma regra de validação de crédito não deve ter que se preocupar com sessões do Hibernate, queries SQL nativas ou conexões de rede. A abstração esconde esse "ruído".


Facilidade de Substituição e Testabilidade (TDD): Se o seu domínio interage com um PaymentGateway (uma abstração), o seu código de negócio não se importa se a implementação real por trás é a AWS, uma API REST externa ou um mock em um teste unitário.

Exemplo Prático: Em vez de expor o cliente diretamente a chamadas de SDK da AWS para persistir dados no DynamoDB, criamos um OrderRepository (abstração). O domínio apenas diz: repository.save(order). Como isso acontece fisicamente (via Spring Data  ou SDK puro) é um detalhe de implementação irrelevante para a regra de negócio.

New chat
Search chats
Images
New
Library
New notebook
Projeto Java Especialist
Zup Java Senior
All notebooks
Couldn't connect
Reload
Conversation with Gemini
Crie respostas contextualizadas para estas perguntas de uma possível entrevista de java expert:



* Você sabe explicar por que uma abstração reduz complexidade de domínio.

* Você entende quando concorrência melhora throughput e quando só aumenta contenção.

* Você não confunde novidade de linguagem com ganho automático de performance.



* Você sabe explicar por que uma abstração reduz complexidade de domínio.

* Você entende quando concorrência melhora throughput e quando só aumenta contenção.

* Você não confunde novidade de linguagem com ganho automático de performance.

Com certeza! Como o seu foco aqui no Projeto Java Especialist está direcionado para arquiteturas escaláveis, microsserviços e ecossistema AWS , vamos estruturar respostas que não apenas demonstrem domínio técnico profundo, mas que também conectem esses conceitos ao dia a dia prático de um engenheiro de software especialista.

Aqui está como você pode responder a essas perguntas em uma entrevista de nível Senior/Specialist:

1. Por que uma abstração reduz a complexidade de domínio?
Uma abstração reduz a complexidade porque ela separa o "o quê" do "como". No design de software, o domínio de negócios deve ser protegido de detalhes de infraestrutura e mecanismos de entrega.


Isolamento do Core Business: Ao criar interfaces e modelos de domínio ricos, expressamos as regras de negócio em uma linguagem ubíqua (DDD), totalmente agnóstica a bancos de dados ou protocolos de comunicação.


Redução da Carga Cognitiva: Um desenvolvedor que precisa implementar uma regra de validação de crédito não deve ter que se preocupar com sessões do Hibernate, queries SQL nativas ou conexões de rede. A abstração esconde esse "ruído".


Facilidade de Substituição e Testabilidade (TDD): Se o seu domínio interage com um PaymentGateway (uma abstração), o seu código de negócio não se importa se a implementação real por trás é a AWS, uma API REST externa ou um mock em um teste unitário.


Exemplo Prático: Em vez de expor o cliente diretamente a chamadas de SDK da AWS para persistir dados no DynamoDB, criamos um OrderRepository (abstração). O domínio apenas diz: repository.save(order). Como isso acontece fisicamente (via Spring Data  ou SDK puro) é um detalhe de implementação irrelevante para a regra de negócio.

2. Quando a concorrência melhora o throughput e quando ela só aumenta a contenção?
A regra de ouro da concorrência é: concorrência não faz o código rodar mais rápido; ela permite que você faça mais coisas ao mesmo tempo.

Concorrência melhora o Throughput quando:

O gargalo é I/O Bound: O sistema passa muito tempo esperando por respostas de rede, consultas ao Postgres/SQL Server, leitura no S3 ou chamadas de microsserviços. Enquanto uma thread espera o I/O bloquear, outras threads processam requisições.


Uso de Virtual Threads (Java 17+): Com as Virtual Threads, o custo de bloqueio de I/O despenca. Podemos escalar o throughput massivamente porque o scheduler do Java desvincula a thread virtual da thread do sistema operacional quando ocorre um bloqueio de I/O.

Tarefas são CPU-Bound e independentes: Se você tem múltiplos cores de CPU livres e precisa processar grandes volumes de dados isolados (ex: processar imagens ou cálculos complexos em paralelo usando o ForkJoinPool).

Concorrência só gera Contenção quando:
Há disputa por recursos compartilhados (Locks): Se muitas threads tentam ler e atualizar o mesmo estado mutável ou recurso limitado (ex: um Connection Pool de banco de dados sobrecarregado ou blocos synchronized), o tempo gasto gerenciando travas e trocas de contexto supera o ganho da execução paralela.

Context Switching excessivo: Em modelos tradicionais de concorrência (Platform Threads), criar milhares de threads para tarefas muito curtas consome muita memória e tempo de CPU apenas alternando as threads no processador.


Exemplo Prático: Em uma arquitetura de mensageria com Kafka, aumentar o número de consumidores concorrentes em um Consumer Group melhora o throughput até o limite do particionamento do tópico. Se você tiver mais consumidores do que partições, ou se todos eles baterem na mesma tabela do Postgres travando linhas via optimistic locking, você terá apenas contenção e lentidão.

3. Não confundir novidade de linguagem com ganho automático de performance

Um erro clássico de desenvolvedores menos experientes é assumir que utilizar a sintaxe mais recente do Java (como Java 17 ou superior ) ou adotar novos paradigmas trará otimizações de performance automáticas sem medir o contexto.

Legibilidade =/ Velocidade: Recursos como Records, Pattern Matching e Switch Expressions foram introduzidos principalmente para aumentar a produtividade do desenvolvedor, reduzir o boilerplate e aumentar a segurança de tipos. 
Eles não alteram a velocidade de execução da JVM de forma milagrosa.

O perigo das Streams: A API de Streams (introduzida no Java 8, mas amplamente usada no 17+ ) é excelente para legibilidade. 
No entanto, para coleções pequenas ou loops simples, um for tradicional geralmente performa melhor e aloca menos objetos na memória do que uma pipeline de Stream com lambdas.

Virtual Threads não aceleram CPU: Como mencionado, as Virtual Threads (Java 21+) evitam o desperdício de threads físicas em bloqueios de I/O, mas se o seu microsserviço realiza cálculos matemáticos pesados (CPU-bound), usar Virtual Threads não trará ganho algum — na verdade, pode até piorar a performance devido ao overhead de agendamento.

Mentalidade de Especialista: Novidades de linguagem resolvem problemas de design, arquitetura, expressividade e manutenibilidade. O ganho real de performance em sistemas corporativos de alta escala quase sempre vem de bom design de algoritmos, modelagem de banco de dados eficiente (indexação de SQL ), caching estratégico (Redis ) e arquitetura assíncrona (Kafka/SQS ).

