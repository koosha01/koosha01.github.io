---
layout: post
title:  "Minimal Assembly Reference"
tags: assembly binary 
excerpt_separator: <!--more-->
---
Assembly language is without doubt one of the most important topics in binary reverse engineering and exploitation. 

Of course Assembly is huge and there are lots of great resources to learn from. This post however represents an abstract of what I've learned during an assembly course. You can find it [here](http://www.securitytube.net/groups?operation=view&groupId=5) or [here](https://www.youtube.com/playlist?list=PLyqno_bgl3e-zLBZGdi_zsPQYPQUlZYe4). It's more appropriate in case you want to look up instructions and read binary.  
Hope it comes useful :)
<!--more-->
## Some words on Memory
---

### Memory Layout

Memory plays a big role in executing a binary. Basically the whole program including the source code, variables and function parameters have to be loaded onto the memory in order to be running. Here is how memory looks like in a general view:

![Simple Memory Layout](/assets/simple-memory-layout.png)

- Stack: Stores function arguments and local variables. 
- Heap: Dynamic memory. Used by functions like malloc().
- BSS Segment(.bss): Uninitialized data.
- Data Segment(.data): Initialized data.
- Code Segment(.text): Program code.

A more detailed view with a practical example:

![Advanced Memory Layout](/assets/advanced-memory-layout.png)

### Registers

Beside computer's memory, registers are fast storage spaces in CPU enabling the processor doing various operations and instructions. Here is a list of important ones, their names and duties:

**General Purpose Registers:**

![General Purpose Registers](/assets/general-registers.png)


**Pointer Registers:**

![Pointer Registers](/assets/pointer-registers.png)

**Segment Registers:**

![Segment Registers](/assets/segment-registers.png)

**Flag Registers:**

![Flag Registers](/assets/flag-registers.png)

## GDB
---
GDB (short for Gnu Debugger) is a powerful debugger you could use for your binary analysis. These are a number of commands which I use more frequently: 

|Command, shorter version [parameters]|Explanation|
|---|---|
|list, l [line number or function name]|Views source code (if available).|
|run, r [program arguments]|Starts the program.|
|disassemble, disas [function name, address]|Disassembles a specified section of memory. <br> With */s* modifier shows source code (if available) and with */r*, the hex instructions.|
|break, b [function name, memory address or ...]|Sets a breakpoint at the specified location.|
|continue, c|Continues program being debugged after a breakpoint.|
|info [variables, registers, etc]|Shows information about different program elements.|
|x|Examines memory. <br> Usage: x/FMT ADDRESS. <br> Example: x/12cb 0x08049024. Shows 12 bytes in character format at the specified address.|
|print|Prints the value of an expression.|
|step [n]|Steps through the program until it reaches a different line. Giving number *n* it'll step n times.|
|set|Sets the variable to the specified expression. <br> Usage: set Var = EXP <br> Example: set step-mode on. Enables stepping through the program line by line.|

## Assembly Language
---

### Structure of an Assembly Program

An Assembly source code can be divided into several segments, giving the code a structure to follow. It's much like a C program in which you declare some headers at the beginning, then define global variables and routines and finally write your primary code into the *main* function.

![Assembly Structure](/assets/assembly-structure.png)

### Linux System Calls

In a nutshell system calls are kernel APIs. An essential interface between a process and the operating system. This way the user(program) can ask for various services from the kernel of the operating system.

**Procedure of using Linux system call:**

1. Put the system call number in EAX.
2. Put system call arguments in EBX, ECX, EDX, etc.
3. Invoke the system call by the relevant interrupt.

**Some useful syscalls:**

|EAX|Name|EBX|ECX|EDX|
|:---:|:---:|:---:|:---:|:---:|
|1  |Exit|int|-|-|
|2  |Fork|struct pt_regs|-|-|
|3  |Read|unsigned int|char *|size_t|
|4  |Write|unsigned int|const char *|size_t|
|5  |Open|const char *|int|int|
|6  |Close|unsigned int|-|-|


**Example:**  
(Exiting a program)

```
movl $1, %eax
movl $0, %ebx
Int $0x80
```

### Compiling an Assembly Program

It's as simple as follow:

```
$> as file.s -o file.o
$> ld file.o -o file
```

So we first made the object file out of the assembly code (with the help of *as* assembler) and then using ld (Gnu linker) built the executable binary.

### Data Types

Data types and how to define them in different segments:

**.data:**

|Data Type|Explanation|
|:-------:|:---------:|
|.byte|One byte|
|.ascii|String|
|.asciz|Null terminated string|
|.int|32 bit integer|
|.short|16 bit integer|
|.float|Single precision floating point number|
|.double|Double precision floating point number|

**.bss:**

|Data Type|Explanation|
|:-------:|:---------:|
|.comm|Common memory area|
|.lcomm|Local memory area|

**Example:**

```
.data:
	HelloWorld:
	.ascii “Hello world!!”

	Int32: 
	.int 2

	Float:
	.float 10.23

.bss
	.comm LargeBuffer, 1000
```

### Moving Data

Using *MOV*s instructions we can transfer data from one location to another.

**Usage format: MOVx source, destination**

Different types of *mov* instructions and their examples:

1. movl: moves a 32 bit value.  
	
	`movl %eax, %ebx`

2. movw: moves a 16 bit value.  

	`movw %ax, %bx`

3. movb: moves a 8 bit value
	
	`movb %ah, %bh`

**Moving Types:**

There are different types of data transition based on the source and destination. Here we look at each with an example:

1. Between Registers:

	`movl %eax, %ebx`

2. Between Registers and Memory:

	```
	location:
		.int 10

	movl %eax, location
	movl location, %ebx
	```
3. Immediate value into Register:

	`movl $10, %ebx`

4. Immediate value into memory location:

	```
	location:
		.byte 0
		
	movb $10, location
	```
5. Moving Data into an Indexed Memory Location:

	Here's the syntax to access a place in the indexed memory location:  
	**BaseAddress(Offset, Index, Size)**.  
	See the example bellow which moves 4 bytes into the third place of the array(I.e 30):
	
	```
	IntegerArray:
	.int 10, 20, 30, 50
	movl %eax, IntegerArray(0, 2, 4)
	```
	
**Indirect Addressing:**

Two ways of indirect addressing:
 
1. Placing *$* before a label name => Getting the memory address of the variable.

	Example:  
	Moving the address of *location* variable to register EDI.

	`movl $location, %edi`

2. Putting parentheses around a register => Pointing to the address which the register contains.

	Example:  
	Moving the integer 9 to the memory location pointed by EDI register.

	`movl $9, (%edi)`

	Example:  
	Moving the integer 9 to the memory location pointed by EDI + 4.

	`movl $9, 4(%edi)`

	Example:  
	Moving integer 9 to the memory location pointed by EDI – 2.
	
	`movl $9, -2(%edi)`
	
### Working with Strings

There are multiple ways around string transmission just like the regular data. We inspect each one's situation:

**Moving strings from one memory location to another:**  

1. ESI points to the source memory location.
2. EDI points to the destination memory location.
3. MOVSx instructions are used:
	- MOVSB: moves a byte (8 bits)
	- MOVSW: moves a word (16 bits)
	- MOVSL: moves a double word (32 bits)
4. DF flag is set appropriately.

**Direction Flag (DF):**

Decides whether to increment or decrement ESI and EDI registers after a MOVSx command.  
If set (DF = 1) ESI and EDI will be decremented.  
If not set (DF = 0) ESI and EDI will be incremented.  
DF can be set using STD instruction and be cleared using CLD command.

Note: All decrements or increments in the string instructions are controlled by Direction Flag. 

**REP Instruction:**

Repeats a string instruction over and over till ECX register has value > 0.  

Usage:  

- Load ECX with string length.
- Use REP MOVSB to copy string from source to destination.

**Loading strings from memory into registers:**

1. Loads into EAX register.
2. String source is pointed by ESI.
3. LOADSx instructions are used:
	- LODSB: loads a byte from memory location into AL.
	- LODSW: loads a word from memory location into AX.
	- LODSL: loads a double word from memory location into EAX.

**Storing strings from registers into memory:**

1. Stores from EAX register into memory location.
2. String destination is pointed by EDI.
3. STOSx instructions are used:
	- STOSB: stores AL into the memory location.
	- STOSW: stores AX into the memory location.
	- STOSL: sotores EAX into the memory location.

**Comparing Strings:**

1. ESI register points to the source memory location.
2. EDI register points to the destination memory location.
3. CMPSx instructions are used:
	- CMPSB: compares a byte value.
	- CMPSW: compares a word value.
	- CMPSL: compares a double word value.

The result can be seen by the flags set in the EFLAG register.

**REPZ and REPNZ Instructions:**

- REPZ: repeats string instruction while zero flag is set.
- REPNZ: repeats string instruction while zero flag is not set. 

### Branching 

**Unconditional branching:**

1. JMP:
	- Can be compared with GOTO in C.
	- syntax: `JMP Label`.
2. Call:
	- Like calling a function in C.
	- syntax: `Call Label`.
	- Call is associated with RET instruction.

**Conditional Branching:**

- JZ: Jump if Zero Flag is set.
- JNZ: Jump if Zero Flag is not set.
- JG: Jump if Greater (for arithmetic purposes).
- JEG: Jump if Greater or Equal.
- JL: Jump if less.
- ...

**Loop Instruction:**

- What it does: Loops through a set of instructions for the x number of times which x is stored in ECX register.
- Simple usage:
	+ Put the number of times to loop in ECX.
	+ Use this sample:
		
		```
		Label:
			<code>
			<code>
			Loop Label
		```

### Functions

**Functions in Assembly:**

- Defining:  
	
	```
	.type MyFunction, @function
	```
	(Which MyFunction is the desired label for the function.)

- Usage:

	```
	MyFunction:
		<code>
		<code>
		ret
	```

- Invoking: It’s done using CALL instruction.

## Resources
---

- Pictures for Memory layout:
	
	```
	http://dmitrysoshnikov.com/compilers/writing-a-memory-allocator  
	http://tetru.blogspot.com/2014/09/the-memory-layout-of-computer-program.html
	```

- Pictures for registers:
	```
	https://wiki.osdev.org/CPU_Registers_x86-64
	```
