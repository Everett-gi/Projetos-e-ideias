# HelpDesk — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../../python/docsage`.

## O que é
Sistema de **tickets de suporte** com papéis (cliente/agente/admin), comentários, anexos e
acompanhamento de **SLA**.

**Valor de portfólio:** CRUD rico + permissões em camadas + regras de negócio = backend completo.

## Stack
Java 21 · Spring Boot 3.3 · Spring Security · Spring Data JPA · Flyway · PostgreSQL 16 ·
Object Storage (anexos — OCI, gratuito) · Maven · JUnit 5.

## Modelo de dados
- **users** (id, email, role[CLIENT|AGENT|ADMIN])
- **tickets** (id, requester_id, assignee_id, subject, status, priority, sla_policy_id, created_at)
- **comments** (id, ticket_id, author_id, body, created_at)
- **attachments** (id, ticket_id, filename, content_type, size, storage_key)
- **sla_policies** (id, name, response_minutes, resolution_minutes)

## Endpoints principais
- CRUD de tickets · atribuição a agente · mudança de status/prioridade
- Comentários e histórico · upload/download de anexos
- Acompanhamento de SLA (prazos, tickets em risco) · busca/filtros

## Foco de segurança
**RBAC** por papel; autorização **por ticket** (cliente só vê os seus — anti-IDOR);
validação/scan de anexos (tipo e tamanho); rate limit na criação.

## Plano de build
1. Modelo tickets/usuários + migrations
2. Ciclo de vida do ticket + papéis
3. Comentários + histórico
4. Anexos (upload validado + storage)
5. SLA (cálculo de prazos + destaque de risco) + notificações
6. Autorização (RBAC + por ticket) + deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.java`). Modele os papéis e a
autorização por ticket desde o início — é onde mora o valor de segurança aqui.
