# `calcy`

### `a tiny calculator hiding a surprisingly serious parser`

<p align="center">

<img src="https://img.shields.io/badge/language-C-A8B9CC?style=for-the-badge&logo=c&logoColor=white">
<img src="https://img.shields.io/badge/platform-Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black">
<img src="https://img.shields.io/badge/build-Make-427819?style=for-the-badge">
<img src="https://img.shields.io/badge/status-building-orange?style=for-the-badge">

</p>

<p align="center">

```text
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   ██████╗ █████╗ ██╗      ██████╗██╗   ██╗            │
│  ██╔════╝██╔══██╗██║     ██╔════╝╚██╗ ██╔╝            │
│  ██║     ███████║██║     ██║      ╚████╔╝             │
│  ██║     ██╔══██║██║     ██║       ╚██╔╝              │
│  ╚██████╗██║  ██║███████╗╚██████╗   ██║               │
│   ╚═════╝╚═╝  ╚═╝╚══════╝ ╚═════╝   ╚═╝               │
│                                                         │
│              terminal mathematics, in C               │
└─────────────────────────────────────────────────────────┘
```

</p>

<p align="center">

**Type an expression. Press Enter. Get the answer. Repeat.**

</p>

---

## `> ./calcy`

```text
$ calcy

   ╭────────────────────────────────────╮
   │  calcy :: interactive calculator   │
   │  type 'exit' or press Ctrl+C       │
   ╰────────────────────────────────────╯

calcy> 1 + 2 + 3
6

calcy> 2 * 4
8

calcy> 2 + 3 * 4
14

calcy> (2 + 3) * 4
20

calcy> sqrt(144)
12

calcy> 2^16
65536

calcy> sin(PI / 2)
1

calcy> exit

$
```

No GUI.

No buttons.

No mouse.

Just:

```text
stdin → parser → evaluator → stdout
```

---

# `what is calcy?`

**Calcy** is a terminal-based scientific calculator implemented from scratch in **C**.

It behaves like a normal Unix command:

```bash
calcy
```

and provides an interactive REPL for evaluating mathematical expressions.

The goal isn't to reinvent mathematics.

The goal is to understand what happens **between**

```text
"2 + sqrt(16) * 3"
```

and

```text
14
```

without outsourcing the interesting part to an expression-evaluation library.

---

# `the pipeline`

```text
                       ┌──────────────┐
                       │    stdin     │
                       └──────┬───────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │       REPL       │
                    │  read → dispatch │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │      LEXER       │
                    │                  │
                    │  "2 + sqrt(16)"  │
                    │        ↓         │
                    │  TOKENS TOKENS   │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │      PARSER      │
                    │                  │
                    │ precedence       │
                    │ associativity    │
                    │ parentheses      │
                    │ functions        │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │    EVALUATOR     │
                    │                  │
                    │  expression → x  │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │      stdout      │
                    └──────────────────┘
```

---

# `the nerdy part`

Consider:

```text
2 + 3 * 4
```

A naive left-to-right implementation could produce:

```text
(2 + 3) * 4
= 20
```

That's wrong.

Calcy understands precedence:

```text
       +
      / \
     2   *
        / \
       3   4
```

Therefore:

```text
2 + (3 * 4)
= 14
```

The parser is built around a recursive-descent expression grammar.

```text
expression
    │
    ├── term
    │     ├── factor
    │     ├── *
    │     └── factor
    │
    ├── +
    │
    └── term
```

The important idea:

```text
lower precedence
        ↓
expression
        ↓
term
        ↓
power
        ↓
unary
        ↓
primary
        ↓
higher precedence
```

---

# `supported mathematics`

### Arithmetic

```text
+     addition
-     subtraction
*     multiplication
/     division
%     modulo
^     exponentiation
```

Examples:

```text
calcy> 2 + 3
5

calcy> 10 - 7
3

calcy> 4 * 8
32

calcy> 20 / 4
5

calcy> 10 % 3
1

calcy> 2^10
1024
```

---

### Parentheses

```text
calcy> (2 + 3) * 4
20

calcy> 2 * (3 + 4)
14

calcy> ((2 + 3) * 4) / 5
4
```

---

### Scientific functions

```text
sqrt(x)
cbrt(x)

pow(x, y)

abs(x)
floor(x)
ceil(x)
round(x)

log(x)
log10(x)
ln(x)

sin(x)
cos(x)
tan(x)

asin(x)
acos(x)
atan(x)
```

Examples:

```text
calcy> sqrt(144)
12

calcy> pow(2, 8)
256

calcy> log10(1000)
3

calcy> sin(PI / 2)
1
```

---

### Constants

```text
PI
E
```

So:

```text
calcy> 2 * PI
6.283185307179586
```

and:

```text
calcy> E^2
7.38905609893065
```

---

# `errors > segfaults`

A calculator shouldn't die because somebody forgot a parenthesis.

```text
calcy> 2 +
error: expected expression

calcy> (2 + 3
error: expected ')'

calcy> 10 / 0
error: division by zero

calcy> sqrt(-1)
error: domain error

calcy> foo(10)
error: unknown function 'foo'
```

The REPL survives and waits for the next expression.

```text
calcy> 10 / 0
error: division by zero

calcy> 10 / 2
5
```

---

# `architecture`

```text
calcy/
│
├── src/
│   ├── main.c
│   │
│   ├── repl.c
│   │   └── terminal interaction
│   │
│   ├── lexer.c
│   │   └── characters → tokens
│   │
│   ├── parser.c
│   │   └── tokens → expression
│   │
│   ├── evaluator.c
│   │   └── expression → result
│   │
│   ├── functions.c
│   │   └── mathematical functions
│   │
│   └── error.c
│       └── diagnostics
│
├── include/
│   ├── lexer.h
│   ├── parser.h
│   ├── evaluator.h
│   ├── functions.h
│   └── error.h
│
├── tests/
│   ├── test_lexer.c
│   ├── test_parser.c
│   └── test_evaluator.c
│
├── Makefile
├── LICENSE
└── README.md
```

---

# `from characters → computation`

For:

```text
sqrt(2^8 + 9)
```

Calcy conceptually processes:

```text
RAW INPUT
    │
    ▼
"sqrt(2^8 + 9)"
    │
    ▼
LEXING
    │
    ├── FUNCTION(sqrt)
    ├── LPAREN
    ├── NUMBER(2)
    ├── POW
    ├── NUMBER(8)
    ├── PLUS
    ├── NUMBER(9)
    └── RPAREN
    │
    ▼
PARSING
    │
    ▼
             sqrt
               │
               +
             /   \
            ^     9
          /   \
         2     8
    │
    ▼
EVALUATION
    │
    ▼
17
    │
    ▼
sqrt(265)
    │
    ▼
16.2788...
```

The actual implementation may use a different internal representation as the project evolves.

---

# `why C?`

Because the point is not just getting the answer.

C makes the machinery visible.

This project touches:

```text
┌───────────────────────────────────────┐
│               C                       │
├───────────────────────────────────────┤
│ pointers                              │
│ arrays                                │
│ strings                               │
│ structs                               │
│ memory management                     │
│ function pointers                     │
│ floating-point arithmetic             │
│ error propagation                     │
│ stdin / stdout                        │
│ POSIX signals                         │
│ command-line arguments                │
│ compilation & linking                 │
└───────────────────────────────────────┘
```

And underneath the C:

```text
source
  ↓
preprocessor
  ↓
compiler
  ↓
assembly
  ↓
object files
  ↓
linker
  ↓
ELF executable
  ↓
Linux process
```

---

# `build`

### Dependencies

```text
gcc / clang
make
libm
Linux / Unix-like environment
```

Build:

```bash
make
```

Or manually:

```bash
gcc -Wall -Wextra -Wpedantic -O2 \
    -o calcy src/*.c -lm
```

Run:

```bash
./calcy
```

---

# `make it a real command`

Once built:

```bash
mkdir -p ~/.local/bin
cp ./calcy ~/.local/bin/
```

Then:

```bash
calcy
```

from anywhere.

```text
~/projects/foo $ calcy

calcy> 42 * 10
420
```

```text
~/Downloads $ calcy

calcy> sqrt(81)
9
```

---

# `CLI mode`

Interactive mode:

```bash
calcy
```

Expression mode:

```bash
calcy "2 + 3 * 4"
```

Pipeline mode:

```bash
echo "sqrt(144)" | calcy
```

Potentially making Calcy usable in shell scripts:

```bash
result=$(calcy "2^16")
echo "$result"
```

---

# `testing`

Calcy should be tested at multiple levels.

### Lexer

```text
"123"
"3.14"
"+"
"-"
"*"
"/"
"sqrt"
"PI"
"("
")"
```

### Parser

```text
1 + 2
2 + 3 * 4
(2 + 3) * 4
2^3^2
-sqrt(16)
```

### Evaluator

```text
1 + 2                 → 3
2 * 3                 → 6
2 + 3 * 4             → 14
(2 + 3) * 4           → 20
sqrt(144)             → 12
```

### Failure cases

```text
1 /
(1 + 2
10 / 0
sqrt(
unknown()
```

Run:

```bash
make test
```

---

# `debugging`

Calcy is also an excuse to get comfortable with actual systems tooling.

### GDB

```bash
gdb ./calcy
```

### AddressSanitizer

```bash
gcc -fsanitize=address -g ...
```

### UndefinedBehaviorSanitizer

```bash
gcc -fsanitize=undefined -g ...
```

### Assembly

```bash
gcc -S src/evaluator.c
```

### Disassembly

```bash
objdump -d ./calcy
```

### ELF inspection

```bash
readelf -h ./calcy
```

```bash
nm ./calcy
```

The calculator becomes a convenient playground for understanding what a C program actually becomes after compilation.

---

# `design principles`

### No `eval()`

The expression must be parsed by Calcy.

### No external expression engine

The parser and evaluator are the project's core.

### Fail gracefully

Bad input should produce diagnostics, not undefined behavior.

### Keep the core small

The project should remain understandable.

### Unix first

`stdin`, `stdout`, pipes, exit codes, signals and `$PATH` are first-class concerns.

### Extend without rewriting

Adding:

```text
sqrt()
sin()
log()
```

should not require rebuilding the parser from scratch.

---

# `roadmap`

```text
[x] REPL
[x] Basic arithmetic
[x] Operator precedence
[x] Parentheses

[ ] Unary operators
[ ] Floating-point literals
[ ] Exponentiation
[ ] Scientific notation

[ ] sqrt()
[ ] cbrt()
[ ] pow()

[ ] sin()
[ ] cos()
[ ] tan()
[ ] asin()
[ ] acos()
[ ] atan()

[ ] log()
[ ] log10()
[ ] ln()

[ ] PI
[ ] E

[ ] Command-line expressions
[ ] Piped input
[ ] help
[ ] version

[ ] Unit tests
[ ] Fuzz testing
[ ] Sanitizer builds
[ ] CI
```

---

# `the philosophy`

Calcy isn't trying to compete with MATLAB.

It isn't trying to replace Python.

It isn't trying to become another programming language.

It's a deliberately small problem with enough depth underneath it to explore:

```text
        mathematics
             │
             ▼
        text input
             │
             ▼
          lexing
             │
             ▼
          parsing
             │
             ▼
        computation
             │
             ▼
        C / memory
             │
             ▼
        Linux process
             │
             ▼
         terminal
```

A tiny command can still be a deep engineering exercise.

---

<p align="center">

### `calcy`

**type → parse → evaluate → repeat**

```text
$ calcy
calcy> _
```

</p>
