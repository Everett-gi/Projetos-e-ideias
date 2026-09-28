# Lição 01 — Python para quem vem do C/C++

> **Objetivo:** montar o modelo mental certo de Python a partir do que você já sabe de C/C++.
> Não é um curso de lógica: é um mapa do que **muda** e das armadilhas de quem chega do C.
>
> **Como estudar:** abra o terminal, rode `python` (o REPL, o modo interativo) e vá
> testando cada trecho. `help(objeto)` mostra a documentação, `dir(objeto)` lista o que
> ele tem, `type(objeto)` mostra o tipo, e `exit()` sai. Todas as saídas mostradas aqui
> foram conferidas no Python 3.12 da sua máquina.

**Sumário**
1. [Modelo de execução](#1-modelo-de-execução)
2. [Variáveis são nomes, não caixas](#2-variáveis-são-nomes-não-caixas)
3. [Tipagem dinâmica e forte](#3-tipagem-dinâmica-e-forte)
4. [Números](#4-números)
5. [str × bytes](#5-str--bytes)
6. [Coleções](#6-coleções)
7. [Controle de fluxo](#7-controle-de-fluxo)
8. [Comprehensions](#8-comprehensions)
9. [Funções](#9-funções)
10. [Exceções](#10-exceções)
11. [`with` é o RAII do Python](#11-with-é-o-raii-do-python)
12. [Módulos e pacotes](#12-módulos-e-pacotes)
13. [Classes](#13-classes)
14. [Iteradores e geradores](#14-iteradores-e-geradores)
15. [Decoradores](#15-decoradores)
16. [Type hints](#16-type-hints)
17. [Desempenho e o GIL](#17-desempenho-e-o-gil)
18. [Estilo (PEP 8)](#18-estilo-pep-8)
19. [Colinha C/C++ → Python](#19-colinha-cc--python)
20. [Exercícios](#20-exercícios)

---

## 1. Modelo de execução

- **Não há `main()` obrigatório.** O arquivo é executado de cima para baixo; `def` e `class`
  são *comandos* que criam funções e classes quando a linha roda.
- **Não há etapa de compilação separada.** Um erro de sintaxe impede o arquivo de rodar, mas
  um nome errado dentro de uma função só explode quando aquela linha executar.
- O equivalente ao `main` é esta convenção:

```python
import sys

def main() -> int:
    print("olá")
    return 0

if __name__ == "__main__":   # verdadeiro só quando o arquivo é executado diretamente
    sys.exit(main())          # o int vira o código de saída do processo, como em C
```

`__name__` vale `"__main__"` quando o arquivo é o programa principal, e vale o nome do módulo
quando ele é importado por outro arquivo. Assim o mesmo arquivo serve de programa e de biblioteca.

---

## 2. Variáveis são nomes, não caixas

**Esta é a mudança de modelo mental mais importante.** Em C, `int x = 5;` reserva uma caixa
de memória chamada `x`. Em Python, **todo valor é um objeto no heap**, e uma variável é só
um **nome ligado a um objeto** — pense num `PyObject*` (e é literalmente isso no CPython).

```python
a = [1, 2, 3]
b = a            # NÃO copia a lista: b aponta para o MESMO objeto
b.append(4)
print(a)         # [1, 2, 3, 4]
print(a is b)    # True  -> "is" compara identidade (os ponteiros)

c = a.copy()     # agora sim, uma cópia (rasa)
print(a == c, a is c)   # True False -> mesmo valor, objetos diferentes
```

Em C++, o `b = a` acima se comporta como `auto& b = a;`, não como uma cópia do `std::vector`.

### Mutáveis × imutáveis

| Imutáveis (não mudam depois de criados) | Mutáveis |
|---|---|
| `int`, `float`, `bool`, `str`, `bytes`, `tuple`, `None` | `list`, `dict`, `set`, `bytearray`, objetos das suas classes |

```python
x = 10
y = x
y += 1       # int é imutável: cria um NOVO objeto (11) e religa o nome y
print(x)     # 10 -> x continua apontando para o 10
```

### Passagem de argumentos

Python passa **a referência por valor** — igual a passar um ponteiro em C:

```python
def adiciona(lista):
    lista.append(99)   # muta o objeto do chamador: visível fora

def religa(lista):
    lista = [0]        # só religa o nome LOCAL: invisível fora

nums = [1]
adiciona(nums)
religa(nums)
print(nums)            # [1, 99]
```

É o mesmo que em C: `void f(int *p) { p[0] = 99; /* visível */  p = NULL; /* invisível */ }`.

---

## 3. Tipagem dinâmica e forte

**Dinâmica:** o tipo pertence ao *objeto*, não à variável. **Forte:** não há conversão
implícita entre tipos incompatíveis — nisso Python é mais rígido que C.

```python
x = 1          # x aponta para um int
x = "texto"    # agora aponta para uma str: permitido
"1" + 1        # TypeError: can only concatenate str (not "int") to str
int("1") + 1   # 2 -> a conversão é sempre explícita
```

**Type hints** são anotações que o interpretador **ignora** na execução. Quem as verifica
são ferramentas como o Pylance, no editor:

```python
def area(largura: float, altura: float) -> float:
    return largura * altura

area("a", 2)   # roda sem erro e devolve 'aa' (str * int repete a string)!
               # O Pylance sublinha em vermelho; o Python não reclama.
```

Mesmo assim, **usamos type hints em tudo** (convenção do portfólio): eles documentam,
alimentam o autocompletar e pegam erros antes da execução.

### Verdadeiro e falso

Valores "vazios" são falsos: `0`, `0.0`, `""`, `[]`, `{}`, `set()` e `None`.

```python
if lista:              # ≈ if (!lista.empty())
    ...
if not texto:          # ≈ if (texto.empty())
    ...
if valor is None:      # None é o NULL/nullptr do Python — compare SEMPRE com "is"
    ...
```

---

## 4. Números

```python
2 ** 100        # 1267650600228229401496703205376 -> int NÃO tem overflow
7 / 2           # 3.5  -> "/" sempre devolve float
7 // 2          # 3    -> divisão inteira
-7 // 2         # -4   !! arredonda para -infinito (em C: -7 / 2 == -3, trunca para zero)
-7 % 2          # 1    !! o resto segue o sinal do divisor (em C: -7 % 2 == -1)
0.1 + 0.2       # 0.30000000000000004 -> float é o double IEEE-754 do C
0xFF, 0b1010, 0o17, 1_000_000   # hex, binário, octal, separador de milhar
x += 1          # não existe x++ nem ++x
```

Os operadores de bits são os mesmos do C (`& | ^ ~ << >>`), e funcionam com inteiros de
qualquer tamanho.

---

## 5. str × bytes

**Fundamental para o PwnCheck.** Em C, `char*` serve tanto para texto quanto para bytes
crus. Python separa os dois:

| Tipo | O que guarda | Equivalente em C |
|---|---|---|
| `str` | sequência de **caracteres Unicode** (code points) | não há equivalente direto |
| `bytes` | sequência imutável de valores 0–255 | `const unsigned char[]` |
| `bytearray` | o mesmo, mas mutável | `unsigned char[]` |

```python
s = "ação"
len(s)                   # 4  -> str conta CARACTERES
b = s.encode("utf-8")    # b'a\xc3\xa7\xc3\xa3o'
len(b)                   # 6  -> bytes conta BYTES: "ç" e "ã" ocupam 2 cada em UTF-8
b.hex()                  # '61c3a7c3a36f'
b[0]                     # 97 -> indexar bytes dá um int (como um unsigned char)
b.decode("utf-8")        # 'ação' -> de volta para str
bytes.fromhex("dead")    # b'\xde\xad'
```

**Regra de ouro:** dentro do programa, texto é `str`. Nas **bordas** (arquivos, rede, hash,
criptografia), é `bytes`. Codifique (`encode`) na saída e decodifique (`decode`) na entrada.
O hash do PwnCheck é calculado sobre os **bytes** UTF-8 da senha, e a codificação faz
diferença: há um teste mostrando que `"é"` tem outro hash se codificado em Latin-1.

### Strings são imutáveis

`s[0] = "x"` dá `TypeError`. Os métodos devolvem strings **novas**:

```python
"a,b,c".split(",")              # ['a', 'b', 'c']
"-".join(["a", "b", "c"])       # 'a-b-c'
"  oi \r\n".strip()             # 'oi'
"chave:valor".partition(":")    # ('chave', ':', 'valor')
"sem-separador".partition(":")  # ('sem-separador', '', '') -> nunca levanta erro
"senha" in "minha senha"        # True -> busca de substring
"abc"[0], "abc"[-1]             # ('a', 'c') -> índice negativo conta do fim
```

### f-strings: o printf do Python

```python
nome, n, pi = "Ada", 42, 3.14159
f"{nome} tem {n} anos"   # 'Ada tem 42 anos'
f"{pi:.2f}"              # '3.14'       ≈ printf("%.2f")
f"{n:08b}"               # '00101010'   binário com 8 dígitos
f"{n:#x}"                # '0x2a'       ≈ printf("%#x")
f"{1234567:,}"           # '1,234,567'
f"{nome!r}"              # "'Ada'"      repr: mostra as aspas, bom para depuração
f"{n=}"                  # 'n=42'       nome + valor: depuração rápida
```

---

## 6. Coleções

### list ≈ `std::vector` (de referências)

```python
nums = [3, 1, 2]
nums.append(4)         # push_back
nums.pop()             # remove e devolve o último
nums.sort()            # ordena no lugar
len(nums)              # size()
nums[0], nums[-1]      # primeiro e último
nums[1:3]              # fatia [1, 3) -> uma NOVA lista (cópia)
nums[::-1]             # invertida
```

As fatias seguem o intervalo semiaberto `[início, fim)`, como os iteradores `begin()`/`end()`
da STL.

### tuple ≈ `std::tuple` imutável

```python
ponto = (10, 20)
x, y = ponto           # desempacotamento ≈ auto [x, y] = ponto;  (C++17)
a, b = b, a            # troca de valores sem variável temporária
```

Funções que "retornam vários valores" na verdade retornam **uma tupla**: o `split_hash` do
PwnCheck retorna `(prefixo, sufixo)`.

### dict ≈ `std::unordered_map`

```python
idades = {"ana": 30, "bia": 25}
idades["caio"] = 40            # insere
idades["zé"]                   # KeyError! (o operator[] do C++ criaria a chave; aqui é erro)
idades.get("zé", 0)            # 0 -> valor padrão, sem exceção
"ana" in idades                # True -> busca pela chave, O(1)
for nome, idade in idades.items():
    print(nome, idade)
```

Desde o Python 3.7, dicts **preservam a ordem de inserção**. O PwnCheck usa
`dict.get(sufixo, 0)`: "quantas vezes vazou, ou 0 se não achar".

### set ≈ `std::unordered_set`

```python
vistos = {1, 2, 3}
vistos | {4}       # união
vistos & {2, 9}    # interseção
vistos - {1}       # diferença
```

---

## 7. Controle de fluxo

**A indentação é a sintaxe** — ela substitui as chaves. O padrão é 4 espaços.

```python
for i in range(0, 10, 2):          # for (int i = 0; i < 10; i += 2)
    ...
for item in lista:                 # for (auto& item : lista)
    ...
for i, item in enumerate(lista):   # índice + valor ao mesmo tempo
    ...
for a, b in zip(lista1, lista2):   # percorre as duas juntas
    ...
while condicao:
    ...

resultado = "par" if n % 2 == 0 else "ímpar"   # ternário (cond ? a : b)
```

Não há `do-while` nem `switch` clássico. No lugar do `switch` existe o `match` (3.10+),
que vai além: casa **estruturas**, não só valores.

```python
match comando.split():
    case ["sair"]:
        ...
    case ["abrir", arquivo]:       # casa uma lista de 2 itens e captura o segundo
        abrir(arquivo)
    case _:                        # default
        print("comando desconhecido")
```

`pass` é o comando vazio (o `;` sozinho do C), útil para blocos ainda não implementados.

---

## 8. Comprehensions

Uma forma compacta de construir coleções, muito usada:

```python
quadrados = [x * x for x in range(10) if x % 2 == 0]   # [0, 4, 16, 36, 64]
por_nome = {u.nome: u for u in usuarios}               # dict comprehension
total = sum(len(s) for s in textos)                    # "generator expression": sem lista intermediária
```

Equivale a um `for` com `push_back` — e costuma ser mais rápida, porque usa uma instrução
de bytecode especializada em vez de procurar e chamar `.append` a cada volta.

---

## 9. Funções

```python
def conectar(host: str, porta: int = 443, *, timeout: float = 5.0) -> bool:
    ...

conectar("exemplo.com")                    # porta=443, timeout=5.0
conectar("exemplo.com", 8080, timeout=1)   # argumento nomeado na chamada
conectar(host="exemplo.com", porta=80)
conectar("x", 80, 1.0)   # TypeError: tudo depois do "*" só pode ser passado pelo nome
```

- **Argumentos nomeados** deixam a chamada legível e a ordem deixa de importar.
- O `*` sozinho torna os parâmetros seguintes *keyword-only*.
- `*args` recebe os posicionais extras numa tupla; `**kwargs`, os nomeados extras num dict
  (é o `...` do C, só que tipado e seguro).

### Funções são objetos

Funções podem ser passadas, guardadas e retornadas — como ponteiros para função ou
`std::function`:

```python
def aplica(f, valor):
    return f(valor)

aplica(len, "abc")             # 3
aplica(lambda x: x * 2, 21)    # 42  ≈ [](auto x) { return x * 2; }
```

### Closures

Uma função definida dentro de outra **captura** as variáveis de fora — como uma lambda C++
com captura por referência `[&]`:

```python
def contador():
    n = 0
    def incrementa():
        nonlocal n        # "n" é a variável da função de fora, não uma nova
        n += 1
        return n
    return incrementa

c = contador()
c(), c(), c()             # (1, 2, 3)
```

O teste do cliente HTTP do PwnCheck usa isso: a função `handler` captura `body`, `status` e
`sent` da função que a cria.

**Armadilha — closures em laço:** a captura é do *nome*, não do valor do momento:

```python
fs = [lambda: i for i in range(3)]
[f() for f in fs]         # [2, 2, 2] — todas enxergam o último valor de i
```

### ⚠️ A armadilha do argumento padrão mutável

O valor padrão é criado **uma única vez**, quando o `def` executa — como uma variável
`static` local em C:

```python
def adiciona(item, itens=[]):    # ❌ a MESMA lista é reaproveitada em toda chamada
    itens.append(item)
    return itens

adiciona(1)                      # [1]
adiciona(2)                      # [1, 2]  !!

def adiciona(item, itens=None):  # ✅ o idioma correto
    if itens is None:
        itens = []
    itens.append(item)
    return itens
```

É por isso que o `fake_api` dos testes do PwnCheck usa `sent: list | None = None`.

---

## 10. Exceções

```python
try:
    n = int(texto)
except ValueError as e:          # ≈ catch (const std::invalid_argument& e)
    print(f"não é número: {e}")
else:
    print("convertido")          # roda só se NÃO houve exceção
finally:
    print("sempre roda")         # limpeza (mas prefira "with", seção 11)

raise ValueError("mensagem")     # ≈ throw std::invalid_argument("mensagem")

class SenhaFracaError(Exception):   # exceção própria: basta herdar de Exception
    pass
```

**Diferença cultural:** em C você checa códigos de retorno; em C++ exceções são para o
excepcional. Em Python, exceções são o **mecanismo normal de erro**, e o estilo idiomático é
o EAFP ("é mais fácil pedir perdão que permissão"): tente, e trate a exceção se falhar.

```python
# LBYL (estilo C): verifica antes
if chave in d:
    v = d[chave]

# EAFP (estilo Python): tenta e trata
try:
    v = d[chave]
except KeyError:
    v = None
```

`raise NovoErro(...) from e` encadeia exceções, preservando a causa original (o DocSage usa
isso no `main.py`).

---

## 11. `with` é o RAII do Python

```python
with open("dados.txt", encoding="utf-8") as f:
    for linha in f:              # lê linha a linha, sem carregar o arquivo inteiro
        processa(linha)
# aqui o arquivo JÁ foi fechado, mesmo que uma exceção tenha acontecido no meio
```

O equivalente em C++:

```cpp
{
    std::ifstream f("dados.txt");
    // ...
}   // o destrutor fecha o arquivo ao sair do escopo
```

Qualquer objeto com os métodos `__enter__` e `__exit__` funciona com `with`. O PwnCheck usa
`with httpx.Client(...) as client:` para garantir que as conexões sejam fechadas.

> ⚠️ **Sempre passe `encoding="utf-8"` para o `open()`.** No Windows, o padrão é a
> codificação regional — na sua máquina, `cp1252`; no Linux, é UTF-8. Sem o `encoding`, um
> arquivo UTF-8 lido no Windows vira texto corrompido, e um arquivo gravado no Windows quebra
> ao ser lido no servidor Linux. O código "funciona" numa máquina e falha na outra.

**Memória:** não existe `free` nem `delete`. O CPython usa **contagem de referências** (o
objeto é liberado quando a última referência some) e um coletor de lixo para ciclos. `del x`
apaga o *nome*, não o objeto. Não conte com `__del__` como destrutor determinístico: para
liberar recursos, use `with`.

---

## 12. Módulos e pacotes

```python
import hashlib                        # o módulo inteiro: hashlib.sha1(...)
from app.kanonymity import sha1_hex   # só um nome
import numpy as np                    # com apelido (convenção da comunidade)
```

**`import` não é `#include`.** O `#include` copia texto em tempo de compilação. O `import`
**executa o arquivo do módulo uma única vez**, guarda o resultado em `sys.modules` e liga um
nome a esse objeto-módulo. Importar de novo só consulta esse cache. Por isso não existem
header, include guard, nem separação entre declaração e implementação.

- **Módulo** = um arquivo `.py`. **Pacote** = uma pasta com `__init__.py` (como o `app/`).
- `python -m app.cli` executa o módulo `app.cli` como programa principal e coloca a pasta
  atual no caminho de busca — é assim que `from app...` funciona.
- `sys.path` é o caminho de busca de módulos, como o *include path* e o *library path* do
  compilador.
- **Não há `private`.** Por convenção, um nome começando com `_` é interno ("não use de fora").
- A ordem dos imports é convencionada: biblioteca padrão → terceiros → seu código, com uma
  linha em branco entre os grupos. O ruff confere (regra `I`).

---

## 13. Classes

```python
class Conta:
    def __init__(self, titular: str, saldo: float = 0.0) -> None:   # ≈ construtor
        self.titular = titular          # atributos nascem aqui, sem declaração prévia
        self._saldo = saldo             # "_" = privado por convenção

    def deposita(self, valor: float) -> None:   # "self" explícito ≈ o "this" do C++
        if valor <= 0:
            raise ValueError("valor deve ser positivo")
        self._saldo += valor

    @property
    def saldo(self) -> float:           # getter: usado como conta.saldo, sem parênteses
        return self._saldo

    def __repr__(self) -> str:          # como o objeto aparece no print e no depurador
        return f"Conta({self.titular!r}, saldo={self._saldo})"


c = Conta("Ada")
c.deposita(100)
print(c, c.saldo)       # Conta('Ada', saldo=100.0) 100.0
```

### Métodos "dunder" = sobrecarga de operadores

| Python | C++ |
|---|---|
| `__init__` | construtor |
| `__repr__` / `__str__` | `operator<<` para `ostream` |
| `__eq__`, `__lt__` | `operator==`, `operator<` |
| `__len__` | `size()` |
| `__getitem__` | `operator[]` |
| `__call__` | `operator()` |
| `__hash__` | especialização de `std::hash<T>` |
| `__enter__` / `__exit__` | construtor e destrutor no RAII |

### `@dataclass` ≈ struct com construtor e `==` gerados

```python
from dataclasses import dataclass

@dataclass(frozen=True)          # frozen=True ≈ todos os membros const
class Ponto:
    x: float
    y: float

    def distancia(self) -> float:
        return (self.x**2 + self.y**2) ** 0.5

p = Ponto(3, 4)
p                     # Ponto(x=3, y=4)   -> __repr__ gerado
p.distancia()         # 5.0
p == Ponto(3, 4)      # True              -> __eq__ gerado (compara campo a campo)
p.x = 10              # FrozenInstanceError: cannot assign to field 'x'
```

### Herança e duck typing

- Todos os métodos são "virtuais"; `super().__init__(...)` chama o construtor da classe base.
- **Duck typing:** qualquer objeto com os métodos certos serve, sem herdar de uma interface
  ("se anda como pato e grasna como pato..."). É parecido com templates do C++, só que
  verificado na execução. Para checagem estática existe o `typing.Protocol`, parecido com os
  *concepts* do C++20.

---

## 14. Iteradores e geradores

O `for` do Python funciona com qualquer **iterável**: ele chama `iter()` uma vez e `next()`
até receber a exceção `StopIteration` (o equivalente ao `it != end()`).

Um **gerador** é uma função com `yield`: cada `yield` entrega um valor e **pausa** a função
exatamente ali, mantendo as variáveis locais. É uma sequência preguiçosa (*lazy*):

```python
def le_em_blocos(caminho: str, tamanho: int = 65536):
    with open(caminho, "rb") as f:          # "rb" = binário, como fopen(..., "rb")
        while bloco := f.read(tamanho):     # ":=" atribui e testa na mesma expressão
            yield bloco                     # entrega um bloco e pausa aqui

for bloco in le_em_blocos("arquivo-grande.iso"):   # nunca carrega o arquivo inteiro
    ...
```

É exatamente o que o FileSentry vai fazer para calcular o hash de arquivos grandes.

```python
def fib():
    a, b = 0, 1
    while True:              # infinito, mas só calcula o que for pedido
        yield a
        a, b = b, a + b

import itertools
list(itertools.islice(fib(), 10))   # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

---

## 15. Decoradores

Um decorador é uma função que **recebe uma função e devolve outra**, normalmente
"embrulhando" a original. A sintaxe `@` é só açúcar:

```python
import functools
import time

def cronometra(func):
    @functools.wraps(func)                 # preserva nome e docstring da original
    def wrapper(*args, **kwargs):
        inicio = time.perf_counter()
        resultado = func(*args, **kwargs)
        print(f"{func.__name__}: {time.perf_counter() - inicio:.3f}s")
        return resultado
    return wrapper

@cronometra                  # equivale a: processa = cronometra(processa)
def processa():
    ...
```

Você vai ver decoradores por toda parte: `@pytest.mark.parametrize` (já usado no PwnCheck),
`@app.get("/check")` (FastAPI, fase 4), `@dataclass`, `@property` e `@lru_cache` (DocSage).

---

## 16. Type hints

| Hint | Significado | C++ |
|---|---|---|
| `int`, `float`, `str`, `bytes`, `bool` | tipos básicos | `int`, `double`, `std::string`... |
| `list[int]` | lista de ints | `std::vector<int>` |
| `dict[str, int]` | dicionário str → int | `std::unordered_map<std::string, int>` |
| `tuple[str, str]` | tupla de 2 strings | `std::pair<std::string, std::string>` |
| `int \| None` | int ou None | `std::optional<int>` |
| `Callable[[int], str]` | função int → str | `std::function<std::string(int)>` |
| `-> None` | não retorna valor | `void` |

---

## 17. Desempenho e o GIL

- Um laço em Python puro costuma ser de **10 a 100 vezes mais lento** que o equivalente em C:
  cada operação passa pelo interpretador, com despacho dinâmico e objetos no heap.
- Na prática, o trabalho pesado fica em código nativo: o `hashlib` chama o OpenSSL, o `numpy`
  é C e Fortran. **Meça antes de otimizar** (`time.perf_counter()`, `python -m cProfile`).
- **GIL** (*Global Interpreter Lock*): na versão padrão do CPython, só uma thread executa
  bytecode Python por vez. Threads ajudam em **E/S** (rede, disco — o GIL é liberado durante a
  espera), mas não paralelizam cálculo. Para cálculo pesado, usa-se `multiprocessing`; para
  muita E/S simultânea, `asyncio` (PortPulse e ThreatScope). As versões 3.13+ oferecem uma
  build opcional sem GIL.

---

## 18. Estilo (PEP 8)

| O quê | Convenção | Exemplo |
|---|---|---|
| funções e variáveis | `snake_case` | `check_password`, `prefix_length` |
| classes | `PascalCase` | `HibpClient` |
| constantes | `MAIUSCULAS` | `PREFIX_LENGTH` (não existe `const`: é só convenção) |
| módulos | `snake_case` curto | `kanonymity.py` |
| indentação | 4 espaços | — |

Você não precisa decorar: o `ruff check` aponta os desvios e o `ruff format` formata sozinho
(o `clang-format` do Python).

---

## 19. Colinha C/C++ → Python

| C/C++ | Python |
|---|---|
| `NULL` / `nullptr` | `None` |
| `true` / `false` | `True` / `False` |
| `&&`, `\|\|`, `!` | `and`, `or`, `not` |
| `x++` | `x += 1` |
| `a ? b : c` | `b if a else c` |
| `printf("%d\n", x)` | `print(x)` ou `print(f"{x}")` |
| `scanf` / `std::cin` | `input()` (sempre devolve `str`) |
| `strlen`, `sizeof`, `.size()` | `len()` |
| `malloc`/`free`, `new`/`delete` | automático (contagem de referências + GC) |
| `struct` | `@dataclass` |
| `std::vector` | `list` |
| `std::unordered_map` | `dict` |
| `std::unordered_set` | `set` |
| `std::pair` / `std::tuple` | `tuple` |
| `std::optional<T>` | `T \| None` |
| `const` | convenção `MAIUSCULAS` |
| `#include` | `import` |
| `namespace` | módulo |
| `switch` | `match` (3.10+) |
| `throw` / `catch` | `raise` / `except` |
| RAII / destrutor | `with` |
| `unsigned char buf[]` | `bytes` / `bytearray` |
| `int main(int argc, char** argv)` | `if __name__ == "__main__":` + `sys.argv` |
| `return 1;` no `main` / `exit(1)` | `sys.exit(1)` |
| `assert()` (aborta o programa) | `assert` (levanta `AssertionError`; em testes, o pytest mostra os valores) |

---

## 20. Exercícios

Faça no REPL (`python`) ou em arquivos `.py` numa pasta de rascunho.

1. **Aliasing.** Antes de rodar, preveja a saída. Depois corrija para `a` não mudar.
   ```python
   a = [1, 2]
   b = a
   b.append(3)
   print(a)
   ```
2. **Divisão.** Calcule `-7 // 2`, `-7 % 2` e `int(-7 / 2)`. Qual das três dá o mesmo
   resultado que `-7 / 2` em C?
3. **Bytes.** Compare `len("ação")` com `len("ação".encode("utf-8"))`. Depois rode
   `"ação".encode("latin-1")` e compare os bytes. Por que o hash de uma senha depende disso?
4. **dict.** Escreva `conta_palavras(texto: str) -> dict[str, int]` usando `dict.get`.
   Depois reescreva com `collections.Counter` (procure na documentação).
5. **Padrão mutável.** Escreva a versão ❌ de `adiciona` da seção 9, chame-a 3 vezes e
   explique o resultado com as palavras "objeto" e "nome".
6. **dataclass.** Crie `@dataclass class Retangulo` com `largura`, `altura` e um método
   `area()`. Teste o `==` entre dois retângulos iguais.
7. **RAII.** Escreva uma classe com `__enter__` e `__exit__` que imprima "abrindo" e
   "fechando". Use-a num `with` que levanta uma exceção no meio: o "fechando" aparece?
8. **Gerador.** Escreva `pares(n)`, um gerador que produz os `n` primeiros números pares,
   e use-o num `for`.

<details>
<summary>Respostas</summary>

1. Imprime `[1, 2, 3]`: `a` e `b` são dois nomes para o mesmo objeto. Correção: `b = a.copy()`
   (ou `b = list(a)`, ou `b = a[:]`).
2. `int(-7 / 2)` dá `-3`, o mesmo que em C: `int()` trunca em direção a zero, enquanto `//`
   arredonda para baixo (`-4`).
3. `4` contra `6`: em UTF-8, "ç" e "ã" ocupam 2 bytes cada. Em Latin-1, cada um ocupa 1 byte, com
   outros valores. O hash é calculado sobre os bytes: codificações diferentes geram hashes
   diferentes, e a senha "não seria encontrada" na base, que usa UTF-8.
4. ```python
   def conta_palavras(texto: str) -> dict[str, int]:
       contagem: dict[str, int] = {}
       for palavra in texto.split():
           contagem[palavra] = contagem.get(palavra, 0) + 1
       return contagem

   from collections import Counter
   Counter("a b a".split())   # Counter({'a': 2, 'b': 1})
   ```
5. Devolve `[1]`, depois `[1, 2]`, depois `[1, 2, 3]`. O objeto lista padrão foi criado uma vez,
   no `def`, e o nome `itens` é ligado a esse **mesmo objeto** em toda chamada sem argumento.
6. ```python
   from dataclasses import dataclass

   @dataclass
   class Retangulo:
       largura: float
       altura: float

       def area(self) -> float:
           return self.largura * self.altura

   Retangulo(2, 3) == Retangulo(2, 3)   # True
   ```
7. Sim: o `__exit__` roda mesmo com exceção, como um destrutor durante o *stack unwinding*.
   ```python
   class Recurso:
       def __enter__(self):
           print("abrindo")
           return self

       def __exit__(self, tipo, valor, traceback):
           print("fechando")
           return False   # False = não engole a exceção; ela continua subindo

   with Recurso():
       raise RuntimeError("falhou no meio")
   ```
8. ```python
   def pares(n: int):
       for i in range(n):
           yield 2 * i

   for p in pares(5):
       print(p)   # 0 2 4 6 8
   ```

</details>
