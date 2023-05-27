# SOPL
Stack-oriented Programming Language

## About

> What is sopl?

Sopl is a lightweight, easily extendable, stack-oriented language.
It is currently in it's very early stages and is syntacically bound to change over time. I'll try to not change it semantically too much but be warned if you are using it to build projects yourself as things may break over versions.
For more info on current versions checkout [version.md](version.md)

**Manual:**
Made for easy navigation around the README.md
- [Plans](#plans)
- [Build platforms](#build)
- [Requirements](#Requirements)
- [Startup](#startup)
- [Flags and versions](#flags-and-versions)
- [How-to-use](#how-to-use)
  - [Hello-World](#hello-world)
  - [Externals](#extern)
  - [functions](#functions)
  - [including](#including)
  - [registers](#registers)
  - [managing-c-functions](#managing-c-functions-and-their-return-values)
  - [strings](#strings)
  - [control-flow](#control-flow)
  - [constants](#constants)
  - [locals](#locals)
  - [interrupts](#interrupts)

## Plans
> Source
- [x] externals
- [x] strings
- [x] register access
- [x] includes
- [x] functions
- [x] constants
- [ ] control flow
  - [x] if statements
  - [x] else statements
  - [ ] else if statements
  - [ ] while statements
  - [ ] for statements
- [x] locals
- [ ] type checking
  - [x] Function type checking
  - [ ] Branching
- [ ] structs
- [ ] struct methods
- [ ] interfaces


## Build
> Build platforms: 

Because of how the project is structured, it can easily be expanded to not only NASM assembly but to virtually any stack machine out there.
Current supports include:
- [x] NASM x86 64
- [ ] FASM x86 64
- [ ] Java bytecode
- [ ] Native simulation

Ideas that are left for discussion and further expansion:
- [ ] ARM assembly
- [ ] C

[NOTE] that whilst registers are really powerful and useful for building with x86 64, they might get 'discontinued' after Java integration (theoretical Java integration). They will have support for nasm but in the future their usage will be warned and their features replaced by more modern ones. Deprecation isn't expected any time soon but when the majority of the boxes under 'plans' are ticked off you probably will expect deprecation of them. 
For anyone wandering things like interrupts are probably going to be handled like so:
```sopl
interrupt 128, 1, 1, "Hello World!" // Print Hello World! on linux
```
^ Regarding previous [NOTE], functions now no longer take in arguments through the stack and have more modern syntax, lowering the practical usage of registers even more [0.11A](version.md#011a)
## Requirements
**NASM**:
- Any nasm version that can support 64 bit or 32 bit assembly.
- Any linker or executable builder (gcc, ld, etc.)
## Startup
```cmd
cargo run (PLATFORM) (File path) -o (Output path)
```
Another thing worth noting:
[NOTE] If you are experiencing an error of the following type:
```
[NOTE] No architecture found for {OS}_{Arch}! Please specify the output architecture!
```
It is probably because the compiler can't find a proper architecture for your current operating system and Arch. To fix this you can use the -arc flag with | and the path to the architecture configuration (examples [here](examples/arcs/), This should fix the error although it could cause issues if the configuration isn't correct) Like so:
```
cargo run nasm_x86_64 myprogram.spl -arc | myarcconfig.json 
```
## Flags and Versions

Whenever something gets added you will see that in 99% of the time [version.md](version.md) gets updated. Thats because it contains the necessary information about any new versions that come out.
Whilst the README is updated oftenly, for the most modern features you might have to check out the [version.md](version.md) since it contains any information on the latest patches and updates. 
It also gives you a timeline of the most recent changes (newest -> oldest).

Flags are really important and if not updated here, you can always check out what flag support there is by just running the compiler without anything (This will display information such as the usage, the currently supported builds and any flags you might want to use).

As of [0.11A](version.md#011a) flags consist of:
```
--------------------------------------------
(output language) (input path) [flags]
    Output Language:
        - nasm_x86_64
    flags:
        -o (output path)                    -> outputs to that file (example: hello.asm in nasm_x86_64 mode). If the output path is not specified it defaults to the modes default (for nasm_x86_64 thats a.asm)
        -r                                  -> builds the program for you if the option is available for that language mode (for example in nasm_x86_64 it calls nasm with gcc to link it to an executeable)
        -noRaxWarn                          -> removes the RAX usage warning for nasm
        -release                            -> builds the program in release mode
        -ntc                                -> (NoTypeChecking) Disable type checking
        -nou (all, funcs, externs, strings) -> Disable unused warns for parameter
        -callstack (size)                   -> Set callstack size.
                                               The name is very deceiving but callstack is now only used for locals as of 0.0.6A (checkout versions.md)
                                               [NOTE] it is planned for -callstack to be deprecated for instead using the stack as a way to store variables with the new function system
        -arc (builtin arc)                  -> builds for a builtin architecture
        -arc | (path to custom arc)         -> builds for a custom architecture following the syntax described in ./examples/arcs
--------------------------------------------
```
## How to use?
*Note that documentation may not cover all of the latest features tho you might expect updates on them shortly after implementation*
### Hello World!
```sopl
extern "C" printf(ptr : int)

func main() {
  printf("Hello World!"c);
}
```
### extern
  * Changed syntax in [0.11A](version.md#011a)
  * Added Files related in [0.7A](version.md#07a):
    - [stdio.spl](src/C/stdio.spl)
    - [stdlib.spl](src/C/stdlib.spl)
  * Added in Pre 0.1A
**Syntax**:
```
extern [TYPE] (name) <(extern contract)>;
```
Extern is a way to access outside features of the language or link to functions from other languages such as IO implementations from C etc.
It has to be noted here that using extern on its own can sometimes lead to unexpected behavior ~~or program slowdowns as most externs don't get type checked automatically~~ (as of 0.11A externs ARE typechecked) and
might not be available for the current mode you are running in like if you try to link to standard C functions when you are in JAVA mode.

Thats also a reason why externs are generally not recommended for direct use (although depending on your version it might be required to use them raw (currently its not required - use stdio.spl, stdlib.spl directly instead of listing externs every time :D. If you want to implement any C library you can do that and share the information online)).

Externs are really powerful if you want to link to libraries that aren't previously implement and are generally really good to use for building things that require platform specific things such as Windows applications etc.

**Example(s)**:
```
extern "C" puts(ptr : int); // Links with puts from C
func main(){
  puts("Hello World"c) // uses it to print a string to the console
}
```

```
extern "C" puts(ptr : int); // Links with puts from C
func main(){
  "Hello World"c
  puts(top) // uses it to print a string to the console (copying it) from the top of the stack
  puts(top)
  puts(top)
  pop
}
```
### functions
* 0.11A Changed how functions are called with the introduction of argument contracts
* 0.6A Changed to now have named parameters
* Pre 0.1A

**Syntax**:
```
func (name) ((contract)) {

}
```
Functions, just like in other languages have a name and a body. In sopl however you have to provide a contract:

A contract is a way to tell the type checker what to expect to have put onto the stack after it calls your function.
The contract contains:
(Input parameters separated by , : Output parameters separated by ,)

Inside of functions you can use the "ret" keyword to return although it is automatically done for you at the end of the functions declaration.

**Example(s)**:
```sopl
func sayHello() {
  printf("Hello World!\n"c)
}
func counter(c: long : long) {
  // Old syntax:
  //RBX pop
  //RCX = 1
  //RBX RCX -
  //RBX push
  //counter
  //RBX pop
  //pop
  //rs RBX push

  // New (planned) syntax:
  c -= 1
  counter(n) 
  rs RAX push
}
```

As of 0.6A, there is a new keyword called 'rs' short for "Return stack"
As of 0.11A, the 'rs' keyword is planned to be deprecated, and instead replaced with the new syntax and C standard

### including
* Pre 0.1A

**Syntax**
```spl
include "Path/To/File"
```
Functions.spl:
```spl
extern "C" puts(ptr : int)
func sayHello() {
  printf("Hello World!")
}
```
HelloWorld.spl:
```spl
include "./Functions.spl"
func main() {
  sayHello()
}
```
### registers

* Added float registers and the last remaining r\<n\> registers in [0.11A](version.md#011a)
* Added the majority of the registers Pre 0.1A

Current register support:
- [x] RAX
  - [x] EAX
    - [x] AX
      - [x] AH
      - [x] AL
- [x] RBX
  - [x] EBX
    - [x] BX
      - [x] BH
      - [x] BL
- [x] RCX
  - [x] ECX
    - [x] CX
      - [x] CH
      - [x] CL
- [x] RDX
  - [x] EDX
    - [x] DX
      - [x] DH
      - [x] DL
- [x] RSI
  - [x] ESI
- [x] RDI
  - [x] EDI
- [x] RSP
  - [x] ESP
- [x] RBP
  - [x] EBP

All of their sub types respectfully:
- [x] R8
- [x] R9
- [x] R10
- [x] R11
- [x] R12
- [x] R13
- [x] R14
- [x] R15

- [x] XMM0
- [x] XMM1
- [x] XMM2
- [x] XMM3


**[NOTE] RAX is used for returning longs/int/shorts/chars for C functions and your functions so usage of it outside of that is not advised** 
**Syntax**
```
(RegisterName) (Op | RegisterName) [RegisterName | Op]
```
**Example(s)**
```sopl
RBX pop   // Pops RBX off the stack
RCX = 4   // sets RCX
RBX RCX + // Adds RCX to RBX
```

Currently supported register operations:
```
(REG1) (REG2) +  -> adds the two registers together (RESULT IN REG1)
(REG1) (REG2) -  -> subtracts the second register from the first register (RESULT IN REG1)
(REG1) (REG2) *  -> multiples the two registers     (RESULT IN REG1)
(REG1) (REG2) == -> compares two registers (RESULT IN REG1)
(REG1) = (REG2)  -> Moves the value from REG2 to REG1
(REG1) push      -> pushes register onto the stack                          
(REG1) pop       -> pops the value off the stack and loads it into register 
pop              -> pops 64 bit value off the stack and loads it into RAX
push             -> pushes RAX onto the stack
```
### managing C functions and their return values

C functions are a little bit different than SOPL ones, As you know C actually copies its parameters when they are passed to a function and DOES NOT CONSUME them off the stack.
It also returns its integers, longs, shorts or chars (this also includes pointers) in the RAX register. 
Parameters to C functions are also passed backwards. As an example here is some C code:
```c
printf("%ld", 5);
```
And here is the same code in sopl:
```sopl
printf("%ld"c, 5l);
```
~~As you can see the parameters get passed in a reversed order (thats because of how your parameters in C get passed to functions normally)~~

~~As for returning values that are higher than 8 bytes, C pushes them on the stack together with everything else.~~



### strings
> Strings  -> Pre 0.1A 

> CStrings -> 0.2A

In sopl there are 2 different types of strings. There are:
```
cstrings 
strings
```
With the main difference being that cstrings don't push their length after the pointer to the string unlike normal strings.
To define a string we use "" (escaping is supported). To define a cstring just add the letter c after the string.

**Examples**

```sopl
func main() {
  printf("Hello World!"c)
  printf("Foo Bar!"c)
}
```

**[NOTE] It is generally recommended to use cstrings whenever possible if the function doesn't use the length as a parameter. In < 0.1A strings had to be pushed and then the length of them popped which is no longer sufficient and may cause your code to be inefficient**


### control flow

> If     -> Pre 0.1A

> Else   -> Pre 0.1A

**Syntax:**
```sopl
if (condition) {(Body)}
```

**Examples:**
```sopl
RBX = 4
RCX = 5
if RBX RCX == {
  "Nice!"c printf pop
}
else {
  "Not nice!"c printf pop
}
```


### constants
* Pre 0.1A

**Syntax:**
```sopl
const (name) = (your constant expersion goes here) ;
```

As in many other languages, Sopl also has a constant system. Constants are expressions that are evaluated at compile time and are therefor very useful for anything that can be evaluated at compile time. The name can be any normal name and the expression is a set of operations that the program can evaluate. The quickest way to understand how they work is to check out the examples:

```sopl
const World = "World"
const HelloWorld = "Hello" World +      // Concatenates strings and expressions can use constants
const A = 4;
const B = 5;
const C = A B + ;                       
const HelloWorldNine = HelloWorld C + ; // Concatenates the integers and strings together
```

In a constants expression can only be things that can be evaluated at compile time, such as Strings, Integers, Longs and other constants. If you try to use something like a function it would tell you that it doesn't recognize it as a constant value. 

### locals
* Pre 0.1A
  * Add  -> Pre 0.1A
  * Set  -> Pre 0.1A
  * Push -> Pre 0.1A
  * Pop  -> Pre 0.1A

**Syntax:**
```sopl
let (name): (type)
```

Whilst the stack is really flexible and neat, and unlike in most stack-oriented languages pretty customizable with the ability to access registers, having local variables is really important. Local variables in SOPL
are allocated on the callstack and therefor after the functions end they are removed off the stack. All local variables have a type and a name. Names are bound to a specific **FUNCTION SCOPE**. There aren't yet local variables for
normal scopes although they are planned for the future. Local variables are planned to be developed along side registers, with the majority of the operations matching in both syntax and usage. Although currently only a few are supported:
```
addition (+)
set (=)
pop
push
```

### interrupts
* 0.4A
  
Interrupts are essential for anything you build on platforms such as linux if you don't want to use libc. Whilst they aren't useful as much on windows machines WSL could be used to make linux syscalls into something which can run on windows.

To use interrupts you can do 'interrupt' followed by the number of the interrupt. So syscall (int 128) is 
```sopl
interrupt 128
```
