# ThreatScope — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../docsage`.

## O que é
Agregador de **threat intelligence**: coleta indicadores de comprometimento (IOCs — IPs,
domínios, hashes) de feeds públicos, deduplica, enriquece e disponibiliza por API/busca.

**Valor de portfólio:** integração de múltiplas fontes + modelagem de dados de segurança —
núcleo do ecossistema de cibersegurança.

## Stack
Python 3.12 · FastAPI · SQLAlchemy 2 · Alembic · **httpx** (async) · APScheduler (coleta
agendada) · PostgreSQL 16 · pytest · ruff.

## Modelo de dados
- **iocs** (id, type[ip|domain|hash], value, source, risk_score, first_seen, last_seen)
- **feeds** (id, name, url, api_key_env, last_synced, enabled)
- **api_keys** (id, owner, key_hash)

## Endpoints principais
- `GET /iocs/search?value=` · `GET /iocs?type=` (paginado)
- `POST /feeds/sync` (dispara coleta) + coleta agendada em background
- score de risco por IOC · API autenticada por key

## Foco de segurança
Chaves de feeds via **env**; rate limit; validação de entrada; cache de consultas;
auditoria de buscas; sem expor as chaves em logs.

## Plano de build
1. Schema de IOCs + Alembic
2. Coletor async + agendador (1 feed público para começar)
3. Deduplicação + enriquecimento (merge por value, atualiza last_seen)
4. API de busca + paginação
5. Autenticação por API key + rate limit
6. Deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.python`). Estrutura de código
espelhando o DocSage (`app/` com config, database, models, schemas, routers/services).
