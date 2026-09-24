# OpsBoard

Painel administrativo de uma operação de e-commerce, construído como estudo prático de arquitetura front-end e integração com uma API própria.

## Objetivo

O projeto simula o trabalho diário de uma operação: acompanhar pedidos, produtos, estoque e indicadores. Ele será desenvolvido em etapas, com decisões técnicas documentadas e histórico de commits intencional.

## Escopo inicial

- Autenticação e autorização por papéis: `admin`, `operator` e `viewer`.
- Dashboard com indicadores e pedidos recentes.
- Pedidos com busca, filtros, ordenação e paginação no servidor.
- Página de detalhes e alteração de status de um pedido.
- Produtos e alerta de estoque baixo.
- Preferências de interface.

## Arquitetura planejada

```text
apps/
  web/       # Interface React + TypeScript
  api/       # REST API NestJS + TypeScript
```

O front-end e o back-end ficam juntos porque formam um único produto. Eles ainda serão aplicações independentes: a interface consome a API por HTTP, da mesma forma que faria em um ambiente de produção.

## Princípios do projeto

- A API é a fonte de verdade para pedidos, produtos e permissões.
- A interface melhora a experiência, mas não é responsável por segurança.
- Filtros, ordenação e paginação vivem na URL e são executados no servidor.
- Dados de desenvolvimento serão reproduzíveis por meio de seeds.
- Cada funcionalidade terá estados de carregamento, erro, vazio e sucesso.

## Tecnologias previstas

- React + TypeScript
- NestJS + TypeScript
- PostgreSQL
- TanStack Query
- React Router
- React Hook Form + Zod
- Vitest, Testing Library e Playwright

## Banco de dados local

O PostgreSQL de desenvolvimento roda em Docker e usa a porta local `5435`, para não disputar a porta padrão com outros projetos.

```bash
nvm use
npm install
npm run db:up
npm run db:migrate --workspace=@opsboard/api -- --name nome-da-migracao
```

Para encerrar o banco local, use `npm run db:down`. Os dados ficam no volume Docker `opsboard-postgres-data` e sobrevivem à parada do container.

## Etapa atual

Fundação do repositório e definição de arquitetura.
