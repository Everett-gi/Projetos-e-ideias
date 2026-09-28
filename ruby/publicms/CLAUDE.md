# PubliCMS — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../../python/docsage`.

## O que é
CMS/blog multiusuário com **papéis editoriais** (autor/editor/admin), editor rich text,
agendamento de publicação e comentários moderados.

**Valor de portfólio:** demonstra papéis, workflow editorial e SEO.

## Stack
Ruby 3.3 · Rails 8 · **Action Text** (rich text) · Devise · Pundit · PostgreSQL 16 ·
RSpec · RuboCop · Brakeman.

## Modelos
- **User** (role: author|editor|admin) · **Post** (status: draft|published|scheduled,
  published_at, rich body) · **Comment** (post, moderado) · **Category**

## Funcionalidades principais
- Rascunho → publicado → agendado (job publica no horário)
- Papéis editoriais (autor cria, editor aprova) · comentários moderados
- Categorias · SEO (meta tags, slug) · feed RSS

## Foco de segurança
**Sanitização** do conteúdo rich text; autorização **por papel** (Pundit); CSRF (nativo);
anti-spam nos comentários; strong params.

## Plano de build
1. Auth + papéis
2. CRUD de posts (Action Text)
3. Workflow editorial (rascunho/agendado/publicado + job)
4. Comentários moderados
5. SEO + RSS
6. Deploy (ver `../../DEPLOY-GERAL.md` — notas de Rails)

## Como começar
Scaffold: `rails new publicms --database=postgresql`. Configure Action Text e um job de
publicação agendada (Solid Queue). Rode Brakeman/bundler-audit no CI.
