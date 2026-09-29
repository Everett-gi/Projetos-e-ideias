# Lição 02 — Um repositório por projeto: git subtree e GitHub CLI

> **Objetivo:** entender por que o portfólio trocou o monorepo por um repositório por projeto,
> como extrair um projeto **sem perder a história** dos commits e como criar repositórios pelo
> terminal com o GitHub CLI. No fim, uma receita para começar cada projeto novo.

**Sumário**
1. [A decisão](#1-a-decisão)
2. [Extraindo um projeto com a história: `git subtree split`](#2-extraindo-um-projeto-com-a-história-git-subtree-split)
3. [O que não veio junto, e por quê](#3-o-que-não-veio-junto-e-por-quê)
4. [GitHub CLI (`gh`)](#4-github-cli-gh)
5. [Receita: começar um projeto novo](#5-receita-começar-um-projeto-novo)
6. [Exercícios](#6-exercícios)

---

## 1. A decisão

No começo, todos os projetos moravam num só repositório (monorepo; veja a Lição 00, seção 8).
Logo na primeira fase, decidimos trocar para **um repositório por projeto**:

| Motivo | Explicação |
|---|---|
| **O portfólio é visto pelo seu perfil** | No GitHub você fixa até 6 repositórios. Cada um aparece com nome, descrição, linguagem e README próprios. Num monorepo, o PwnCheck ficaria escondido em `python/pwncheck/`. |
| **Os projetos não compartilham código** | Monorepo compensa quando vários projetos usam as mesmas bibliotecas internas. Os nossos são independentes: pagaríamos o custo sem o benefício. |
| **CI e deploy mais simples** | Sem filtro de `paths` nem `working-directory` (o filtro chegou a impedir o CI do DocSage de rodar). No servidor, clona-se só o projeto que vai ao ar. |

Duas regras vieram junto:

- **Repositório só quando o projeto começa.** Vinte e dois repositórios só com um blueprint
  pareceriam abandonados. O próprio README diz: *4 projetos completos e no ar valem mais que 24
  inacabados*.
- **Um repositório por linguagem foi descartado:** continuaria sendo um monorepo por dentro, e
  agrupar por linguagem não diz nada a quem avalia o portfólio.

### Como ficou

```
GitHub                                   Sua máquina
──────                                   ───────────
Everett-gi/Projetos-e-ideias  (central)  C:\Users\gmnas\OneDrive\Documentos\Projetos e ideias
Everett-gi/docsage                       C:\dev\docsage       (clone quando for estudá-lo)
Everett-gi/pwncheck                      C:\dev\pwncheck
Everett-gi/authhub                       C:\dev\authhub
Everett-gi/secvault                      C:\dev\secvault
Everett-gi/reconkit                      C:\dev\reconkit
Everett-gi/mercadolite                   C:\dev\mercadolite
```

Os seis são os **projetos principais**: um para cada repositório que o GitHub deixa fixar no
perfil. Os quatro últimos foram criados logo depois do PwnCheck, com o mesmo `subtree split`,
para serem construídos em sessões do Claude Code na nuvem. Por isso o `CLAUDE.md` de cada um
é **autossuficiente**: traz o blueprint, a base de segurança, as convenções e o modo tutorial,
sem depender de nada da central.

O **Projetos-e-ideias** virou a central do portfólio: guarda os blueprints dos projetos que
ainda não começaram, as convenções (README), o guia de deploy e esta trilha de aprendizado.
Quando um projeto começa, ele ganha repositório próprio e sai da central.

Os projetos com código ficam em `C:\dev`, **fora do OneDrive** (Lição 00, seção 9). A central
pode continuar no OneDrive: só tem Markdown, sem `.venv`.

---

## 2. Extraindo um projeto com a história: `git subtree split`

Copiar a pasta `python/pwncheck` para um repositório novo funcionaria, mas perderia a história:
o primeiro commit do repositório novo seria "tudo de uma vez". O Git consegue fazer melhor.

### Um pouco de como o Git guarda as coisas

- Um **commit** aponta para uma **árvore** (*tree*: o snapshot das pastas e arquivos), para o
  commit anterior (o *pai*) e guarda autor, data e mensagem.
- O **hash** do commit (ex.: `4e24532`) é calculado sobre tudo isso. Mude qualquer coisa — um
  caminho de arquivo, o pai — e o hash muda. É como um checksum encadeado: cada commit "assina"
  toda a história até ele.

### O comando

```powershell
cd "C:\Users\gmnas\OneDrive\Documentos\Projetos e ideias"
git subtree split --prefix=python/pwncheck -b pwncheck-split
```

O que ele faz:

1. Percorre a história e seleciona **só os commits que tocaram em `python/pwncheck`**.
2. Para cada um, cria um commit **novo**, cuja árvore é só o conteúdo daquela pasta — como se
   `python/pwncheck` fosse a raiz.
3. Encadeia os commits novos e cria o branch `pwncheck-split` apontando para o último.

É como extrair uma biblioteca de dentro de um projeto C grande para distribuí-la separada,
levando junto o histórico de commits dela. O resultado:

```
e6365a5 fix(pwncheck): tira docs/ do alcance do ruff e corrige trecho da lição
723d32b docs: trilha de aprendizado — lições 00, 01 e fase 1 do PwnCheck
64f00b0 feat(pwncheck): cliente k-anonymity com testes (fase 1)
9e1c293 chore: importa os blueprints do portfólio (Java, Python, Ruby)
```

Os 4 commits do monorepo que mexeram na pasta vieram com autor, data e mensagem originais —
mas **com hashes novos**: o `feat(pwncheck)` era `4e24532` no monorepo e virou `64f00b0`.
Os caminhos mudaram, então o conteúdo mudou, então o hash mudou.

### Do branch para uma pasta nova

```powershell
git clone --branch pwncheck-split --single-branch "C:\Users\gmnas\OneDrive\Documentos\Projetos e ideias" C:\dev\pwncheck
cd C:\dev\pwncheck
git branch -m pwncheck-split main     # renomeia o branch para main
git remote remove origin              # o "origin" apontava para a pasta da central
```

- **`git clone` aceita uma pasta local como origem**, não só uma URL. Para o Git, um "remote"
  é só um lugar de onde buscar commits: pode ser o GitHub, um servidor SSH ou outra pasta.
- `--single-branch` traz só o branch extraído, sem o resto do monorepo.

> Para reescritas mais complexas (tirar um arquivo de toda a história, juntar várias pastas),
> a ferramenta recomendada é o `git filter-repo`, que precisa ser instalado à parte. Para
> extrair uma pasta, o `subtree`, que já vem com o Git para Windows, basta.

---

## 3. O que não veio junto, e por quê

O `subtree split` só traz o que estava **dentro** da pasta. Três coisas moravam na raiz do
monorepo e precisaram ser recriadas no repositório novo:

| Arquivo | O que mudou |
|---|---|
| `.gitignore` | versão só com o que um projeto Python precisa (segredos, `.venv`, caches) |
| `.gitattributes` | igual ao da central (tudo com LF) |
| `.github/workflows/ci.yml` | **mais simples**: sai o filtro `paths:` e o `working-directory`, porque o repositório inteiro é o projeto |

Duas outras adaptações:

- **As regras do modo tutorial foram copiadas para o `CLAUDE.md` do projeto.** O Claude Code lê
  o `CLAUDE.md` da pasta aberta e das pastas acima dela. No monorepo, o `CLAUDE.md` da raiz
  valia para tudo; em `C:\dev\pwncheck` ele não existe, então as regras precisam estar no
  próprio projeto.
- **Links relativos viraram URLs.** A lição da fase 1 apontava para `../../../../tutorial/...`,
  um caminho que só existia dentro do monorepo.

Por fim, o `.venv` foi **recriado do zero** em `C:\dev\pwncheck` a partir do
`requirements-dev.txt`, e os 17 testes passaram. É o teste de reprodutibilidade da Lição 00:
o ambiente não é copiado, é reconstruído.

---

## 4. GitHub CLI (`gh`)

O `gh` traz o GitHub para o terminal: repositórios, pull requests, issues, execuções do CI,
releases. Tudo o que você faria clicando no site.

### Instalação e login

```powershell
winget install -e --id GitHub.cli     # versão portátil, só para o seu usuário, sem admin
gh auth login                         # abra um terminal NOVO antes (o PATH mudou)
```

No `gh auth login`:

| Pergunta | Resposta | Por quê |
|---|---|---|
| Where do you use GitHub? | GitHub.com | — |
| Preferred protocol | HTTPS | o mesmo que o Git já usa |
| Authenticate Git with your GitHub credentials? | **No** | o Git já autentica pelo Git Credential Manager; responder "No" não mexe na sua configuração do Git |
| How would you like to authenticate? | Login with a web browser | você autoriza no navegador com um código de 8 caracteres |

O `gh` guarda o token no Gerenciador de Credenciais do Windows. Confira com `gh auth status`.

### Criando o repositório

```powershell
cd C:\dev\pwncheck
gh repo create Everett-gi/pwncheck --public --source . --remote origin `
  --description "Verifica se uma senha vazou sem enviá-la a ninguém (k-anonymity + HaveIBeenPwned)"
git push -u origin main
```

| Opção | O que faz |
|---|---|
| `--public` | repositório público (é portfólio) |
| `--source .` | usa o repositório Git da pasta atual |
| `--remote origin` | cadastra o novo repositório como o remote `origin` |

### Por que o push é feito pelo `git`, e não pelo `gh`

O `gh repo create` tem a opção `--push`, que cria e envia de uma vez. Na primeira tentativa
usamos essa opção, e o GitHub **recusou** o push:

```
! [remote rejected] HEAD -> main (refusing to allow an OAuth App to create or update
  workflow `.github/workflows/ci.yml` without `workflow` scope)
```

O repositório foi criado, mas os commits não subiram. O motivo: com `--push`, o `gh` envia
usando o **token dele**, que tem os escopos (*scopes*, permissões) `repo`, `read:org` e `gist`.
Criar ou alterar arquivos em `.github/workflows/` exige o escopo extra **`workflow`**. O GitHub
protege esses arquivos à parte porque um workflow executa código com acesso aos segredos do
repositório: quem pode alterá-los tem, na prática, acesso a esses segredos.

Duas saídas:

1. **Enviar pelo `git push` normal** (o que fizemos): o Git autentica pelo Git Credential
   Manager, cujo token já tem o escopo `workflow`.
2. Dar o escopo ao `gh`: `gh auth refresh -s workflow` (autoriza de novo pelo navegador).

É o princípio do menor privilégio visto do outro lado: cada token só faz o que o seu escopo
permite. Veja os escopos do seu com `gh auth status`.

### Comandos úteis no dia a dia

```powershell
gh repo view --web                    # abre o repositório no navegador
gh run list                           # últimas execuções do CI
gh run watch                          # acompanha a execução atual ao vivo
gh run view --log-failed              # mostra só o log dos passos que falharam
gh auth status                        # quem está logado
```

O `gh run view --log-failed` teria poupado trabalho na fase 1: quando o CI falhou, a API do
GitHub recusou mostrar o log sem login ("Must have admin rights"), e a causa precisou ser
reproduzida localmente.

---

## 5. Receita: começar um projeto novo

Quando for a vez do próximo projeto (digamos, o FileSentry):

1. Crie a pasta e o repositório local:
   ```powershell
   mkdir C:\dev\filesentry
   cd C:\dev\filesentry
   git init
   ```
2. Copie o blueprint `python\filesentry\CLAUDE.md` da central para a pasta nova e acrescente a
   seção **Modo tutorial** (use a do PwnCheck como modelo).
3. Crie os arquivos-base usando o PwnCheck como referência: `.gitignore`, `.gitattributes`,
   `pyproject.toml`, `requirements.txt`, `requirements-dev.txt`, `.github/workflows/ci.yml`.
   Leia cada um antes de copiar: você já sabe o que cada linha faz.
4. Crie o ambiente: `python -m venv .venv` e instale as dependências.
5. Primeiro commit. Depois crie o repositório e envie (o push pelo `git`, por causa do escopo
   `workflow` — seção 4):
   ```powershell
   gh repo create Everett-gi/filesentry --public --source . --remote origin
   git push -u origin main
   ```
6. Na central: apague `python/filesentry/` (o blueprint agora mora no repositório do projeto) e
   atualize a tabela de status do README com o link.

---

## 6. Exercícios

1. Rode `git log --oneline` em `C:\dev\pwncheck` e na central, e ache o commit
   `feat(pwncheck)` nos dois. Os hashes são iguais? Por quê?
2. Rode `gh run list -R Everett-gi/pwncheck`. Quantas execuções do CI existem, e qual o status?
3. No seu perfil do GitHub, use *Customize your pins* e fixe o `pwncheck`.
4. Na central, rode `git branch`. O branch `pwncheck-split` ainda existe? Ele ainda é necessário?
5. Explique com suas palavras por que o `.venv` não foi copiado de uma pasta para a outra.

<details>
<summary>Respostas</summary>

1. São diferentes (`4e24532` na central, `64f00b0` no pwncheck). O hash é calculado sobre a
   árvore de arquivos e o pai do commit. O `subtree split` reescreveu os caminhos (tirou o
   `python/pwncheck/` da frente) e os pais mudaram, então todos os hashes mudaram. A mensagem,
   o autor e a data continuam iguais.
2. Depende de quando você rodar; a primeira execução acontece no push que criou o repositório.
3. Pratique: é o que um recrutador vê primeiro.
4. Foi apagado depois do push: os commits dele já estão no GitHub, em `Everett-gi/pwncheck`.
   Se existisse, seria só um rótulo apontando para commits que a central não usa.
5. O `.venv` tem caminhos absolutos embutidos (veja o `pyvenv.cfg` e os `.exe` de `Scripts\`)
   e é regenerável pelo `requirements*.txt`. Copiar seria frágil; recriar prova que o projeto é
   reproduzível.

</details>
