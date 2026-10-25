# Projeto de Arquitetura: Implementação do Padrão Saga em E-commerce Distribuído

## 1. Visão Geral e Contexto do Negócio

Durante a migração do ecossistema monolítico do nosso e-commerce para uma arquitetura distribuída baseada em microsserviços, deparámo-nos com um desafio fundamental: a impossibilidade de utilizar transações ACID tradicionais entre bases de dados independentes. 

No cenário atual, a compra envolve três domínios desacoplados:
* **Order Service**: Responsável pelo registo e gestão do ciclo de vida dos pedidos.
* **Inventory Service**: Responsável pelo controlo de stock e reservas temporárias.
* **Payment Service**: Responsável pelo processamento financeiro com gateways externos.

Para garantir a consistência final dos dados sem comprometer o desempenho e a disponibilidade do sistema, implementámos o **Padrão Saga**. Desta forma, cada serviço executa as suas operações locais e, em caso de falha numa etapa posterior, são acionadas transações compensatórias automáticas para reverter o estado do sistema.

---

## 2. Decisão Arquitetural: Saga por Orquestração

Para este projeto, optámos expressamente pela **Saga Baseada em Orquestração** em detrimento da Coreografia. 

### Justificativa da Escolha:
* **Centralização do Fluxo e Menor Acoplamento Cíclico**: Como o processo de compra envolve passos críticos que podem expandir no futuro (ex.: faturação, transporte, notificações), a orquestração evita que os serviços precisem de conhecer o fluxo completo. O controlo do estado reside unicamente no orquestrador.
* **Clareza nas Transações Compensatórias**: Em cenários de falha, o orquestrador sabe exatamente quais os passos que foram concluídos com sucesso e quais necessitam de ser desfeitos, executando as chamadas de compensação na ordem inversa de forma determinística.
* **Monitorização e Observabilidade**: A tabela de estado do orquestrador permite identificar instantaneamente em que etapa um pedido se encontra ou onde ocorreu um eventual problema.

---

## 3. Estrutura do Repositório

O repositório encontra-se organizado da seguinte forma:

```text
.
├── README.md                           # Documentação executiva do projeto
├── diagrams/                           # Desenhos arquiteturais do sistema
│   ├── saga-architecture-overview.png  # Visão geral da arquitetura e mensageria
│   ├── sequence-success-flow.png       # Fluxo feliz (sucesso)
│   └── sequence-compensation-flow.png  # Fluxo de falha e compensação
└── docs/                               # Documentação técnica detalhada
    └── decisoes-arquiteturais.md       # Análise de riscos, resiliência e padrões
