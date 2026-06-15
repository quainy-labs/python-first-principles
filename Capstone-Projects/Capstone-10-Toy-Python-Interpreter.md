# Capstone 10 - Toy Python Interpreter

## Project Brief

In this project, you will build a tiny Python-like interpreter.

An interpreter is a program that reads source code, understands its structure, and executes it according to the rules of a language.

This capstone does not try to re-create CPython.

That would be a much larger project.

Instead, you will build a small language inspired by Python.

The final project will support:

```text
numbers
strings
booleans
None
variables
arithmetic expressions
comparison expressions
if statements
while loops
print
function definitions
function calls
return statements
lexical environments
runtime errors
a REPL
tests for lexer, parser, AST, evaluator, functions, and scope
```

The project will introduce the full interpreter pipeline:

```text
source code
    -> tokens
    -> abstract syntax tree
    -> evaluation
    -> output and runtime state
```

By the end, the reader should understand how a programming language can be implemented, why parsing matters, how scope works, why functions need environments, and how much of a language runtime is made from ordinary data structures and careful rules.

---

# Why This Project Matters

The book has already explained many Python internals.

It has discussed objects, names, bytecode, virtual machines, functions, scope, descriptors, metaclasses, and CPython architecture.

Those topics are easier to respect when the reader has built a small interpreter.

Before this project, a statement like:

```python
x = 10 + 20
```

may feel obvious.

After this project, the reader sees the hidden stages:

```text
characters are grouped into tokens
tokens are organized into syntax
syntax becomes an AST
the AST is evaluated
the expression 10 + 20 produces 30
the name x is bound in an environment
```

The interpreter turns code from text into behavior.

That idea is deep.

It also makes Python itself feel less magical.

Python is larger, faster, older, and far more sophisticated than the toy interpreter.

But both systems share the same conceptual path:

```text
read code
represent code
execute code
manage state
report errors
```

---

# The Mental Model

A language implementation has layers.

The first layer is lexical analysis.

It turns characters into tokens.

Example source:

```python
x = 10 + 20
```

Tokens:

```text
NAME("x")
EQUAL("=")
NUMBER(10)
PLUS("+")
NUMBER(20)
EOF
```

The second layer is parsing.

It turns tokens into structure.

The assignment above becomes something like:

```text
Assign(
    name="x",
    value=Binary(left=Number(10), op="+", right=Number(20))
)
```

The third layer is evaluation.

It walks that structure and performs meaning.

The evaluator computes `10 + 20`.

Then it stores `x = 30`.

The fourth layer is runtime state.

The interpreter needs environments, values, functions, and errors.

The project will build each layer deliberately.

---

# What You Will Build

You will build a package named `toypy`.

Suggested structure:

```text
toypy/
    __init__.py
    tokens.py
    lexer.py
    ast.py
    parser.py
    environment.py
    values.py
    interpreter.py
    builtins.py
    repl.py
    errors.py
tests/
    test_lexer.py
    test_parser.py
    test_expressions.py
    test_statements.py
    test_functions.py
    test_scope.py
    test_errors.py
```

The language should be small enough to explain, but large enough to feel real.

Example program:

```python
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(6))
```

Another example:

```python
i = 0
total = 0

while i < 5:
    total = total + i
    i = i + 1

print(total)
```

This is enough to teach expressions, statements, scope, loops, functions, and calls.

---

# Requirements

The interpreter must support:

```text
integer and floating-point numbers
strings
True
False
None
variable assignment
name lookup
arithmetic: +, -, *, /
comparisons: ==, !=, <, <=, >, >=
boolean operators: and, or, not
if / else
while
function definitions
function calls
return
print as a built-in
comments
clear syntax errors
clear runtime errors
a REPL
```

The language should use Python-like indentation for blocks if you want a deeper challenge.

However, indentation parsing adds complexity.

For a first implementation, braces or explicit `end` markers are easier.

Because this is a Python book, the recommended path is to implement indentation after a simpler block parser is understood.

The chapter should explain both options.

---

# Non-Requirements

The interpreter does not need classes.

It does not need imports.

It does not need decorators.

It does not need comprehensions.

It does not need generators.

It does not need exceptions inside the toy language.

It does not need async.

It does not need bytecode.

It does not need optimization.

It does not need a garbage collector.

It does not need perfect Python compatibility.

Those are fascinating topics.

They are intentionally outside the first interpreter.

The goal is to build a language core that the reader can understand end to end.

---

# Syntax Choice

The project has a design choice.

Option one is a Python-like syntax with indentation:

```python
if x > 0:
    print(x)
else:
    print(0)
```

Option two is a simpler explicit block syntax:

```text
if x > 0:
    print(x)
else:
    print(0)
end
```

Option three is braces:

```text
if x > 0 {
    print(x)
} else {
    print(0)
}
```

For this capstone, use Python-like keywords and expression rules, but allow `end` to close blocks.

That keeps the parser manageable while preserving a familiar feel.

The reader can add indentation as an extension.

This is a good engineering compromise.

It keeps the project about interpretation rather than getting trapped in indentation tokenization.

---

# Tokens

Tokens are categorized pieces of source code.

The lexer reads characters and emits tokens.

Token types:

```text
NUMBER
STRING
NAME
NEWLINE
EOF
PLUS
MINUS
STAR
SLASH
EQUAL
EQUAL_EQUAL
BANG_EQUAL
LESS
LESS_EQUAL
GREATER
GREATER_EQUAL
LPAREN
RPAREN
COMMA
COLON
IF
ELSE
WHILE
DEF
RETURN
TRUE
FALSE
NONE
AND
OR
NOT
END
```

A token should store:

```text
type
lexeme
literal value
line
column
```

Line and column matter for error messages.

An interpreter that says:

```text
Syntax error
```

is frustrating.

An interpreter that says:

```text
line 3, column 12: expected expression after '+'
```

teaches better debugging.

---

# Lexer

The lexer is a scanner.

It moves through the source one character at a time.

It groups characters into tokens.

Rules:

```text
digits become numbers
quotes begin strings
letters and underscores begin names or keywords
spaces and tabs separate tokens
newlines become NEWLINE tokens
# begins a comment until end of line
operators become operator tokens
unknown characters become syntax errors
```

Example:

```python
answer = 40 + 2
```

The lexer emits:

```text
NAME(answer)
EQUAL
NUMBER(40)
PLUS
NUMBER(2)
NEWLINE
EOF
```

The parser should not care about raw characters.

It should consume tokens.

This separation keeps the code understandable.

---

# Numbers

Support integers and floats.

Rules:

```text
123 -> int
12.5 -> float
0.25 -> float
```

You do not need scientific notation in the first version.

The lexer can read digits.

If it sees a decimal point followed by more digits, it reads a float.

Examples:

```text
42
3.14
0.5
```

The literal value stored in the token should be the Python numeric value.

That means the evaluator does not need to parse number text again.

---

# Strings

Support double-quoted strings first.

Example:

```python
name = "Narendra"
```

The lexer should read until the closing quote.

It should report an error for an unterminated string.

Basic escape sequences can be an extension.

The first version can support:

```text
\n
\t
\"
\\
```

Or it can reject escapes clearly.

Choose one behavior and test it.

Strings teach that a token has both source text and literal meaning.

The lexeme includes quotes.

The literal value does not.

---

# Keywords And Names

Names are identifiers:

```text
x
total
fib
user_name
```

Keywords are reserved names:

```text
if
else
while
def
return
True
False
None
and
or
not
end
```

The lexer can read a name and then check a keyword table:

```python
KEYWORDS = {
    "if": IF,
    "else": ELSE,
    "while": WHILE,
    "def": DEF,
    "return": RETURN,
    "True": TRUE,
    "False": FALSE,
    "None": NONE,
    "and": AND,
    "or": OR,
    "not": NOT,
    "end": END,
}
```

This is how many lexers work.

The shape is simple and powerful.

---

# Abstract Syntax Tree

The AST is a tree representation of code.

It removes unnecessary source details.

For example:

```python
1 + 2 * 3
```

Should parse as:

```text
Binary(
    left=Number(1),
    op="+",
    right=Binary(
        left=Number(2),
        op="*",
        right=Number(3)
    )
)
```

The AST captures meaning.

Multiplication has higher precedence than addition.

The tree shape expresses that.

Use dataclasses for AST nodes.

```python
@dataclass
class Binary:
    left: Expr
    operator: Token
    right: Expr
```

AST nodes are ordinary Python objects.

That is one of the best lessons in this project.

Code can be represented as data.

---

# Expression Nodes

Expression nodes produce values.

Suggested expression nodes:

```text
Literal
Name
Unary
Binary
Logical
Call
Grouping
```

Examples:

```python
42
"hello"
x
-x
x + y
a and b
fib(5)
(1 + 2)
```

Expressions can appear inside statements.

For example:

```python
print(1 + 2)
```

The `1 + 2` part is an expression.

The `print(...)` call is also an expression.

The whole line is an expression statement or built-in call, depending on your design.

---

# Statement Nodes

Statements perform actions.

Suggested statement nodes:

```text
ExpressionStatement
Assign
If
While
FunctionDef
Return
Block
```

Examples:

```python
x = 10
if x > 0:
    print(x)
end
while x > 0:
    x = x - 1
end
def add(a, b):
    return a + b
end
```

Statements usually do not produce a user-visible value.

They change control flow, bind names, call functions, or manage blocks.

---

# Parsing Strategy

Use recursive descent parsing.

A recursive descent parser is a set of functions that mirror the grammar.

For expressions:

```text
expression     -> assignment
assignment     -> NAME "=" assignment | logic_or
logic_or       -> logic_and ("or" logic_and)*
logic_and      -> equality ("and" equality)*
equality       -> comparison (("==" | "!=") comparison)*
comparison     -> term ((">" | ">=" | "<" | "<=") term)*
term           -> factor (("+" | "-") factor)*
factor         -> unary (("*" | "/") unary)*
unary          -> ("-" | "not") unary | call
call           -> primary ("(" arguments? ")")*
primary        -> NUMBER | STRING | True | False | None | NAME | "(" expression ")"
```

This grammar encodes precedence.

`factor` binds more tightly than `term`.

That means multiplication happens before addition.

The parser creates AST nodes while following these rules.

---

# Statement Grammar

Statements can use rules like:

```text
program        -> statement* EOF
statement      -> if_stmt
                | while_stmt
                | function_def
                | return_stmt
                | simple_stmt

simple_stmt    -> expression NEWLINE
if_stmt        -> "if" expression ":" block ("else" ":" block)? "end"
while_stmt     -> "while" expression ":" block "end"
function_def   -> "def" NAME "(" parameters? ")" ":" block "end"
return_stmt    -> "return" expression? NEWLINE
block          -> statement*
```

Because the first version uses `end`, the parser knows where a block finishes.

This is easier than indentation handling.

It still teaches nested structure.

Example:

```python
if x > 0:
    while x > 0:
        x = x - 1
    end
end
```

The parser recursively parses the inner while block before finishing the if block.

---

# Assignment

Assignment binds a name.

```python
x = 10
```

You can parse assignment as a statement or as an expression.

Python treats assignment as a statement.

For this toy language, keep assignment as a statement first.

That means the parser can recognize:

```text
NAME "=" expression NEWLINE
```

This avoids trickier assignment expression rules.

Later, assignment expressions can be an extension.

The evaluator will:

```text
evaluate the right side
store the value in the current environment
```

---

# Environment

An environment maps names to values.

```python
class Environment:
    def __init__(self, parent=None):
        self.parent = parent
        self.values = {}
```

Lookup:

```python
def get(self, name):
    if name in self.values:
        return self.values[name]
    if self.parent is not None:
        return self.parent.get(name)
    raise RuntimeError(f"undefined name: {name}")
```

Assignment:

```python
def set(self, name, value):
    self.values[name] = value
```

This simple parent chain teaches lexical scope.

Inner scopes can see outer scopes.

Outer scopes do not automatically see inner local variables.

---

# Values

The first version can use native Python values:

```text
int
float
str
bool
None
```

This keeps evaluation simple.

Functions need a custom value type:

```python
@dataclass
class ToyFunction:
    name: str
    params: list[str]
    body: list[Stmt]
    closure: Environment
```

Using Python values is fine for a toy interpreter.

A more advanced interpreter may define its own object model.

That would allow custom attribute lookup, methods, classes, and memory management.

But for this project, native values keep the focus on interpretation.

---

# Truthiness

The interpreter needs truthiness for `if`, `while`, `and`, `or`, and `not`.

You can follow Python's basic truthiness:

```text
False is false
None is false
0 is false
0.0 is false
"" is false
everything else is true
```

Implement a helper:

```python
def is_truthy(value):
    return bool(value)
```

This uses Python's own truthiness for native values.

If the toy language later adds custom objects, truthiness rules can become explicit.

---

# Expression Evaluation

The interpreter evaluates expression nodes recursively.

For a literal:

```text
return literal.value
```

For a name:

```text
look up name in environment
```

For a binary expression:

```text
evaluate left
evaluate right
apply operator
```

Example:

```python
1 + 2 * 3
```

Evaluation:

```text
evaluate left of +
    1
evaluate right of +
    evaluate left of *
        2
    evaluate right of *
        3
    apply * -> 6
apply + -> 7
```

The AST shape controls the order.

That is why parsing matters.

---

# Short-Circuit Logic

`and` and `or` should short-circuit.

For `and`:

```text
if left is false, return left without evaluating right
otherwise evaluate and return right
```

For `or`:

```text
if left is true, return left without evaluating right
otherwise evaluate and return right
```

This means:

```python
False and expensive()
```

does not call `expensive`.

Short-circuiting is a language behavior, not an optimization detail.

The evaluator must implement it deliberately.

---

# Statements

Statement execution changes the environment or control flow.

Assignment:

```text
evaluate value
bind name
```

If:

```text
evaluate condition
execute then block if truthy
otherwise execute else block if present
```

While:

```text
while condition is truthy:
    execute body
```

Function definition:

```text
create ToyFunction
bind it to the function name
```

Return:

```text
exit the current function with a value
```

Return is the only one that needs special control-flow handling.

---

# Return Control Flow

In an interpreter, `return` jumps out of a function.

One simple implementation uses an internal exception.

```python
class ReturnSignal(Exception):
    def __init__(self, value):
        self.value = value
```

When executing a return statement:

```python
value = evaluate(expr) if expr else None
raise ReturnSignal(value)
```

When calling a function:

```python
try:
    execute_block(function.body, local_env)
except ReturnSignal as signal:
    return signal.value
return None
```

This exception is not an error.

It is a control-flow signal inside the interpreter implementation.

This pattern also helps explain how interpreters often use internal mechanisms that are not visible to user programs.

---

# Function Calls

A function call evaluates:

```text
callee
arguments
```

Then it invokes the callable.

Example:

```python
add(2, 3)
```

Evaluation:

```text
look up add
evaluate 2
evaluate 3
call add with [2, 3]
```

A toy function call creates a new environment:

```text
parent = function.closure
bind parameter names to argument values
execute function body
```

The closure matters.

It lets functions remember the environment where they were defined.

---

# Closures

Even if the language does not support nested functions heavily, closures are worth explaining.

Example:

```python
def make_adder(x):
    def add(y):
        return x + y
    end
    return add
end

add_five = make_adder(5)
print(add_five(10))
```

This requires the inner function to remember `x`.

The `ToyFunction` stores a `closure` environment.

When called later, it uses that environment as the parent of the local call environment.

This connects directly to Python closures from Volume I and advanced function behavior from Volume II.

If nested functions feel too much for the first implementation, leave closure tests as an extension.

But design `ToyFunction` with a closure field from the start.

---

# Built-In Functions

The language needs built-ins.

Start with `print`.

Represent a built-in as a callable object:

```python
class BuiltinFunction:
    def __init__(self, name, func, arity=None):
        self.name = name
        self.func = func
        self.arity = arity

    def call(self, interpreter, args):
        if self.arity is not None and len(args) != self.arity:
            raise RuntimeError(...)
        return self.func(*args)
```

`print` should write to an output stream.

Do not hard-code global stdout in tests.

Let the interpreter receive an output object.

```python
output = io.StringIO()
interpreter = Interpreter(output=output)
```

This makes print testable.

---

# Runtime Errors

Runtime errors happen while executing valid syntax.

Examples:

```text
undefined name
calling a non-function
wrong number of arguments
adding incompatible values
division by zero
return outside function
```

Define clear error classes:

```python
class ToyPyError(Exception):
    pass

class SyntaxError(ToyPyError):
    pass

class RuntimeError(ToyPyError):
    pass
```

Avoid shadowing Python's built-in names in public APIs if that becomes confusing.

Names like `ToySyntaxError` and `ToyRuntimeError` are clearer.

Include token location when possible.

Good errors make interpreters feel professional.

---

# Parser Errors

Parser errors should explain what was expected.

Examples:

```text
line 1, column 8: expected expression after '='
line 4, column 1: expected 'end' after if block
line 2, column 12: expected ')' after arguments
```

Parser methods can use:

```python
def consume(self, token_type, message):
    if self.check(token_type):
        return self.advance()
    raise self.error(self.peek(), message)
```

This pattern keeps parser error handling consistent.

---

# REPL

A REPL is a read-eval-print loop.

It lets the user type code interactively.

Simple shape:

```python
while True:
    line = input("toypy> ")
    if line in {"exit", "quit"}:
        break
    run(line)
```

A single-line REPL is easy.

Multi-line functions and blocks are harder because the interpreter needs to know when the block is complete.

For the first version, the REPL can support single-line expressions and statements.

For full programs, run from a file:

```bash
python -m toypy examples/fib.toy
```

This is a good compromise.

---

# Testing The Lexer

Lexer tests should cover:

```text
single-character tokens
two-character operators
numbers
floats
strings
names
keywords
comments
newlines
line and column tracking
unknown character errors
unterminated string errors
```

Example:

```python
tokens = lex("x = 1 + 2")
assert [token.type for token in tokens] == [
    NAME, EQUAL, NUMBER, PLUS, NUMBER, EOF
]
```

Lexer tests are usually precise.

They protect the rest of the interpreter.

If tokenization is wrong, parsing cannot be trusted.

---

# Testing The Parser

Parser tests should inspect AST shape.

For:

```python
1 + 2 * 3
```

Assert:

```text
top node is Binary '+'
left is Literal 1
right is Binary '*'
```

This proves precedence.

Test:

```text
parentheses override precedence
function calls parse arguments
if parses then and else blocks
while parses body
function definition parses parameters and body
return parses optional value
syntax errors point to useful tokens
```

AST tests may feel verbose.

They are worth it.

They prove the language structure before execution begins.

---

# Testing Evaluation

Evaluation tests should run small programs.

Examples:

```python
x = 10
print(x)
```

Expected output:

```text
10
```

Test arithmetic:

```python
print(1 + 2 * 3)
```

Expected:

```text
7
```

Test conditionals:

```python
if True:
    print("yes")
else:
    print("no")
end
```

Test loops:

```python
i = 0
while i < 3:
    print(i)
    i = i + 1
end
```

Evaluation tests are the user's view of the language.

---

# Testing Functions

Function tests should cover:

```text
definition binds function name
calling returns value
parameters bind arguments
wrong arity errors
return exits early
function without return returns None
recursion works
local variables do not leak globally
functions can read outer names
```

Example:

```python
def add(a, b):
    return a + b
end

print(add(2, 3))
```

Expected output:

```text
5
```

Recursion test:

```python
def fact(n):
    if n <= 1:
        return 1
    end
    return n * fact(n - 1)
end

print(fact(5))
```

Expected:

```text
120
```

---

# Testing Errors

Error tests are as important as success tests.

Test:

```text
undefined name
invalid operator use
wrong argument count
calling non-callable value
division by zero
return outside function
missing end
missing closing parenthesis
```

The interpreter should fail clearly.

It should not produce Python tracebacks for normal user mistakes.

Internal tracebacks are useful for interpreter developers.

Language users need language-level errors.

---

# Milestone 1 - Tokens And Lexer

Create token types and the token dataclass.

Implement the lexer.

Support:

```text
numbers
strings
names
keywords
operators
comments
newlines
EOF
```

Write lexer tests before moving on.

This milestone turns source text into structured input.

---

# Milestone 2 - AST Nodes

Create expression and statement dataclasses.

Expressions:

```text
Literal
Name
Unary
Binary
Logical
Call
Grouping
```

Statements:

```text
ExpressionStatement
Assign
If
While
FunctionDef
Return
Block
```

Do not write evaluator logic yet.

First define the shapes of the language.

---

# Milestone 3 - Expression Parser

Implement recursive descent expression parsing.

Support:

```text
operator precedence
grouping
unary operators
boolean operators
function calls
```

Write AST shape tests.

This milestone is where the grammar starts to feel real.

---

# Milestone 4 - Statement Parser

Parse:

```text
assignments
expression statements
if / else / end
while / end
def / end
return
blocks
```

Add syntax error tests.

Make sure nested blocks work.

A parser that handles nesting correctly is a real parser.

---

# Milestone 5 - Environment And Literals

Implement `Environment`.

Implement evaluation for:

```text
literals
names
assignment
expression statements
print built-in
```

At this stage, the interpreter can run:

```python
x = 10
print(x)
```

That is a major step.

---

# Milestone 6 - Operators

Implement:

```text
arithmetic
comparisons
truthiness
not
and
or
```

Test precedence through full program output.

Test type errors.

The interpreter should reject nonsense clearly.

---

# Milestone 7 - Control Flow

Implement:

```text
if
else
while
blocks
```

Test nested loops and nested conditionals.

Protect tests from infinite loops by using small programs.

As an extension, add an execution step limit for safety.

---

# Milestone 8 - Functions

Implement:

```text
ToyFunction
function definition
function call
local environment
return signal
arity checking
recursion
```

This milestone gives the language real power.

Once functions work, the interpreter feels alive.

---

# Milestone 9 - REPL And File Runner

Add:

```text
run(source)
run_file(path)
repl()
```

The command-line interface can support:

```bash
python -m toypy program.toy
python -m toypy
```

Single-line REPL support is enough.

Document limitations for multi-line blocks.

---

# Milestone 10 - Polish And Documentation

Add:

```text
better error messages
examples directory
language reference
README
```

Example programs:

```text
hello.toy
sum.toy
factorial.toy
fibonacci.toy
scope.toy
```

The reader should finish with both an interpreter and a small language guide.

---

# Common Mistake 1 - Parsing Without A Grammar

Do not start by guessing with string operations.

A language needs a grammar.

Even a small grammar gives the parser a map.

Without a grammar, every new feature becomes a tangle of special cases.

Write the grammar down.

Then implement parser functions that match it.

---

# Common Mistake 2 - Ignoring Precedence

If parsing is too naive, this:

```python
1 + 2 * 3
```

may become:

```text
(1 + 2) * 3
```

That produces 9 instead of 7.

Operator precedence is not optional.

Recursive descent handles it cleanly when each precedence level has its own method.

---

# Common Mistake 3 - Using One Global Dictionary For Everything

A single global dictionary works for the first few variables.

It fails once functions exist.

Functions need local scope.

Nested functions need outer scope.

Use environment objects with parent links.

That gives the interpreter room to grow.

---

# Common Mistake 4 - Treating Return As A Normal Value

`return` is not just a value.

It changes control flow.

If a return happens inside an if block inside a while loop inside a function, it must exit the whole function.

An internal `ReturnSignal` makes that behavior manageable.

Do not leak this signal to user code.

It is an implementation detail.

---

# Common Mistake 5 - Showing Python Tracebacks For User Errors

If the toy program has an undefined name, the user should see a toy language error.

They should not see a long Python traceback from inside the interpreter.

Catch expected interpreter errors at the top level.

Display clean messages.

Keep Python tracebacks for bugs in the interpreter itself.

---

# Common Mistake 6 - Testing Only Final Output

Output tests are useful.

They are not enough.

Test the lexer separately.

Test the parser separately.

Test the environment separately.

Then test full programs.

Layered tests make interpreter bugs much easier to locate.

---

# Extension Ideas

Add indentation-sensitive blocks.

Add lists.

Add dictionaries.

Add indexing.

Add `for` loops.

Add `break` and `continue`.

Add exceptions in the toy language.

Add classes.

Add methods.

Add modules and imports.

Add bytecode compilation.

Add a stack-based virtual machine.

Add disassembly.

Add lexical closures fully.

Add decorators.

Add a static resolver pass.

Add type checking.

Add better diagnostics with source snippets.

Add a formatter.

Add a language server.

Add a debugger.

Every extension connects to topics from the main book.

The toy interpreter can become a playground for understanding Python from the inside.

---

# What This Project Teaches

This project teaches language implementation.

It teaches lexical analysis.

It teaches parsing.

It teaches abstract syntax trees.

It teaches recursive descent.

It teaches operator precedence.

It teaches environments.

It teaches expression evaluation.

It teaches statements and control flow.

It teaches function calls.

It teaches local scope and closures.

It teaches runtime errors.

It teaches REPL design.

It connects many earlier ideas:

```text
strings become source code
dataclasses become AST nodes
lists become token streams
dictionaries become environments
recursion becomes parsing and evaluation
exceptions become internal control flow
functions become interpreted values
scope becomes environment chains
tests become confidence in a language runtime
```

The reader should leave with this mental model:

```text
An interpreter turns code into data, then walks that data according to language rules.
```

That mental model makes Python itself easier to study.

---

# Completion Checklist

The project is complete when:

```text
Source code can be tokenized.
Tokens include location information.
Lexer handles numbers, strings, names, keywords, operators, comments, and EOF.
Parser builds AST nodes.
Expression precedence works.
Assignments bind names.
Name lookup uses environments.
Arithmetic works.
Comparisons work.
Boolean logic short-circuits.
If statements choose the correct branch.
While loops repeat while truthy.
Function definitions bind function values.
Function calls bind parameters.
Return exits the current function.
Recursive functions work.
Built-in print is testable.
Runtime errors are clear.
Syntax errors are clear.
REPL runs simple statements.
File runner executes programs.
Tests cover lexer, parser, evaluator, functions, scope, and errors.
Documentation explains syntax and limitations.
```

When all of these are true, the reader has built a real tiny interpreter.

---

# Exercises

1. Create the `toypy` package structure.

2. Define token types.

3. Implement the `Token` dataclass.

4. Implement number lexing.

5. Implement string lexing.

6. Implement names and keywords.

7. Implement comments and newlines.

8. Add lexer error tests.

9. Define expression AST nodes.

10. Define statement AST nodes.

11. Write the expression grammar.

12. Implement precedence parsing.

13. Parse grouping.

14. Parse unary expressions.

15. Parse function calls.

16. Parse assignments.

17. Parse if statements.

18. Parse while statements.

19. Parse function definitions.

20. Parse return statements.

21. Implement environments.

22. Evaluate literals and names.

23. Evaluate arithmetic.

24. Evaluate comparisons.

25. Implement truthiness.

26. Implement short-circuit logic.

27. Execute assignment statements.

28. Execute if statements.

29. Execute while loops.

30. Implement `ToyFunction`.

31. Implement function calls.

32. Implement return using an internal signal.

33. Add recursion tests.

34. Add the print built-in.

35. Add the REPL.

36. Add a file runner.

37. Write a language reference.

---

# Preview of Capstone 11

Capstone 10 built a Toy Python Interpreter.

It introduced tokenization, parsing, ASTs, recursive descent, environments, expression evaluation, statements, functions, scope, runtime errors, and REPL design.

Capstone 11 will build a Distributed Scheduler.

The Distributed Scheduler project will combine many themes from the book and capstones into one larger system.

It will introduce scheduled jobs, distributed coordination, worker heartbeats, leases, persistence, retries, idempotency, observability, failure recovery, and operational design.

The transition is:

```text
Toy Python Interpreter explains how code can be executed
Distributed Scheduler explains how work can be coordinated across machines
```

The final capstone will ask the reader to think like a Python engineer building a real production system.
