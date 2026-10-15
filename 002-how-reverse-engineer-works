# Software Reverse Engineering: First Steps in Practice

This repository documents my studies in **Software Reverse Engineering**. My university course is very theory-heavy, so I set up my own environment to practice: compiling a simple C program, reading the generated assembly, and following its execution instruction by instruction with a debugger.

## Goals

- Understand how a debugger works.
- Learn to read x86-64 assembly (Intel syntax).
- Connect C code to machine instructions and to what happens in registers and on the stack.

## Environment

- Windows with **WSL** (Windows Subsystem for Linux)
- **Ubuntu** inside WSL
- Tools: `gcc`, `gdb`, `objdump` (packages `build-essential`, `gdb` and `binutils`)

---

## 1. Installing WSL and Ubuntu

In PowerShell (as administrator):

```powershell
wsl --update
wsl --install -d Ubuntu
```

After restarting the computer, I opened the **Ubuntu** app from the Start menu and created a username and password.

> **Problem I ran into:** the app simply called "WSL" in the Start menu opened and closed immediately. It is only the base program, and it exits when no Linux distribution is installed. The fix was to install Ubuntu explicitly with the command above and open the **Ubuntu** app instead.
>
> To see what is installed: `wsl --list --verbose`. If you get a virtualization error, enable virtualization in the BIOS and check that "Virtual Machine Platform" and "Windows Subsystem for Linux" are turned on in `optionalfeatures`.

## 2. Installing the tools

Inside Ubuntu:

```bash
apt update
apt install -y build-essential gdb binutils nano
```

> **Problem I ran into:** when compiling, I got `fatal error: stdio.h: No such file or directory`. `gcc` was installed, but the C standard library headers were missing. The `build-essential` package fixed it.

## 3. The test program

```bash
mkdir ~/reversa
cd ~/reversa
nano teste.c
```

```c
#include <stdio.h>

int soma(int a, int b) {
    return a + b;
}

int main() {
    int r = soma(3, 4);
    if (r > 5)
        printf("maior\n");
    else
        printf("menor\n");
    return 0;
}
```

(`soma` means "sum", `maior` means "greater" and `menor` means "smaller".)

## 4. Compiling and running

```bash
gcc -g -O0 -o teste teste.c
./teste
```

Expected output: `maior`

- `-O0` turns off optimizations so the assembly stays faithful to the source code.
- `-g` includes debug information.

## 5. Reading the assembly

```bash
objdump -d -M intel teste | grep -A 12 "<soma>:"
```

The `soma` function produces something like:

```asm
push   rbp
mov    rbp,rsp
mov    DWORD PTR [rbp-4],edi
mov    DWORD PTR [rbp-8],esi
mov    edx,DWORD PTR [rbp-4]
mov    eax,DWORD PTR [rbp-8]
add    eax,edx
pop    rbp
ret
```

| Instruction | What it does |
|---|---|
| `push rbp` / `mov rbp,rsp` | Sets up the function's stack frame |
| `mov [rbp-4],edi` / `mov [rbp-8],esi` | Stores the arguments `a` and `b`, which arrive in `edi` and `esi`, on the stack |
| `mov edx,...` / `mov eax,...` | Loads the values back into registers |
| `add eax,edx` | Performs the addition; the result ends up in `eax` |
| `pop rbp` / `ret` | Tears down the stack frame and returns; the return value is in `eax` |

## 6. Debugging with gdb

```bash
gdb ./teste
```

```text
set disassembly-flavor intel
break soma
run
info registers rdi rsi
disassemble
stepi
info registers rip
info registers rax
x/8xg $rsp
finish
continue
quit
```

What to observe:

- When stopped in `soma`, `rdi` is 3 and `rsi` is 4: these are the arguments of the call `soma(3, 4)`.
- With each `stepi`, one instruction executes and the registers change.
- After the `add`, `rax` is 7.

## How a debugger works (summary)

1. **Breakpoint:** the debugger replaces the first byte of the instruction with `0xCC` (`int3`). When the CPU executes it, the operating system pauses the program and notifies the debugger, which restores the original byte.
2. **Single-step:** the CPU has a flag that makes it stop after every instruction. This is what `stepi` uses.
3. **Reading and writing memory and registers** of the stopped process, on Linux through the `ptrace` system call.

## Next steps

- [ ] Analyze the `main` function and identify the `cmp` and the conditional jump that correspond to the `if`
- [ ] Repeat the exercise with `for` loops, arrays and `struct`s
- [ ] Use [Compiler Explorer](https://godbolt.org) to compare C and assembly side by side
- [ ] Open a binary in **Ghidra** and compare the pseudo-C with the original code
- [ ] Solve beginner-level crackmes (crackmes.one, picoCTF, pwn.college)

## References

- [Compiler Explorer](https://godbolt.org)
- [Ghidra](https://ghidra-sre.org)
- [picoCTF](https://picoctf.org)
- [pwn.college](https://pwn.college)
