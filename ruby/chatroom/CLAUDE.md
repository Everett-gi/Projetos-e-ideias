# ChatRoom — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: [DocSage](https://github.com/Everett-gi/docsage).

## O que é
Salas de **chat em tempo real** com mensagens ao vivo, indicador de presença/digitando,
histórico e menções/notificações — via **Action Cable** (WebSockets).

**Valor de portfólio:** WebSockets é vitrine de tempo real e demonstra arquitetura assíncrona.

## Stack
Ruby 3.3 · Rails 8 · **Action Cable** · **Solid Cable** (ou Redis) · Devise · Pundit ·
PostgreSQL 16 · RSpec · RuboCop · Brakeman.

## Modelos
- **Room** (nome, tipo: public|private, owner) · **Membership** (room, user)
- **Message** (room, user, body, created_at)

## Funcionalidades principais
- Salas públicas e privadas · mensagens ao vivo (Action Cable)
- Presença e indicador de "digitando" · histórico · menções e notificações

## Foco de segurança
Autorização de **entrada em sala** (só membros nas privadas); **sanitização** das mensagens;
**rate limit anti-flood**; escopo por sala (não vazar mensagens de outras salas).

## Plano de build
1. Salas e mensagens (CRUD)
2. Tempo real (Action Cable — broadcast das mensagens)
3. Presença + "digitando"
4. Salas privadas (autorização de entrada)
5. Menções/notificações + autenticação
6. Deploy (ver `../../DEPLOY-GERAL.md` — notas de Rails)

## Como começar
Scaffold: `rails new chatroom --database=postgresql --css=tailwind`. Comece pelo envio/
persistência das mensagens; depois plugue o broadcast via Action Cable.
