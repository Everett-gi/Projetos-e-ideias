# Projetos e ideias — contexto do repositório

> Lido automaticamente pelo Claude Code. Convenções gerais no `README.md`;
> cada projeto tem o próprio `CLAUDE.md` com o blueprint e o plano de build.

## Modo tutorial (vale para todo o repositório)

Este repositório é também a trilha de aprendizado do autor. Ele vem de **C e C++**
(domina lógica, ponteiros, memória, compilação) e está aprendendo Python — depois
virão Java e Ruby.

- Explique cada passo e o **porquê**. Use analogias com C/C++ quando ajudarem
  (ex.: `with` ≈ RAII, `list` ≈ `std::vector`, `bytes` ≈ `unsigned char[]`).
- Não explique lógica de programação básica; foque no que é novo: idiomas da
  linguagem, bibliotecas, ferramentas, arquitetura e segurança.
- Cada fase concluída de um projeto ganha uma lição em `<projeto>/docs/tutorial/`
  e uma entrada no índice `tutorial/README.md` (o progresso é marcado lá).
- Termine cada lição com exercícios. Deixe o autor rodar os comandos sempre que possível.

## Organização: central + um repositório por projeto

- Este repositório é a **central**: blueprints dos projetos que ainda não começaram,
  convenções (README), DEPLOY-GERAL e a trilha (`tutorial/`). Não há código aqui.
- Projeto que começa ganha **repositório próprio** (`Everett-gi/<projeto>`), clonado em
  `C:\dev\<projeto>` (fora do OneDrive). O `CLAUDE.md` do blueprint vai para lá, ganha as
  seções de modo tutorial e convenções, e a pasta sai daqui. Receita em
  `tutorial/02-um-repositorio-por-projeto.md`.
- Repositórios dos 6 projetos principais (tabela no README): `docsage` (referência de
  qualidade), `pwncheck`, `authhub`, `secvault`, `reconkit` e `mercadolite`, todos em
  `https://github.com/Everett-gi/<projeto>`. O `CLAUDE.md` de cada um é autossuficiente.
- Lições de cada fase ficam em `docs/tutorial/` do repositório do projeto; o índice
  `tutorial/README.md` daqui aponta para elas com URLs.
- GitHub CLI (`gh`) instalado e logado. O token dele não tem o escopo `workflow`: faça o push
  com `git push` (Credential Manager), não com `gh repo create --push`.
- Commits: Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `ci:`, `chore:`).

## Ambiente do autor

Windows 11 · PowerShell · VS Code · Python 3.12 (python.org, com o launcher `py`) · Git.
Comandos em lições e respostas: sintaxe PowerShell.
