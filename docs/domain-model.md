# Modelo de domínio do OpsBoard

## Objetivo

Este documento descreve os dados que o sistema administra antes da implementação do banco ou da API. A API será a fonte de verdade para todos esses dados.

## Visão das relações

```text
User ───────< OrderStatusHistory >────── Order >────── Customer
                                      │
                                      └──────< OrderItem >────── Product
```

- Um usuário interno pode alterar o status de vários pedidos.
- Um cliente pode possuir vários pedidos.
- Um pedido possui um ou mais itens.
- Um produto pode aparecer em vários itens de pedidos.

## Entidades

### User

Representa uma pessoa interna que acessa o painel.

| Campo | Tipo | Regra |
| --- | --- | --- |
| `id` | UUID | Identificador interno. |
| `name` | texto | Nome exibido na aplicação. |
| `email` | texto | Único; usado no login. |
| `passwordHash` | texto | Senha protegida com hash; a senha pura nunca é armazenada. |
| `role` | enum | `ADMIN`, `OPERATOR` ou `VIEWER`. |
| `createdAt` | data/hora | Registro de criação. |
| `updatedAt` | data/hora | Última atualização. |

### Customer

Representa quem realizou uma compra. Clientes não acessam este painel no escopo inicial.

| Campo | Tipo | Regra |
| --- | --- | --- |
| `id` | UUID | Identificador interno. |
| `name` | texto | Nome da pessoa compradora. |
| `email` | texto | Contato para pedido. |
| `phone` | texto opcional | Contato adicional. |
| `createdAt` | data/hora | Registro de criação. |
| `updatedAt` | data/hora | Última atualização. |

### Product

Representa um item do catálogo e seu estoque atual.

| Campo | Tipo | Regra |
| --- | --- | --- |
| `id` | UUID | Identificador interno. |
| `sku` | texto | Único; código operacional do produto. |
| `name` | texto | Nome exibido. |
| `description` | texto opcional | Descrição curta. |
| `priceInCents` | inteiro | Preço em centavos; evita erros de ponto flutuante em dinheiro. |
| `stockQuantity` | inteiro | Estoque disponível atual. |
| `lowStockThreshold` | inteiro | Quantidade a partir da qual o produto entra no alerta. |
| `isActive` | booleano | Produto disponível ou arquivado. |
| `createdAt` | data/hora | Registro de criação. |
| `updatedAt` | data/hora | Última atualização. |

### Order

Representa uma compra. O número do pedido é usado na operação; o UUID é usado internamente pela API.

| Campo | Tipo | Regra |
| --- | --- | --- |
| `id` | UUID | Identificador interno. |
| `number` | texto | Único e legível, por exemplo `OPS-1024`. |
| `customerId` | UUID | Referência ao cliente. |
| `status` | enum | `PENDING`, `PROCESSING`, `SHIPPED`, `DELIVERED` ou `CANCELLED`. |
| `paymentStatus` | enum | `PENDING`, `PAID`, `REFUNDED` ou `FAILED`. |
| `subtotalInCents` | inteiro | Soma dos itens antes do frete. |
| `shippingInCents` | inteiro | Valor do frete. |
| `totalInCents` | inteiro | Subtotal + frete; gravado para consulta eficiente. |
| `createdAt` | data/hora | Momento em que o pedido foi realizado. |
| `updatedAt` | data/hora | Última atualização. |

### OrderItem

Representa cada produto comprado dentro de um pedido.

| Campo | Tipo | Regra |
| --- | --- | --- |
| `id` | UUID | Identificador interno. |
| `orderId` | UUID | Referência ao pedido. |
| `productId` | UUID | Referência ao produto atual. |
| `productName` | texto | Fotografia do nome no momento da compra. |
| `sku` | texto | Fotografia do SKU no momento da compra. |
| `unitPriceInCents` | inteiro | Fotografia do preço no momento da compra. |
| `quantity` | inteiro | Sempre maior que zero. |
| `totalInCents` | inteiro | Preço unitário × quantidade. |

Os campos de fotografia são intencionais. Se o nome ou preço do produto mudar amanhã, um pedido antigo deve continuar fiel ao que foi comprado.

### OrderStatusHistory

Registra uma mudança de status. É a base de auditoria operacional do pedido.

| Campo | Tipo | Regra |
| --- | --- | --- |
| `id` | UUID | Identificador interno. |
| `orderId` | UUID | Pedido alterado. |
| `changedByUserId` | UUID | Usuário interno que fez a alteração. |
| `fromStatus` | enum opcional | Nulo apenas na criação do pedido. |
| `toStatus` | enum | Novo status. |
| `createdAt` | data/hora | Quando ocorreu a mudança. |

## Regras de negócio iniciais

1. `VIEWER` pode consultar, mas não pode alterar dados.
2. `OPERATOR` pode alterar o status de pedidos; `ADMIN` pode executar todas as ações do painel.
3. Um pedido deve ter ao menos um `OrderItem`.
4. Valores financeiros são armazenados como inteiros em centavos.
5. Uma alteração de status cria um registro de histórico na mesma operação de banco.
6. Um produto com `stockQuantity <= lowStockThreshold` aparece na lista de estoque baixo.
7. O status de um pedido só pode avançar por transições permitidas; essas transições serão implementadas e testadas na API.

## Fora do escopo inicial

- Pagamento real.
- Endereço e cálculo de frete real.
- Login de clientes.
- Devolução detalhada por item.
- Multiestoque e múltiplos depósitos.
