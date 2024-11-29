---
title: Javascript Engine
description: Javascript is a high-level, interpreted language. Before it can be run, it must be compiled to bytecode and then to machine code.
pubDate: 2024-11-29
---

# Javascript Engine

## Overview

Javascript is a high-level, interpreted language. Before it can be run, it must be compiled to **bytecode** and then to **machine code**. (It differs therefore from lower-level languages like C or Go which are compiled directly to machine code.) This compilation is handled by the Javascript engine, and different execution environments use different Javascript engines. Chrome and NodeJS use V8, Safari uses JavascriptCore, and Firefox uses SpiderMonkey.

In addition to simply compiling the code, the engine performs memory allocation, garbage collection and performance optimization. This work is happening in real time whenever and wherever Javascript code is running, and that means that Javascript programs will always have a memory/CPU overhead that lower-level, compiled languages do not have. However the engine's performance optimizations mean that Javascript can still be very fast in practice.

Code execution happens in a few steps. First the Javascript code is converted into the **AST** or **Abstract Syntax Tree**. This syntax tree is then converted into **bytecode**, which is run by the **interpreter**. Then as the program runs, the engine monitors which code paths are run most frequently, and selectively compiles those **hot paths** to machine code. This is called a **tiered compilation** strategy, and it allows for a fast startup speed without compromising too much on performance.

Although these are simplified representations, this is what the code may look like at each of these steps:

**Javascript**

```javascript
function add(a, b) {
  return a + b;
}
add(1, 2);
```

**Abstract Syntax Tree (AST)**

```json
{
  "type": "Program",
  "body": [
    {
      "type": "FunctionDeclaration",
      "id": {
        "type": "Identifier",
        "name": "add"
      },
      "params": [
        {
          "type": "Identifier",
          "name": "a"
        },
        {
          "type": "Identifier",
          "name": "b"
        }
      ],
      "body": {
        "type": "BlockStatement",
        "body": [
          {
            "type": "ReturnStatement",
            "argument": {
              "type": "BinaryExpression",
              "operator": "+",
              "left": {
                "type": "Identifier",
                "name": "a"
              },
              "right": {
                "type": "Identifier",
                "name": "b"
              }
            }
          }
        ]
      }
    },
    {
      "type": "ExpressionStatement",
      "expression": {
        "type": "CallExpression",
        "callee": {
          "type": "Identifier",
          "name": "add"
        },
        "arguments": [
          {
            "type": "Literal",
            "value": 1
          },
          {
            "type": "Literal",
            "value": 2
          }
        ]
      }
    }
  ]
}
```

**V8 Bytecode (Simplified)**

```
// Function add
LdaZero                 // Load accumulator with 0
Star r0                 // Store accumulator to register 0
LdaSmi [1]             // Load Small Integer 1
Add r0                  // Add register 0 to accumulator
Return                  // Return with accumulator

// Calling code
LdaGlobal [0]          // Load 'add' function
Star r1                // Store in register 1
LdaSmi [1]             // Load argument 1
Star r2                // Store in register 2
LdaSmi [2]             // Load argument 2
Star r3                // Store in register 3
Call r1, r2, r3        // Call add(1,2)
```

**x86-64 Machine Code (Simplified)**

```nasm
; Function prologue
push    rbp
mov     rbp, rsp

; Load parameters
mov     rax, QWORD PTR [rbp+16]    ; Load first parameter
mov     rcx, QWORD PTR [rbp+24]    ; Load second parameter

; Add numbers
add     rax, rcx                   ; Add the numbers

; Function epilogue
pop     rbp
ret

; Calling code
mov     edi, 1                     ; First argument
mov     esi, 2                     ; Second argument
call    add                        ; Call the function
```

## Memory Management

When we code in Javascript, we don't have to think too much about memory. That's one of Javascripts main advantages -- memory management can be laborious and error-prone, so leaving that work to the engine means we can focus on application logic. However, the engine isn't perfect, and it's possible to write code that impedes the engine's work. So it's helpful to understand some practical details about how memory management happens.

### Heap and Stack

As with all other programs, memory in Javascript programs is managed in two areas: the **stack** and the **heap**. In Javascript's case, the **stack** manages the execution context of the running code, as well as primitive values and object references. The **heap** stores basically everything else, so: all non-primitives such as objects, arrays, and functions. (Note that in practice, both arrays and functions are actually objects too.) Here are some examples of values stored on the **stack** and **heap** respectively:

```javascript
// Stack storage
let number = 42; // Primitive stored directly
let str = "hello"; // Primitive stored directly
let obj = { name: "John" }; // Reference stored (object in heap)

// Heap allocations
let array = new Array(1000); // Large array in heap
let obj = {
  // Object in heap
  data: new ArrayBuffer(1000), // Large buffer in heap
};
```

Mostly we can ignore this as an implementation detail. But there are a couple of cases where you might want to take note of distinction. One is **memory leaks**. Stack-allocated data is automatically freed when it goes out of scope, but heap-allocated data is not. So it's fairly common to create memory leaks by neglecting to free references to heap-allocated objects.

```javascript
function doMath() {
  let x = 1;
  let y = 2;
  return x + y;
}

// Potential memory leak - heap allocated objects not being cleaned up
function addHandler() {
  const bigData = {
    /* lots of data */
  };
  element.addEventListener("click", () => {
    console.log(bigData); // bigData can't be garbage collected
    // while this handler exists
  });
}
```

The second is **caching**. If you're having to frequently access heap-allocated data, you can limit your overhead by caching it on the stack.

```javascript
// Less efficient - repeated heap access
function sum(obj) {
  let sum = 0;
  for (let i = 0; i < 1000; i++) {
    sum += obj.value; // accessing heap each time
  }
  return sum;
}

// More efficient - single heap access
function sum(obj) {
  let sum = 0;
  const value = obj.value; // cache heap value in stack
  for (let i = 0; i < 1000; i++) {
    sum += value;
  }
  return sum;
}
```
