# Calcy

> A lightweight, extensible terminal calculator written from scratch in C.

**Calcy** is an interactive command-line calculator designed to evaluate mathematical expressions directly from the terminal. It provides a persistent REPL interface with support for arithmetic operations, operator precedence, parentheses, mathematical functions, constants, and robust error handling.

The project is implemented from scratch in **C**, with no external expression-evaluation libraries.

```text
$ calcy

calcy> 1 + 2 * 3
7

calcy> (10 + 5) * 2
30

calcy> sqrt(144)
12

calcy> 2^10
1024

calcy> sin(PI / 2)
1

calcy> exit
$
```

---

## Features

### Interactive REPL

Launch Calcy directly from the terminal:

```bash
calcy
```

Expressions are evaluated immediately and the calculator remains ready for the next input.

```text
calcy> 25 * 4
100

calcy> 100 / 5
20

calcy> 7^3
343
```

Exit using:

```text
exit
```

or:

```text
Ctrl+C
```

### Arithmetic

Supported operators include:

| Operator | Operation      | Example  |
| -------- | -------------- | -------- |
| `+`      | Addition       | `5 + 3`  |
| `-`      | Subtraction    | `5 - 3`  |
| `*`      | Multiplication | `5 * 3`  |
| `/`      | Division       | `10 / 2` |
| `%`      | Modulo         | `10 % 3` |
| `^`      | Exponentiation | `2 ^ 8`  |

### Operator Precedence

Calcy follows standard mathematical precedence rules.

```text
calcy> 2 + 3 * 4
14

calcy> (2 + 3) * 4
20
```

### Parentheses

Expressions can be grouped using parentheses:

```text
calcy> (10 + 5) / 3
5

calcy> 2 * (3 + 4)
14

calcy> sqrt((5 + 4) * 4)
6
```

### Mathematical Functions

Calcy is designed to support common scientific-calculator functions, including:

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

Example:

```text
calcy> sqrt(144)
12

calcy> pow(2, 10)
1024

calcy> log10(1000)
3

calcy> sin(PI / 2)
1
```

### Mathematical Constants

Built-in constants include:

```text
PI
E
```

Example:

```text
calcy> PI
3.141592653589793

calcy> 2 * PI
6.283185307179586
```

---

# Architecture

Calcy is structured as a small expression-processing pipeline:

```text
                    User Input
                        │
                        ▼
                ┌───────────────┐
                │      REPL     │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │     Lexer     │
                └───────┬───────┘
                        │
                     Tokens
                        │
                        ▼
                ┌───────────────┐
                │     Parser    │
                └───────┬───────┘
                        │
                  Expression Tree
                        │
                        ▼
                ┌───────────────┐
                │   Evaluator   │
                └───────┬───────┘
                        │
                        ▼
                     Result
```

## REPL

The REPL is responsible for:

* Reading input from `stdin`
* Displaying the prompt
* Detecting `exit`
* Handling `Ctrl+C`
* Passing expressions to the parser
* Printing results
* Continuing until termination

---

## Lexer

The lexer converts raw input into tokens.

For example:

```text
2 + sqrt(16) * 3
```

becomes approximately:

```text
NUMBER(2)
PLUS
FUNCTION(sqrt)
LPAREN
NUMBER(16)
RPAREN
MULTIPLY
NUMBER(3)
```

The lexer is responsible for identifying:

* Numbers
* Operators
* Parentheses
* Function names
* Constants
* Invalid characters

---

## Parser

The parser converts the token stream into an expression structure while enforcing operator precedence and associativity.

A recursive-descent grammar can be represented as:

```text
expression
    → term (("+" | "-") term)*

term
    → power (("*" | "/" | "%") power)*

power
    → unary ("^" power)?

unary
    → ("+" | "-") unary
    → primary

primary
    → NUMBER
    → CONSTANT
    → FUNCTION "(" expression ")"
    → "(" expression ")"
```

This allows expressions such as:

```text
2 + 3 * 4
```

to be interpreted as:

```text
2 + (3 * 4)
```

rather than:

```text
(2 + 3) * 4
```

---

## Evaluator

The evaluator walks the parsed expression and computes the final numerical result.

For example:

```text
sqrt(2^8 + 9)
```

is evaluated as:

```text
          sqrt
           │
           +
         /   \
        ^     9
      /   \
     2     8
```

The evaluator recursively resolves the expression from the leaves upward.

---

# Error Handling

Calcy is designed to report invalid input without crashing the process.

Examples:

```text
calcy> 10 / 0
Error: division by zero
```

```text
calcy> 2 + *
Error: expected expression
```

```text
calcy> (2 + 3
Error: expected ')'
```

```text
calcy> sqrt(-1)
Error: domain error
```

```text
calcy> hello
Error: unexpected identifier 'hello'
```

Errors should provide enough context to identify what went wrong while keeping the REPL alive.

---

# Building

## Requirements

* Linux / Unix-like operating system
* GCC or Clang
* GNU Make
* Standard C library
* Standard math library

Check your compiler:

```bash
gcc --version
```

or:

```bash
clang --version
```

## Clone

```bash
git clone https://github.com/<username>/calcy.git
cd calcy
```

## Build

Using Make:

```bash
make
```

Or directly:

```bash
gcc -Wall -Wextra -Wpedantic -O2 -o calcy src/*.c -lm
```

---

# Usage

Run the executable:

```bash
./calcy
```

Example:

```text
$ ./calcy

calcy> 1 + 2 + 3
6

calcy> 2 * 8
16

calcy> 2 + 3 * 4
14

calcy> (2 + 3) * 4
20

calcy> sqrt(81)
9

calcy> 2^16
65536
```

---

# Installing as a Terminal Command

To make `calcy` available globally for your user:

```bash
mkdir -p ~/.local/bin
cp ./calcy ~/.local/bin/calcy
```

Ensure the directory is in your `PATH`:

```bash
echo $PATH
```

Then Calcy can be launched from anywhere:

```bash
$ calcy
```

---

# Testing

The project includes expression-level tests covering:

### Arithmetic

```text
1 + 2
10 - 5
4 * 8
20 / 4
10 % 3
2 ^ 10
```

### Precedence

```text
2 + 3 * 4
10 - 2 * 3
2 * 3 ^ 2
```

### Parentheses

```text
(2 + 3) * 4
2 * (3 + 4)
((2 + 3) * 4)
```

### Functions

```text
sqrt(16)
pow(2, 8)
abs(-42)
log10(100)
sin(PI / 2)
```

### Invalid Expressions

```text
2 +
(2 + 3
10 / 0
sqrt(
unknown(10)
```

Run the test suite with:

```bash
make test
```

---

# Project Structure

The implementation is intentionally modular:

```text
calcy/
├── src/
│   ├── main.c
│   ├── repl.c
│   ├── lexer.c
│   ├── parser.c
│   ├── evaluator.c
│   ├── functions.c
│   └── error.c
│
├── include/
│   ├── lexer.h
│   ├── parser.h
│   ├── evaluator.h
│   ├── functions.h
│   └── error.h
│
├── tests/
│   ├── test_parser.c
│   ├── test_evaluator.c
│   └── test_functions.c
│
├── Makefile
├── LICENSE
└── README.md
```

The exact structure may evolve as the implementation develops.

---

# Design Goals

Calcy is intentionally built around a few principles:

### 1. From-scratch implementation

The expression engine is implemented directly in C rather than delegating expression parsing to an external library.

### 2. Unix-native behavior

Calcy should behave like a normal command-line utility:

```bash
calcy
```

It should work naturally with:

* `stdin`
* `stdout`
* pipes
* shell scripts
* command-line arguments
* Unix signals

### 3. Small and understandable

The implementation should remain compact enough to understand from source code without sacrificing correctness.

### 4. Extensible parser

The parser should make it straightforward to add new operators, functions, constants, and expression types.

### 5. Correctness over convenience

Malformed expressions should produce useful errors rather than crashes or silently incorrect results.

---

# Technical Concepts

This project explores several low-level and systems-programming concepts:

* C programming
* Pointers and memory
* Character and string processing
* Tokenization
* Recursive-descent parsing
* Operator precedence
* Expression trees
* Function dispatch
* Floating-point arithmetic
* Error propagation
* Unix standard input/output
* POSIX signals
* Command-line arguments
* Process exit codes
* Build systems
* Unit testing
* Debugging with GDB

---

# Roadmap

### Core

* [x] Interactive REPL
* [x] Basic arithmetic
* [x] Operator precedence
* [x] Parentheses
* [ ] Unary operators
* [ ] Floating-point support
* [ ] Scientific notation
* [ ] Robust error reporting

### Scientific Functions

* [ ] `sqrt`
* [ ] `cbrt`
* [ ] `pow`
* [ ] `abs`
* [ ] `floor`
* [ ] `ceil`
* [ ] `round`
* [ ] `log`
* [ ] `log10`
* [ ] `ln`
* [ ] `sin`
* [ ] `cos`
* [ ] `tan`
* [ ] `asin`
* [ ] `acos`
* [ ] `atan`

### Constants

* [ ] `PI`
* [ ] `E`

### CLI

* [ ] `./calcy`
* [ ] Command-line expression evaluation
* [ ] Piped input
* [ ] Exit status codes
* [ ] `help`
* [ ] `version`

### Quality

* [ ] Unit tests
* [ ] Parser fuzz testing
* [ ] Memory-safety testing
* [ ] GDB debugging workflow
* [ ] Sanitizer builds
* [ ] CI

---

# Example

```text
$ calcy

Calcy — terminal calculator

calcy> 12 + 8 * 2
28

calcy> sqrt(144) + 2^5
44

calcy> sin(PI / 2)
1

calcy> log10(1000) * 5
15

calcy> exit

$
```

---

# License

This project is licensed under the MIT License. See [`LICENSE`](LICENSE) for details.

---

## Status

Calcy is an actively developed personal systems-programming project focused on implementing a practical mathematical expression engine and terminal interface from first principles in C.
