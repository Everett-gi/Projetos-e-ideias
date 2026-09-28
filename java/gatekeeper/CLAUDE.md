# GateKeeper — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../../python/docsage`.

## O que é
Um **API Gateway** que autentica, aplica **rate limiting** e **quotas**, gerencia API keys e
roteia chamadas para serviços internos. Proteção de borda.

**Valor de portfólio:** demonstra arquitetura de microsserviços e proteção contra abuso.

## Stack
Java 21 · Spring Boot 3.3 · **Spring Cloud Gateway** · Redis (contadores de rate limit) ·
PostgreSQL 16 (API keys e quotas) · Flyway · Maven · JUnit 5.

## Modelo de dados
- **api_keys** (id, owner, key_hash, name, active, created_at, rotated_at)
- **quotas** (api_key_id, requests_per_day, requests_per_minute)
- **usage_counters** (api_key_id, window, count) — pode viver no Redis
- **routes** (id, path_prefix, target_uri, requires_key)

## Endpoints principais
- Gestão de keys: emitir, listar, revogar, rotacionar
- Gateway: proxy com verificação de key + rate limit + quota por chave/IP
- `GET /usage` — métricas de uso por chave

## Foco de segurança
**Rate limiting** e **throttling** (por chave e por IP); validação de origem; **rotação** de
keys; quotas; logging sem vazar segredos; chaves armazenadas como hash.

## Plano de build
1. Gestão de API keys (emissão/hash/revogação) + migrations
2. Filtro de autenticação por key no gateway
3. Rate limiting + quotas (Redis)
4. Roteamento para 2 serviços demo
5. Métricas de uso
6. Deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.java`). Adicione um serviço
Redis ao `docker-compose.yml`. Suba 2 backends fictícios para o gateway rotear.
