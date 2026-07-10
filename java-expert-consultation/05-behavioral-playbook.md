# Roteiro comportamental e narrativa de impacto

## O que revisar

Prepare histórias reais com foco em contexto, decisão, trade-off, impacto e aprendizado. Em nível expert, o entrevistador quer ver liderança técnica sem cargo formal e capacidade de conduzir incidentes.

## Estrutura recomendada

Use a forma curta:

* Contexto
* Problema
* Sua ação
* Resultado
* O que faria diferente

## Modelo de resposta para incidentes

```text
1. Identifiquei o sintoma principal e o impacto para o negócio.
2. Cortei hipóteses pouco prováveis e priorizei o gargalo dominante.
3. Apliquei a contenção mais segura para restaurar o serviço.
4. Registrei evidências para pós-incidente e prevenção estrutural.
```

## Modelo de resposta para decisão técnica impopular

```text
1. Mostrei o problema com dados.
2. Comparei alternativas e seus riscos.
3. Propus uma decisão com custo e benefício explícitos.
4. Combinei acompanhamento com métrica clara.
```

## Perguntas para treinar

* Conte um incidente que você liderou de ponta a ponta.
* Qual decisão técnica você sustentou mesmo com resistência?
* Como você equilibra velocidade de entrega e qualidade?
* Como você orienta pessoas menos experientes sem microgerenciar?

## Exemplos STAR para entrevista Java Expert

### 1) Conte um incidente que você liderou de ponta a ponta

**S (Situation):** Em uma sexta-feira de fechamento financeiro, a API de pagamentos apresentou picos de latencia e erro 5xx. O impacto foi de 18% das transações com timeout por 22 minutos.

**T (Task):** Como referencia tecnica do squad, precisei restaurar a estabilidade rapidamente e reduzir risco de reincidencia no mesmo ciclo de faturamento.

**A (Action):** Conduzi a sala de incidente, centralizei comunicacao com produto e SRE, identifiquei saturacao no pool JDBC e efeito N+1 em endpoint critico. Apliquei mitigacao imediata com feature flag para reduzir carga, limitei concorrencia no endpoint afetado e publiquei hotfix com Join Fetch e ajuste de timeout.

**R (Result):** Restabelecemos o servico em 14 minutos apos a mitigacao. Na semana seguinte, reduzimos o p95 de 1.9s para 280ms e zeramos o erro de timeout no fluxo principal. Documentei post-mortem com plano de prevenção e dono por ação.

### 2) Qual decisao tecnica voce sustentou mesmo com resistencia

**S (Situation):** O time queria migrar rapidamente um fluxo sincrono para assincromo sem redesenhar idempotencia, para cumprir prazo comercial.

**T (Task):** Eu precisava evitar risco de duplicidade financeira e defender uma entrega segura sem bloquear totalmente o roadmap.

**A (Action):** Apresentei dados de incidentes anteriores, modelei opcoes com trade-offs e propus rollout em duas etapas: primeiro outbox + consumidor idempotente, depois otimizar throughput. Defini metricas de sucesso e alertei sobre risco regulatorio caso houvesse reprocessamento sem controle.

**R (Result):** A proposta foi aceita. Entramos em producao sem duplicidade de cobranca e com 37 por cento de ganho de throughput em 30 dias. O modelo virou padrao para novos fluxos de evento no dominio.

### 3) Como voce equilibra velocidade de entrega e qualidade

**S (Situation):** Em lancamento de parceria, havia pressao para liberar API em duas semanas com requisitos ainda evoluindo.

**T (Task):** Entregar no prazo sem comprometer confiabilidade minima de um servico critico de autorizacao.

**A (Action):** Negociei escopo com produto usando matriz de risco, separei requisitos obrigatorios de seguranca/observabilidade, implementei testes de contrato e smoke test no pipeline, e deixei funcionalidades menos criticas atras de feature flag.

**R (Result):** Entrega no prazo, sem rollback em producao e sem incidente severo no primeiro mes. O lead time caiu 22 por cento nas entregas seguintes porque padronizamos o processo de release seguro.

### 4) Como voce orienta pessoas menos experientes sem microgerenciar

**S (Situation):** O time cresceu e dois desenvolvedores juniores passaram a atuar em servicos de alta criticidade.

**T (Task):** Elevar autonomia tecnica do time preservando qualidade de codigo e tempo de resposta a incidentes.

**A (Action):** Criei trilha de onboarding por contexto de negocio, pair programming com rotacao semanal, templates de PR com criterios claros e sessoes curtas de revisao de arquitetura. Em vez de dizer como fazer, eu alinhava objetivo, restricoes e sinais de qualidade esperados.

**R (Result):** Em dois meses, o tempo medio de revisao de PR caiu 35 por cento e os juniores passaram a resolver chamados de baixa e media complexidade sem escalacao. O time ganhou velocidade sem perda de padrao tecnico.

## Dicas para personalizar os exemplos

* Troque os numeros pelos seus dados reais (latencia, taxa de erro, tempo de restauracao).
* Nomeie tecnologias que voce realmente usou (Spring Boot, Kafka, Redis, Kubernetes, AWS).
* Termine cada historia com aprendizado e mudanca de processo.
* Mantenha cada resposta entre 60 e 120 segundos.

## Dica final

Não tente soar perfeito. Em posições expert, é melhor mostrar clareza, método e maturidade do que uma resposta decorada.
