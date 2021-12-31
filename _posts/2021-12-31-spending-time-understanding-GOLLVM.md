---
title: "Spending time understanding GOLLVM"
date: "2021-12-31"
layout: single
tags:
- LLVM-IR
categories:
- low-level-exploration
---


## Why this blog?

Well, hello everyone! I hope everyone is doing great, so after my first blog on exploring and trying to understand LLVM-IR, which was basically an outcome of a conversation at a discord server of Rust Language, similarly this is also an outcome of a private conversation which was focused on feedbacks regarding my [blog](https://un-devs.github.io/low-level-exploration/journey-to-understanding-llvm-ir/), I would like to thank [x0r19x91](https://twitter.com/x0r19x91) for  actually provoking my monkey brain with the fact that *Golang doesn't use LLVM based optimizer, rather Go has it's own compiler which has it's own optimizer which works in SSA*  behind writing this blog and exploring the GOLLVM Compiler and the Go compiler, and their respective IRs. Before getting started would definitely like to include this that GoLLVM is currently in development as per [these docs](https://go.googlesource.com/gollvm/#building) & I am not a subject matter expert when it comes to compilers, please feel free to correct me, I would be happy to receive positive criticism.


## Contents

- A quick recap. 
- What is GoLLVM?
- Setting up the environment.
- Building & Getting GoLLVM working.
- Running GoLLVM.
- Why GoLLVM?
- GC vS GOLLVM
- What about SSA-IR ?
- Better IR?
- We do a little bit of reverse engineering
- Inlining, Vectorization, register allocation.
- Understanding the IR.
- Author's two cents
- Resources

## A quick recap

So, before moving ahead to the original blog I would like to include few basic understanding regarding LLVM , what's LLVM IR?, what's an optimizer?. In extremely simple words LLVM is a compiler framework which priovides a target independent optimizer, static single assignment based compilation strategy capable of supporting both static and dynamic compilation of programming languages. Now what's an optimizer? Basically, it is a way to improve the Intermediate Code in order to increase the speed and performace of the program without altering the meaning of the program, also the process of optimization should not delay the compilation process. Now, you might be curious like *aren't like Optimizations are of different types*, Well, yes the different types of optimizations are :


- **Machine Independent Optimization** : As it sounds "machine" independent, optimizations which are done regardless of the processor's architecture, mostly in machine independent optimizations loop optimization is performed for *reducing the number of loops* & *reducing the number of times loops in a program should execute* there are various techniques of loop optimization like frequency reduction where a block of code is moved from a high frequency zone to a low frequency zone, where high frequency means `inside the code`  and low frequency means `outside the code`, now let us understand it this way:


```c

// Before applying frequency reduction
i = 0;
while ( i<= 300)
{
    a = 3*sin(x) / 2*cos(x) * i;
    i++

}
```

```c
j = 3*sin(x) / 2*cos(x);
i = 0;
while(i < 300)
{
    j * i;
    i++
}

```

Here, in the above original code, the value of `3sin(x)/2cos(x)` is being calculated multiple times, it is taking more time, everytime the loop runs till end, so why not just move that part of code from the `loop` which is a `high frequency zone` to outside of the loop to `low frequency region` and store the value inside a variable and then perform multiplication operation which definitely saves time. Then after this we go ahead to another method of loop optimization that is `Loop Unrolling` where the number of comparisions are decreased, after optimization is performed, let us understand it this way:

```c

//Before applying code unrolling
while ( i < 10){


x[i] = 0;
i++;
}
```


```c

// After applying code unrolling
while (i < 10){

    x[i] = 0;
    i++;
    x[i] = 0;
    i++;
}
```

Here, in the very first code, where the comparision happens 10 times and loop condition is checked 10 times, in the exact other loop the loop condition is checked 5 times. Other one technique for loop optimization is loop jamming where the optimization works by reducing the number of loops of the high level program. Now moving ahead to other machine independent optimization techniques, we are now at `Folding` where an expression should be reduced based on the fact that they can be easily calculated at the compile time by the it's value. Let us understand it this way:

`A+B+1+2+3+4+C` this can be easily `folded` to `A+B+C+10` where four expressions were ommited, now moving ahead to the last topic of machine independent optimization that is redundancy elimination which simply focuses on omitting re-writting of expression, for an example:

**Case 1:**

```
M = N + O
D = N + O + 9 + L
```

**Case 2:**

```
M =  N + O
D = M + 9 + L

```

Above in the very first set of expressions the `N+O` which is already equivalent to `M` is re-written, but in the next case, it is replaced with just `M` . So we saw a few cases of optimization and how it happens now moving on to the last part of our quick recap section that is what actually is an *IR* or *intermediate representation* , we can just simply understand it as representation somewhere between source and target language. You can learn more about IR in my previous blog and from this [site](https://cs.lmu.edu/~ray/notes/ir/).



## So what is GoLLVM???

![Gollvm](https://i.imgur.com/1hWAhAe.png)

Well going with the documentation and available information online at blogs and papers, would say GOLLVM is a compiler based project which uses the LLVM infrastructure, It incorporates “gofrontend” (a Go language front end written in C++ and shared with GCCGO), a bridge component (which translates from gofrontend IR to LLVM IR), and a driver that sends the resulting IR through the LLVM back end. this is a sub-project of [llvm](https://llvm.org/), now lets set up an environment to move ahead.


## Setting up environment

For setting up the environment inside a linux machine, the very first thing we could do is lurk into this [repository](https://github.com/thanm/dragongo) which helps and lays down a few commands for setting up the environment,next is to clone [gofrontend repository](https://go.googlesource.com/gofrontend).
```bash
mkdir work-space
cd gollvm ; git clone https://github.com/llvm/llvm-project.git
```

![making directory&installingllvm]()


```bash
cd llvm-project/llvm/tools
git clone https://github.com/thanm/dragongo
cd dragongo/llvm-gofrontend
git clone https://go.googlesource.com/gofrontend
cd ../../../../../
```

With the help of the above steps we would be able to set up the environment now lets build the gollvm & get it running . As Gollvm is under development we dont have any pre build gollvm binary, we need to build it with the help of *cmake* and *ninja*.

## Building & Getting GoLLVM Working

before building this make sure you have updated version of cmake and ninja.as well as a C/C++ compiler (V10.0 or later for Clang, or V6.0 or later of GCC), and a working copy of ‘m4’. lets first create a build directory and keep it seperate from the llvm-tree. 

```
cd work-space
mkdir build-debug
cd build-debug
cmake -DCMAKE_BUILD_TYPE=Debug -DLLVM_USE_LINKER=gold -G Ninja ../llvm-project/llvm
ninja gollvm
``` 

with this you would be able to build the gollvm now lets Installing gollvm


A gollvm installation will contain ‘llvm-goc’ (the compiler driver), the libgo standard Go libraries, and the standard Go tools (“go”, “vet”, “cgo”, etc).

The installation directory for gollvm needs to be specified when invoking cmake prior to the build so lets create a directory and move on with installation

```
mkdir build.rel
cd build.rel
cmake -DCMAKE_INSTALL_PREFIX=/my/install/dir -DCMAKE_BUILD_TYPE=Release -DLLVM_USE_LINKER=gold -G Ninja ../llvm-project/llvm

// Build all of gollvm
ninja gollvm

// Install gollvm to "/my/install/dir"
ninja install-gollvm

```

Now lets use the installed copy of gollvm to run a go program but before that we Programs build with the Gollvm Go compiler default to shared linkage, which means that they need to pick up the Go runtime library via LD_LIBRARY_PATH, so lets first set the path and run it:

`export LD_LIBRARY_PATH=/tmp/gollvm-install/lib64`
`export PATH=/tmp/gollvm-install/bin:$PATH`

## Running GoLLVM


Now, after installing and getting LLVM to work, let's try to write and compile a simple Go Program:

```go
package main
  
import "fmt"

func main() {
  
    fmt.Println("welcome to undev!")
}

```


```bash

$ go run hello.go

welcome to undev!

```

Great, we finally got it working!
