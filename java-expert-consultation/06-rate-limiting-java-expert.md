# Rate Limiting para Java Software Engineer Expert

## Visao geral

Rate Limiting e um padrao de resiliencia para limitar a taxa de requisicoes por cliente em uma janela de tempo. O objetivo principal e proteger APIs contra abuso, preservar recursos compartilhados e manter previsibilidade de latencia para todos os consumidores.

Em entrevistas de nivel Expert, o tema normalmente aparece junto com:

- resiliencia e estabilidade de APIs publicas;
- protecao de recursos de infraestrutura (CPU, memoria, pool de conexoes, filas);
- arquitetura distribuida e consistencia de contadores entre multiplas instancias;
- experiencia do cliente ao receber status 429 e cabecalhos informativos.

## Por que o uso sem limite e perigoso

Sem controle de taxa, um unico cliente pode monopolizar a API e provocar:

- aumento de latencia para os demais usuarios;
- falhas em cascata por exaustao de recursos;
- queda parcial ou total dos servicos (downtime).

Em termos de engenharia, isso costuma degradar primeiro os componentes mais sensiveis a burst: pool JDBC, thread pools, dependencias externas e fila de processamento.

## Onde aplicar o Rate Limiting

A limitacao pode existir em mais de uma camada:

1. API Gateway: controle global de borda e protecao inicial.
2. Cliente consumidor: autocontrole para evitar estouros e retries agressivos.
3. Aplicacao servidora (Spring Boot): controle por regra de negocio e identidade.

Para Java Expert, o ponto chave e combinar camadas, nao depender de apenas uma.

## Como funciona internamente

Um algoritmo classico e o Fixed Window (janela fixa):

- define uma janela de tempo fixa (exemplo: 1 minuto);
- define capacidade maxima por identificador (exemplo: 10 requisicoes);
- ao estourar a cota, bloqueia novas chamadas ate a proxima janela.

Quando o limite e excedido, a API deve responder com:

- HTTP 429 Too Many Requests.

## Ambientes distribuidos: necessidade de cache compartilhado

Em cluster com varias instancias da API, limitar localmente por processo e insuficiente. Um cliente pode alternar entre nos e burlar o limite real.

A solucao e armazenar o estado do consumo em cache distribuido para sincronizar contadores em tempo quase real.

### Por que evitar banco relacional para o contador de janela

- latencia maior para operacoes de altissima frequencia;
- lock/contention em picos;
- custo alto para leitura/escrita de contador em cada request.

### Alternativas praticas

- Hazelcast
- Redis

Ambos trabalham em memoria e sao adequados para baixa latencia em cenarios de alto throughput.

## Implementacao pratica no Spring Boot (Bucket4j + Hazelcast)

Cenario: API de mensagens.

- Usuario anonimo: pode apenas consultar mensagens (GET).
- Usuario autenticado: pode criar mensagens (POST).

### 1) Dependencias no pom.xml

```xml
<dependency>
    <groupId>com.giffing.bucket4j.spring.boot.starter</groupId>
    <artifactId>bucket4j-spring-boot-starter</artifactId>
    <version>0.15.2</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast-spring</artifactId>
    <version>5.1.1</version>
</dependency>
```

### 2) Configuracao do Hazelcast (src/main/resources/hazelcast.xml)

```xml
<hazelcast xmlns="http://www.hazelcast.com/schema/config"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.hazelcast.com/schema/config
                               http://www.hazelcast.com/schema/config/hazelcast-config-5.1.xsd">
    <map name="mensagemRateLimit">
    </map>
</hazelcast>
```

Tambem habilite cache na aplicacao:

```java
@EnableCaching
@SpringBootApplication
public class ApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiApplication.class, args);
    }
}
```

### 3) Configuracao do Bucket4j no application.yml

```yaml
spring:
  cache:
    type: jcache
    provider: com.hazelcast.client.impl.cache.HazelcastClientCachingProvider

bucket4j:
  enabled: true
  filters:
    - cache-name: mensagemRateLimit
      url: /mensagens.*
      filter-order: -1000
      rate-limits:
        - execute-condition: "@segurancaService.isAnonimo()"
          expression: "getRemoteAddr()"
          bandwidths:
            - capacity: 5
              time: 1
              unit: minutes

        - execute-condition: "!@segurancaService.isAnonimo()"
          expression: "@segurancaService.getUsuarioAutenticado()"
          bandwidths:
            - capacity: 10
              time: 1
              unit: minutes
```

Nota tecnica sobre filtro: em alguns cenarios, o Bucket4j pode executar antes do contexto do Spring Security estar disponivel. Ajustar filter-order para rodar apos a cadeia de seguranca evita acesso nulo ao usuario autenticado.

## Validacao de comportamento e cabecalhos HTTP

Ao aplicar o filtro, os cabecalhos ajudam o cliente a se autorregular:

- X-Rate-Limit-Remaining: restante da cota na janela atual.
- X-Rate-Limit-Retry-After-Seconds: segundos para tentar novamente apos bloqueio.

Quando a cota estoura:

- resposta HTTP 429 Too Many Requests.

## Como provar distribuicao em laboratorio

1. Suba duas instancias da API (exemplo: 8080 e 8081).
2. Gere chamadas alternando entre as duas portas para o mesmo identificador.
3. Verifique que o consumo e compartilhado entre instancias.
4. Confirme que, ao exceder a cota em uma instancia, o bloqueio vale para as demais.

## O que um entrevistador Expert espera ouvir

- Qual identidade usar para chave de limite (IP, usuario, client_id, tenant).
- Como evitar bypass em ambiente distribuido.
- Como responder com 429 e cabecalhos para UX de integracao.
- Como escolher capacidade/janela por tipo de consumidor e endpoint.
- Como combinar Rate Limiting com Retry, Circuit Breaker e observabilidade.

## Trade-offs importantes

- Limite muito baixo: protege infraestrutura, mas penaliza clientes legitimos.
- Limite muito alto: melhora UX imediata, mas aumenta risco de saturacao.
- Chave por IP: simples, mas pode ser injusta atras de NAT/proxy.
- Chave por usuario/client_id: mais precisa, porem depende de autenticacao confiavel.

## Checklist rapido para producao

- Definir politica por endpoint e perfil de cliente.
- Garantir armazenamento distribuido do estado.
- Configurar resposta 429 e cabecalhos padrao.
- Instrumentar metricas (bloqueios por rota, por cliente e por janela).
- Criar alertas para aumentos de 429 e quedas de throughput.
- Testar comportamento sob burst e em cenarios multi-instancia.
