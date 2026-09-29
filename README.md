# Portfólio — Java · Python · Ruby

24 aplicações full-stack com foco em segurança. Cada projeto usa **PostgreSQL**,
**Docker**, roda em **hospedagem gratuita** e segue a mesma base de segurança.

## Organização

Este repositório é a **central do portfólio**. Cada projeto em construção tem
**repositório próprio**; aqui ficam os blueprints dos projetos que ainda não começaram, as
convenções compartilhadas (este README), o guia de deploy e a trilha de aprendizado.

```
Projetos-e-ideias/
├── README.md            (este arquivo — convenções compartilhadas)
├── CLAUDE.md            (contexto do repositório para o Claude Code)
├── DEPLOY-GERAL.md      (deploy compartilhado na Oracle Cloud + notas por linguagem)
├── tutorial/            (trilha de aprendizado — comece por tutorial/README.md)
├── java/                (blueprints: backends de segurança — Spring Boot)
│   ├── securebank/      · siem-lite/   · audittrail/
│   └── gatekeeper/      · helpdesk/    · linkshield/
├── python/              (blueprints: ferramentas de segurança + IA — FastAPI)
│   ├── threatscope/     · portpulse/   · filesentry/
│   └── loglens/         · phishguard/
└── ruby/                (blueprints: produtos web — Rails)
    ├── devforum/        · publicms/    · flowboard/    · chatroom/
    └── edupath/         · agendaja/    · rachaconta/
```

Cada pasta de blueprint tem um **`CLAUDE.md`**: o handoff completo (o que é, stack,
modelo de dados, funcionalidades, segurança e plano de build). Quando o projeto começa,
esse `CLAUDE.md` vai para o repositório dele e a pasta sai daqui (receita na
[Lição 02](tutorial/02-um-repositorio-por-projeto.md)).

## Repositórios dos projetos

| Projeto | Linguagem | Repositório | Status |
|---|---|---|---|
| DocSage | Python | [Everett-gi/docsage](https://github.com/Everett-gi/docsage) | ✅ Completo — referência de qualidade para os outros |
| PwnCheck | Python | [Everett-gi/pwncheck](https://github.com/Everett-gi/pwncheck) | 🚧 Em construção (fase 1 de 6) |
| AuthHub | Java | [Everett-gi/authhub](https://github.com/Everett-gi/authhub) | 📋 Blueprint no repositório — construção começando |
| SecVault | Java | [Everett-gi/secvault](https://github.com/Everett-gi/secvault) | 📋 Blueprint no repositório — construção começando |
| ReconKit | Python | [Everett-gi/reconkit](https://github.com/Everett-gi/reconkit) | 📋 Blueprint no repositório — construção começando |
| MercadoLite | Ruby | [Everett-gi/mercadolite](https://github.com/Everett-gi/mercadolite) | 📋 Blueprint no repositório — construção começando |
| Os outros 18 | — | — | 📋 Blueprint pronto, nesta central |

Esses seis são os **projetos principais** — um por slot de repositório fixado no perfil do
GitHub — e cobrem autenticação, criptografia, OSINT, privacidade, IA e e-commerce nas três
linguagens. O `CLAUDE.md` de cada um é autossuficiente (blueprint, base de segurança,
convenções e modo tutorial), pronto para ser trabalhado em qualquer ambiente, inclusive na
nuvem.

Clone os projetos em `C:\dev`, **fora do OneDrive** (motivos na
[Lição 00](tutorial/00-ambiente.md#9-onedrive-uma-recomendação)):

```powershell
cd C:\dev
git clone https://github.com/Everett-gi/docsage.git     # fica em C:\dev\docsage
```

## Trilha de aprendizado

Os projetos são construídos em **modo tutorial**: cada fase concluída vira uma lição
explicando o que foi feito e por quê. O índice completo, com a ordem de estudo e o progresso,
está em **[tutorial/README.md](tutorial/README.md)**. Os tutoriais disponíveis:

| Tutorial | Repositório | Para quê |
|---|---|---|
| [00 — Ambiente](tutorial/00-ambiente.md) | central | Python, venv, pip, VS Code, Git — o que foi instalado e como funciona |
| [01 — Python para quem vem do C/C++](tutorial/01-python-para-quem-vem-do-c.md) | central | o modelo mental do Python a partir do C/C++, com exercícios |
| [02 — Um repositório por projeto](tutorial/02-um-repositorio-por-projeto.md) | central | a organização do portfólio, `git subtree` e GitHub CLI |
| [PwnCheck — Fase 1: cliente k-anonymity](https://github.com/Everett-gi/pwncheck/blob/main/docs/tutorial/fase-1-k-anonymity.md) | pwncheck | hashing, `str` × `bytes`, pytest, mocks de rede, ruff e CI |
| [DocSage — Passo a passo](https://github.com/Everett-gi/docsage/blob/main/docs/PASSO-A-PASSO.md) | docsage | **usar o DocSage**: WSL 2 + Docker, chave da API, `.env`, rodar e testar |
| [DocSage — Deploy](https://github.com/Everett-gi/docsage/blob/main/docs/DEPLOY.md) | docsage | colocar um projeto no ar na Oracle Cloud (grátis), com HTTPS — serve para todos |

## Como cada projeto é construído (fluxo)

1. **Ler o `CLAUDE.md`** do projeto — ele é o mapa.
2. **Criar o repositório do projeto** e o scaffold, seguindo o DocSage e o PwnCheck como
   referência (receita na [Lição 02](tutorial/02-um-repositorio-por-projeto.md)). Cada
   arquivo de configuração é explicado na lição correspondente — nada de copiar template
   sem entender.
3. **Construir com o Claude Code** no repositório do projeto, seguindo as fases do build.
4. **Testar** localmente (`pytest`, depois `docker compose up`) e versionar.
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
- Commits no padrão *Conventional Commits*: `feat: ...`, `fix: ...`, `docs: ...`,
  `test: ...`, `ci: ...`, `chore: ...`.
- **Um repositório por projeto**, com o CI em `.github/workflows/ci.yml` e um `CLAUDE.md`
  que traz o blueprint, as convenções e as regras do modo tutorial — cada repositório tem que
  fazer sentido sozinho.

## Sugestão de ordem

**Carros-chefe primeiro** (1 por linguagem, completos e no ar):
[DocSage](https://github.com/Everett-gi/docsage) ✅ → [AuthHub](https://github.com/Everett-gi/authhub) → [MercadoLite](https://github.com/Everett-gi/mercadolite).
Em paralelo, a trilha Python de aprendizado começou pelo [PwnCheck](https://github.com/Everett-gi/pwncheck).

Depois, o restante por afinidade — Java para aprofundar segurança de backend,
Python para o ferramental de segurança, Ruby para produtos web.
