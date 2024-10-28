---
author: "tianyi"
date: "2024-10-28"
labels: ["zink", "evm"]
description: "How zink translate WASM to EVM bytecode"
title: "Rust EVM contracts with zink"
---

For those who may not know, Rust is a systems programming language known for its memory safety features and performance.
EVM (Ethereum Virtual Machine) is the runtime environment for Ethereum smart contracts, responsible for executing bytecode
written in languages like Solidity.

So, what makes it possible to write Rust EVM smart contracts with Zink? Let's dive into the "dark magics" of Zink!

## 1. What is Zink?

[Zink](https://github.com/zink-lang/zink) is a rustic smart contract langauge for EVM, it uses WebAssembly (WASM) as an
intermediate format before generating EVM bytecode.

### 1.1. Why using WASM ?

We're choosing WASM as the intermediate representation with the following reasons:

1. WASM can be compiled from rust source code directly
2. WASM has simple instruction set, 172 instructions in the MVP implementation (127 of them are numeric instructions).
3. WASM is backed by Mozilla and the Rust Team and widely used by the leading companies in the world, it has powerful toolchains.
4. WASM is designed for stack machine, it's perfectly matched with EVM.

### 1.2. Challenges translating WASM to EVM bytecode

Even though WASM is designed for stack based machine, it can not be executed by EVM directly for several reasons:

1. **Instruction Set**: EVM has its own instruction set, which is optimized for executing smart contracts. WASM, on the other
hand, has a different instruction set that is designed for general-purpose computation.
2. **Memory Model**: EVM uses a memory model that is specific to Ethereum's blockchain architecture. WASM, being a stack-based 
virtual machine, has its own memory model that is not compatible with EVM's memory layout.
3. **Function Call Conventions**: EVM has its own function call conventions, which are designed for executing Solidity smart 
contracts. WASM, being a stack-based VM, uses different function call conventions that would need to be adapted for execution 
on the EVM.

## 2. Instruction Mappings

As mentioned above, WASM and EVM bytecode have different ISA (see [Webassembly Opcodes][wasm-ist] and
[EVM Opcode Opcodes Interactive Reference][evm-ist]), what we need to do first is actually mapping the 
shared opcodes from WASM to EVM directly, once all of the WASM opcodes can be mapped to EVM opcodes safely
and correctly, any of WASM can be translated to EVM bytecode.

### 2.1. Control Flow ( 0x00 - 0x1f )

| WASM Opcode Hex | WASM         | EVM          | Description                                |
|-----------------|--------------|--------------|--------------------------------------------|
| 0x00            | unreachable  | INVALID      | -                                          |
| 0x01            | nop          | JUMPDEST     | JUMPDEST in EVM perfroms nop as well       |
| 0x02 - 0x1f     | control flow | JUMP & JUMPI | need structured instructions based on JUMP |

As we can see in the control flow instruction set, some of the opcodes can be mapped to EVM opcodes in per opcode level, like
`unreachable` and `nop`, others are required **structured instructions**, see [select][select] as an example.

### 2.2. Memory & Locals ( 0x20 - 0x44 )

These opcodes can not be simply mapped since WASM has different memory model with EVM, they will be translated to different
instructions in different cases, see `3.` for more details.

1. **local** will be translated to memory or calldata operations like `MLOAD` or `CALLDATALOAD`.
2. **const** will be translated to `PUSH(n) + {value}`
3. **global** fetches bytes from data section and then do stack pushing like **const**

Some opcodes are banned since they are not adapatable, for example `global.get`, `memory.x`, once zink compiler reaches these
opcodes, panic occurs, however, zink provides different approaches for replacing the logic which can generate these opcodes,
see `5.` for more details.


### 2.3. Numberic operations ( 0x44 - 0xbf )

WASM supports `i32` and `i64` as number types, for EVM, it's `u256`, and since EVM only has `u256` as number (bigger than `i64`),
all WASM numbers and their operations can be perfectly mapped to EVM in per opcode level or chained instructions.


## 3. Memory Allocator

## 4. Function Call

## 5. Language Features

[select]: https://docs.zink-lang.org/compiler/control-flow.html#select
[wasm-ist]: https://pengowray.github.io/wasm-ops/
[evm-ist]: https://www.evm.codes/
