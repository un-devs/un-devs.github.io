# x86_64 Assembly Speed Run using NASM 
## _Just another handy cheatsheet_


## Contents

- Hello World
- Registers
- Syscalls
- Sections
- Labels
- EFLAGS
- Pointers
- Jumps(Signed & Unsigned)
- Subroutine
- Calls
- Arithmetic
- Stack
- Files
- Macros










## Hello World

```nasm

section .data
message db "My First line in x86_64", 10

section .text
global _start

_start:

mov rax, 1
mov rdi, 1
mov rsi, message
mov rdx, 23
syscall

mov rax, 60
mov rdi, 0
syscall
```


## So, we wrote our first ```Hello World``` , now what are those `rax` , `rdi` , `rsi` ?? :no_mouth: 

These above terms are known as Registers, I feel this definition is one of the most crystal clear explanations of what registers actually are in terms of assembly language. [Click Here.](https://electronics.stackexchange.com/a/491586) Once you are familiar with the definition of the term ```Registers```, we look forward to some of them:


| Register(64-BIT) |32-bit  | 16-bit | 8-bit |
|------------------|--------|--------|-------|
| RAX              |  EAX   | AX     | AL    |   
| RBX              |  EBX   | BX     | BL    |
| RCX              |  ECX   | CX     | CL    |
| RDX              |  EDX   | DX     | DL    |
| RSI              |  ESI   | SI     | SIL   |
| RDI              |  EDI   | DI     | DIL   |    
| RBP              |  EBP   | BP     | BPL   |                
| RSP              |  ESP   | SP     | SPL   |
| R8               |  R8D   | R8W    | R8B   |
| R9               |  R9D   | R9W    | R9B   |
| R10              |  R10D  | R10W   | R10B  |
| R11              |  R11D  | R11W   | R11B  |
| R12              |  R12D  | R12W   | R12B  |
| R13              |  R13D  | R13W   | R13B  |
| R14              |  R14D  | R14W   | R14B  |
| R15              |  R15D  | R15W   | R15B  |
|------------------|--------|--------|-------| 

### Bonus : Wait, what is this (E) prefix :thinking: and (D), (w) , (B) suffixes etc ..?
-> The prefix E stands for ```E```xtended versions of 16-bit registers, that is we are given extra 16 bits along with the 16-bit registers, (X) also can be termed as E```x```tended or implying 16 as in hexadecimal, the **X** suffixed registers are the extension of 8-bit registers, that is we are given extra 8 bits along with the 8-bit registers, the (L) suffix in 8-bit registers mean low. Also along with these some additional suffixes you might encounter (B) which also mean"LOW" , please remember L is not long here, the suffix (D) mean double, and last but not the least (W) stands for word, as you probably be knowing a word takes up 16 bits and a Double takes up 32 bits. Hopefully the confusion regarding the suffix and prefixes are clear :) 

Citations : 

[Stack Overflow](https://stackoverflow.com/questions/1753602/what-are-the-names-of-the-new-x86-64-processors-registers)

[Stack Overflow](https://stackoverflow.com/a/43933932)

### What is this ```mov``` stuff ? :thinking: 

-> MOV here is an instruction that moves contents from the first operand to second operand the contents can sometimes be memory addresses, registers contents, or sometimes value such as in the above code snippet we see ```mov rax, 1``` . Let us simplify it a bit:

```nasm

mov eax, ecx ;moves contents of ecx into eax
mov [some_memory_address], eax ; moves the contents of eax into [some_memory_address]
mov eax, [esi+2] ; move two bytes at memory address *(esi+2) into eax. 

```
There are lots of other instructions which are present. You can check [this](https://ax1al.com/projects/rez/index.html) out for a dedicated place for other instructions for x86_64 architecture set.

### What are these .data & .text section :thinking:

-> The text section is the region which contains the instructions and the ```.data``` section is the region where data elements are stored or the non-stack based variables and constants lie. 

Cited from [Stack Overflow](https://stackoverflow.com/a/14544134)

### What is this term syscall and why are we moving random values like ```1``` , ```message``` to the registers :thinking: ?

-> In layman words using a syscall instruction means we are asking the operating system to perform some task as requested may be like read, write, sendfile, or get the current process ID of a running process, there are lots of them which can be held accountable of doing these tasks. Now let us understand this from the above example:

```nasm

mov rax, 60 ;first argument
mov rdi, 0  ;first and only argument
syscall ;invoking of syscall

```

Syscalls apparently take arguments, now these registers are accountable for holding these arguments but which of them and whow can you know them?

| Argument         |  Registers| 
|------------------|-----------|
| ID               | RAX       | 
| 1st Argument     | RDI       | 
| 2nd Argument     | RSI       |    
| 3rd Argument     | RDX       |                
| 4th Argument     | R10       |
| 5th Argument     | R8        |
| 6th Argument     | R9        |
|------------------|-----------|

Now as per our small program, we can see that the ```RAX``` stores the syscall ID which is ```60``` here, a bit of googling and we land up that the name of this syscall is ```exit()``` syscall which is responsible for termination of the program and it takes only argument that is the exit error code .

```c
void _exit(int status)
```

Here, the value which is stored in the ```rdi``` register is zero, so here it just exists with no error :-) . Wait we did not explain the first part of program :eyes: where the syscall ID is ```1``` , I will leave this upto you to figure out the arguments, just as a small hint the syscall with ID = 1 is ```sys_write``` . 

### What actually is the ```_start``` ? 

-> This is a label a part of code which is assigned the current value of the active location, during assembly. I would also suggest checking [this definition](https://qr.ae/pGM19t) out which I found as one of the easy to understand definition regarding labels. Labels are mostly of two types
  - Symbolic Labels: The ones which consists of an identifier or symbols in layman terms a ```name``` followed by a colon ```:```, they are defined only once. For example we have the label in our code ```_start``` which is a symbolic label.
  
  - Numeric Labels : The ones which contains of single digit from the range [0-9] also followed by a colon.
  
  Citations: [Docs](https://docs.oracle.com/cd/E19120-01/open.solaris/817-5477/esqaq/index.html)
  
  ### What if we rename the label _start to something else :thinking: ?
  
  Yes, when we are writing our first program, we might be curious why not change the _start to something else ? But wait it pops up with an error ```cannot find entry symbol _start``` , check this out to know why ? [Check Out](https://www.linuxquestions.org/questions/programming-9/ld-warning-cannot-find-entry-symbol-_start-833944/). 
  
  Finally , we completed understanding each and every part of our first basic program , now we will move ahead with understanding Jumps, calls, comparision, subroutines, stack, macros !
  
 ```nasm
section .data

message db  "We will now go through Jumps", 10
jumped_message dw "Jump has successfully taken place", 10
rcxval db "Test", 10

section .text
global _start

_start:

	
        mov rax, 1
	mov rdi, 1
	mov rsi, message
	mov rdx, 29
	syscall

	xor eax, eax
	cmp eax, 0
	je _jumpexecute
_exit:

	mov rax, 60
	mov rdi, 0
	syscall 

_jumpexecute:
	mov rax, 1
	mov rdi, 1
	mov rsi, jumped_message
	mov rdx, 35
	syscall
	mov eax, 0
	cmp eax, 0
	je _valueofrcx

_valueofrcx:
	mov rax, 1
	mov rdi, 1
	lea rcx, [rcxval]
	mov rsi,rcx
	mov rdx, 5
	syscall
	mov eax, 0
	cmp eax, 0
	je  _exit	

```
[![asciicast](https://asciinema.org/a/pA79uIjEuSCvOXdNYhXWmRuwt.svg)](https://asciinema.org/a/pA79uIjEuSCvOXdNYhXWmRuwt)

Before we move ahead, we will modify our previous hello world program a bit, assemble it with NASM and then link it which will print us the message:

```
 We will now go through Jumps
Jump has successfully taken place
Test

```

Explanation:

























```nasm
 1	section .data
       
     2	;message db "Hello World!" ,10
     3	greet dw "Hey there, What is your name? ", 10
     4	text2  dw "My name is, ", 10
     5	askage dw "Hey what's your age? ", 10
     6	soage dw "So your age is, ", 10
       
     7	section .bss
       
     8	yourname resb 20
     9	age resb 5
       
    10	section .text
    11	    global _start
       
    12	_start:
    13	    call _greet2
    14	    call _getthename
    15	    call _text2
    16	    call _printname
    17	    call _asktheage
    18	    call _getage
    19	    call _text3
    20	    call _printage
    21	    call _exit
       
       
    22	_greet2: 
       
    23	        mov rax, 1
    24	        mov rdi, 1
    25	        mov rsi, greet
    26	        mov rdx, 31
    27	        syscall
    28	        ret
       
    29	_getthename:
       
    30	    mov rax, 0
    31	    mov rdi, 0
    32	    mov rsi, yourname
    33	    mov rdx, 20
    34	    syscall
    35	    ret
       
    36	_text2:
    37	    mov rax, 1
    38	    mov rdi, 1
    39	    mov rsi, text2
    40	    mov rdx, 11
    41	    syscall
    42	    ret
       
       
       
    43	_printname:
    44	    mov rax, 1
    45	    mov rdi, 1
    46	    mov rsi, yourname
    47	    mov rdx, 20
    48	    syscall
    49	    ret
       
       
    50	_asktheage:
       
    51		mov rax, 1
    52		mov rdi, 1
    53		mov rsi,askage
    54		mov rdx,22
    55		syscall
    56		ret
       
    57	_getage:
       
    58		mov rax, 0
    59		mov rdi, 0
    60		mov rsi,age
    61		mov rdx, 5
    62		syscall
    63		ret
       
    64	_text3:
       
    65		mov rax, 1
    66		mov rdi, 1
    67		mov rsi, soage 
    68		mov rdx, 17
    69		syscall
    70		ret
       
    71	_printage:
    72		mov rax, 1
    73		mov rdi, 1
    74		mov rsi, age
    75		mov rdx, 5
    76		syscall
    77		ret
       
    78	_exit:
    79	    mov rax, 60
    80	    mov rdi, 0
    81	    syscall
```


  

