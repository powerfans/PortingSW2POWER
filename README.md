# PortingSW2POWER
The methodologies of software porting to POWER

## 1、The methodologies of software porting

### 1.1 Program execution process

Software is the set of instructions that tell the processor what to do. All software is created through the process of programming with programming languages. The programming language will be translated into machine language on the computer and executed on the processor in the form of instructions.

### 1.2 Instruction difference between POWER processor and x86 processor

The instruction set, also called ISA (instruction set architecture), is part of a computer that pertains to programming, which is more or less machine language. The instruction set provides commands to the processor, to tell it what it needs to do. The instruction sets used by processors of different architectures are difference. The POWER architecture processor uses the RISC-based (Reduced Instruction Set Computing) ISA instruction set architecture, while the x86 architecture uses the CISC-based (Complex Instruction Set Computing) instruction set architecture.

Therefore, the software under the x86 architecture must be ported before it can run on the POWER platform

### 1.3 Common issues during porting

#### 1.	Compatibility issues of make file and configure file

Makefile tell make what to do and also tells make how to compile and link a program. Building executables in different architecture require to change the variable definitions or with different compilers or compiler flags. 

For example, developer requires to modify configure.guess file to support POWER architecture in python2.7.

    ppc64le:Linux:*:*)
        GUESS=powerpc64le-unknown-linux-$LIBC 
        ;;  
    ppcle:Linux:*:*)
        GUESS=powerpcle-unknown-linux-$LIBC  
        ;;  
    vax:Linux:*:*)
        GUESS=$UNAME_MACHINE-dec-linux-$LIBC 
        ;;  
    x86_64:Linux:*:*)
    set_cc_for_build  
    GUESS=$UNAME_MACHINE-pc-linux-$LIBCABI

#### 2.	Compatibility issues of assembly instructions

Each architecture has its own assembly language. Usually a Compiler can only create code for one architecture, and there might be optional flags which enable compiler to build executables on other platforms. When these flags get enabled, the program will usually only run on processors that support these extensions.

Such as the function of acquiring CPU serial number, the implementation of the two platforms is completely different. The assembly instructions of the X86 platform need to be replaced to POWER platform accordingly.

X86：

    asm volatile
        (
         "movl $0x01, %%eax; \n\t"
         "xorl %%edx, %%edx; \n\t"
         "cpuid; \n\t"
         "movl %%edx, %0; \n\t"
         "movl %%eax, %1; \n\t"
         : "=m"(s1), "=m"(s2)
        );

PPC64LE：

    Read from system file "/proc/device-tree/vpd/"

#### 3.	Compatibility issues of SIMD instructions

Single Instruction, Multiple Data (SIMD) units refer to hardware components that perform the same operation on multiple data operands concurrently. SIMD units have been available in most architectures. But the instruction sets provided by one architecture platform is completely different to another. For example, Intel introduced the advent of the MMX, SSE (Streaming SIMD Extensions), and AVX (Advanced Vector Extensions) ISA extensions. But IBM introduced VSX instructions.

For example, the instruction of popcnt, developers need to replace the X86 SSE instruction set with the PPC64le instruction in the code.

X86：

    #define popcnt(x) asm("popcnt %[x], %[val]” : [val] "+r" (x) : : "cc")

PPC64LE:

    #define popcnt(x) __builtin_popcount(x)

#### 4.	Compatibility issues of Memory page size

Physical memory partitioned into equal sized page frame. Memory only allocated in page frame sized increments. The memory page frame size used by different processor is different. The Intel processor generates a page size of 4k, and the POWER processor generates a page size of 64k by default. Allocating wrong page size in the application may cause unpredictable problems.

For example, the page size was set to 4K in mozjs17 code. It will trigger a segment fault while mozjs17 runs on platform with 64K page size. Developers need to modify the code and set the page size to 64K.

    #if defined(SOLARIS) && (defined(__sparc) || defined(__sparcv9))
        const size_t PageShift = 13;
        const size_t ArenaShift = PageShift;
    #elif defined(__powerpc__) || defined(__aarch64__)
        const size_t PageShift = 16;


