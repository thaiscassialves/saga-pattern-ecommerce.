# Decisões Arquiteturais e Mitigação de Riscos (ADR)

## 1. Introdução
Este documento detalha as diretrizes de engenharia, os padrões de resiliência e as estratégias de mitigação de riscos adotadas na arquitetura de microsserviços do e-commerce, com foco particular na implementação do **Padrão Saga por Orquestração**.

## 2. Riscos Mitigados e Estratégias de Resiliência

### 2.1. Falhas de Rede e Transitórias (Retry com Exponential Backoff)
Em ambientes distribuídos, falhas de rede temporárias entre o Orquestrador e os microsserviços participantes (*Inventory Service* e *Payment Service*) são esperadas.
* **Mitigação:** Implementação de políticas de nova tentativa (*Retry*) com espaçamento exponencial e *jitter* aleatório. Isto evita o efeito de tempestade de tráfego (*thundering herd*) no barramento de mensagens.

### 2.2. Esgotamento de Recursos e Falhas em Cascata (Circuit Breaker)
O acoplamento com gateways de pagamento externos ou serviços de terceiros apresenta riscos de latência elevada ou indisponibilidade.
* **Mitigação:** Aplicação do padrão **Circuit Breaker** nas chamadas críticas. Se o *Payment Service* ou o gateway externo registarem uma taxa de erro elevada, o circuito abre-se temporariamente, falhando de imediato pedidos futuros e poupando recursos do sistema até que o serviço recupere.

### 2.3. Mensagens Duplicadas e Reentências (Idempotent Consumer)
Devido a retransmissões automáticas no barramento de mensagens, um comando (como `ReserveStock`) pode ser entregue mais do que uma vez.
* **Mitigação:** Todos os serviços participantes implementam o padrão de Consumidor Idempotente. Cada transação possui um identificador único (`Saga-ID` e `Correlation-ID`). O estado é validado antes de qualquer execução de escrita na base de dados local.

### 2.4. Timeouts e Consistência Eventual
Caso um microsserviço demore excessivamente a responder, o sistema não pode ficar bloqueado indefinidamente à espera de uma resposta.
* **Mitigação:** O orquestrador mantém um monitor de prazos limite (*Timeout Monitor*). Se o tempo limite de resposta para uma etapa for excedido, o orquestrador assume o falhanço e aciona de imediato a cadeia de transações compensatórias.

## 3. Tratamento de Erros e Dead-Letter Queues (DLQ)
* Mensagens que falhem repetidamente após esgotarem todas as tentativas de reprocessamento (*max retries*) não são descartadas nem mantidas em ciclo infinito.
* São encaminhadas automaticamente para uma **Dead-Letter Queue (DLQ)** dedicada. A equipa de engenharia dispõe de ferramentas de monitorização para inspecionar o payload, corrigir eventuais inconsistências de dados e reprocessar a mensagem manualmente quando o problema técnico for resolvido.
