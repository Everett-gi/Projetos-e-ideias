# AuditTrail — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: [DocSage](https://github.com/Everett-gi/docsage).

## O que é
Serviço de **trilha de auditoria à prova de adulteração**: cada registro é encadeado por
hash ao anterior (tamper-evident), permitindo verificar a integridade de toda a cadeia.

**Valor de portfólio:** conceito de integridade/append-only que conversa com blockchain-light
e compliance — assunto que impressiona.

## Stack
Java 21 · Spring Boot 3.3 · Spring Data JPA · SHA-256 (encadeamento) · assinatura digital
(exportação) · Flyway · PostgreSQL 16 · Maven · JUnit 5.

## Modelo de dados
- **audit_events** (id, seq, prev_hash, hash, actor, resource, action, payload JSONB, created_at)

> `hash = SHA256(seq || prev_hash || actor || resource || action || payload || created_at)`.
> A tabela é **append-only**: sem UPDATE, sem DELETE.

## Endpoints principais
- `POST /events` — registra um evento (calcula o hash encadeado)
- `GET /events` — consulta por ator/recurso/período
- `GET /verify` — recomputa e valida a cadeia inteira, apontando onde quebrou (se quebrar)
- `GET /export` — exportação **assinada** para verificação externa

## Foco de segurança
**Append-only** (garantido no código e por permissões do banco); verificação criptográfica
da cadeia; **assinatura** dos exports; RBAC de leitura; payload sem segredos.

## Plano de build
1. Modelo de evento + migrations
2. Cálculo do hash encadeado + gravação append-only
3. API de registro e consulta
4. Verificação de integridade da cadeia (com teste que adultera e detecta)
5. Exportação assinada
6. Integração: usar como serviço de auditoria de outros projetos + deploy

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.java`). Escreva cedo um teste
que insere N eventos, adultera um registro no banco e confirma que `/verify` acusa a quebra.
