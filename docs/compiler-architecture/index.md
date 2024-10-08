---
layout: default
title: Compiler Architecture
description: Technical overview of the Theta Compiler and its workflow, including links to Lexer, Parser, TypeChecker, and Code Generator.
---

# Theta Compiler

The **Theta Compiler** is responsible for transforming raw source code into WebAssembly. It orchestrates the different stages of compilation—lexing, parsing, type checking, and code generation—while handling exceptions, optimizations, and file outputs. This page provides an overview of the compiler’s workflow and links to the components that handle each phase.

## Table of Contents
1. [Overview](#overview)
2. [Core Components](#core-components)
   - [Lexing](#lexing)
   - [Parsing](#parsing)
   - [Type Checking](#type-checking)
   - [Code Generation](#code-generation)
   - [Optimization](#optimization)
3. [Detailed Compilation Process](#detailed-compilation-process)
4. [Design Considerations](#design-considerations)
5. [Example Workflow](#example-workflow)

---

## Overview

The Theta compiler processes input source files and produces WebAssembly binaries. It follows a structured compilation pipeline:
1. **Lexing**: Convert source code into tokens. See [Lexer](./lexer).
2. **Parsing**: Convert tokens into an abstract syntax tree (AST). See [Parser](./parser).
3. **Type Checking**: Validate types and ensure correctness. See [TypeChecker](./typechecker).
4. **Code Generation**: Convert the AST into WebAssembly. See [Code Generation](./codegen).

Along the way, the compiler handles errors, applies optimizations, and outputs the generated binaries or WebAssembly Text (WAT) representations.

---

## Core Components

### Lexing

The first step in the compilation process is handled by the **Lexer**, which transforms raw source code into a sequence of tokens. The tokens represent keywords, operators, literals, and other syntactical elements. The lexer operates on a character-by-character basis and outputs tokens ready for the next stage.

For more details, visit the [Lexer](./lexer) page.

### Parsing

Once the tokens are generated, the **Parser** takes over. It transforms the sequence of tokens into an **abstract syntax tree (AST)**, which represents the syntactical structure of the program. The AST is a tree-like structure where each node corresponds to a construct in the language, such as expressions, functions, or control flow statements.

For more information on parsing, visit the [Parser](./parser) page.

### Type Checking

After the AST is built, the **TypeChecker** ensures that all types in the program are correct and consistent. This includes checking function parameter types, variable assignments, and return types to ensure they conform to the language's type system.

If type errors are encountered, the TypeChecker reports them, preventing further stages from executing until the errors are resolved.

Visit the [TypeChecker](./typechecker) page for details on how types are validated.

### Code Generation

Once the AST has been validated, the **Code Generator** transforms the AST into WebAssembly (Wasm) binaries. This phase includes generating the low-level instructions that map to the nodes of the AST. Optionally, the compiler can emit WebAssembly Text (WAT) for debugging and inspection.

For more details, see the [Code Generation](./codegen) page.

### Optimization

Before generating code, the compiler applies several optimization passes to the AST. Each optimization pass improves the efficiency of the generated WebAssembly, reducing the size and execution time. Optimizations may include constant folding, dead code elimination, and control flow optimizations.

---

## Detailed Compilation Process

### 1. Build AST

The compiler begins by building the AST from the source file:

```cpp
shared_ptr<ASTNode> programAST = buildAST(entrypoint);
```

This involves lexing the source code and passing the tokens to the parser, which returns the AST. The compiler can also emit the lexed tokens or the parsed AST for debugging purposes.

### 2. Optimize AST

After the AST is built, the compiler runs a series of optimization passes:

```cpp
if (!optimizeAST(programAST)) return;
```

Each pass optimizes specific patterns in the AST, such as constant expressions or unreachable code. If errors are encountered during optimization, they are reported, and the compilation process halts.

### 3. Type Checking

The TypeChecker validates that the types within the AST are correct. If the types are valid, the compilation continues; otherwise, type errors are displayed, and the process halts:

```cpp
TypeChecker typeChecker;
bool isTypeValid = typeChecker.checkAST(programAST);

if (!isTypeValid) return;
```

### 4. Code Generation

Finally, the **CodeGen** component generates WebAssembly from the optimized and type-checked AST. If requested, the compiler can also emit WAT (WebAssembly Text) for debugging:

```cpp
CodeGen codeGen;
BinaryenModuleRef module = codeGen.generateWasmFromAST(programAST);

if (isEmitWAT) {
  BinaryenModulePrint(module);
}
```

The generated WebAssembly is either written to a file or returned as a buffer.

---

## Design Considerations

The compiler is designed with the following principles in mind:
1. **Modularity**: Each phase of the compiler (lexing, parsing, type checking, code generation) is handled by a separate module, which allows for easier maintenance and extension.
2. **Error Handling**: The compiler provides detailed error messages at every stage. Errors are collected and displayed in a user-friendly manner.
3. **Optimization-Driven**: Optimization passes are modular and can be extended. They ensure that the generated WebAssembly is efficient and performant.
4. **Emit Modes**: The compiler can emit intermediate representations, such as tokens and ASTs, for debugging and testing purposes.

