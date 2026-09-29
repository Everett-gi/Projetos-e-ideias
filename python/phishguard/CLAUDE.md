# PhishGuard — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: [DocSage](https://github.com/Everett-gi/docsage).

## O que é
API que avalia se uma URL é provavelmente **phishing**, combinando heurísticas com um
**classificador de ML**, devolvendo um score e a explicação dos fatores.

**Valor de portfólio:** ML aplicado a segurança, com uma API utilizável de verdade.

## Stack
Python 3.12 · FastAPI · **scikit-learn** · SQLAlchemy 2 · Alembic · PostgreSQL 16 ·
pytest · ruff.

## Modelo de dados
- **url_checks** (id, url_hash, score, verdict, features JSONB, created_at)
- **model_versions** (id, version, metrics JSONB, active, created_at)
- **feedback** (id, url_hash, label, created_at)

## Endpoints principais
- `POST /check` — recebe uma URL, extrai features, retorna score + explicação
- `POST /feedback` — usuário marca acerto/erro (para retreino futuro)
- Versionamento do modelo

## Features (exemplos)
Idade do domínio, comprimento e entropia da URL, uso de IP no lugar de domínio, TLD suspeito,
número de subdomínios, presença de palavras-isca, HTTPS ou não.

## Foco de segurança
**Anti-SSRF** ao analisar/baixar a URL (não seguir para redes internas); rate limit;
cache; **modelo versionado**; nunca executar conteúdo da URL.

## Plano de build
1. Extração de features (funções puras + testes) + Alembic
2. Treino/avaliação do classificador (dataset público)
3. API de score + explicação
4. Feedback + caminho de retreino
5. Autenticação + rate limit
6. Deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.python`). As features são o
coração — implemente-as como funções puras testáveis antes de treinar qualquer modelo.
