# ReconKit — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: [DocSage](https://github.com/Everett-gi/docsage).

## ⚠️ Ética e escopo
Ferramenta de **OSINT passivo**. Deve operar **apenas em alvos que você possui ou tem
autorização escrita para investigar** — o padrão profissional (HackerX). Deixe isso explícito
no README e exija confirmação de autorização por alvo.

## O que é
Painel que agrega **WHOIS, DNS, enumeração passiva de subdomínios, cabeçalhos HTTP de
segurança e checagem de vazamentos** para um alvo autorizado, com relatório exportável.

**Valor de portfólio:** alinhado direto à sua certificação (OSINT); segurança ofensiva
responsável.

## Stack
Python 3.12 · FastAPI · **dnspython** · **python-whois** · httpx · SQLAlchemy 2 · Alembic ·
PostgreSQL 16 (histórico de scans) · pytest · ruff.

## Modelo de dados
- **scans** (id, target, authorized_by, created_at, status)
- **findings** (id, scan_id, category[whois|dns|headers|subdomains|breaches], data JSONB)

## Endpoints principais
- `POST /scans` (alvo + confirmação de autorização) · `GET /scans/{id}`
- Coleta: WHOIS, registros DNS, headers de segurança, subdomínios (fontes passivas)
- `GET /scans/{id}/report` (PDF/JSON) · histórico

## Foco de segurança
**Somente coleta passiva** (sem varredura ativa/intrusiva); **autorização explícita** por alvo;
rate limit; não armazenar dado sensível além do necessário.

## Plano de build
1. Módulos WHOIS + DNS + Alembic
2. Headers de segurança + subdomínios (passivo)
3. Persistência de scans/findings
4. Relatório (PDF/JSON)
5. Autenticação + registro de autorização
6. Deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.python`). Comece com um
módulo por vez (WHOIS, depois DNS), cada um testável isoladamente.
