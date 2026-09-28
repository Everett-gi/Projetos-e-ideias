# LinkShield — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../../python/docsage`.

## O que é
Encurtador de URL com **analytics**, links com expiração/senha e **proteção contra links
maliciosos**. O clássico, elevado por uma camada de segurança.

**Valor de portfólio:** mostra que você pega um projeto comum e adiciona rigor de segurança.

## Stack
Java 21 · Spring Boot 3.3 · Spring Data JPA · Cache (Caffeine ou Redis) · reputação de URL ·
Flyway · PostgreSQL 16 · Maven · JUnit 5.

## Modelo de dados
- **links** (id, code único, target_url, owner_id, expires_at, password_hash, active, created_at)
- **clicks** (id, link_id, clicked_at, referrer, country, user_agent_hash)
- **api_keys** (id, owner_id, key_hash)

## Endpoints principais
- `POST /links` (encurtar) · `GET /{code}` (redirecionar)
- `GET /links/{id}/analytics` (cliques, referrer, geografia)
- Links com expiração e/ou senha · checagem de URL maliciosa na criação
- API autenticada por key

## Foco de segurança
Validação **anti-SSRF** e **anti-open-redirect** (só http/https, sem IPs internos);
rate limit; **detecção de phishing/malware** (lista de reputação); CAPTCHA opcional;
sanitização de entrada.

## Plano de build
1. Encurtamento + redirect + migrations
2. Analytics de cliques
3. Expiração + proteção por senha
4. Checagem de reputação/malícia da URL
5. Autenticação por API key
6. Cache + deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.java`). Implemente a validação
anti-SSRF/open-redirect junto com o encurtamento — não como um "extra" depois.
