# FlowBoard — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: [DocSage](https://github.com/Everett-gi/docsage).

## O que é
Gestão de projetos estilo **Kanban** (Trello): quadros, colunas, cards, drag-and-drop e
atualização **em tempo real** — sem JavaScript pesado, via Hotwire/Turbo Streams.

**Valor de portfólio:** mostra Turbo Streams e colaboração em tempo real, um destaque de Rails 8.

## Stack
Ruby 3.3 · Rails 8 · **Turbo Streams** + **Stimulus** · Devise · Pundit · PostgreSQL 16 ·
RSpec · RuboCop · Brakeman.

## Modelos
- **Board** (owner) · **Membership** (board, user, role) · **Column** (board, posição)
- **Card** (column, posição, título, descrição) · **Comment** (card, user)

## Funcionalidades principais
- Quadros, colunas e cards · **drag-and-drop** (Stimulus + posição persistida)
- Tempo real (Turbo Streams — todos veem a mudança) · membros e papéis
- Comentários e checklists nos cards

## Foco de segurança
Autorização **por quadro** (Pundit — só membros acessam); validação da movimentação de cards;
rate limit; **escopo de dados** por usuário (nunca vazar quadros de terceiros).

## Plano de build
1. Quadros, colunas e cards (CRUD)
2. Drag-and-drop com persistência de posição
3. Tempo real (Turbo Streams)
4. Membros e permissões (Pundit)
5. Comentários/checklists + autenticação
6. Deploy (ver `../../DEPLOY-GERAL.md` — notas de Rails)

## Como começar
Scaffold: `rails new flowboard --database=postgresql --css=tailwind`. Faça o CRUD e o
drag-and-drop primeiro; só então adicione o broadcast via Turbo Streams.
