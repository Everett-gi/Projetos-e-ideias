# SecureBank API — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../../python/docsage`.

## O que é
API bancária com **ledger contábil de partidas dobradas**, transferências idempotentes e
extrato. O foco é **integridade transacional** sob concorrência.

**Valor de portfólio:** consistência financeira, idempotência e concorrência são temas
sérios de backend que aparecem em entrevistas.

## Stack
Java 21 · Spring Boot 3.3 · Spring Data JPA · Flyway · PostgreSQL 16 (transações ACID) ·
Maven · JUnit 5 + Testcontainers.

## Modelo de dados
- **accounts** (id, owner_id, currency, status, created_at)
- **ledger_entries** (id, transaction_id, account_id, direction[DEBIT|CREDIT], amount, created_at)
- **transactions** (id, type, status, idempotency_key único, created_at)
- **idempotency_keys** (key, request_hash, response, created_at)

> Regra do ledger: toda transação gera lançamentos que **somam zero** (débito = crédito).
> O saldo é a soma dos lançamentos da conta — nunca um campo mutável solto.

## Endpoints principais
- `POST /accounts` · `GET /accounts/{id}/balance`
- `POST /accounts/{id}/deposit` · `POST /accounts/{id}/withdraw`
- `POST /transfers` (idempotente, via header `Idempotency-Key`)
- `GET /accounts/{id}/statement` (paginado)

## Foco de segurança
**Idempotência** (evita débito duplicado); **locking** otimista/pessimista contra corrida;
autorização por titular da conta (**anti-IDOR**); ledger **imutável** (sem update/delete);
validação de valores (sem negativos); auditoria de transações.

## Plano de build
1. Modelo de contas + ledger + migrations
2. Depósito/saque com lançamentos balanceados
3. Transferência com locking (transação atômica)
4. Idempotency keys + tratamento de erros idempotente
5. Extrato paginado + autorização por titular
6. Testes de concorrência (Testcontainers, transações simultâneas)
7. Deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.java`). Priorize os testes
de concorrência — são o diferencial deste projeto.
