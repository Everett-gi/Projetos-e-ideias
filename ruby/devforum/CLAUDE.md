# DevForum — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: [DocSage](https://github.com/Everett-gi/docsage).

## O que é
Fórum/comunidade com tópicos, respostas, **votos**, tags, reputação e **moderação**.

**Valor de portfólio:** modela relacionamentos complexos, votação e moderação — CRUD avançado
em Rails.

## Stack
Ruby 3.3 · Rails 8 · Hotwire (Turbo) · **Devise** (auth) · **Pundit** (autorização) ·
rack-attack (rate limit) · PostgreSQL 16 · RSpec · RuboCop · Brakeman.

## Modelos
- **User** (Devise) · **Topic** (título, corpo, user, tags) · **Post** (resposta, topic, user)
- **Vote** (votable polimórfico, user, value) · **Tag** + **Taggings** · **Report** (moderação)

## Funcionalidades principais
- Criar/editar tópicos e respostas · votos e cálculo de reputação
- Tags e busca · moderação (flag/remover/banir) · notificações

## Foco de segurança
**Pundit** (autorização por ação); **strong params**; **rack-attack** (rate limit anti-abuso);
anti-spam; **sanitização** do Markdown/HTML; CSRF (nativo do Rails).

## Plano de build
1. Auth com Devise
2. Tópicos e respostas (CRUD + Hotwire)
3. Votos e reputação
4. Tags e busca
5. Moderação (Pundit) + notificações
6. Deploy (ver `../../DEPLOY-GERAL.md` — notas de Rails)

## Como começar
Scaffold: `rails new devforum --database=postgresql --css=tailwind` e traga do
`../../../projeto-template` o `.gitignore`, CI e o padrão de `docker-compose`. Rode
**Brakeman** e **bundler-audit** desde o começo.
