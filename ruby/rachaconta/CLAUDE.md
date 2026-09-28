# RachaConta — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../../python/docsage`.

## O que é
Divisor de despesas estilo **Splitwise**: grupos, despesas compartilhadas, cálculo de saldos
e **simplificação de dívidas** (quem deve quanto a quem, com o mínimo de transações).

**Valor de portfólio:** lógica financeira sobre um grafo de dívidas — interessante e não trivial.

## Stack
Ruby 3.3 · Rails 8 · Devise · Pundit · Hotwire · PostgreSQL 16 · RSpec · RuboCop · Brakeman.

## Modelos
- **Group** · **Membership** (group, user) · **Expense** (group, payer, amount, description)
- **Split** (expense, user, share) · **Settlement** (group, from_user, to_user, amount)

## Funcionalidades principais
- Grupos e membros · despesas com divisão (igual ou percentual)
- Cálculo de saldos por membro · **simplificação de dívidas** · histórico

## Foco de segurança
Autorização **por grupo** (só membros); **validação de valores/divisões** (a soma das partes
tem que fechar com o total); escopo de dados; impedir manipulação de saldo pelo cliente.

## Plano de build
1. Grupos e membros
2. Despesas + divisão (igual/percentual, com validação da soma)
3. Cálculo de saldos
4. Simplificação de dívidas (algoritmo de minimização)
5. Autenticação + histórico
6. Deploy (ver `../../DEPLOY-GERAL.md` — notas de Rails)

## Como começar
Scaffold: `rails new rachaconta --database=postgresql --css=tailwind`. O cálculo de saldos e a
simplificação são lógica pura — implemente em objetos de serviço testáveis, com cobertura alta.
