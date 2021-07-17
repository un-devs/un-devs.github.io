---
title: "A Journey to understand LLVM-IR!"
date: "2021-07-16"
layout: single
tags:
- LLVM-IR 
categories:
- llvm
---

## Why trying to understand LLVM-IR

![my encounter with the term](/assets/images/posts/picture1.png "my encounter with the term")

So, a few days ago I saw some tweet saying about Reversing a Rust binary claiming it as a pretty tedious task, indeed it is to be honest, previously I tried to understand the disassembly of a rust binary but due to extremely irritating symbols in Rust, I would say I just gave up, but yes this time I was pretty determined to do it, so I wrote a small Rust program, and installed the [cargo-disasm](https://github.com/ExPixel/cargo-disasm) crate which helped me out to understand the disassembly of the exact functions without spitting out tons of weird symbol names, anyways [I figured out later](https://twitter.com/ElementalX2/status/1414035130855960576?s=20), but during this period I encountered macros in this small program, so someone at the Rust Programming language Discord introduced to me the term ```LLVM IR``` , till date I just knew there exists LLVM, so this *IR* was a new term for me. So I just became quite curious and just started to learn more how do I read an LLVM IR because it looks extremely weird. 

![confused weird stuff from Rust Playground](/assets/images/posts/picture2.png "confusing weird stuff from Rust Playground")


## How did I start the journey?

Just after a simple google search using the keyword [understanding LLVM IR Code](https://lmgtfy.app/?q=understanding+LLMV+IR+code), I encounted this cool site known as [freecompilercamp](https://freecompilercamp.org/llvm-ir/) 
