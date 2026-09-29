# Trilha de aprendizado

Este repositório é construído em **modo tutorial**: cada fase de cada projeto vira uma lição
que explica o que foi feito e **por quê**. As lições assumem que você já programa em C/C++ —
lógica de programação não é explicada; o foco é no que é novo: a linguagem, as bibliotecas,
as ferramentas, a arquitetura e a segurança.

## Como estudar cada lição

1. **Leia a lição inteira** antes de mexer no código: ela explica o raciocínio.
2. **Abra o código ao lado** e confira cada trecho explicado.
3. **Rode os comandos você mesmo** (os blocos `powershell`).
4. **Faça os exercícios do fim.** É ali que o aprendizado fixa.

## Fundamentos

| # | Lição | Status |
|---|---|---|
| 00 | [Ambiente: Python, venv, pip, VS Code e Git](00-ambiente.md) | ✅ |
| 01 | [Python para quem vem do C/C++](01-python-para-quem-vem-do-c.md) | ✅ |

## Trilha Python — projetos em ordem de estudo

A ordem foi pensada para que cada projeto traga poucos conceitos novos e reaproveite os
anteriores.

| # | Projeto | O que ele ensina de novo | Status |
|---|---|---|---|
| 1 | **PwnCheck** | hashlib, bytes × str, pytest, mocks, httpx, FastAPI, Pydantic, SQLAlchemy, Alembic, Docker | 🚧 fase 1 de 6 |
| 2 | FileSentry | pathlib, leitura de arquivos em blocos, threads (watchdog), API externa com chave | 📋 |
| 3 | ThreatScope | async/await com httpx, agendamento (APScheduler), upsert, paginação, API keys | 📋 |
| 4 | ReconKit | DNS e WHOIS, cabeçalhos HTTP de segurança, arquitetura modular, relatórios | 📋 |
| 5 | PortPulse | asyncio a fundo, sockets TCP, semáforos, timeouts, interface web com HTMX | 📋 |
| 6 | PhishGuard | machine learning: features, scikit-learn, avaliação e versionamento de modelo | 📋 |
| 7 | LogLens | pandas, estatística, detecção de anomalias (Isolation Forest), dashboard | 📋 |
| — | [DocSage](https://github.com/Everett-gi/docsage) | já está pronto (repositório próprio): vira um estudo guiado do código depois do PwnCheck (RAG, pgvector, LLM) | ✅ pronto |

**Por que essa ordem?** O PwnCheck tem o menor escopo e passa pela pilha inteira uma vez
(API, banco, Docker, CI). O FileSentry reaproveita os hashes e acrescenta arquivos e threads.
O ThreatScope introduz programação assíncrona, que o ReconKit usa e o PortPulse leva ao
limite com sockets. Por último vêm os dois de machine learning, que pedem mais
conceitos novos de uma vez.

### PwnCheck

| Fase | Lição | Status |
|---|---|---|
| 1 | [Cliente k-anonymity: hashing, testes e mocks](../python/pwncheck/docs/tutorial/fase-1-k-anonymity.md) | ✅ |
| 2 | Política de força de senha | ⬜ |
| 3 | Cache de prefixos: PostgreSQL, SQLAlchemy, Alembic e Docker | ⬜ |
| 4 | API REST com FastAPI + autenticação | ⬜ |
| 5 | Rate limit e métricas | ⬜ |
| 6 | Deploy com HTTPS | ⬜ |

## Pendências de ambiente

- **Docker Desktop** — necessário a partir da fase 3 do PwnCheck. Ele depende do WSL 2
  (Linux dentro do Windows), que ainda não está instalado nesta máquina. Instalamos juntos
  quando chegar a hora.
