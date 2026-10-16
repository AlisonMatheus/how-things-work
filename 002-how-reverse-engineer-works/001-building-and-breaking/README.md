# Crackme0: Building and Breaking a Naive Password Check

This is the first entry in my hands-on reverse engineering series. The goal was simple: write my own tiny "license check" program, then attack it myself, to understand both sides of the problem — how a protection is built, and how it falls apart.

This level is intentionally naive. It exists to demonstrate the most basic mistake a developer can make: **storing a secret in plaintext inside the compiled binary.**

## The program

```c
#include <stdio.h>
#include <string.h>

int main() {
    char senha[50];
    printf("Digite a senha: ");
    scanf("%49s", senha);

    if (strcmp(senha, "supersenha123") == 0)
        printf("Acesso liberado!\n");
    else
        printf("Senha incorreta.\n");

    return 0;
}
```

It asks for a password and compares it directly, with `strcmp`, against a hardcoded string. That direct comparison is the vulnerability.

## Build

Compiled on Ubuntu (WSL) for Linux:

```bash
gcc -g -O0 -o crackme0 crackme0.c
```

And cross-compiled for Windows, to analyze it with a GUI debugger:

```bash
sudo apt install -y mingw-w64
x86_64-w64-mingw32-gcc -g -O0 -o crackme0.exe crackme0.c
```

## Attack 1: static analysis (no debugger needed)

Before even opening a debugger, a simple strings scan reveals the password:

```bash
strings crackme0 | grep -i senha
```

The literal string `supersenha123` sits right there in the binary. This alone is often enough to break this kind of check — no execution required.

## Attack 2: dynamic analysis with x64dbg

To see the logic actually execute, I opened `crackme0.exe` in **x64dbg**.

1. Ran past the initial system breakpoint (`ntdll.dll` loader code) with **F9** until execution reached the `crackme0` module itself.
2. Located the hardcoded strings via right-click on the disassembly → *Search for* → *Current Module* → *String references*. Both `"Acesso liberado!"` and `"Senha incorreta."` showed up, each referenced from a specific address.
3. Jumped to the reference for `"Acesso liberado!"` and scrolled up a few lines to find the decision point, right after the call to `strcmp`:

```asm
test eax, eax
jne  crackme0.7FF7EBF514DA   ; jump taken -> "Senha incorreta."
lea  rax, [...62024]         ; "Acesso liberado!"
mov  rcx, rax
call puts
jmp  crackme0.7FF7EBF514E9
```

`strcmp` returns 0 when the strings match, so `test eax, eax` followed by `jne` reads as: *"if the result is non-zero (strings differ), jump to the failure path; otherwise, fall through to success."*

## Breaking it two different ways

**A. Register manipulation (no code changes)**

- Set a breakpoint on `test eax, eax`.
- Ran the program with a deliberately wrong password.
- When execution paused at the breakpoint, modified `RAX` to `0` directly in the Registers panel.
- Resumed execution: the `jne` was no longer taken, and the program printed `"Acesso liberado!"` despite the wrong password.

This shows that an attacker doesn't need to understand or reverse the comparison logic at all — just lie about its result at the right moment.

**B. Binary patching (permanent bypass)**

- Edited the `jne` instruction directly in the disassembly (Assemble, `Space` key), changing it to `nop` so the jump to the failure path never happens.
- Used *Patch file...* (`Ctrl+P`) to write the modification back to a new `.exe` on disk.
- The patched binary now always reports success, regardless of the password entered — permanently, with no debugger attached.

## Key takeaway

Every check that runs entirely inside the attacker's own machine can be observed and altered, because the attacker controls the CPU, the memory, and the instruction flow. Storing the secret in cleartext made this trivial; even hiding or hashing it (later levels in this series) doesn't fix the deeper issue — the final decision (`if` / conditional jump) still lives on a machine the attacker fully controls. Real protection has to move that decision somewhere the attacker doesn't control, such as a remote server.

## Tools used

- GCC / MinGW-w64 (compilation)
- `strings`, `objdump` (static analysis, Linux)
- [x64dbg](https://x64dbg.com/) (dynamic analysis and patching, Windows)

## Next steps in this series

- `crackme1`: password hidden from `strings` (byte-shifted array instead of a literal)
- `crackme2`: the same check duplicated in multiple places to resist a single patch
- `crackme3`: self-debugger detection via `ptrace`
- `crackme4`: hash-based comparison instead of a reversible secret
