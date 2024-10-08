---
layout: home
title: Home 
nav_order: 1
description: Developer documentation for the Theta programming language, aimed at maintainers and contributors.
---

# Welcome to the Theta Developer Documentation

This site is dedicated to **developers** and **maintainers** working on the core of the **Theta programming language**. It serves as an internal guide for tracking design decisions, architecture, and the inner workings of Theta. If you're looking for information on how to **use** the language, please refer to our [user documentation](https://docs.theta-lang.org).

---

## Purpose

The purpose of this documentation is to:

- **Document key design decisions**: Track why certain features exist and how they are implemented.
- **Explain the architecture**: Provide insights into how different parts of the language are structured and how they work together.
- **Ensure consistency**: Maintain a unified approach to the development and evolution of the language.
- **Collaborate effectively**: Help current and future contributors get up to speed with how Theta is built and maintained.

---

## What You'll Find Here

This documentation is organized around the following topics:

### 1. **Core Language Design**
   - Decisions regarding the syntax and semantics of Theta.
   - Rationale behind key language features and implementation details.

### 2. **Compiler Architecture**
   - Explanation of the various compiler stages (lexing, parsing, code generation).
   - Details about optimizations and compilation targets (including WebAssembly).

### 3. **Memory Management**
   - Design and implementation of Theta's memory model and garbage collection strategy.
   - Detailed documentation of the copying garbage collector and memory handling across function boundaries.

### 4. **Interfacing with WebAssembly**
   - Integration points between Theta and WebAssembly, including how WebAssembly features are leveraged in the language.

---

## Getting Started

If you're new to contributing to Theta, please begin by reading the [Contributing Guide](https://github.com/ThetaLang/Theta/blob/master/CONTRIBUTING.md). It outlines how to set up your development environment and introduces our processes for building, testing, and reviewing changes.

For a deep dive into the architecture, start with [Core Language Design](./docs/core-language-design.html) and [Compiler Architecture](./docs/compiler-architecture/).

---

## Related Resources

- **Theta User Documentation**: [docs.theta-lang.org](https://docs.theta-lang.org)
- **Theta GitHub Repository**: [github.com/theta-lang](https://github.com/ThetaLang/Theta)
