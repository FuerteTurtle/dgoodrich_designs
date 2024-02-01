# Lexer Design

I have chosen to use ply for my lexer. As such most of my design will touch on how I will integrate with ply. The design also contains my notes from reading the ply documentation.

## Ply

ply has two modules namely lex and yacc. I will be using the lex module for lexing. I can use the yac module going forward to create an abstract syntax tree.

## Example Ply lex code

from https://ply.readthedocs.io/en/latest/ply.html#introduction

```python
# ------------------------------------------------------------
#
# tokenizer for a simple expression evaluator for
# numbers and +,-,*,/
# ------------------------------------------------------------
import ply.lex as lex

# List of token names.   This is always required
tokens = (
   'NUMBER',
   'PLUS',
   'MINUS',
   'TIMES',
   'DIVIDE',
   'LPAREN',
   'RPAREN',
)

# Regular expression rules for simple tokens
t_PLUS    = r'\+'
t_MINUS   = r'-'
t_TIMES   = r'\*'
t_DIVIDE  = r'/'
t_LPAREN  = r'\('
t_RPAREN  = r'\)'

# A regular expression rule with some action code
def t_NUMBER(t):
    r'\d+'
    t.value = int(t.value)
    return t

# Define a rule so we can track line numbers
def t_newline(t):
    r'\n+'
    t.lexer.lineno += len(t.value)

# A string containing ignored characters (spaces and tabs)
t_ignore  = ' \t'

# Error handling rule
def t_error(t):
    print("Illegal character '%s'" % t.value[0])
    t.lexer.skip(1)

# Build the lexer
lexer = lex.lex()
```

ply documentation also has an example of reading in a loop

```python
# Test it out
data = '''
3 + 4 * 10
  + -20 *2
'''

# Give the lexer some input
lexer.input(data)

# Tokenize
for tok in lexer:
    print(tok)
```

The docs mention that the tokens list is used by yacc. I should keep this in mind as I organize my files. The tokens list is a shared resource.

I could convert number values into literal integers in this step. remember to return any changes to the token object `t`

```python
def t_NUMBER(t):
    r'\d+'
    t.value = int(t.value)
    return t
```

In ply reserved words get a special rule similar to

```python
reserved = {
   'if' : 'IF',
   'then' : 'THEN',
   'else' : 'ELSE',
   'while' : 'WHILE',
   ...
}

tokens = ['LPAREN','RPAREN',...,'ID'] + list(reserved.values())

def t_ID(t):
    r'[a-zA-Z_][a-zA-Z_0-9]*'
    t.type = reserved.get(t.value,'ID')    # Check for reserved words
    return t
```

You can discard a token by returning no value for a token rule.

I should probably use function definition rule types since that explicitly controls the precedence where string rules are sorted by regular expression length.

literals can be used for single character token rules

```python
literals = [ '+','-','*','/' ]
```

I must update lineno myself, lex guarantees lineno will be a property on the object instance, but never updates the value itself.

## Organizing code into different files

when you call `lex.lex()` you may use the module argument to create an instance from an imported module. You can define the lexer as a class, as a file, or as a closure. For my project I will be using a class.
