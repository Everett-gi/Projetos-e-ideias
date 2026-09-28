# PortPulse — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../docsage`.

## ⚠️ Ética e escopo
Scanner de portas para **auditar a sua própria infraestrutura**. Só escaneia alvos numa
**allowlist** de sistemas que você possui ou tem autorização escrita para testar. Sem isso,
não roda. Deixe explícito no README.

## O que é
Scanner de portas **assíncrono** com UI web, detecção de serviço (banner), diff entre scans e
agendamento — para monitorar a exposição da sua própria infra.

**Valor de portfólio:** redes, programação assíncrona e ferramenta de segurança na prática.

## Stack
Python 3.12 · FastAPI · **asyncio** · SQLAlchemy 2 · Alembic · PostgreSQL 16 ·
frontend simples (HTMX ou React) · pytest · ruff.

## Modelo de dados
- **allowlist** (id, target, authorized_by, added_at)
- **scans** (id, target, started_at, finished_at, status)
- **results** (id, scan_id, port, state, service, banner)
- **schedules** (id, target, cron, enabled)

## Endpoints principais
- `POST /scans` (alvo **precisa** estar na allowlist) · `GET /scans/{id}`
- Fingerprint de serviço/banner · diff entre dois scans (portas novas/fechadas)
- Agendamento de scans recorrentes

## Foco de segurança
**Allowlist** obrigatória de alvos autorizados; **limites de taxa** de scan (não floodar);
log de quem escaneou o quê; confirmação explícita antes de cada scan.

## Plano de build
1. Engine async de scan (com timeouts e limite de concorrência)
2. Detecção de serviço/banner
3. Persistência + diff entre scans
4. UI web
5. Allowlist + autenticação + rate limit
6. Agendamento + deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.python`). Implemente a
allowlist **antes** da engine de scan — é a trava de segurança que torna o projeto responsável.
