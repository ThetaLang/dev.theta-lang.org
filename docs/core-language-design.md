---
layout: default
title: Core Language Design
nav_order: 2
description: Detailed design of the core language syntax and structure for the Theta programming language.
---

# Core Language Design

The following document outlines the core design of the Theta programming language using Backus-Naur Format (BNF). It captures the fundamental syntax and structure of the language, guiding developers and maintainers through the core rules that define valid Theta code.

This BNF serves as the foundation for the language parser and informs the behavior of the lexer, parser, and type checker.

## Table of Contents

1. [Functions](#functions)
2. [Control Flow](#control-flow)
3. [Data Structures](#data-structures)
4. [Expressions](#expressions)
5. [Types](#types)
6. [Comments](#comments)

---

## Functions

### Function Invocation
```
<function-invocation> ::= <identifier> "(" ((identifier | literal) ","?)* ")" ;
```
A function invocation consists of an identifier followed by a list of arguments inside parentheses. These arguments can either be identifiers or literals, and they are optional.

### Function Definition
```
<function-definition> ::= "(" <identifier-declaration>* ")" "->" (<expression> | <block>)
                        | <identifier-declaration>* "->" (<expression> | <block>) ;
```
Functions are defined using parameter declarations followed by a `->` and a block or an expression. Parameters are optional.

---

## Control Flow

### If Statement
```
<if-statement> ::= "if" <expression> <block> <else-if-statement>* <else-statement>? ;
```
The `if` statement checks an expression and executes the corresponding block if the expression evaluates to true.

### Else-If Statement
```
<else-if-statement> ::= "else if" <expression> <block> ;
```
Optional branches of an `if` statement that are evaluated if the preceding condition is false.

### Else Statement
```
<else-statement> ::= "else" <block> ;
```
The `else` statement executes if no previous `if` or `else-if` conditions were met.

---

## Data Structures

### Enums
```
<enum> ::= "enum" <identifier> "{" (<symbol> ","?)* "}" ;
```
Enums are a way of defining named constants with associated symbols.

### Structs
```
<struct> ::= "struct" <identifier> "{" (<identifier> <type> ","?)* "}" ;
```
Structs define a collection of named fields and their associated types.

### Capsules
```
<capsule> ::= "capsule" <identifier> "{" <assignment>* <struct>* <enum>* <function-definition>* "}" ;
```
A capsule groups related functions, enums, and structs into a modular unit.

---

## Expressions

### Assignment
```
<assignment> ::= <identifier-declaration> "=" <expression>
               | <identifier-declaration> "=" <function-definition>;
```
Assignments associate an identifier with either an expression or a function.

### Expressions
```
<expression> ::= <identifier>
               | <literal>
               | "!"+ (<identifier> | <boolean> | <comparative-expression>)
               | <numeric-expression>
               | "-" (<number> | "(" <numeric-expression> ")")
               | <string> "+" <string>
               | <comparative-expression>
               | <function-invocation>
               | <pipeline> ;
```
Expressions are the core unit of logic in Theta, capable of representing identifiers, literals, function calls, and more.

---

## Types
```
<type> ::= "<" <uppercase-letter>+ <letter>* <type>? ">" ;
```
Types are defined using uppercase letters followed by optional lowercase letters.

---

## Comments

### Single-Line Comment
```
<single-line-comment> ::= "//" <string> "
" ;
```
Single-line comments begin with `//` and continue until the end of the line.

### Multi-Line Comment
```
<multi-line-comment> ::= "/-" <string> "
"* "-/" ;
```
Multi-line comments start with `/-` and end with `-/`, spanning one or more lines.

---

## Arithmetic and Comparison Operators

### Arithmetic Operators
```
<arithmetic-operator> ::= "+" | "-" | "*" | "/" | "%" ;
```
Theta supports common arithmetic operators.

### Comparative Operators
```
<comparative-operator> ::= "<=" | ">=" | "==" | "!=" | ">" | "<" ;
```
Comparison operators are used to compare values in Theta.

---

## Example Workflow

To understand how the language design fits together, consider the following example of a function in Theta:

```typescript
capsule MyCapsule {
  struct Point {
    x Number,
    y Number
  }

  distance<Number> = (p1<Point>, p2<Point>) -> {
    return Math.sqrt((p2.x - p1.x)^2 + (p2.y - p1.y)^2)
  }
}
```

In this example:
- A capsule named `MyCapsule` encapsulates a struct and a function.
- The `Point` struct defines two fields: `x` and `y`, both of type `Number`.
- The `distance` function calculates the distance between two `Point` objects.
