# FileSentry — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: [DocSage](https://github.com/Everett-gi/docsage).

## O que é
Monitor de **integridade de arquivos** (FIM/HIDS leve): estabelece um baseline de hashes,
detecta criação/alteração/remoção e consulta **reputação de hashes** (estilo VirusTotal).

**Valor de portfólio:** ferramenta blue team prática, com integração externa.

## Stack
Python 3.12 · FastAPI · **watchdog** · hashlib · httpx · SQLAlchemy 2 · Alembic ·
PostgreSQL 16 · pytest · ruff.

## Modelo de dados
- **baselines** (id, path, hash, size, recorded_at)
- **changes** (id, path, change_type[created|modified|deleted], old_hash, new_hash, detected_at)
- **hash_lookups** (id, hash, reputation JSONB, checked_at)
- **alerts** (id, change_id, severity, created_at)

## Endpoints principais
- `POST /baseline` — define/atualiza o baseline de um diretório
- Detecção de mudanças (watcher em background) · `GET /changes`
- `GET /hash/{hash}/reputation` — consulta reputação externa · alertas

## Foco de segurança
Integridade por **hash**; chaves de API externas via **env**; controle de acesso; log de
todas as alterações detectadas; não confiar em caminhos fora do escopo configurado.

## Plano de build
1. Cálculo de hash + baseline (funções puras + testes) + Alembic
2. Watcher de diretório (detecção de created/modified/deleted)
3. Consulta de reputação de hash
4. Alertas
5. Autenticação + rate limit
6. Deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.python`). O cálculo de hash e
a comparação com o baseline são funções puras — comece por elas, com testes.
