# LogLens — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: [DocSage](https://github.com/Everett-gi/docsage).

## O que é
Ingere logs, extrai padrões e **sinaliza anomalias** (picos, sequências suspeitas) com um
modelo simples de ML, exibindo tudo num dashboard com alertas.

**Valor de portfólio:** ponte entre segurança e dados/ML — um diferencial forte.

## Stack
Python 3.12 · FastAPI · **pandas** · **scikit-learn** (Isolation Forest) · SQLAlchemy 2 ·
Alembic · PostgreSQL 16 · pytest · ruff.

## Modelo de dados
- **log_sources** (id, name, format)
- **log_entries** (id, source_id, timestamp, level, message, parsed JSONB)
- **baselines** (id, source_id, metric, mean, stddev, computed_at)
- **anomalies** (id, source_id, detected_at, score, context JSONB)

## Endpoints principais
- `POST /ingest` ou upload de arquivo de log · parsing configurável por fonte
- `GET /stats` (métricas/baseline) · detecção de anomalia (Isolation Forest)
- Dashboard + alertas

## Foco de segurança
**Sanitização** de logs; controle de acesso ao dashboard; **sem PII em claro**;
rate limit de ingestão.

## Plano de build
1. Ingestão + parsing configurável + Alembic
2. Métricas e baseline por fonte
3. Modelo de anomalia (Isolation Forest) sobre features dos logs
4. Dashboard (visualização das anomalias)
5. Autenticação + alertas
6. Deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.python`). O parsing e as
features são funções puras — coloque em um módulo separado e cubra com testes (como o
`text_utils` do DocSage).
