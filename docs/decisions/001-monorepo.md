# ADR 001 — Monorepo com npm workspaces

## Contexto

OpsBoard possui uma interface web e uma API. Elas precisam evoluir juntas, mas devem continuar independentes em execução e comunicação: a interface chama a API por HTTP.

## Decisão

Usaremos npm workspaces com duas aplicações:

- `apps/web`: React + TypeScript.
- `apps/api`: NestJS + TypeScript.

## Consequências

- Um único `npm install` na raiz instala dependências das duas aplicações.
- A raiz expõe comandos consistentes, como `npm run build` e `npm run test`.
- Cada app continua com seu próprio `package.json`, scripts e responsabilidades.
- Se no futuro houver código realmente compartilhado, ele será criado em `packages/`; não vamos antecipar essa abstração.
