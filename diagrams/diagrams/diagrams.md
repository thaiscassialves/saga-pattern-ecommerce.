# Diagramas de Arquitetura da Saga

Abaixo encontram-se representados os diagramas arquiteturais e de sequência do sistema, estruturados através de representação visual suportada por código.

## 1. Visão Geral da Arquitetura
```mermaid
graph TD
    Client -->|HTTP POST| Gateway[API Gateway]
    Gateway --> OrderSvc[Order Service / Orquestrador]
    
    subgraph Barramento de Mensagens (Broker)
        Broker[RabbitMQ / Kafka]
    end

    OrderSvc -->|Comandos| Broker
    Broker --> InventorySvc[Inventory Service]
    Broker --> PaymentSvc[Payment Service]
    
    InventorySvc --> DB1[(BD Stock)]
    PaymentSvc --> DB2[(BD Pagamentos)]
    OrderSvc --> DB3[(BD Pedidos & Saga)]

sequenceDiagram
    autonumber
    actor Client
    participant Order as Order Service (Orquestrador)
    participant Inventory as Inventory Service
    participant Payment as Payment Service

    Client->>Order: POST /orders (Criar Pedido)
    Note over Order: Regista estado PENDING
    Order->>Inventory: Command: Reserve Stock
    Inventory-->>Order: Event: Stock Reserved (OK)
    Order->>Payment: Command: Process Payment
    Payment-->>Order: Event: Payment Approved (OK)
    Note over Order: Atualiza estado para CONFIRMED
    Order-->>Client: 201 Created (Pedido Confirmado)

sequenceDiagram
    autonumber
    actor Client
    participant Order as Order Service (Orquestrador)
    participant Inventory as Inventory Service
    participant Payment as Payment Service

    Client->>Order: POST /orders
    Order->>Inventory: Command: Reserve Stock
    Inventory-->>Order: Event: Stock Reserved (OK)
    Order->>Payment: Command: Process Payment
    Payment-->>Order: Event: Payment Failed (Recusado)
    
    Note over Order: INÍCIO DA COMPENSAÇÃO (Rollback)
    Order->>Inventory: Command: Release Stock (Compensação)
    Inventory-->>Order: Event: Stock Released
    
    Note over Order: Atualiza estado para CANCELLED
    Order-->>Client: 422 Unprocessable Entity
