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

## What is Zink?

[Zink](https://github.com/zink-lang/zink) is a rustic smart contract langauge for EVM, it uses WebAssembly (WASM) as an 
intermediate format before generating EVM bytecode.

We're choosing WASM as the intermediate representation with the following reasons:

1. WASM can be compiled from rust source code directly
2. WASM has simple instruction set,  172 instructions in the MVP implementation (127 of them are numeric instructions).
3. WASM is backed by Mozilla and the Rust Team and it's widely used in the world with powerful toolchains. 

WASM can not be executed by EVM directly for several reasons:

1. **Instruction Set**: EVM has its own instruction set, which is optimized for executing Solidity smart contracts. WASM, 
on the other hand, has a different instruction set that is designed for general-purpose computation.

2. **Memory Model**: EVM uses a memory model that is specific to Ethereum's blockchain architecture. WASM, being a stack-based 
virtual machine, has its own memory model that is not compatible with EVM's memory layout.

3. **Function Call Conventions**: EVM has its own function call conventions, which are designed for executing Solidity smart 
contracts. WASM, being a stack-based VM, uses different function call conventions that would need to be adapted for execution 
on the EVM.

