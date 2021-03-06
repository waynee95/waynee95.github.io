---
title: Symbol Tables
date: 2021-04-22
---

## Overview

- The [[abstract-syntax-tree]]# is the result of top-down or bottom-up parsing
  - Such a parser is not enough to fully compile a modern programming language
  - To fully compile a program the compiler needs to take _context_ into consideration
- Most PL allow declaration, definition and use of symbolic names to represent constants, variables, methods, types and objects
  - The compiler needs to verify if such names are used correctly
- The symbol table is used to answer two questions:
  1. Given a declaration of a name, is there already a declaration with the same name in the current scope?
  2. Given the use of a name, to which declaration does it correspond, or is it undeclared?
- Generally, the symbol table is only needed to answer thoses two questions
  - Once all declarations have been processed to build the symbol table,
  - And all uses of names have been linked to their corresponding declaration node in the AST
  - Then the symbol table itself is no longer needed (because no more lookups on names will be performed)

## Symbol Table Construction

- Created by a pass over the AST to:
  1. Process symbol declarations
  2. Connect each symbol reference with its corresponding declaration
- An AST node that mentions a symbol name will also have a field with a reference to the symbol name's entry in the symbol table
  - If such connection cannot be made, then the symbol was not declared properly and an error will be thrown
  - Otherwise, other passes can use the symbol table reference to obtain information like type, storage requirements,... about the symbol
- Most languages have scopes that bind the availability of symbols to a specific region of the program (See [[static-vs-dynamic-scope]]#)

## Symbol Table Interface

- `openScope()`: Opens a new scope in the symbol table. New symbols are entered in this scope
- `closeScope()`: Closes the most recently opened scope.
- `enterSymbol(name, type)`: Enters `name` in the current scope.
- `RetrieveSymbol(name)`: Returns the current valid declaration for `name`. Might return null, if no declaration can be found.
- `DeclaredLocally(name)`: Test whether _name_ is present in the current (innermost) scope.

## Handling Scopes

- Every symbol reference in an AST node occurs in the context of defined scopes
- The scope defined by the _innermost_ such contexts is known as _current scope_
- The scopes defined by the current scope and its surrounding scopes are called _open scopes_ or _currently active scopes_
  - All other scopes are said to be _closed_
  - Current, open and closed are not fixed attributes but rather based on the current position in the program
- Common scope rules:
  1.  At any point in the program, accessible names are those that are declared in the current and all other open scopes
  2.  If a name is declared in more than one open scope, then a reference is resolved to the _innermost_ scope (this is called [shadowing](https://en.wikipedia.org/wiki/Variable_shadowing))
  3.  New declarations can only be made in the current scope

## One Symbol Table or Many?

- A symbol table may be associated with each scope or all symbols may be entered in a single _global_ table
  - A single table needs specific mechanisms to handle multiple active declarations of the same symbol
  - However, search for a symbol in a single table may be faster

## References

- [Crafting A Compiler, Chapter 8](https://www.goodreads.com/book/show/6152082-crafting-a-compiler) by Fischer et al.
- [Lecture notes](https://www.d.umn.edu/~rmaclin/cs5641/Notes/L15_SymbolTable.pdf) from CS 5641 @ UMN
