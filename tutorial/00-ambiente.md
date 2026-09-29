# Lição 00 — Ambiente: Python, venv, pip, VS Code e Git

> **Objetivo:** entender cada ferramenta instalada e configurada, o que ela faz por baixo
> dos panos e como usá-la no dia a dia. Tudo aqui foi feito na primeira sessão; esta lição
> é o registro, com as explicações.

**Sumário**
1. [O que foi instalado](#1-o-que-foi-instalado)
2. [Como o Python executa seu código](#2-como-o-python-executa-seu-código)
3. [A instalação do Python no Windows](#3-a-instalação-do-python-no-windows)
4. [pip e PyPI: instalando bibliotecas](#4-pip-e-pypi-instalando-bibliotecas)
5. [Ambientes virtuais (venv)](#5-ambientes-virtuais-venv)
6. [requirements.txt: as dependências do projeto](#6-requirementstxt-as-dependências-do-projeto)
7. [VS Code](#7-vs-code)
8. [Git e o monorepo](#8-git-e-o-monorepo)
9. [OneDrive: uma recomendação](#9-onedrive-uma-recomendação)
10. [Exercícios](#10-exercícios)

---

## 1. O que foi instalado

| Ferramenta | Versão | Papel | Equivalente em C/C++ |
|---|---|---|---|
| Python (CPython) | 3.12.10 | interpretador da linguagem | compilador + runtime |
| pip | 26.2 | instala pacotes do PyPI | vcpkg / Conan |
| venv | vem com o Python | ambiente isolado por projeto | uma pasta `third_party/` por projeto |
| Extensão Python + Pylance | — | autocompletar, checagem de tipos, navegação | IntelliSense / clangd |
| Python Debugger (debugpy) | — | breakpoints, execução passo a passo | gdb / depurador do Visual Studio |
| Ruff (extensão + pacote) | 0.16 | lint e formatação | clang-tidy + clang-format |
| pytest (pacote) | 9.1 | testes automatizados | GoogleTest / Catch2 |

---

## 2. Como o Python executa seu código

Em C você compila e linka (`gcc main.c -o main`): sai código de máquina, e o executável
roda sem o compilador. No Python é diferente:

```
arquivo.py ──(compila)──> bytecode (.pyc) ──(interpreta)──> máquina virtual do Python
```

- O `python.exe` é o **CPython**, a implementação oficial da linguagem — um programa escrito
  **em C**. Ele compila o seu `.py` para *bytecode* (instruções de uma máquina virtual,
  parecido com o da JVM) e executa esse bytecode num grande laço escrito em C.
- O bytecode fica em cache nas pastas `__pycache__/` (arquivos `.pyc`) para não recompilar
  sem necessidade. Por isso elas estão no `.gitignore`: são artefato de build, como os `.o`.
- **Não existe link nem executável final.** Para rodar um programa Python, a máquina precisa
  do interpretador. (É um dos motivos de usarmos Docker em produção: a imagem leva o
  interpretador junto.)
- **Consequência importante:** a maioria dos erros que o compilador C pegaria (nome errado,
  tipo errado) só aparece **quando a linha executa**. Quem faz o papel do compilador são as
  ferramentas de análise — o **Pylance** no editor e o **ruff** no terminal e no CI — e,
  principalmente, os **testes**.

**Curiosidade para quem vem de C:** a pasta de instalação tem um `include\Python.h` — é a
API em C do interpretador, usada para escrever extensões nativas. Muita biblioteca Python é
C por dentro. O `hashlib`, que usamos no PwnCheck, é o arquivo `DLLs\_hashlib.pyd` — e um
`.pyd` é simplesmente uma **DLL** — que chama o **OpenSSL 3.0.16**.

---

## 3. A instalação do Python no Windows

O comando usado (o `winget` é o gerenciador de pacotes do Windows, como o `apt` no Linux):

```powershell
winget install -e --id Python.Python.3.12 --source winget --scope user `
  --accept-package-agreements --accept-source-agreements `
  --override "/quiet InstallAllUsers=0 PrependPath=1 Include_test=0 Include_launcher=1 InstallLauncherAllUsers=0"
```

| Parte | Significado |
|---|---|
| `-e --id Python.Python.3.12` | pacote exato, sem busca aproximada |
| `--scope user` | instala só para o seu usuário: não precisa de administrador |
| `--accept-...-agreements` | aceita a licença (PSF-2.0) sem perguntar |
| `--override "..."` | argumentos repassados direto ao instalador oficial do python.org: |
| ↳ `/quiet` | sem janelas |
| ↳ `InstallAllUsers=0` | instala em `%LOCALAPPDATA%\Programs\Python\Python312` |
| ↳ `PrependPath=1` | coloca o Python **no começo** do PATH do usuário |
| ↳ `Include_test=0` | pula a suíte de testes do próprio Python (economiza espaço) |
| ↳ `Include_launcher=1 InstallLauncherAllUsers=0` | instala o launcher `py`, só para você |

**Por que 3.12 e não a versão mais nova?** Porque é a versão definida para todos os projetos
Python do portfólio: a mesma do `Dockerfile` (`python:3.12-slim`) e do CI. Mesma versão em
todo lugar significa menos "funciona na minha máquina". O 3.12.10 foi a última versão do 3.12
com instalador para Windows; hoje o 3.12 só recebe correções de segurança, o que não afeta
o nosso dia a dia.

### A armadilha da Microsoft Store

Antes da instalação, o comando `python` já "existia": era um **atalho da Microsoft Store**
(`...\WindowsApps\python.exe`) que só abre a loja. Veja a ordem de busca agora:

```powershell
where.exe python
# C:\Users\gmnas\AppData\Local\Programs\Python\Python312\python.exe   <- o real (primeiro)
# C:\Users\gmnas\AppData\Local\Microsoft\WindowsApps\python.exe       <- o atalho
```

O Windows procura executáveis nas pastas do `PATH` **em ordem** e usa o primeiro que achar,
igual ao Linux. O `PrependPath=1` colocou o Python real na frente do atalho.

> ⚠️ O PATH é lido quando o terminal abre: cada processo herda o ambiente do processo pai
> (como o `envp` do `main` em C). Um terminal aberto **antes** da instalação não enxerga o
> Python. Feche e abra de novo.

### O launcher `py`

```powershell
py --list            # lista as versões de Python instaladas
py -3.12 --version   # roda uma versão específica
```

Ajuda quando houver várias versões instaladas. No dia a dia, dentro de um projeto, você vai
usar o `python` do ambiente virtual (seção 5).

---

## 4. pip e PyPI: instalando bibliotecas

- **PyPI** (pypi.org) é o repositório central de pacotes Python.
- **pip** é o instalador: baixa do PyPI e instala numa pasta chamada `site-packages`.

```powershell
python -m pip install httpx       # instala
python -m pip list                # lista o que está instalado
python -m pip show httpx          # detalhes: versão, dependências, onde está
python -m pip uninstall httpx     # remove
```

**Por que `python -m pip` e não só `pip`?** O `-m` roda o módulo `pip` *do interpretador que
você chamou*. Com várias instalações de Python na máquina (a global, a de cada venv...), o
`pip` sozinho pode ser de uma e o `python` de outra, e o pacote vai parar no lugar errado.
Com `python -m pip` isso não acontece.

Os pacotes geralmente vêm como **wheels** (`.whl`): um zip pronto para instalar, que pode
incluir código nativo **pré-compilado** para o seu sistema. Por isso você não precisou de um
compilador C para instalar nada — o `ruff.exe` do seu projeto, por exemplo, é um binário
nativo de 26 MB (o ruff é escrito em Rust).

---

## 5. Ambientes virtuais (venv)

### O problema

Sem venv, todo pacote é instalado no `site-packages` global, compartilhado por **todos** os
projetos. Se o PwnCheck precisar do httpx 0.28 e outro projeto do 0.23, um quebra o outro.
É o "DLL hell": o mesmo que todos os projetos C da máquina linkarem contra a mesma versão de
uma biblioteca em `/usr/lib`.

### A solução

Um **venv** é uma pasta com um `site-packages` exclusivo daquele projeto:

```powershell
cd python\pwncheck
python -m venv .venv      # cria o ambiente na pasta .venv
```

```
.venv\
├── pyvenv.cfg            <- aponta para o Python "de verdade" (home = ...\Python312)
├── Scripts\
│   ├── python.exe        <- lançador: usa o Python base + as bibliotecas deste venv
│   ├── Activate.ps1      <- script de ativação para PowerShell
│   └── pip.exe, pytest.exe, ruff.exe...   <- comandos dos pacotes instalados
└── Lib\site-packages\    <- as bibliotecas DESTE projeto
```

O `pyvenv.cfg` do seu PwnCheck (abra e confira):

```ini
home = C:\Users\gmnas\AppData\Local\Programs\Python\Python312
include-system-site-packages = false
version = 3.12.10
```

A linha `include-system-site-packages = false` é o isolamento: este ambiente **não enxerga**
os pacotes globais.

### Ativar — e o que isso realmente faz

```powershell
.\.venv\Scripts\Activate.ps1   # o prompt ganha o prefixo (.venv)
python --version               # agora "python" é o do .venv
deactivate                     # volta ao normal
```

Ativar **não tem mágica**: o script coloca `.venv\Scripts` no **começo do PATH**, define a
variável `VIRTUAL_ENV` e muda o prompt. Por isso, depois de ativar, `python`, `pip`, `pytest`
e `ruff` passam a ser os do projeto. Sem ativar, dá no mesmo chamar pelo caminho completo:
`.\.venv\Scripts\python.exe -m pytest`.

> ⚠️ **Apareceu "a execução de scripts foi desabilitada neste sistema"?** Você está no
> *Windows PowerShell 5.1* (ícone azul), que por padrão bloqueia scripts `.ps1`. Na sua
> máquina, o **PowerShell 7** (perfil "PowerShell", ícone preto, no Windows Terminal — e o
> terminal padrão do VS Code) já permite scripts locais. Alternativas:
> 1. Usar o PowerShell 7 (recomendado).
> 2. Não ativar: chamar `.\.venv\Scripts\python.exe` direto.
> 3. Liberar scripts locais para o seu usuário: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.
>    É uma configuração de **segurança** do Windows, então a decisão é sua: ela permite rodar
>    scripts criados na sua máquina e continua exigindo assinatura digital dos baixados da internet.

### Regras do venv

- **Um `.venv` por projeto**, dentro da pasta do projeto (`python/pwncheck/.venv`).
- **Nunca vai para o Git** (está no `.gitignore`): ele é recriável a partir do
  `requirements.txt`, como uma pasta `build/`. Quebrou? Apague a pasta e recrie.
- O `.venv` do PwnCheck tem ~59 MB e ~2.200 arquivos. Guarde esse número para a seção 9.

---

## 6. requirements.txt: as dependências do projeto

```
requirements.txt        -> o que a APLICAÇÃO precisa para rodar (vai para produção)
requirements-dev.txt    -> tudo acima + ferramentas de desenvolvimento (pytest, ruff)
```

- A linha `-r requirements.txt` dentro do `-dev` funciona como um `#include`.
- `httpx==0.28.*` aceita qualquer 0.28.x (correções), mas não a 0.29, que pode mudar a API.
- **Dependências transitivas:** pedimos 3 pacotes (httpx, pytest e ruff) e o pip instalou 15.
  O httpx precisa de `httpcore`, `h11`, `anyio`, `certifi`, `idna`... Veja com
  `python -m pip list`. É uma árvore de dependências, como uma lib C que depende de outras libs.

Para recriar o ambiente do zero, em qualquer máquina:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements-dev.txt
```

---

## 7. VS Code

Extensões instaladas:

| Extensão | O que faz |
|---|---|
| **Python** (`ms-python.python`) | integração com o interpretador, os venvs e os testes |
| **Pylance** | autocompletar, "ir para definição", checagem de tipos pelos *type hints* |
| **Python Debugger** | breakpoints e execução passo a passo (F5) |
| **Python Environments** | gerencia ambientes virtuais pela interface |
| **Ruff** (`charliermarsh.ruff`) | mostra os avisos do lint enquanto você digita e formata o código |

**Abra a pasta do PROJETO, não a raiz do repositório:**

```powershell
code "C:\Users\gmnas\OneDrive\Documentos\Projetos e ideias\python\pwncheck"
```

Assim o VS Code encontra o `.venv` sozinho. Confira no canto inferior direito se aparece
`3.12.10 ('.venv')`; se não aparecer: `Ctrl+Shift+P` → **Python: Select Interpreter** →
escolha o do `.venv`.

**Testes pela interface:** ícone do frasco (*Testing*) na barra lateral → *Configure Python
Tests* → **pytest** → pasta `tests`. Dá para rodar e depurar cada teste com um clique.

**Formatar ao salvar (recomendado):** `Ctrl+Shift+P` → *Preferences: Open User Settings
(JSON)* e acrescente:

```json
"[python]": {
  "editor.defaultFormatter": "charliermarsh.ruff",
  "editor.formatOnSave": true
}
```

---

## 8. Git e o monorepo

### O que foi feito

```powershell
git init                          # cria o repositório (a pasta oculta .git)
git add .                         # coloca tudo na "staging area" (o próximo commit)
git status                        # CONFERE o que vai entrar — sempre, antes de commitar
git commit -m "chore: ..."        # grava o snapshot
git remote add origin https://github.com/Everett-gi/Projetos-e-ideias.git
git push -u origin main           # envia; o -u associa main a origin/main
```

No primeiro `push`, o **Git Credential Manager** (vem com o Git para Windows) abre o navegador
para você autorizar o acesso ao GitHub. O token fica guardado no Gerenciador de Credenciais
do Windows, e os próximos pushes não pedem nada.

### `.gitignore`

Lista o que o Git deve fingir que não existe. Os itens mais importantes do nosso:

| Padrão | Por quê |
|---|---|
| `.env`, `*.key`, `*.pem` | **segredos**. Num repositório público, um segredo commitado deve ser considerado vazado em minutos: robôs varrem o GitHub o tempo todo. |
| `!.env.example` | o `!` abre uma exceção: o *modelo*, com valores falsos, pode ir |
| `.venv/` | recriável pelo `requirements.txt` |
| `__pycache__/`, `*.pyc` | bytecode — artefato de build, como `.o` |
| `.pytest_cache/`, `.ruff_cache/` | caches das ferramentas |

### `.gitattributes` e os finais de linha

O Windows termina linhas com `\r\n` (CRLF); o Linux, com `\n` (LF). É a mesma conversão que
o `fopen` em **modo texto** faz em C no Windows. Nossos projetos rodam em Linux (Docker, CI,
servidor), e um script com CRLF quebra lá (`/bin/sh^M: bad interpreter`). A linha
`* text=auto eol=lf` faz o Git **guardar tudo com LF**, mesmo que algum editor do Windows
salve com CRLF. Os `.ps1` são exceção: precisam de CRLF para rodar no Windows.

### Monorepo

Você escolheu **um repositório para todos os projetos** (monorepo), em vez de um repositório
por projeto (polyrepo).

| | Monorepo (o nosso) | Um repositório por projeto |
|---|---|---|
| Visão do portfólio | tudo num lugar só | espalhado |
| Configuração compartilhada | feita uma vez (`.gitignore`, convenções) | repetida em cada repositório |
| CI | um workflow por projeto, na raiz, filtrado por pasta | um por repositório, mais simples |
| Histórico | mistura commits de todos os projetos | separado |

A consequência prática: o **GitHub só lê workflows em `.github/workflows/`, na raiz**. O CI do
DocSage estava em `python/docsage/.github/` e nunca rodaria. Ele foi movido para
`.github/workflows/docsage-ci.yml`, com um filtro `paths:` para rodar só quando a pasta do
DocSage muda. (O CI é explicado em detalhe na lição da fase 1 do PwnCheck.)

> **Atualização:** depois, o DocSage ganhou repositório próprio,
> [Everett-gi/docsage](https://github.com/Everett-gi/docsage). Lá, a raiz do repositório
> **é** a raiz do projeto, então o CI voltou a ser um `.github/workflows/ci.yml` simples,
> sem filtro `paths:`. O `docsage-ci.yml` saiu do monorepo. É a tabela acima na prática:
> num repositório por projeto, o CI fica mais simples.

### Conventional Commits

Todas as mensagens de commit seguem o formato `tipo(escopo): descrição`:

| Tipo | Quando usar |
|---|---|
| `feat` | funcionalidade nova |
| `fix` | correção de bug |
| `test` | só testes |
| `docs` | só documentação |
| `ci` | pipeline de CI |
| `chore` | manutenção (configurações, estrutura) |

O escopo é o projeto: `feat(pwncheck): cliente k-anonymity`. Veja o histórico com
`git log --oneline`.

### O e-mail dos commits

Todo commit grava o nome e o e-mail do autor, e num repositório público isso fica visível
para sempre (qualquer um vê com `git log`). O seu Gmail já aparece nos commits de outros
repositórios seus, então usá-lo aqui não expõe nada novo. O risco de um e-mail público são os
robôs que coletam endereços do GitHub para spam e phishing direcionado.

Se um dia quiser parar de expor o e-mail **daqui para frente** (os commits antigos ficam como
estão):

1. GitHub → *Settings → Emails* → marque **Keep my email addresses private**.
2. No repositório: `git config user.email "185654721+Everett-gi@users.noreply.github.com"`.
   Sem `--global`, a configuração vale só para este repositório.

### Proteções do GitHub (faça você, no site)

No repositório, abra **Settings** e procure a seção de segurança (*Code security* ou
*Advanced Security*, conforme a versão da interface). Confirme que estão ativos:

- **Secret scanning** e **Push protection** — bloqueiam o push se detectarem uma chave.
- **Dependabot alerts** — avisam sobre dependências com vulnerabilidades conhecidas.

---

## 9. OneDrive: uma recomendação

A pasta `Documentos` do seu Windows fica dentro do **OneDrive**, que sincroniza com a nuvem
cada arquivo alterado. Com código, isso atrapalha:

- cada `.venv` tem milhares de arquivos (o do PwnCheck: ~2.200 arquivos, 59 MB), e os projetos
  de machine learning terão ambientes bem maiores;
- o OneDrive pode **travar um arquivo** enquanto sincroniza, e aí o `pip` ou o `git` falham com
  "arquivo em uso" (`WinError 32`);
- sincronizar a pasta `.git` no meio de uma operação pode corromper o repositório.

Como tudo agora está no GitHub, **o GitHub já é o seu backup**. Quando quiser, passe a
trabalhar numa pasta fora do OneDrive:

```powershell
mkdir C:\dev
cd C:\dev
git clone https://github.com/Everett-gi/Projetos-e-ideias.git
```

Depois recrie o `.venv` de cada projeto lá (seção 6). A pasta antiga pode ser apagada.

---

## 10. Exercícios

1. Num terminal **novo**, rode `where.exe python` e `py --list`. Explique por que o atalho da
   Microsoft Store não é mais usado.
2. Abra o `.venv\pyvenv.cfg` do PwnCheck. O que mudaria se `include-system-site-packages`
   fosse `true`?
3. Crie um ambiente descartável e destrua-o, para perder o medo:
   ```powershell
   cd $env:TEMP
   python -m venv teste-venv
   .\teste-venv\Scripts\python.exe -m pip install requests
   .\teste-venv\Scripts\python.exe -m pip list      # quantos pacotes vieram junto?
   Remove-Item -Recurse -Force teste-venv
   ```
4. Ative o `.venv` do PwnCheck e rode `Get-Command python`. Rode `deactivate` e repita.
   Compare os dois resultados.
5. Na raiz do repositório, rode `git log --stat` e leia cada commit: o que entrou em cada um?
6. No GitHub, abra a aba **Actions** do repositório e acompanhe os workflows depois do push.

<details>
<summary>Respostas</summary>

1. O instalador colocou as pastas do Python **antes** da pasta `WindowsApps` no PATH do
   usuário. O Windows usa o primeiro `python.exe` que encontra.
2. O venv passaria a enxergar também os pacotes do `site-packages` global. Isso quebra o
   isolamento: o projeto poderia funcionar na sua máquina só porque um pacote está instalado
   globalmente, e falhar no CI.
3. O `requests` traz 4 dependências (`certifi`, `charset-normalizer`, `idna` e `urllib3`).
   Com o próprio `pip`, a lista mostra 6 pacotes.
4. Ativado: `...\pwncheck\.venv\Scripts\python.exe`. Depois do `deactivate`: o Python global,
   em `...\Programs\Python\Python312\python.exe`. A ativação só mexe no PATH.
5. Veja você mesmo: a ideia é se acostumar a ler histórico.
6. O workflow PwnCheck CI deve aparecer com ✅. (O DocSage CI agora roda na aba Actions do
   repositório próprio dele, `Everett-gi/docsage`.)

</details>
