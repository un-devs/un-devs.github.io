---
title: "A Journey to understand LLVM-IR!"
date: "2021-07-16"
layout: single
tags:
- LLVM-IR 
categories:
- llvm
---

# Why trying to understand LLVM-IR

![my encounter with the term](/assets/images/posts/picture1.png "my encounter with the term")

So, a few days ago I saw some tweet saying about Reversing a Rust binary claiming it as a pretty tedious task, indeed it is to be honest, previously I tried to understand the disassembly of a rust binary but due to extremely irritating symbols in Rust, I would say I just gave up, but yes this time I was pretty determined to do it, so I wrote a small Rust program, and installed the [cargo-disasm](https://github.com/ExPixel/cargo-disasm) crate which helped me out to understand the disassembly of the exact functions without spitting out tons of weird symbol names, anyways [I figured out later](https://twitter.com/ElementalX2/status/1414035130855960576?s=20), but during this period I encountered macros in this small program, so someone at the Rust Programming language Discord introduced to me the term ```LLVM IR``` , till date I just knew there exists LLVM, so this *IR* was a new term for me. 