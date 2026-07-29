# Shopping - E-commerce com Microsserviços (.NET 6)

Aplicação de e-commerce construída em .NET 6 seguindo uma arquitetura de microsserviços, com comunicação síncrona via API Gateway e comunicação assíncrona via mensageria, autenticação centralizada e persistência isolada por serviço (database-per-service).

## Arquitetura

```
                                   ┌──────────────────┐
                                   │   Shopping.Web    │  (Frontend MVC)
                                   └─────────┬─────────┘
                                             │
                                   ┌─────────▼─────────┐
                                   │  API Gateway       │  (Ocelot)
                                   └─────────┬─────────┘
              ┌──────────────┬───────────────┼───────────────┬──────────────┐
              │              │               │               │              │
        ┌─────▼────┐  ┌──────▼─────┐  ┌──────▼─────┐  ┌──────▼─────┐ ┌──────▼─────┐
        │ Product  │  │   Cart      │ │  Coupon    │  │    Order   │ │   Payment  │
        │   API    │  │    API      │ │    API     │  │     API    │ │     API    │
        └─────┬────┘  └──────┬─────┘  └──────┬─────┘  └──────┬─────┘ └──────┬─────┘
              │              │               │               │              │
          [MySQL]        [MySQL]         [MySQL]       ┌───────▼──────┐  [MySQL]
                                                       │  RabbitMQ    │◄────┘
                                                       │ (MessageBus) │
                                                       └───────┬──────┘
                                                       ┌───────▼──────┐
                                                       │   Payment    │
                                                       │   Processor  │
                                                       └───────┬──────┘
                                                       ┌───────▼──────┐
                                                       │    Email     │
                                                       └──────────────┘

        ┌──────────────────┐
        │  IdentityServer  │  (Autenticação/Autorização - OAuth2/OIDC)
        └──────────────────┘
```

Todos os serviços autenticam requisições via JWT emitido pelo **IdentityServer**, e o **API Gateway (Ocelot)** é o único ponto de entrada exposto ao cliente, roteando as chamadas para o microsserviço correto.

## Serviços

| Serviço | Responsabilidade | Comunicação |
|---|---|---|
| `Shopping.Web` | Frontend MVC (catálogo, carrinho, checkout) | HTTP → API Gateway |
| `Shopping.APIGateway` | Ponto único de entrada, roteamento e autenticação (Ocelot + JWT) | HTTP |
| `Shopping.ProductAPI` | Catálogo de produtos | HTTP |
| `Shopping.CartAPI` | Carrinho de compras | HTTP + RabbitMQ (checkout) |
| `Shopping.CouponAPI` | Cupons de desconto | HTTP |
| `Shopping.OrderAPI` | Criação e consulta de pedidos | HTTP + RabbitMQ |
| `Shopping.PaymentAPI` | Orquestração de pagamento do pedido | HTTP + RabbitMQ |
| `Shopping.PaymentProcessor` | Processamento assíncrono de pagamentos (worker) | RabbitMQ |
| `Shopping.Email` | Envio de notificações por e-mail (confirmação de pedido/pagamento) | RabbitMQ |
| `Shopping.MessageBus` | Abstração de mensageria compartilhada entre serviços | — |
| `IdentityServer` | Emissão/validação de tokens (OAuth2/OpenID Connect) | HTTP |
| `DatabaseMigrations` | Aplica as migrations em todos os DbContexts na subida do projeto | — |

## Stack Técnica

- **.NET 6 / ASP.NET Core** — Web APIs e MVC
- **Ocelot** — API Gateway
- **IdentityServer** — Autenticação e autorização (OAuth2/OIDC), com emissão de JWT
- **Entity Framework Core + Pomelo (MySQL)** — persistência, um banco por serviço
- **RabbitMQ** — mensageria assíncrona entre Cart, Order, Payment, PaymentProcessor e Email
- **AutoMapper** — mapeamento entre entidades e DTOs
- **Swagger / Swashbuckle** — documentação interativa de cada API
- **Docker Compose** — infraestrutura local (MySQL + RabbitMQ)

## Principais decisões de arquitetura

- **Database-per-service:** cada microsserviço tem seu próprio schema MySQL, evitando acoplamento via banco de dados compartilhado.
- **Mensageria assíncrona para o fluxo de checkout:** a confirmação de pedido, processamento de pagamento e disparo de e-mail acontecem via RabbitMQ, desacoplando os serviços e evitando chamadas síncronas encadeadas (que aumentariam a latência e o acoplamento temporal).
- **API Gateway como fachada única:** o cliente (Shopping.Web) nunca conhece os endereços dos microsserviços individualmente — toda a comunicação passa pelo Ocelot, que também centraliza a validação do token JWT.
- **Autenticação centralizada:** o IdentityServer é o único emissor de tokens, permitindo que todos os serviços validem a mesma identidade sem duplicar lógica de autenticação.

## Como rodar o projeto

### Pré-requisitos
- .NET 6 SDK
- Docker e Docker Compose
- Visual Studio (ou outra IDE com suporte a múltiplos projetos de inicialização)
- Um cliente MySQL (ex: HeidiSQL, DBeaver) para criar os bancos

### Passo a passo

1. Suba a infraestrutura local (MySQL + RabbitMQ):
   ```bash
   docker-compose up -d
   ```

2. Conecte-se ao MySQL e crie os bancos de cada serviço:
   ```
   Host: 127.0.0.1
   Porta: 3306
   Usuário: root
   Senha: rootpwd
   ```
   Bancos necessários:
   - `shopping_product_api`
   - `shopping_order_api`
   - `shopping_email`
   - `shopping_coupon_api`
   - `shopping_cart_api`
   - `shopping_identity_server`

3. No Visual Studio, configure **"Multiple startup projects"** e defina como `Start` todos os projetos, **exceto**:
   - `DatabaseMigrations` (executa uma vez e aplica as migrations automaticamente)
   - `PaymentProcessor` e `MessageBus` (são class libraries, não executáveis)

4. Rode a solução. As migrations serão aplicadas automaticamente na primeira subida.

5. Acesse o Swagger de cada API para explorar os endpoints, ou o `Shopping.Web` para o fluxo completo de compra.

## Estrutura do repositório

```
microservices-dotnet6/
├── DatabaseMigrations/       # Aplica migrations em todos os DbContexts
├── IdentityServer/           # Autenticação e emissão de tokens
├── Shopping.APIGateway/      # Ocelot - roteamento e ponto único de entrada
├── Shopping.CartAPI/
├── Shopping.CouponAPI/
├── Shopping.Email/
├── Shopping.MessageBus/      # Abstração de mensageria (RabbitMQ)
├── Shopping.OrderAPI/
├── Shopping.PaymentAPI/
├── Shopping.PaymentProcessor/
├── Shopping.ProductAPI/
└── Shopping.Web/             # Frontend MVC
```

## Possíveis evoluções

- Adicionar testes automatizados (unitários e de integração) — hoje o repositório não possui suíte de testes.
- Circuit breaker / retry policy (ex: Polly) nas chamadas entre API Gateway e serviços.
- Observabilidade centralizada (logs estruturados + tracing distribuído, ex: OpenTelemetry).
- Health checks por serviço, expostos via API Gateway.

---

Projeto de estudo de arquitetura de microsserviços com .NET 6, cobrindo comunicação síncrona e assíncrona, autenticação centralizada e persistência isolada por serviço.
