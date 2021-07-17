---
title: "A Journey to understand LLVM-IR!"
date: "2021-07-16"
layout: single
tags:
- LLVM-IR 
categories:
- low-level-exploration
---

## Why trying to understand LLVM-IR

![my encounter with the term](/assets/images/posts/picture1.png "my encounter with the term")

So, a few days ago I saw some tweet saying about Reversing a Rust binary claiming it as a pretty tedious task, indeed it is to be honest, previously I tried to understand the disassembly of a rust binary but due to extremely irritating symbols in Rust, I would say I just gave up, but yes this time I was pretty determined to do it, so I wrote a small Rust program, and installed the [cargo-disasm](https://github.com/ExPixel/cargo-disasm) crate which helped me out to understand the disassembly of the exact functions without spitting out tons of weird symbol names, anyways [I figured out later](https://twitter.com/ElementalX2/status/1414035130855960576?s=20), but during this period I encountered macros in this small program, so someone at the Rust Programming language Discord introduced to me the term ```LLVM IR``` , till date I just knew there exists LLVM, so this *IR* was a new term for me. So I just became quite curious and just started to learn more how do I read an LLVM IR because it looks extremely weird. 

![confused weird stuff from Rust Playground](/assets/images/posts/picture2.png "confusing weird stuff from Rust Playground")


## How did I start the journey?

Just after a simple google search using the keyword [understanding LLVM IR Code](https://lmgtfy.app/?q=understanding+LLMV+IR+code), I encounted this cool site known as [freecompilercamp](https://freecompilercamp.org/llvm-ir/), which helped me to get to the know that LLVM IR is based on static single assignment representation, wait what the hell is a static single assignment representation? Let me explain this with a simple example:

```
// Source Language Representation : 1 
a = c + d
b = a * k
a = b + i
b = d - n
a = a * b 
f = c + d 
```

The above example is a simple [three add code](https://www.geeksforgeeks.org/three-address-code-compiler/), basically variables and operators, so now let us convert this into static single assignment representation, just remember we add a suffix to variables if we see they are being assigned and used multiple times. 

```

//Target IR Representation : 2
a(1) = c(1) + d(1)
b(1) = a(1) * k 
a(2) = b(1) + i
b(2) = d(1) - n
a(3) = a(2) + b(2)
f(1) = a(1)
```

Okay, so if we check out the above representation 1 the value inside **a(1)**  & **f(1)** are same so we do not need any re-computation of that **f** variables, which somehow is a kind of optimization because we saved time to re-compute those variable and store them inside **f** , so this was just a small example of SSA-IR, LLVM-IR is based on this. 
