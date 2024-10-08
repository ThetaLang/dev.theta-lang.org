---
layout: default
title: Lexer
nav_order: 2
description: Technical overview of the Theta lexer and its implementation details.
---

# Theta Lexer

The **Theta lexer** is responsible for converting raw source code into a sequence of **tokens** that can be processed by the parser. This step is crucial for analyzing and structuring code according to the language's syntax rules. The lexer scans the source string character by character, generates tokens for recognized patterns, and ignores irrelevant characters such as comments and whitespace.

This document details the architecture and flow of the lexer, providing insights into how tokens are generated, accumulated, and emitted.

## Table of Contents
1. [Overview](#overview)
2. [Core Components](#core-components)
   - [Token Creation](#token-creation)
   - [Accumulated Tokens](#accumulated-tokens)
   - [Handling Special Cases](#handling-special-cases)
   - [Ignored Tokens](#ignored-tokens)
3. [Detailed Lexing Process](#detailed-lexing-process)
   - [Character Scanning](#character-scanning)
   - [Tokenization Flow](#tokenization-flow)
4. [Design Considerations](#design-considerations)
5. [Example Workflow](#example-workflow)

---

## Overview

The Theta lexer reads through the source code character by character, converting the code into a **deque of tokens**. Each token represents a meaningful symbol or keyword in the source code, such as operators, keywords, identifiers, or literals. These tokens are passed on to the **parser** for further syntactical analysis and AST construction.

---

## Core Components

### Token Creation

At the core of the lexing process is the **`makeToken()`** function, which generates tokens by matching the current character (and potentially the next character) in the source string to known patterns.

**Key Responsibilities**:
- **Character Matching**: The lexer uses `attemptLex()` to match known lexemes (symbols, operators, keywords, etc.). The matching order is critical because multi-character symbols must be prioritized over single-character tokens to ensure correct tokenization.
- **Token Generation**: For each recognized symbol or pattern, the lexer generates a corresponding token with the appropriate **type** and **lexeme** (the string representation of the token).
- **Tracking Position**: The lexer tracks the current **line** and **column** positions in the source code, associating this information with each token for error reporting and debugging.

### Accumulated Tokens

Certain types of tokens, such as **identifiers**, **strings**, **numbers**, and **comments**, are made up of multiple characters. These tokens are generated using **accumulation functions** that gather characters until a stopping condition is met.

- **`accumulateUntilNext()`**: Collects characters until a specific terminal string (like a string delimiter or comment end symbol) is encountered.
- **`accumulateUntilAnyOf()`**: Collects characters until one of a set of specified characters is encountered (e.g., space or operator).
- **`accumulateUntilCondition()`**: A generalized accumulation function that continues accumulating characters as long as a provided condition is true.

#### Types of Accumulated Tokens:
- **Identifiers**: Accumulated until a non-identifier character is encountered.
- **Strings**: Accumulated between string delimiters.
- **Numbers**: Accumulated until a non-numeric character is encountered.

### Handling Special Cases

The lexer is also responsible for handling **special cases** that require unique processing rules:
- **Multiline Comments**: These are accumulated until the closing comment delimiter (`*/`).
- **String Literals**: Strings are enclosed in delimiters, and the lexer continues accumulating characters until the closing delimiter is found, while handling escape sequences and newlines within strings.
- **Newlines and Whitespace**: While these tokens are recognized, they are typically **ignored** by the parser.

### Ignored Tokens

Not all tokens are meant to be emitted or processed by the parser. The lexer defines a set of **non-emitted tokens**, such as:
- **Whitespace**: Single spaces and tabs.
- **Comments**: Both single-line and multiline comments are recognized but not emitted.
- **Newlines**: Newline characters, while important for tracking line numbers, are not emitted as tokens.

This filtering is handled by the **`shouldEmitToken()`** function, which checks whether the token should be added to the token deque or discarded.

---

## Detailed Lexing Process

### Character Scanning

The **lexing process** starts by scanning through the source string one character at a time. For each character, the lexer performs the following steps:
1. **Identify the Token Type**: The lexer tries to match the current character (or the current and next character together) against known token patterns using the `attemptLex()` function.
2. **Generate the Token**: If a match is found, the corresponding token is generated.
3. **Handle Accumulation**: If the token type requires accumulation (e.g., for strings, identifiers, or numbers), the lexer continues gathering characters until a valid stopping condition is met.
4. **Emit or Discard the Token**: After generating a token, the lexer checks whether the token should be emitted (added to the deque) or discarded (e.g., whitespace, comments).

### Tokenization Flow

The key part of the lexing process is the **tokenization flow**, which is handled in the **`lex()`** function. This function:
- Iterates through each character of the source code.
- Calls **`makeToken()`** to generate a token for each character or group of characters.
- Tracks the position in the source (line and column numbers) for accurate error reporting.
- Accumulates multi-character tokens like identifiers and numbers when necessary.

#### Token Matching Logic:
The lexer attempts to match tokens in a specific order to avoid misinterpretation of characters:
- **Multi-Character Operators**: Operators like `==`, `<=`, and `!=` are prioritized over their single-character counterparts (`=`, `<`, `>`).
- **Literals and Keywords**: Identifiers are parsed until a non-identifier character is found. If the identifier matches a reserved keyword, it is converted into a keyword token.
- **Numbers**: Numeric tokens are accumulated until a non-numeric character is encountered (with support for floating-point numbers).

---

## Design Considerations

The lexer is designed to handle complex tokenization patterns while maintaining flexibility and performance. Key design considerations include:

1. **Sequential Lexing**: By lexing in a sequential manner (character by character), the lexer can efficiently handle multi-character tokens without ambiguity.
2. **Priority-Based Matching**: The order in which tokens are matched is crucial. Multi-character symbols (like `==` or `!=`) must be recognized before their single-character equivalents (like `=`), ensuring correct tokenization.
3. **Accumulation Handling**: For tokens that span multiple characters (like identifiers, strings, and numbers), accumulation functions (`accumulateUntilNext()`, `accumulateUntilAnyOf()`) are used to gather the full lexeme before emitting the token.
4. **Error Reporting**: The lexer tracks line and column numbers to associate each token with its exact position in the source code. This is critical for providing accurate error messages during the compilation process.

---

## Example Workflow

### Lexing a Source String

Here’s a step-by-step example of how the Theta lexer processes a simple source string:

**Source**:
```cpp
if (x == 10) {
  return x;
}
```

1. Tokenization:
  - The lexer begins scanning the source string from left to right.
  - It recognizes the if keyword, generates a KEYWORD token, and advances to the next character.
  - It identifies the opening parenthesis ( and generates a PAREN_OPEN token.
  - It processes x, recognizes it as an identifier, and generates an IDENTIFIER token.
  - The == operator is recognized as an equality operator, and an OPERATOR token is generated.
  - The number 10 is identified as a numeric literal, and a NUMBER token is generated.
  - The closing parenthesis ) is identified, followed by the opening brace {.
  - The return keyword is recognized, and its corresponding KEYWORD token is generated.
  - Finally, the closing brace } is recognized and tokenized.
2. Token Emission:
  - After generating each token, the lexer determines whether the token should be emitted (added to the token deque) or ignored (whitespace, comments).
3. Resulting Tokens:
    - The token deque for this source string would contain the following tokens:
        - `KEYWORD: if`
        - `PAREN_OPEN: (`
        - `IDENTIFIER: x`
        - `OPERATOR: ==`
        - `NUMBER: 10`
        - `PAREN_CLOSE: )`
        - `BRACE_OPEN: {`
        - `KEYWORD: return`
        - `IDENTIFIER: x`
        - `BRACE_CLOSE: }`
