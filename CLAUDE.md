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

## Monorepo

- CI: um workflow por projeto em `.github/workflows/<projeto>-ci.yml`, com filtro
  `paths:` e `defaults.run.working-directory`. O GitHub ignora `.github/` em subpastas.
- Python: cada projeto tem o próprio `.venv` dentro da pasta do projeto.
- Commits: Conventional Commits com escopo, ex.: `feat(pwncheck): ...`.

## Ambiente do autor

Windows 11 · PowerShell · VS Code · Python 3.12 (python.org, com o launcher `py`) · Git.
Comandos em lições e respostas: sintaxe PowerShell.
