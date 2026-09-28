# AgendaJá — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../../python/docsage`.

## O que é
Sistema de **agendamento/reservas**: disponibilidade, marcação com checagem de conflito,
lembretes por e-mail, cancelamento/remarcação e suporte a fuso horário.

**Valor de portfólio:** lógica de calendário e concorrência (evitar overbooking) é um bom desafio.

## Stack
Ruby 3.3 · Rails 8 · Devise · Pundit · **Solid Queue** (jobs em background) · Action Mailer ·
PostgreSQL 16 · RSpec · RuboCop · Brakeman.

## Modelos
- **Provider** (user que oferece horários) · **Availability** (provider, dia, janela)
- **Booking** (provider, client, starts_at, ends_at, status) · **Reminder** (booking, enviado)

## Funcionalidades principais
- Definição de disponibilidade · reserva com **checagem de conflito**
- Lembretes por e-mail (job agendado) · cancelar/remarcar · fuso horário do cliente

## Foco de segurança
**Locking** para evitar double-booking (transação + trava na faixa de horário); autorização
das reservas (cada um vê as suas); rate limit; validação de horários (sem reservar no passado).

## Plano de build
1. Disponibilidade
2. Reserva + checagem de conflito (com teste de duas reservas simultâneas)
3. Lembretes por e-mail (Solid Queue)
4. Cancelamento/remarcação
5. Autenticação + fuso horário
6. Deploy (ver `../../DEPLOY-GERAL.md` — notas de Rails)

## Como começar
Scaffold: `rails new agendaja --database=postgresql`. O teste-chave é o de concorrência:
duas reservas para o mesmo horário — só uma pode passar.
