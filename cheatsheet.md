## Skeleton of a Typical Assembly Program (x86\_64, Linux, AT&T Syntax)

Here’s the minimal structure of an assembly program that runs on Linux using the `syscall` interface. This is typically written using the GNU assembler (GAS) syntax.

```asm
.section .data
# Define initialized global data here

.section .bss
# Reserve space for uninitialized data (optional)

.section .text
.globl _start         # Declare _start as the entry point (visible to linker)
_start:
    # Your code begins here
    mov $60, %rax      # syscall number for exit
    xor %rdi, %rdi     # exit status 0
    syscall            # make the system call
```

### Explanation of Each Part

#### `.text`

- This section contains your executable code.
- All instructions go here.

#### `.data`

- This section is for **initialized global variables**.
- Data declared here is copied into the final binary.

#### `.bss`

- This section holds **uninitialized global variables**.
- The size is reserved in the binary, but the content is zero-initialized at runtime.

#### `.globl _start`

- Declares `_start` as a global label so the linker can recognize it as the program entry point.
- Required when not linking with the C runtime (e.g., no `main` function).

#### `_start:`

- This is the entry point of your program.
- When your program is executed, the instruction pointer begins here.

---

This is a minimal, freestanding setup — perfect for learning how low-level Linux programs work without relying on a high-level runtime.

---

## Linux System Call Convention (x86\_64)

When invoking a system call on Linux using the `syscall` instruction, the following registers are used to pass information between your program and the kernel.

### Register Usage

| Register | Purpose                      |
| -------- | ---------------------------- |
| `%rax`   | System call number           |
| `%rdi`   | 1st argument                 |
| `%rsi`   | 2nd argument                 |
| `%rdx`   | 3rd argument                 |
| `%r10`   | 4th argument                 |
| `%r8`    | 5th argument                 |
| `%r9`    | 6th argument                 |
| `%rax`   | Return value (after syscall) |

### Notes

- You **must** place the syscall number in `%rax` before executing `syscall`.
- Return value is also in `%rax`. If the syscall returns an error, it will be a negative value (e.g., `-1` for `EPERM`, `-2` for `ENOENT`, etc.).
- The 4th argument is passed via `%r10` (not `%rcx`) because `syscall` clobbers `%rcx` and `%r11`.

### Example: `write` syscall

```asm
mov $1, %rax        # syscall number for write
mov $1, %rdi        # file descriptor (stdout)
mov $msg, %rsi      # pointer to message
mov $len, %rdx      # length of message
syscall
```

This will write `len` bytes from `msg` to standard output.

---

## x86\_64 General-Purpose Registers and Size Variants

Each general-purpose register in x86\_64 has variants for different operand sizes:

```
+------------+----------+----------+--------+
| 64-bit     | 32-bit   | 16-bit   | 8-bit  |
+------------+----------+----------+--------+
| %rax       | %eax     | %ax      | %al    |
| %rbx       | %ebx     | %bx      | %bl    |
| %rcx       | %ecx     | %cx      | %cl    |
| %rdx       | %edx     | %dx      | %dl    |
| %rsi       | %esi     | %si      | %sil   |
| %rdi       | %edi     | %di      | %dil   |
| %rbp       | %ebp     | %bp      | %bpl   |
| %rsp       | %esp     | %sp      | %spl   |
| %r8        | %r8d     | %r8w     | %r8b   |
| %r9        | %r9d     | %r9w     | %r9b   |
| %r10       | %r10d    | %r10w    | %r10b  |
| %r11       | %r11d    | %r11w    | %r11b  |
| %r12       | %r12d    | %r12w    | %r12b  |
| %r13       | %r13d    | %r13w    | %r13b  |
| %r14       | %r14d    | %r14w    | %r14b  |
| %r15       | %r15d    | %r15w    | %r15b  |
+------------+----------+----------+--------+
```

### Size and AT&T Suffix Mapping

| Operand Size | Name        | AT&T Suffix |
| ------------ | ----------- | ----------- |
| 8-bit        | Byte        | `b`         |
| 16-bit       | Word        | `w`         |
| 32-bit       | Double word | `l` (long)  |
| 64-bit       | Quad word   | `q`         |

Examples:

- `movb` = move 8-bit
- `movw` = move 16-bit
- `movl` = move 32-bit
- `movq` = move 64-bit

## GDB Cheat Sheet for Debugging Assembly Programs

GDB is a powerful tool for debugging assembly programs. Here's a reference for common tasks and commands.

### Starting GDB

```bash
gdb ./program_name       # Start GDB with the binary
gdb -tui ./program_name  # Start in TUI (text user interface) mode
```

To toggle TUI layout during a session:

```
Ctrl + x, then a         # Switch to TUI assembly + source layout
layout asm               # Show assembly-only view
layout regs              # Show registers view
```

---

### Breakpoints & Execution Control

| Command           | Description                 |
| ----------------- | --------------------------- |
| `break _start`    | Set breakpoint at label     |
| `break *0x401000` | Set breakpoint at address   |
| `run`             | Start the program           |
| `continue` / `c`  | Continue execution          |
| `stepi` / `si`    | Step one instruction (into) |
| `nexti` / `ni`    | Step over next instruction  |
| `finish`          | Run until function returns  |
| `quit` / `q`      | Exit GDB                    |

---

### Viewing Registers

| Command          | Description                        |
| ---------------- | ---------------------------------- |
| `info registers` | Show all general-purpose registers |
| `p/x $rax`       | Print value in hexadecimal         |
| `p $rip`         | Show current instruction pointer   |

---

### Viewing Memory

| Command      | Description                            |
| ------------ | -------------------------------------- |
| `x/Nb addr`  | Examine N bytes from addr              |
| `x/4x $rsp`  | Show 4 words in hex from stack pointer |
| `x/s addr`   | Display string at address              |
| `x/i $rip`   | Disassemble current instruction        |
| `x/16xb msg` | View 16 bytes from label in hex        |

---

### Disassembly

| Command                        | Description                  |
| ------------------------------ | ---------------------------- |
| `disas`                        | Disassemble current function |
| `x/10i $rip`                   | Show next 10 instructions    |
| `set disassembly-flavor intel` | Switch to Intel syntax       |

---

### Miscellaneous

| Command              | Description                      |
| -------------------- | -------------------------------- |
| `info files`         | Shows segments, entry point, etc |
| `info address label` | Get address of a label/symbol    |
| `info breakpoints`   | List breakpoints                 |
| `delete`             | Delete all breakpoints           |
| `set pagination off` | Disable --More-- prompts         |

---

These commands should cover most basic use cases when stepping through and inspecting an assembly-level program using GDB.

## x86\_64 Memory Addressing Modes (with Examples)

In x86\_64 assembly, memory operands can use various combinations of base registers, index registers, scaling, and displacement values.

The general form is:

```
(offset)(base, index, scale)
```

Where:

- `offset`: a constant displacement (can be omitted)
- `base`: a register holding the base address
- `index`: a register multiplied by the scale factor
- `scale`: must be 1, 2, 4, or 8

---

### Common Addressing Modes

#### 1. **Direct Addressing**

```asm
mov (%rax), %rbx      # load value at address in %rax into %rbx
```

#### 2. **Displacement (Offset) Addressing**

```asm
mov 8(%rbp), %rax     # load from address [%rbp + 8]
```

#### 3. **Base + Index**

```asm
mov (%rbx,%rcx), %rax # load from [%rbx + %rcx]
```

#### 4. **Base + Index × Scale**

```asm
mov (%rbx,%rcx,4), %rax  # load from [%rbx + %rcx * 4]
```

#### 5. **Displacement + Base + Index × Scale**

```asm
mov 16(%rbx,%rcx,4), %rax  # load from [%rbx + %rcx * 4 + 16]
```

---

### ### Diagram of Addressing a Struct Field in Memory

Let’s assume a struct is located at address `0x600000` and has the following layout:

```c
struct Point {
    int x;     // offset 0
    int y;     // offset 4
    int z;     // offset 8
};
```

Now, assume `%rdi` holds the base address of a `Point` object in memory.

Here’s how the memory looks:

```
Memory Layout (starting at 0x600000):

  Address      Value     Field
  ---------    -------   -----------
  0x600000     0000002A  x (int, 4 bytes)
  0x600004     0000003C  y (int, 4 bytes)
  0x600008     00000055  z (int, 4 bytes)
```

### Accessing `y` field from assembly

```asm
mov 4(%rdi), %eax    # Load the 'y' field (offset 4) into %eax
```

### Visualization

```
    %rdi → ┌────────────┐  ← Address: 0x600000
           │    x = 42  │  ← offset 0
           ├────────────┤
           │    y = 60  │  ← offset 4
           ├────────────┤
           │    z = 85  │  ← offset 8
           └────────────┘

Instruction:
    mov 4(%rdi), %eax   → loads value 60 from y
```

This shows how displacement-based addressing can be used to access fields in a struct using base register + offset.

### Useful Tips

- If `base` is omitted, you can still use displacement + index × scale.
- You can use `lea` (load effective address) to compute addresses without performing memory access.
  ```asm
  lea 8(%rbx,%rcx,2), %rax  # %rax = %rbx + %rcx * 2 + 8
  ```

---

These modes allow compact and flexible memory access — especially for arrays and structs.

## AT&T Memory Addressing Modes Reference

In AT&T syntax (used by GNU Assembler), memory addressing operands follow this general form:

```
disp(base, index, scale)
```

Where:

- `disp`: Constant displacement (optional)
- `base`: Base register holding a memory address (optional)
- `index`: Index register (optional)
- `scale`: Multiplier for index register (1, 2, 4, or 8)

### Addressing Modes Table

| Syntax            | Description                               | Example                     |
| ----------------- | ----------------------------------------- | --------------------------- |
| `(%rax)`          | Indirect via base register                | `mov (%rax), %rbx`          |
| `8(%rbp)`         | Base + displacement                       | `mov 8(%rbp), %rax`         |
| `(%rbx,%rcx,1)`   | Base + index                              | `mov (%rbx,%rcx), %rax`     |
| `(%rbx,%rcx,4)`   | Base + (index × scale)                    | `mov (%rbx,%rcx,4), %rax`   |
| `16(%rbx,%rcx,4)` | disp + base + (index × scale)             | `mov 16(%rbx,%rcx,4), %rax` |
| `(,%rcx,4)`       | Index × scale (no base)                   | `mov (,%rcx,4), %rax`       |
| `0x601000`        | Absolute address (needs relocation setup) | `mov 0x601000, %eax`        |

### AT&T Syntax Rules Recap

- Constants come before registers: `mov $5, %eax`
- Memory operands are always wrapped in parentheses
- Scale must be 1, 2, 4, or 8
- Square brackets (`[]`) are not used — this is Intel syntax

This table covers all valid memory addressing forms in AT&T syntax used in x86\_64 assembly.



## Arithmetic and Bitwise Instructions Reference

This cheat sheet covers common arithmetic and bitwise operations in x86_64 assembly (AT&T syntax) and the flags they affect in the RFLAGS register.

### Arithmetic Instructions

| Instruction     | Description                                                      | Flags Affected                               |
|----------------|------------------------------------------------------------------|----------------------------------------------|
| `add src, dst` | dst = dst + src                                                  | CF, PF, AF, ZF, SF, OF                       |
| `sub src, dst` | dst = dst - src                                                  | CF, PF, AF, ZF, SF, OF                       |
| `inc dst`      | dst = dst + 1                                                    | PF, AF, ZF, SF, OF (not CF)                  |
| `dec dst`      | dst = dst - 1                                                    | PF, AF, ZF, SF, OF (not CF)                  |
| `imul src, dst`| Signed multiply                                                  | CF, OF (if result overflows destination)     |
| `idiv src`     | Signed divide: RDX:RAX ÷ src → Quotient in RAX, Remainder in RDX | #DE on divide-by-zero or overflow           |
| `mul src`      | Unsigned multiply: RAX × src → Result in RDX:RAX                 | CF, OF (if upper 64 bits in RDX are nonzero) |
| `div src`      | Unsigned divide: RDX:RAX ÷ src → RAX = quotient, RDX = remainder | #DE on divide-by-zero or overflow           |


### Bitwise Instructions

| Instruction     | Description               | Flags Affected                     |
|----------------|---------------------------|------------------------------------|
| `and src, dst` | dst = dst & src           | PF, ZF, SF (clears CF and OF)      |
| `or src, dst`  | dst = dst | src           | PF, ZF, SF (clears CF and OF)      |
| `xor src, dst` | dst = dst ^ src           | PF, ZF, SF (clears CF and OF)      |
| `not dst`      | dst = ~dst                | None                               |
| `shl n, dst`   | dst = dst << n            | CF (last bit shifted out), ZF, SF  |
| `shr n, dst`   | dst = dst >> n (logical)  | CF, ZF, SF                         |
| `sar n, dst`   | dst = dst >> n (arithmetic)| CF, ZF, SF                         |


### Example:
```asm
mov $0b1101, %al
and $0b0101, %al      # Result: %al = 0b0101, ZF = 0
```

### Notes
- Flags commonly used for conditionals: **ZF (Zero Flag)**, **SF (Sign Flag)**, **CF (Carry Flag)**, **OF (Overflow Flag)**.
- `test` is like `and`, but does not modify the destination.

```asm
test %rax, %rax       # Set ZF if %rax is zero
```

Use this as a reference for understanding what each instruction does and how it influences branching or conditional logic. 

---

