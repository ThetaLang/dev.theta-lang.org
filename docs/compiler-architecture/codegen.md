---
layout: default
title: Code Generation
nav_order: 5
description: Technical overview of the Theta code generator and its implementation details.
parent: Compiler Architecture
---

# Theta Code Generation

The **Code Generation (CodeGen)** component in Theta is responsible for translating the parsed AST (Abstract Syntax Tree) into WebAssembly (WASM) code. This process involves traversing the AST, generating appropriate WASM constructs, and organizing the output into a binary module that can be executed efficiently.

This page provides an in-depth breakdown of the code generation architecture, walking through key functions, WASM memory management, closure generation, and special handling for different AST nodes.

---

## Table of Contents
1. [Overview](#overview)
2. [Core Components](#core-components)
   - [Module Initialization](#module-initialization)
   - [WASM Generation](#wasm-generation)
   - [Memory Management](#memory-management)
3. [Detailed Code Generation Process](#detailed-code-generation-process)
   - [Closure Handling](#closure-handling)
   - [AST Node Generation](#ast-node-generation)
4. [Optimizations](#optimizations)
5. [Example Workflow](#example-workflow)

---

## Overview

The Theta code generation process transforms the parsed AST into WebAssembly (WASM). Each AST node type corresponds to a WASM construct, such as a function, block, or assignment. The **CodeGen** system handles this translation by recursively walking the AST, generating WASM instructions, and managing memory offsets, function tables, and closures.

At a high level, the code generation flow looks like this:
1. **Initialization**: A WASM module is initialized with memory and core function imports.
2. **AST Traversal**: The AST is recursively traversed, and for each node, a corresponding WASM expression is generated.
3. **WASM Output**: The resulting WASM module is serialized and exported for execution.

---

## Core Components

### Module Initialization

The code generation process begins by initializing a WebAssembly module. This is handled by the **`initializeWasmModule()`** function, which imports necessary WASM features and sets up memory.

Key points:
- **Memory Setup**: The memory is set up with an initial size of 1 page (64 KB) and a maximum size of 10 pages.
- **Tables**: A string reference table and function reference table are added to the module to manage indirect calls and strings in WASM.

### WASM Generation

Once the WASM module is initialized, the **`generate()`** function recursively traverses the AST, generating corresponding WASM expressions for each node. The generator takes into account various AST node types such as assignments, function invocations, control flow structures, and literals.

- **Function Declarations**: Handled specially, as functions declared inside other functions generate closures with their own captured scope.
- **Binary Operations**: Math operations like addition, subtraction, and multiplication are directly translated to WASM instructions.
- **Control Flow**: Conditional structures (e.g., if-else) are flattened and translated into WASM `if` and `block` expressions.

### Memory Management

Memory in WASM is represented in pages, with each page being 64 KB. Theta manages memory by keeping track of **memory offsets**, where various data structures (closures, function arguments, etc.) are stored. This ensures efficient use of memory and prevents overwriting critical data.

- **Global Memory Offset**: Memory starts at an offset (e.g., 0), and as data is stored, the offset is incremented.
- **Function Pointers**: Theta functions and closures are stored in the function table, which allows for dynamic invocation via indirect calls.

---

## Detailed Code Generation Process

### Closure Handling

In Theta, closures are generated when functions are declared inside other functions. The **`generateClosureFunctionDeclaration()`** function is responsible for lifting a lambda function into a WASM closure. This includes capturing the current scope and ensuring that function calls can be made dynamically.

- **Function Hashing**: To avoid naming collisions, each function is hashed and stored in the function table.
- **Capturing Scope**: If a function references variables from its outer scope, these variables are captured and included in the generated closure function in WebAssembly. 

### AST Node Generation

Different AST node types require different WASM constructs. For example:

- **Assignments**: These translate to `local.set` or `global.set` WASM instructions.
- **Binary Operations**: Translated into appropriate WASM binary instructions (e.g., `i64.add`, `i64.sub`).
- **Function Calls**: Depending on the context, function calls may be direct or indirect (via the function table).

---

## Optimizations

The Theta code generator includes several optimizations to improve performance and memory efficiency:

1. **Automatic Dropping**: Unused values on the WASM stack are automatically dropped using `BinaryenModuleAutoDrop(module)`.
2. **Closure Templates**: Closures are stored as templates, allowing for efficient reuse of the same closure structure with different arguments.
3. **Hoisting Function Declarations**: Functions declared at the top level of a module are hoisted and stored globally, avoiding re-declaration.

---

## Example Workflow

Here’s a simplified example of the Theta code generation process:

### Source Code

```theta
capsule MyCapsule {
    x<Number> = 42
    y<Number> = x + 8
}
```

### AST Representation

The AST might look like this:

```json
{
    "type": "Source", 
    "links": [] , 
    "value": {
        "type": "Capsule", 
        "name": "MyCapsule", 
        "value": {
            "type": "Block", 
            "elements": [
                {
                    "type": "Assignment",
                    "left": {
                        "type": "Identifier", 
                        "value": "x", 
                        "variableType": {"type": "TypeDeclaration","declaredType": "Number","value": null}
                    }, 
                    "right": {"type": "NumberLiteral", "value": "42" }
                },
                {
                    "type": "Assignment", 
                    "left": {
                        "type": "Identifier", 
                        "value": "y", 
                        "variableType": {"type": "TypeDeclaration","declaredType": "Number","value": null}
                    }, 
                    "right": {
                        "type": "BinaryOperation", 
                        "operator": "+", 
                        "left": {"type": "Identifier", "value": "x" }, 
                        "right": {"type": "NumberLiteral", "value": "8" }
                    }
                }, 
                {
                    "type": "Assignment", 
                    "left": {
                        "type": "Identifier", 
                        "value": "main", 
                        "variableType": {"type": "TypeDeclaration","declaredType": "Function","value": {"type": "TypeDeclaration","declaredType": "Number","value": null}}
                    },
                    "right": {
                        "type": "FunctionDeclaration", 
                        "parameters": [] , 
                        "definition": {
                            "type": "Block", 
                            "elements": [
                                {
                                    "type": "BinaryOperation", 
                                    "operator": "+", 
                                    "left": {"type": "Identifier", "value": "x" }, 
                                    "right": {"type": "Identifier", "value": "y", "variableType": null}
                                }
                            ] 
                        }
                    }
                }
            ]
        }
    }
}
```

### WASM Generation

1. **Initialize Module**: A new WASM module is created, memory and tables are set up.
2. **Generate Code**:
    - The assignments `x = 42` and `y = x + 8` are translated into `local.set` and `i64.add` instructions.
    - The function declaration is translated into a `func` instruction, and added to the function table.
    - The `x + y` is translated into an `i64.add` instruction.
3. **Emit Module**: The WASM module is finalized, optimized, and emitted as a binary.

