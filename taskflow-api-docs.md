# TaskFlow API — Documentação do Projeto

Documentação de como o projeto foi construído usando spec-kit + opencode (Spec-Driven Development).

## O que é o projeto

API REST para gerenciamento de tarefas. Usuários se cadastram, criam projetos, criam tarefas dentro desses projetos, atribuem responsáveis, comentam e acompanham status/prioridade.

## Stack

- Node.js + TypeScript
- Express
- PostgreSQL + Prisma
- JWT para autenticação
- Jest + Supertest para testes
- Docker / docker-compose

## Como foi feito (fluxo SDD)

O projeto foi desenvolvido seguindo o fluxo de comandos do spec-kit, rodados dentro do opencode:

1. **`/speckit.constitution`** — definiu os princípios do projeto: simplicidade, API-first, segurança por padrão (JWT), testes obrigatórios, observabilidade e versionamento semântico.

2. **`/speckit.specify`** — descreveu o que o sistema deveria fazer: cadastro/login, CRUD de tarefas, projetos, colaboração, comentários, notificações internas, filtros e paginação. Sem entrar em detalhes técnicos.

3. **`/speckit.plan`** — traduziu a especificação em decisões técnicas: arquitetura em camadas (routes → controllers → services → repositories), Prisma como ORM, Swagger pra documentar a API, Docker pra rodar tudo.

4. **`/speckit.tasks`** — quebrou o plano em tarefas numeradas e ordenadas (setup, models, auth, rotas, testes, etc.).

5. **`/speckit.taskstoissues`** — criou as issues no GitHub a partir das tarefas, uma por uma.

6. **`/speckit.implement`** — implementou o código de fato, tarefa por tarefa, seguindo o que estava no `tasks.md`.

7. **`/speckit.analyze`** — rodado depois do implement, pra checar consistência entre spec, plano e código.

## Estrutura de pastas gerada

```
taskflow-api/
├── src/                # código da API (routes, controllers, services, etc.)
├── prisma/             # schema e migrations do banco
├── tests/              # testes unitários e de integração
├── specs/              # spec.md, plan.md, tasks.md gerados pelo spec-kit
├── .specify/            # configuração interna do spec-kit
├── docker-compose.yml
└── package.json
```

## Como rodar localmente

```bash
npm install
npm test
docker-compose up
```

## Observações / problemas encontrados

- O comando `/speckit.taskstoissues` precisa do GitHub CLI (`gh`) autenticado. Sem isso, o comando falha.
- Os testes (`npm test`) falharam inicialmente com erro de `localStorage` do Jest — causado por incompatibilidade com versão recente do Node.js. Resolvido desativando a flag experimental de webstorage do Node ou usando Node 20 LTS.
- A ordem usada na prática foi: constitution → specify → plan → tasks → taskstoissues → implement → analyze (o ideal seria rodar o analyze antes do implement, mas rodar depois também funciona, só serve pra revisão).

## Status

Implementação gerada via `/speckit.implement`. Pendente: corrigir testes e validar endpoints manualmente.
