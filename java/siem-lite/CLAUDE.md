# SIEM-Lite — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../../python/docsage`.

## O que é
Recebe logs de várias fontes, **normaliza**, indexa e dispara **alertas por regras**
(ex.: N falhas de login em X minutos). Um SIEM enxuto, para blue team.

**Valor de portfólio:** pipeline de dados + detecção; projeto de segurança de alto impacto.

## Stack
Java 21 · Spring Boot 3.3 · Spring Data JPA · PostgreSQL 16 (**JSONB** para eventos) ·
Spring Scheduler (correlação) · Flyway · Maven · JUnit 5.

## Modelo de dados
- **sources** (id, name, api_key_hash, active)
- **events** (id, source_id, timestamp, severity, raw JSONB, normalized JSONB)
- **rules** (id, name, condition, window_seconds, threshold, enabled)
- **alerts** (id, rule_id, triggered_at, details JSONB, status)

## Endpoints principais
- `POST /ingest` (autenticado por fonte) — recebe eventos
- `GET /events` (busca/filtros por severidade, fonte, período; paginado)
- CRUD de **rules** · `GET /alerts`

## Foco de segurança
Autenticação de fontes (**API key**, com caminho para mTLS); validação de payload;
isolamento por tenant/fonte; retenção configurável; sem dados sensíveis em log.

## Plano de build
1. Ingestão + schema com JSONB + migrations
2. Normalização de eventos (campos comuns a partir do raw)
3. Busca/filtros + paginação
4. Motor de regras + alertas (avaliação por janela de tempo)
5. Autenticação de fontes + isolamento
6. Notificação de alerta (e-mail/webhook) + deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.java`). Modele o `events`
com JSONB desde o início — a flexibilidade do schema é o que faz o SIEM funcionar.
