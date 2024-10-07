---
layout: default
title: Garbage Collection System
description: Technical overview of the copying garbage collection system in Theta.
---

# Garbage Collection System in Theta

The following document outlines the design and detailed implementation of Theta’s **copying garbage collector**. This document captures every important design decision and architectural element to help guide implementation and ongoing maintenance.

## Table of Contents
1. [Overview](#overview)
2. [Core Components](#core-components)
   - [Shadow Stack](#shadow-stack)
   - [Temporary Offloading Region](#temporary-offloading-region)
   - [GC Epoch System](#gc-epoch-system)
3. [Detailed Garbage Collection Process](#detailed-garbage-collection-process)
   - [Triggering GC](#triggering-gc)
   - [Object Relocation](#object-relocation)
   - [Stack Frame Updates](#stack-frame-updates)
   - [Rehydrating the Stack](#rehydrating-the-stack)
4. [Design Considerations and Trade-offs](#design-considerations-and-trade-offs)
5. [Full Example Workflow](#full-example-workflow)

---

## Overview

Theta’s garbage collection system manages heap memory using a **copying garbage collector** that moves live objects from one memory space to another (from-space to to-space). The system ensures efficiency in WebAssembly’s constrained environment, addressing challenges such as the opacity of WebAssembly’s stack, while ensuring precise management of heap references across function calls.

The garbage collector is designed to:
- Use a **shadow stack** to track heap references for all active function frames.
- Employ a **GC Epoch system** to avoid redundant updates to function stack frames.
- Utilize a **temporary offloading region** to store non-heap values temporarily during garbage collection.
- Trigger garbage collection at the next function boundary once memory fills 50%.

The following sections describe each of the system’s components in detail and outline how they work together.

---

## Core Components

### Shadow Stack

The **shadow stack** is a critical data structure that mirrors heap references in the WebAssembly stack. It enables the garbage collector to access and update heap references, even though WebAssembly’s internal stack is not directly accessible. 

#### Responsibilities:
1. **Track Heap References Only**: 
   - The shadow stack only tracks **heap references** (e.g., pointers to heap-allocated objects).
   - Non-heap values (such as local variables, function arguments, and primitives) are **not tracked** by the shadow stack.

2. **Function Entry and Exit**:
   - Whenever a function is entered, any heap references used by that function are pushed onto the shadow stack.
   - When the function returns, the references are popped from the shadow stack.

3. **Maintain Consistency During GC**:
   - The shadow stack ensures that all active heap references are available during garbage collection.
   - Heap references are updated in the shadow stack whenever objects are moved by the garbage collector.

#### Why Only Track Heap References?
- **Efficiency**: By only tracking heap references, the shadow stack avoids the overhead of managing non-heap values (such as primitive types) that do not require updates during GC.
- **Memory Performance**: Focusing solely on heap references ensures that the shadow stack remains compact, reducing the memory overhead associated with tracking large function call stacks.

### Temporary Offloading Region

The **temporary offloading region** is a **pre-allocated 8 KB block** of memory located at the beginning of the memory space. It is designed to handle **non-heap values** during garbage collection.

#### Responsibilities:
1. **Offload Non-Heap Values During GC**:
   - When GC is triggered, non-heap values (e.g., local variables, arguments, and intermediate results) are offloaded into this region, allowing the WebAssembly stack to be cleared for heap management.

2. **Rehydrate the WebAssembly Stack**:
   - After GC is complete, non-heap values from the offloading region are restored to their respective locations in the WebAssembly stack, ensuring that function frames are fully restored after GC.

#### Why Use a Pre-Allocated 8 KB Region?
- **Worst-case Stack Frame Handling**: The fixed size is large enough to accommodate even large stack frames, ensuring that memory management remains predictable.
- **Simplified Memory Management**: By pre-allocating the region, we avoid the complexity of dynamically resizing memory during GC, leading to more predictable performance.

### GC Epoch System

The **GC Epoch system** tracks which function stack frames need to be updated during garbage collection, ensuring that each frame is only updated **once per GC cycle**, even if GC runs multiple times within nested function calls.

#### Responsibilities:
1. **Track the Global GC Epoch**:
   - The system maintains a **global GC Epoch counter**, which increments each time GC is triggered.

2. **Per-Frame Epoch Tracking**:
   - Each function stack frame is associated with a **GC Epoch** value, representing the GC cycle in which the frame was last updated.

3. **Update Stack Frames Lazily**:
   - When a function returns, the stack frame is checked against the current global GC Epoch. If the frame’s epoch is **older** than the current epoch, its heap references are updated. If the frame’s epoch matches the current epoch, no updates are needed.

#### Benefits of the GC Epoch System:
- **Prevents Redundant Updates**: Each stack frame is updated only once per GC cycle, ensuring that functions are not re-updated unnecessarily.
- **Efficient Stack Frame Management**: The GC Epoch system reduces the overhead of managing large or deeply nested call stacks during garbage collection.

---

## Detailed Garbage Collection Process

### Triggering GC

Garbage collection is triggered when **memory usage reaches 50%** of the allocated heap. To avoid interrupting mid-function execution, GC runs only at the next **function boundary** (either function entry or exit).

- **Trigger Condition**: When heap memory is half-full, a flag is set to indicate that garbage collection should be run at the next function boundary.
- **Function Boundary Execution**: GC is delayed until a function boundary to minimize disruption and avoid interrupting ongoing calculations.

### Object Relocation

During garbage collection, live heap objects are moved from the **from-space** to the **to-space**, and all heap references in the shadow stack are updated to reflect the new addresses.

#### Steps in Object Relocation:
1. **Identify Live Objects**: Traverse the shadow stack to identify all heap objects that are still referenced.
2. **Move Objects to To-Space**: All live objects are moved to the to-space memory region.
3. **Update Heap References**: After the move, heap references in the shadow stack are updated to point to the new memory locations.

### Stack Frame Updates

Once heap objects have been relocated, stack frames are updated **lazily** using the **GC Epoch system**. This ensures that each stack frame is only updated when necessary.

- **Epoch-Based Updates**: When a function returns, its stack frame is checked. If the frame’s epoch is older than the current global epoch, its heap references are updated; otherwise, no updates are made.
- **Updating References**: When a stack frame needs updating, the references in the WebAssembly stack are synchronized with the updated references in the shadow stack.

### Rehydrating the Stack

Once garbage collection is complete, the WebAssembly stack must be **rehydrated** to restore both heap and non-heap values.

#### Rehydration Process:
1. **Restore Heap References**: Updated heap references are restored from the shadow stack to the WebAssembly stack.
2. **Restore Non-Heap Values**: Non-heap values stored in the offloading region are restored to their original locations on the WebAssembly stack.

---

## Design Considerations and Trade-offs

### Shadow Stack Design
- **Focus on Heap References**: By limiting the shadow stack to heap references only, the system avoids the overhead of tracking non-heap values, optimizing memory usage and performance.

### Pre-Allocated 8 KB Offloading Region
- **Predictable Memory Usage**: The fixed size of the offloading region ensures predictable behavior during GC, eliminating the need for dynamic memory resizing.
- **Worst-Case Handling**: The size is chosen to handle large stack frames, ensuring robust behavior even under heavy load.

### GC Epoch System
- **Avoid Redundant Updates**: The GC Epoch system minimizes unnecessary updates, ensuring that each stack frame is updated only once per GC cycle.
- **Efficient for Deep Stacks**: In deeply nested function calls, the system reduces the cost of repeatedly checking and updating frames by maintaining the epoch for each frame.

### Deferred GC Execution
- **Minimize Disruption**: By deferring garbage collection to the next function boundary, we ensure that garbage collection does not interrupt ongoing computations, improving the user experience and avoiding latency spikes.

---

## Full Example Workflow

### Function Call Sequence: A Calls B, Calls C

#### 1. Initial State:
- **Function A** allocates memory and stores a heap reference in its stack frame.
- **Function B** is called, also allocating memory. The heap references from **Function A** and **Function B** are pushed onto the shadow stack.
- **Function C** is called, allocating further memory. 

#### 2. Garbage Collection Triggered:
- Memory usage reaches 50%, and a flag is set to trigger garbage collection.
- **Function C** completes, and garbage collection starts at the function exit boundary. 
- The **global GC Epoch** increments from **1 to 2**.

#### 3. Object Relocation:
- The garbage collector moves live objects to the to-space, updating the shadow stack with the new memory addresses.

#### 4. Function Returns:
- **Function B** returns to **Function A**. The GC Epoch system checks that **Function B**’s stack frame has an epoch of 1 (outdated), and the frame is updated to reflect the new heap references. Its epoch is set to **2**.
- When **Function A**'s frame is checked, it is similarly updated with the new heap references and assigned the new epoch.

#### 5. Rehydration:
- Once garbage collection completes, heap references are rehydrated from the shadow stack back into the WebAssembly stack.
- Non-heap values stored in the offloading region are restored to their original locations in the stack.

