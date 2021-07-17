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

![confused weird stuff from Rust Playground](/assets/image/posts/picture2.png "confusing weird stuff from Rust Playground")


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


## Taking up a very basic example using C 

So, in the above section I just tried to make SSA representation quite simple, now it's time to write a simple C program and understand the overview of the LLVM-IR dump, I will follow the steps from the freecompilercamp and let's see what happens:


### Step 1: Clang & LLVM Optimizer.

You need to check out if clang is installed on your machine, if not `sudo apt install clang` command will do the job for you, next check for the clang version `clang --version` hopefully your output will be [something similar](/assets/image/posts/picture3.png "something similar"), then next go ahead and check for LLVM's optimizer version `opt --version`, if it is not installed can install this with the same command as above used to install clang. 


### Step 2: Writing a simple C program & generate the optimized LLVM-IR.

Once am done with checking the versions, the next step is to write a simple C program, compile it with Clang and then generate the optimized LLVM-IR

Tiny-C program:

```c
#include <stdio.h>

int add(void){

int a = 5;
int b = 8;

int result = a + b;
return result;

}

int main(){

puts("Jump to Void");

add();

}

```



**Compile it with it Clang the frontend of LLVM & emit un-optimized IR**

`clang -S -emit-llvm program.c` 

**Optimize the LLVM-IR using opt**

`opt -S -mem2reg -instnamer program.ll` 

where, `-mem2reg` flag stands for move as many variables to registers as possible & `-instnamer`
flag stands for assign names to anonymous  instructions.


Now after, running this command I was left with a file named `program.ll` where the output is stored, the optimized output is as follows:
```


; ModuleID = 'program.c'
source_filename = "program.c"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-pc-linux-gnu"

@.str = private unnamed_addr constant [13 x i8] c"Jump to Void\00", align 1

; Function Attrs: noinline nounwind optnone uwtable
define dso_local i32 @add() #0 {
  %1 = alloca i32, align 4
  %2 = alloca i32, align 4
  %3 = alloca i32, align 4
  store i32 5, i32* %1, align 4
  store i32 8, i32* %2, align 4
  %4 = load i32, i32* %1, align 4
  %5 = load i32, i32* %2, align 4
  %6 = add nsw i32 %4, %5
  store i32 %6, i32* %3, align 4
  %7 = load i32, i32* %3, align 4
  ret i32 %7
}

; Function Attrs: noinline nounwind optnone uwtable
define dso_local i32 @main() #0 {
  %1 = call i32 @puts(i8* getelementptr inbounds ([13 x i8], [13 x i8]* @.str, i64 0, i64 0))
  %2 = call i32 @add()
  ret i32 0
}

declare dso_local i32 @puts(i8*) #1

attributes #0 = { noinline nounwind optnone uwtable "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "frame-pointer"="all" "less-precise-fpmad"="false" "min-legal-vector-width"="0" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }
attributes #1 = { "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "frame-pointer"="all" "less-precise-fpmad"="false" "no-infs-fp-math"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }

!llvm.module.flags = !{!0}
!llvm.ident = !{!1}

!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{!"clang version 10.0.0-4ubuntu1 "}


```


### Step 3:Understanding the above LLVM-IR

Now let us understand the above content, from 

```
; ModuleID = 'program.c'
source_filename = "program.c"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-pc-linux-gnu"
```

Module ID is just a refernce to the current module then the next goes for source file name, the target data layout etc. The `e` specifies that the target laid out is in little-endian form, then `-m:e` states that llvm symbols are mangled and type of mangling is _ELF Mangling_ here, then `-i64:64` states the alignment for an integer type of given bit, `f80:128` alignment of floating type in given bit, then `-n8:16:32:64` refers to the native integer widths for the target CPU in bits here it goes for _X86-64_ then `S128` determines the natural alignment of the stack in bits. The last `target_triple` states about the _target-Vendor-OS_.



```
@.str = private unnamed_addr constant [13 x i8] c"Jump to Void\00", align 1

; Function Attrs: noinline nounwind optnone uwtable
define dso_local i32 @add() #0 {
  %1 = alloca i32, align 4
  %2 = alloca i32, align 4
  %3 = alloca i32, align 4
  store i32 5, i32* %1, align 4
  store i32 8, i32* %2, align 4
  %4 = load i32, i32* %1, align 4
  %5 = load i32, i32* %2, align 4
  %6 = add nsw i32 %4, %5
  store i32 %6, i32* %3, align 4
  %7 = load i32, i32* %3, align 4
  ret i32 %7
}

```

Now the `@.str = private unnamed_addr constant [13 x i8] c"Jump to Void\00", align 1` states that a string constant known as _.@str_ is declared as a global constant, and the size of character is 8 bits so `[13 characters of 8 bit]` are initialized to that constant known as **"Jump to Void\00"** and `private` denotes that linkage are only accessible by objects in current module, `unnamed_address` denotes that the address of that string is exactly not known within the module, then `define dso_local i32 @add() #0 ` is the function definition for add `add()`, this `define` keyword is used while declaring functions in LLVM, `dso_local` specifies that the a function will resolve to a symbol within the same linkage unit, then `@add ` is the name of function and `#0` is a function attribute, which denotes that the inliner should never inline this function in any [situation](https://stackoverflow.com/questions/22767523/what-inline-attribute-always-inline-means-in-the-function). Then there are these temporary or local variables `%1 , %2, %3, %4, %5, %6, %7, ` which perform operations of allocating using `alloca` which reserves 4 a space of 4 bytes on the stack frame of the function `add()` then the other temporary variable do the same, then writing to memory is performed using `store` instruction the store instruction has two arguments a value which we want to store and a value to be stored in a certain memory address,here `5` is the value of type `i32` and is to be stored to a pointer of type `i32`  named `%1` . The next instruction at the next instruction of the integer pointer `%1` is now loaded to the variable named `%4` then the value of the integer pointer `%2` is loaded to the variable `%5` then both the values of 4 & 5 are now now added and stored inside variable `%6` now if we remember that variable we declared named `%3` but we never used, so now the value inside `%6` is stored inside the pointer `%3` , then the last variable `%7` the value stored in the pointer variable `*%3` is now stored to variable named `%7` and finally `%7` is returned.


### Trying to re-write this a bit similar to the original source code. 

Remember, the SSA IR? Yes, we are going to use the same suffix notation here so don't get confused with the terms.

```
define dso_local i32 @add() #0 {

a(1) = alloca i32, align 4
b(1) = alloca i32, allign 4
a(2) = alloca i32, allign 4
store i32 5, i32 *a(1) , allign 4
store i32 6, i32 *b(1), align 4
b(2) = load i32, i32 *a(1) , align 4
a(3) = load i32, i32 *b(1), align 4
a(4) = add nsw i32 b(2) + a(3)
store i32 a(4), i32 *a(2), align 4
result(1) = load i32, i32 *a(2), align 4
ret i32 result
}
```
I wrote this above code snippet to make it quite easy to relate to the original source code. 





```
define dso_local i32 @main() #0 {
  %1 = call i32 @puts(i8* getelementptr inbounds ([13 x i8], [13 x i8]* @.str, i64 0, i64 0))
  %2 = call i32 @add()
  ret i32 0
}

declare dso_local i32 @puts(i8*) #1


```



Now after understanding the add() function, we are at the main() function where we have two variables `%1` & `%2` which are definitely temporary vbariables, in the first temporary variable we call the `puts()` function which has a return type `i32` and which 









Finally, I learnt to understand LLVM-IR, not sure I can write it well, may be the next blog, who knows :laugh: !Let's apply this on a different programming language known as Rust and see if we can understand without any discomfort.




## Writing a basic Rust program and understand
## Summary