# Crackme1 — Building and Breaking a Windows Login Program

A hands-on **reverse engineering** project: build a Win32 program with a hash-protected login screen, then break it using static analysis, dynamic analysis, a runtime bypass and a binary patch.

Part of the *Building and Breaking* series, a practical study of software reverse engineering (college course), evolving from a simple crackme (`crackme0`) to this one, with stronger defenses.

---

## 1. Goal

- Build a program with a GUI (Windows-style login window), with `User` and `Password` fields.
- Protect the password with a hash instead of plain text.
- Use a debugger (**x64dbg**) to analyze the binary, understand the validation logic, and bypass the protection in two different ways: a runtime bypass (changing a register in memory) and a binary patch (changing the instruction inside the `.exe` file).

---

## 2. Tools used

| Tool | Purpose |
|---|---|
| **C (Win32 API)** | Language and API used to build the program |
| **MinGW-w64** (`x86_64-w64-mingw32-gcc`) | Cross-platform compiler, run from WSL (Ubuntu), producing a Windows `.exe` |
| **WSL (Ubuntu)** | Development and build environment |
| **VS Code** | Code editor |
| **x64dbg** | Windows debugger, used in the breaking phase |

---

## 3. Building — constructing the program

### 3.1 Functionality

`crackme1.c` creates a login window with:
- **User** field (text)
- **Password** field (hidden text, `ES_PASSWORD`)
- **Login** button

If the credentials are correct, the login window is destroyed and replaced with an "Access granted!" screen. If they are wrong, a `MessageBox` shows "Wrong password", with a growing delay on each attempt and a limit of 3 attempts before the program exits.

### 3.2 Protecting the password: hashing

Instead of storing the password as plain text in the binary (vulnerable to a simple `strings crackme1.exe`), the program computes an **FNV-1a (64-bit) hash** of the string `"cm1$" + username + ":" + password` at runtime, and compares it against a fixed value (`EXPECTED`) embedded in the code.

```c
static uint64_t hash_cred(const char *user, const char *pass)
{
    uint64_t h = 14695981039346656037ULL;   /* FNV offset basis */
    const char *parts[4] = { "cm1$", user, ":", pass };

    for (int i = 0; i < 4; i++)
        for (const unsigned char *p = (const unsigned char *)parts[i]; *p; p++) {
            h ^= *p;
            h *= 1099511628211ULL;          /* FNV prime */
        }
    return h;
}
```

Other security measures applied:
- `SecureZeroMemory` wipes the password from memory after use.
- The password field is cleared on screen after each failed attempt.
- A growing delay (`Sleep(500 * tries)`) makes manual brute-forcing harder.
- A 3-attempt limit shuts the program down.

### 3.3 Compilation

```bash
x86_64-w64-mingw32-gcc crackme1.c -o crackme1.exe -mwindows -s
```

- `-mwindows`: produces a GUI executable (no console).
- `-s`: strips debug symbols from the binary, making analysis harder.

---

## 4. Breaking — reverse engineering

### 4.1 Static analysis — finding the check

1. Opened `crackme1.exe` in x64dbg.
2. Under **Search for → Current Module → String references**, located the string `"Wrong password"`.
3. From there, navigated the disassembly to the decision block.

### 4.2 The validation logic in assembly

```asm
call crackme1.7FF763A61480     ; hash_cred(user, pass) -> result in RAX
mov  rdx, 1959D720A77E0742     ; EXPECTED (hash of the correct password)
cmp  rax, rdx                  ; compare the typed hash with the expected one
jne  crackme1.7FF763A61794     ; if different, jump to "Wrong password"
...
call crackme1.7FF763A61525     ; if equal, fall through to show_system()
```

This sequence maps directly to the `if (hash_cred(user, pass) == EXPECTED)` from the C source. Notable detail: the expected value (`1959D720A77E0742`) appears **in plain text** in the assembly — this makes a direct comparison attack easier.

### 4.3 Dynamic analysis — watching the comparison live

1. Set a breakpoint (`F2`) on the `cmp rax, rdx` instruction.
2. Ran (`F9`) until the window opened.
3. Logged in with `admin` / `abc123` (wrong password, on purpose).
4. The debugger paused at the breakpoint; inspected the registers:
   - `RDX = 1959D720A77E0742` → expected hash (correct one)
   - `RAX = B8FFE2309D264F77` → hash computed from the typed (wrong) credentials

This confirmed, live, exactly where and how the decision is made.

### 4.4 Runtime bypass

With the program still paused at the breakpoint, I edited the **RAX** register directly in memory, replacing it with the value of `RDX` (`1959D720A77E0742`). Resuming execution (`F9`), the `cmp` found equal values, and the program granted access even with the wrong password (`abc123`).

This technique only works while the debugger is attached — it does not modify the `.exe` file.

### 4.5 Binary patch — permanent bypass

To make the bypass permanent (no debugger required), the conditional jump was changed directly in the binary:

| Before | After |
|---|---|
| `jne crackme1.7FF763A61794` (bytes `75 11`) | `je crackme1.7FF763A61794` (bytes `74 11`) |

The logic was inverted: the jump to "Wrong password" now only happens when the hashes **are equal** (i.e. when the password is correct). Since the hashes are normally different, the program now always falls through to the success path, regardless of the password typed.

Process in x64dbg:
1. Selected the `jne` instruction → `Space` (Assemble) → `je crackme1.7FF763A61794`.
2. Right-click menu → **Patches...** → `Patch File`.
3. Saved the new binary as `crackme1_patched.exe`, keeping the original intact.

**Result:** `crackme1_patched.exe`, run outside the debugger, accepts any password and shows "Access granted!".

---

## 5. Technical summary

| Stage | Technique | Tool |
|---|---|---|
| Build | Win32 GUI + hashing (salted FNV-1a) | C, MinGW |
| Reconnaissance | String references | x64dbg |
| Analysis | Breakpoint + register inspection | x64dbg |
| Temporary bypass | Editing a register at runtime | x64dbg |
| Permanent bypass | Instruction patch (`jne` → `je`) in the binary | x64dbg |

---

## 6. Lessons learned

- Comparing hashes isn't enough if the decision logic (`cmp` + conditional jump) can be located and altered; the weakness isn't the hash itself, but exposing the control flow.
- Hash values embedded as plain text in the binary make direct-comparison attacks easier.
- A single verification point (`if`) is an easy target — this highlights the importance of redundant checks and obfuscation in real protections.
- `strip` (`-s`) removes symbols, but doesn't prevent string analysis or control-flow analysis.

---

## 7. Next steps (crackme2)

- Stronger hash (e.g. SHA-256) instead of FNV-1a.
- Multiple password-check points, not just one `if`.
- Anti-debugger check (`IsDebuggerPresent` or similar).
- Obfuscation of the expected value (avoid leaving it in plain text in the binary).

---

## Project files

- `crackme1.c` — source code of the program (hash-protected login).
- `gen_hash.c` — utility to generate the expected hash from a username/password.
- `crackme1.exe` — original compiled binary.
- `crackme1_patched.exe` — binary after the patch (accepts any password).
