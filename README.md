# Portfólio — Java · Python · Ruby

24 aplicações full-stack com foco em segurança. Cada projeto usa **PostgreSQL**,
**Docker**, roda em **hospedagem gratuita** e segue a mesma base de segurança.

## Organização

Todos os projetos vivem **neste repositório** (um *monorepo*).

```
Projetos-e-ideias/
├── README.md            (este arquivo — convenções compartilhadas)
├── CLAUDE.md            (contexto do repositório para o Claude Code)
├── DEPLOY-GERAL.md      (deploy compartilhado na Oracle Cloud + notas por linguagem)
├── tutorial/            (trilha de aprendizado — comece por tutorial/README.md)
├── .github/workflows/   (CI de TODOS os projetos: um <projeto>-ci.yml para cada)
├── java/                (backends de segurança — Spring Boot)
│   ├── authhub/         · securebank/  · secvault/   · siem-lite/
│   └── audittrail/      · gatekeeper/  · helpdesk/    · linkshield/
├── python/              (ferramentas de segurança + IA — FastAPI)
│   ├── docsage/  ✅      · threatscope/ · reconkit/    · portpulse/
│   └── loglens/         · phishguard/  · pwncheck/ 🚧  · filesentry/
└── ruby/                (produtos web — Rails)
    ├── devforum/        · publicms/    · mercadolite/  · flowboard/
    └── edupath/         · agendaja/    · rachaconta/    · chatroom/
```

Cada pasta de projeto tem um **`CLAUDE.md`**: o handoff completo (o que é, stack,
modelo de dados, funcionalidades, segurança e plano de build). O DocSage já está
**construído por inteiro** e serve de referência viva para todos os outros.

## Trilha de aprendizado

Os projetos são construídos em **modo tutorial**: cada fase concluída vira uma lição
explicando o que foi feito e por quê. O índice completo, na ordem de estudo, está em
**[tutorial/README.md](tutorial/README.md)**.

## Status

| Projeto | Linguagem | Status |
|---|---|---|
| DocSage | Python | ✅ Completo |
| PwnCheck | Python | 🚧 Em construção |
| Todos os demais (22) | — | 📋 Blueprint pronto (a construir) |

## Como cada projeto é construído (fluxo)

1. **Ler o `CLAUDE.md`** do projeto — ele é o mapa.
2. **Scaffold** da estrutura seguindo o DocSage como referência (`app/`, `tests/`,
   `pyproject.toml`, `requirements.txt`...). Cada arquivo de configuração é explicado na
   lição correspondente — nada de copiar template sem entender.
3. **Construir com o Claude Code** dentro da pasta do projeto, seguindo as fases do build.
4. **Testar** localmente (`pytest`, depois `docker compose up`) e versionar neste repositório.
5. **Publicar** seguindo o `DEPLOY-GERAL.md`.

> **Um projeto por vez, do começo ao fim.** Um portfólio de 4 projetos completos e no ar
> vale mais que 24 repositórios inacabados.

## Prioridades (todos os projetos)

**1) Segurança · 2) Clareza do código · 3) Funcionalidade.** Nessa ordem.

## Baseline de segurança (aplicado igualmente a todos)

Isto é o "Definition of Done" de segurança de qualquer projeto aqui:

- **Segredos** só em variáveis de ambiente (`.env` fora do Git e da imagem). Nunca hardcoded.
- **Senhas** com hash Argon2/BCrypt. **Nunca** texto puro ou hash fraco.
- **Autenticação** por JWT (access + refresh com rotação) onde houver login.
- **Autorização** com verificação de posse por recurso (anti-IDOR) e RBAC quando aplicável.
- **SQL Injection**: acesso ao banco só via ORM / parâmetros ligados. Nunca concatenar SQL.
- **Validação de entrada** com schema/DTO e limite de tamanho de payload.
- **Rate limiting** em login e endpoints sensíveis.
- **Cabeçalhos de segurança** (HSTS, CSP, X-Content-Type-Options) e **HTTPS** em produção.
- **Dependências** escaneadas no CI (por linguagem — ver abaixo) + Dependabot.
- **Logs** sem segredos nem dados pessoais.
- **LGPD**: coleta mínima, aceite de termos e exclusão de conta quando houver dados de usuário.
- Cada repositório tem um **`SECURITY.md`** descrevendo essas medidas aplicadas a ele.

## Ferramentas por linguagem

| | Java (Spring Boot 3.3 / JDK 21) | Python (FastAPI / 3.12) | Ruby (Rails 8 / 3.3) |
|---|---|---|---|
| Migrations | Flyway | Alembic | Active Record |
| Auth | Spring Security | python-jose / Authlib | Devise + Pundit |
| Testes | JUnit 5 + Testcontainers | pytest | RSpec |
| Lint | Checkstyle / SpotBugs | ruff | RuboCop |
| Scan de segurança | OWASP Dependency-Check | pip-audit + bandit | Brakeman + bundler-audit |

## Convenções de código

- Comentários e docstrings em **português**; nomes de código em **inglês**.
- Type hints / tipos explícitos sempre.
- Toda função pura nova vem acompanhada de teste.
- Mensagens de erro da API em português (é o usuário que lê).
- Commits no padrão *Conventional Commits*, com o projeto como escopo:
  `feat(pwncheck): ...`, `fix(docsage): ...`, `docs: ...`, `test: ...`, `ci: ...`, `chore: ...`.
- **Monorepo:** cada projeto tem seu próprio workflow em `.github/workflows/<projeto>-ci.yml`
  (o GitHub só lê workflows na raiz), filtrado por `paths:` para rodar só quando a pasta
  daquele projeto muda.

## Sugestão de ordem

**Carros-chefe primeiro** (1 por linguagem, completos e no ar):
`python/docsage` ✅ → `java/authhub` → `ruby/mercadolite` (ou `ruby/flowboard`).

Depois, o restante por afinidade — Java para aprofundar segurança de backend,
Python para o ferramental de segurança, Ruby para produtos web.
