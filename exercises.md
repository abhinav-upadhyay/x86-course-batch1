## Intro
**Exercise 1: Fix the Crashing Program**

The following assembly program crashes when executed. Your task is to modify it to terminate gracefully by invoking the appropriate system call to exit the program.

```asm
.data

.text
.globl _start
_start:
    movq $10, %rdi
    movq $32, %rsi
    addq %rsi, %rdi

    # Your code goes here
```

### Instructions:

- The system call number for `exit` is `60`.
- The `exit` syscall takes one argument: the exit code.
- The exit code is already in the `%rdi` register, so we don't need to update it.
- The syscall number should be placed in the `%rax` register.
- Use the `syscall` instruction to invoke the system call after setting up the arguments.

Make sure your program terminates properly by calling `exit` after it finishes the computation.

### To assemble, link, and run the program:

```bash
as -o prog.o prog.s      # Assemble
ld -o prog prog.o        # Link
./prog                   # Run
```

If everything is correct, the program should now exit cleanly instead of crashing.

---

**Exercise 2: Call ****************************************************************************************************************************************************************`getpid`**************************************************************************************************************************************************************** and Exit with the Return Value**

Extend your knowledge of syscalls with this exercise. Your task is to invoke the `getpid` system call and then exit the program using the return value of `getpid` as the exit code.

### Instructions:

- Find out the system call number for `getpid`. You can search online for "Linux x86\_64 syscall table".
- `getpid` takes no arguments.
- The return value of the syscall will be in `%rax`.
- Store this return value in `%rdi`, and then invoke the `exit` syscall with it as the argument.

This exercise will help you get comfortable with calling a syscall, retrieving its return value, and using it in another syscall.

### Assemble, link, and run as before:

```bash
as -o prog.o prog.s
ld -o prog prog.o
./prog
```

After running, you can check the program's exit code using:

```bash
echo $?
```

It should match the PID of the process.

---
## Working with Registers

**Exercise 3: Explore x86 Register Variants and Sizes**

In this series of small tasks, you will explore the different register sizes in x86-64 and learn how values behave when moved between them. We'll also simulate a system-like scenario to make the learning more concrete.

### Part A: Write a 16-bit Value

- Write a 16-bit immediate value (e.g., `0x1234`) into a 16-bit register such as `%bx`.
- Inspect the value using GDB to confirm that the upper bits of the full 64-bit register (`%rbx`) remain unaffected.

### Part B: Move and Extend Values

- Move the 16-bit value into a 64-bit register using:
  - Zero extension (e.g., `movzwq`)
  - Sign extension (e.g., `movswq`)
- Print and compare the results in GDB to understand how zero and sign extension differ.

### Part C: Trigger a Carry or Overflow

- Perform an arithmetic operation (e.g., add two large values in 8-bit registers) that causes a carry or overflow.
- Use GDB to inspect the `%rflags` register and observe the Carry Flag (CF) and Overflow Flag (OF).

### Part D: Simulate Status Flag Extraction

- Suppose a simulated device register returns a 64-bit status word: set a register to `0x0000000100000001`
- Use masking and shifts to extract specific status bits from it (e.g., lowest byte and 33rd bit).
- Print values in gdb to check your bit manipulations.

### Part E: Clear Lower 8 Bits

- Clear the lower 8 bits of a register without affecting the upper bits.
- Hint: Use an AND operation with a mask that zeroes just the lower byte.

### Part F: 32-bit Writes Zero the Upper 32 Bits

- Set a 64-bit register to `-1` (all bits set).
- Write a 32-bit value into `%eax` and inspect `%rax` in GDB. Observe how the upper 32 bits are cleared automatically.

### Part G: Modify `%ah`, Observe `%al` and `%ax`

- Load a known 16-bit value into `%ax` (e.g., `0xABCD`).
- Modify just `%ah` and see how it affects `%ax` and `%al`.

### Part H: Overflow Behavior with Signed vs. Unsigned

- Add two signed 8-bit values that overflow and inspect the Overflow and Sign flags in `%rflags`.
- Then try similar unsigned arithmetic to observe when the Carry flag is affected.

### Part I: 8-bit Writes Do Not Affect Higher Bits

- Write a value into `%al`, then inspect `%ax`, `%eax`, and `%rax`. See how writing 8 bits does not alter the rest of the register.

### Part J: Inspect Zero and Sign Flags

- Perform a subtraction that results in zero (e.g., `sub %rax, %rax`) and inspect the Zero Flag (ZF) in the `%rflags` register.
- Perform a subtraction or move a negative number into a register (e.g., `mov $-1, %al`) and check how the Sign Flag (SF) is set.
- Try operations that clear or set the flags and confirm changes using GDB (`info registers` or `p/x $eflags`).

---

Each of these tasks will help you get comfortable with:

- Register names and their aliases (e.g., `%rax`, `%eax`, `%ax`, `%al`, `%ah`)
- Word sizes (8, 16, 32, and 64 bits)
- Size suffixes in AT&T syntax: `b` (byte), `w` (word), `l` (long), `q` (quad)
- Bit manipulation and how arithmetic affects flags — as used in low-level systems

Use GDB extensively to inspect register states after each operation.

```bash
as -g -o reg.o reg.s
ld -o reg reg.o
gdb ./reg
```

## Accessing Memory

**Exercise 4: Basic Addressing - Direct and Register-Based Access**

Let’s start with a simple exercise to understand how to access memory using both direct addressing and register-based addressing.

### Objective:

Access a static value stored in memory using two different techniques:

1. Directly with the label.
2. Indirectly via a register holding the address.

---

### Part A: Direct Addressing

- Define a label in the `.data` section for a 64-bit value (e.g., `.quad 12345`).
- In your program, access that value directly using the label and move it into the rdi register.
- Then exit the program with the exit status equal to that value

This exercise helps you understand how labels point to memory and how data can be accessed using label names alone.

### Part B: Indirect Addressing via Register

- Use the `leaq` instruction to move the address of your label into a register (e.g., `%rsi`).
- Then dereference the register to load the value into another register (e.g., `%rax`).
- Exit the program after loading the value.

This will help you understand how indirect memory access works using register-based addressing.

Use GDB to verify the loaded value in `%rax`.

---

**Exercise 5: Addressing Modes with User Input**

Let's combine addressing modes with user input to build a more engaging exercise.

### Objective:

Write a program that:

1. Reads one character from standard input (representing an index).
2. Converts the ASCII digit (e.g., `'3'`) into the corresponding integer (i.e., `3`).
3. Uses that value as an index into a static array of 10 values.
4. Exits the program with the value at that index as the exit code.

---

### Starter Code Skeleton:

```asm
.data
array:      .quad 10, 20, 30, 40, 50, 60, 70, 80, 90, 100
input_buf:  .space 1

.text
.globl _start
_start:
    # Read 1 byte from stdin into input_buf
    movq $0, %rax         # syscall: read
    movq $0, %rdi         # stdin
    movq $input_buf, %rsi # buffer
    movq $1, %rdx         # size = 1
    syscall

    # Your code goes here

    movq $60, %rax         # syscall: exit
    xor %rdi, %rdi         # placeholder exit code
    syscall
```

---

### Instructions:

- The input will be a digit character between `'0'` and `'9'`.
- Convert the ASCII character to its integer value by subtracting the ASCII code of `'0'` (i.e., 48).
- Use the resulting index to fetch a value from `array` using scaled indexing (`array(, %reg, 8)`).
- Move the value into `%rdi` as the exit code and terminate the program.

### Tips:

- Use `movzbq` to safely load the byte from the buffer.
- You may use a temporary register like `%rcx` or `%rsi` to hold the index.
- Check your result with `echo $?` after running.

### To test:

```bash
as -o addr.o addr.s
ld -o addr addr.o
./addr
```

Then type a digit (like `4`) and press Enter. The program should exit with code `50` (i.e., the 5th value in `array`).

Use GDB to inspect memory and registers:

```bash
gdb ./addr
```



Let’s make memory layout more fun! In this exercise, you’ll work with a struct-like layout in the `.data` section. You’ll access and extract values from it using offset-based addressing.

### Scenario:

You’ve been given a "record" representing a user profile:

```asm
requrires .data
user1:
    .quad 42           # user ID
    .quad 18           # age
    .quad 9876543210   # phone number
```

Your job is to:

1. Load each field from the struct into a different register.
2. Exit the program using the user’s age as the exit code.

### Instructions:

- `user1` is the start of the struct.
- The `age` is stored 8 bytes after the `user ID`, and `phone number` is another 8 bytes after that.
- Use offsets like `user1 + 8` to get to the `age`, or use register + displacement style.

Bonus: Try copying the whole struct into another region in memory (e.g., a `.bss` buffer) using three separate moves.

```bash
as -g -o struct.o struct.s
ld -o struct struct.o
gdb ./struct
```

Use GDB to inspect the struct fields and verify correct values were loaded. 



**Exercise 6: Offset-Based Addressing with a Monetary Value Struct**

Time to get financially accurate—literally. In this exercise, you’ll work with a `struct` that represents a monetary value with two integer fields: the whole part (e.g., rupees or dollars) and the fractional part (e.g., paise or cents).

### Scenario:

You are handed a monetary value stored in memory:

```asm
.data
amount:
    .quad 42      # whole part (e.g., 42 dollars)
    .quad 37      # fraction part (e.g., 37 cents)
```

Your task is to:

1. Load both the whole and fractional parts from memory using offset-based addressing.
2. Compute the total value in cents (e.g., `42*100 + 37 = 4237`).
3. Exit the program using this computed value as the exit code.

### Instructions:

- Use `leaq` to load the address of `amount` into a register (e.g., `%rsi`).
- Use `movq 0(%rsi), %rax` to read the whole part.
- Use `movq 8(%rsi), %rbx` to read the fractional part.
- Multiply the whole part by 100 (you can use `imulq`) and then add the fraction.
- Move the result into `%rdi`, set `%rax` to 60, and call `syscall` to exit with that value.

### To Test:

```bash
as -o money.o money.s
ld -o money money.o
./money

echo $?
```

You should see 4237 if the values are `42` and `37`.

This exercise helps you understand struct layout, offset navigation, and simple integer arithmetic in a systems programming context. to interpret fixed-format records using struct offsets and reinforces alignment, byte layout, and field access in low-level code.

## Control-flow 

**Exercise 7: Control Flow with ************************************************************************************`fork`************************************************************************************ and Process Identification**

In this exercise, you’ll explore process control by invoking the `fork` system call.

### Objective:

1. Use the `fork` syscall to create a child process.
2. Print a message indicating whether the process is the parent or the child.
3. Call `getpid` to obtain the PID.
4. Exit with the PID as the status code.

### Instructions:

- Look up the syscall number for `fork` on the x86\_64 Linux syscall table.
- `fork` returns `0` in the child process, and the PID of the child in the parent process.
- Use the return value of `fork` to distinguish between the two processes.
- Use `write` syscall to print "child
  " or "parent
  " to stdout (`fd = 1`).
- Then call `getpid` and exit using the PID as the exit code.

### Hints:

- `write` syscall number is 1. It takes 3 arguments: file descriptor, buffer address, and length.
- You can store the message as `.asciz "child
  "` and `.asciz "parent
  "` in the `.data` section.

### Tools:

Use GDB to observe how both processes execute independently.

### Test:

```bash
as -o fork.o fork.s
ld -o fork fork.o
./fork
```

Use `echo $?` to observe the PID-based exit code from each process. Run multiple times to see how the PIDs change. 

**Exercise 8: Branching with UID Check – Are You Root?**

This exercise introduces conditional branching through a real-world scenario: checking whether the program is running with root privileges.

### Scenario:

Your task is to:

1. Call the `getuid` system call to get the UID of the process.
2. If the UID is `0`, print the message: `"You are root
   "`.
3. If the UID is not `0`, print: `"You are not root
   "`.
4. Exit with status code `0` if root, or `1` otherwise.

### Instructions:

- Look up the syscall number for `getuid` on the x86\_64 Linux syscall table.
- `getuid` returns the UID in `%rax`.
- Use `cmpq` to compare the return value with 0.
- Use conditional jumps (`je`, `jne`) to choose which message to write.
- Use `write` syscall (number `1`) to print to stdout.
- Store both messages in the `.data` section using `.asciz`.

###

### Test:

```bash
as -o uid.o uid.s
ld -o uid uid.o
./uid
```

Try running it with `sudo ./uid` to see the message change based on privilege level.

### Learnings:

- Real use of comparison and conditional jumps
- Branching based on syscall return values
- How UID affects process privileges in Unix systems

**Exercise 9: Using ************************************************************`cpuid`************************************************************ to Detect Logical CPU Count**

In this exercise, you'll use the `cpuid` instruction to find out how many logical processors (hardware threads) are available on the system, and use that to classify the system.

### Scenario:

Call the `cpuid` instruction with `eax = 1`. This tells the CPU to return feature and configuration information in the output registers. One of the returned values, stored in bits 23:16 of `%ebx`, encodes the number of logical processors per physical package. This means how many hardware threads (e.g., cores or SMT threads) the processor exposes.

To extract that field, shift `%ebx` right by 16 bits and mask the lower byte:

```asm
shr $16, %ebx
and $0xFF, %ebx
```

Based on that value:

- If it's `1`, print: `"Single-core system"` and exit with code `1`
- If it's `2–4`, print: `"Moderate CPU"` and exit with code `2`
- Else, print: `"High-performance system"` and exit with code `3`

### Instructions:

- Load `1` into `%eax` and execute `cpuid`

- Extract bits 23:16 of `%ebx` using:

  ```asm
  shr $16, %ebx
  and $0xFF, %ebx
  ```

- Then compare:

  - `cmp $1, %ebx` → jump to single-core branch
  - `cmp $5, %ebx` → if less, jump to moderate branch
  - Else → high-performance branch

- Use the `write` syscall to print appropriate message

- Exit with the number of threads as the exit status code

### Messages in `.data`:

```asm
single_msg:     .asciz "Single-core system
"
moderate_msg:   .asciz "Moderate CPU
"
highperf_msg:   .asciz "High-performance system
"
```

### Test:

```bash
as -o cpu.o cpu.s
ld -o cpu cpu.o
./cpu
```

Use `echo $?` to see the classification exit code.

### Learnings:

- Using `cpuid` for system introspection
- Bit-shifting and masking for field extraction
- Multi-way control flow with `if-elseif-else`

**Exercise 10: Traversing a Linked List Stored in the ****************************************`.data`**************************************** Section**

This exercise introduces you to struct-like layout, pointers, and traversal logic in memory.

### Scenario:

You are given a statically defined singly linked list in the `.data` section. Each node contains:

1. A pointer to the next node (or `0` if it's the last node)
2. A pointer to a string
3. An integer value (we won't use it yet)

Your job is to:

- Start from the `head` pointer
- Traverse each node one by one
- Print the string pointed to by each node
- Stop when the next pointer is `0`

### Data layout example:

```asm
.data
str1:      .asciz "node one
"
str2:      .asciz "node two
"
str3:      .asciz "node three
"

node3:
    .quad 0        # next = null
    .quad str3     # string
    .quad 300      # value

node2:
    .quad node3
    .quad str2
    .quad 200

node1:
    .quad node2
    .quad str1
    .quad 100

head:
    .quad node1
```

### Instructions:

- Load the address from `head`
- Follow the `next` pointer using `movq (%reg), %reg` to walk the list
- At each node:
  - Load the string pointer from offset 8
  - Use `write` syscall (fd = 1) to print it
- Continue until `next == 0`

This exercise does not require dynamic memory or function calls—it’s pure pointer arithmetic and fixed memory layout traversal.

### Learnings:

- Struct layout in memory
- Pointer dereferencing
- Walking memory structures with offset-based addressing
- Practical use of multiple field accesses per node

### Bonus:

You can store the integer value in `%rdi` and exit with the last node’s value just for fun! 

---

## Loop Exercises

### Exercise 3: "Array Maximum"
**Task:** Find the maximum value in an array of signed integers.

```asm
.data
values: .long -5, 12, -9, 24, 16, -3, 8, 19, -14
count:  .quad 9

.text
.globl _start
_start:
    # Initialize max with first element
    # Loop through remaining elements
    # Compare each with max and update if larger
    # Exit with max value as status
```

**Skills:** Loops with comparison, conditional updates, signed comparisons


### Exercise: "Memory Fill"
**Task:** Implement functionality similar to `memset` that fills a memory region with a specified byte.

```asm
.data
buffer: .space 100    # 100-byte buffer to fill
fill_byte: .byte 0x41 # ASCII 'A'
fill_size: .quad 100  # Number of bytes to fill

.text
.globl _start
_start:
    # Your code here to fill buffer with fill_byte for fill_size bytes
    # Use a loop to iterate through the memory region
    # Exit with 0 on success
```

**Skills:** Basic loop with counter, memory writes, pointer arithmetic

### Exercise: "String Length"
**Task:** Calculate the length of a null-terminated string (like `strlen`).

```asm
.data
test_string: .asciz "Hello, assembly programmer!"

.text
.globl _start
_start:
    # Load address of test_string
    # Loop through bytes until you find a zero byte
    # Count in a register
    # Exit with the string length as status code
```

**Skills:** Pointer traversal, null-termination check, counter

### Exercise: "Count Set Bits"
**Task:** Count the number of set bits (1s) in a 64-bit value.

```asm
.data
value: .quad 0x1234567890ABCDEF

.text
.globl _start
_start:
    # Load the value
    # Loop 64 times (for each bit)
    # Use shift and test/branch to count bits
    # Exit with the count as status code
```

**Skills:** Bit manipulation, shift operations, conditional logic in loops

### Exercise: "Reverse Array In-Place"
**Task:** Reverse an array of 64-bit integers in-place.

```asm
.data
array: .quad 1, 2, 3, 4, 5, 6, 7, 8
array_size: .quad 8

.text
.globl _start
_start:
    # Swap first with last, second with second-to-last, etc.
    # Exit with 0 on success
```

**Skills:** Array manipulation, pointer arithmetic, loop with multiple indices

# Number to ASCII String Conversion Exercise

**Task:** Convert a 64-bit unsigned integer into its ASCII string representation.

```asm
.data
value:      .quad 12345678901234   # The number to convert
buffer:     .zero 32              # Output buffer for ASCII string
digits:     .ascii "0123456789"    # Digit lookup table

.text
.globl _start
_start:
    # Your code here to convert value to ASCII string in buffer
    # Exit with the string length as status code
```

### Approach:

1. Initialize a pointer to the end of the buffer (we'll build the string backward)
2. Place a null terminator at the end
3. Loop:
   - Divide the number by 10 to get quotient and remainder
   - Use remainder to look up the ASCII digit
   - Store digit at current buffer position and decrement pointer
   - Update the number with the quotient
   - Repeat until number becomes zero
4. Optionally print the resulting string
5. Calculate string length and exit with that value

### Challenges to Consider:

- Handle signed values (with leading minus sign for negatives)

### Example Implementation Hints:

```asm
# Division approach:
# Loop:
#   mov number, %rax
#   mov $10, %rbx
#   xor %rdx, %rdx    # Clear upper bits for division
#   div %rbx          # Divide by 10, quotient in rax, remainder in rdx
#   # Use rdx as an index into the digits table
#   # Store digit at current buffer position
#   # Continue until number is 0
```

### Expected Output:

For the value 12345678901234, the buffer should contain the ASCII string "12345678901234", and the program should exit with status code 14 (the string length).

### Learning Outcomes:

- Division and modulus operations in assembly
- Backwards string construction technique
- Lookup table usage
- Pointer manipulation within a buffer
- Managing iteration conditions in loops

This exercise is particularly valuable because:
1. It's a practical algorithm used in all programming languages
2. It exercises arithmetic, memory access, and looping
3. It demonstrates how to convert between internal binary representation and human-readable format
4. It can be extended in multiple ways for additional challenges

### Exercise: "Simple File Copy"
**Task:** Copy contents from one file to another, 512 bytes at a time.

```asm
.data
source_path: .asciz "/etc/passwd"
dest_path:   .asciz "/tmp/passwd.copy"
buffer:      .space 512

.bss
source_fd:   .space 8
dest_fd:     .space 8

.text
.globl _start
_start:
    # Open source file for reading
    # Open dest file for writing (creating if needed)
    # Loop: Read up to 512 bytes, write to dest file
    # Continue until read returns 0 bytes
    # Close both files and exit
```
**Skills:** File I/O syscalls, read/write loop, buffer management

## Mini Project

# Process Resource Statistics with getrusage

## Background: The getrusage System Call

The `getrusage` system call provides a powerful way to gather resource usage information about a running process. It fills a data structure with statistics about CPU time, memory usage, I/O operations, and other system resource consumption.

On x86-64 Linux, the system call number for getrusage is 98. It takes two arguments:
1. An integer specifying which process to examine (0 for current process, 1 for children)
2. A pointer to a buffer where the data will be stored

## The rusage Structure

The kernel fills a `struct rusage` data structure that looks like this in C:

```c
struct rusage {
    struct timeval ru_utime;    /* user CPU time used */
    struct timeval ru_stime;    /* system CPU time used */
    long ru_maxrss;             /* maximum resident set size (KB) */
    long ru_ixrss;              /* integral shared memory size */
    long ru_idrss;              /* integral unshared data size */
    long ru_isrss;              /* integral unshared stack size */
    long ru_minflt;             /* page reclaims (soft page faults) */
    long ru_majflt;             /* page faults (hard page faults) */
    long ru_nswap;              /* swaps */
    long ru_inblock;            /* block input operations */
    long ru_oublock;            /* block output operations */
    long ru_msgsnd;             /* IPC messages sent */
    long ru_msgrcv;             /* IPC messages received */
    long ru_nsignals;           /* signals received */
    long ru_nvcsw;              /* voluntary context switches */
    long ru_nivcsw;             /* involuntary context switches */
};

struct timeval {
    long tv_sec;                /* seconds */
    long tv_usec;               /* microseconds */
};
```

In memory, each `long` value takes 8 bytes on x86-64, and the fields appear in the order shown. The `struct timeval` values each occupy 16 bytes (8 for seconds, 8 for microseconds).

## Your Task: Create a Resource Monitor

Build a utility that displays five key process resource statistics:

1. **User CPU time**: Time spent executing user code
2. **System CPU time**: Time spent in kernel mode on behalf of the process
3. **Maximum Resident Set Size**: Peak memory usage in kilobytes
4. **Page Faults**: Number of times the process needed to load memory from disk
5. **Context Switches**: Number of times the CPU switched to/from this process

### Implementation Requirements:

1. Allocate a buffer for the rusage structure (144 bytes is sufficient)
2. Call the getrusage system call with RUSAGE_SELF (0) as the first argument
3. Extract each statistic from the structure at the appropriate offset:
   - User time: offset 0 (first 8 bytes for seconds)
   - System time: offset 16 (8 bytes for seconds)
   - Max RSS: offset 32 (8 bytes)
   - Page faults: combine values at offsets 40 and 48
   - Context switches: combine values at offsets 112 and 120
4. Display each value with an appropriate label
5. Exit with status code 0

### Memory Layout of Key Fields:

```
Offset  Field
------  -----
0       ru_utime.tv_sec (user time seconds)
8       ru_utime.tv_usec (user time microseconds)
16      ru_stime.tv_sec (system time seconds)
24      ru_stime.tv_usec (system time microseconds)
32      ru_maxrss (maximum resident set size in KB)
40      ru_minflt (minor page faults)
48      ru_majflt (major page faults)
...
112     ru_nvcsw (voluntary context switches)
120     ru_nivcsw (involuntary context switches)
```

### Starter Code:

```asm
.data
# Headers for each statistic
user_time:      .asciz "User time (sec): "
system_time:    .asciz "System time (sec): "
max_rss:        .asciz "Memory usage (KB): "
page_faults:    .asciz "Page faults: "
ctx_switches:   .asciz "Context switches: "
newline:        .asciz "\n"

# Buffer for rusage struct
rusage:         .space 144

# Buffer for number conversion
num_buffer:     .space 32

.text
.globl _start
_start:
    # Call getrusage
    # ...your code here...
    
    # Process and display the statistics
    # ...your code here...
    
    # Exit
    # ...your code here...

# Helper function to convert numbers to strings
# ...your code here, if needed...
```

### Expected Output:

```
User time (sec): 0
System time (sec): 0
Memory usage (KB): 1024
Page faults: 74
Context switches: 3
```

This exercise combines practical system monitoring with fundamental assembly programming concepts: system calls, memory structures, offset calculations, and formatted output. It creates a genuinely useful tool while reinforcing core skills.

---

## Main Project

### Exercise 11: A Minimal Shell Using `fork` and `execve`

In this final project-style exercise, you’ll build a minimal shell that can launch real programs using `fork` and `execve`. You’ll also use memory allocation, syscall sequencing, and string copying — bringing together everything you’ve learned so far.

---

### 💡 Scenario:

Your shell should:

1. Read a command like `ls` or `date` from stdin (you may assume it's a single word)
2. Manually construct a new string: `/usr/bin/<command>`
3. Fork the current process using the `fork` syscall
4. In the child process, call `execve` with the constructed full path and an `argv[]` array containing just the command name
5. In the parent process, use `wait4` to wait for the child to finish
6. Exit with the child’s exit status

---

### 🛠️ Requirements:

- Use `brk` to allocate memory for the input buffer and for the `/usr/bin/` + command string
- You will need to copy bytes manually to concatenate `/usr/bin/` with the input string (excluding the newline)
- Set up `argv[]` with the command name (copied again into a separate area), and `envp = 0`

---

### Hints:

- Read 1 line of input from stdin using the `read` syscall (just like in Exercise 5)
- The prefix `/usr/bin/` is 10 bytes including the null terminator
- Assume commands will not exceed 16 characters (allocate 32 bytes total for the full path)
- Use a loop to copy the prefix and the user input (byte by byte)
- Strip the trailing newline before appending `�`
- Pass the final buffer as the path to `execve`

---

### Stretch Goals:

- Support a second word as a single argument (e.g., `echo hello`) by manually splitting on the space character
- Allow the shell to run multiple commands in a loop (after covering loops)

---

This is your first real systems program. You are directly manipulating memory, handling processes, and launching real executables without any help from the C standard library or runtime. Welcome to systems programming.

Use GDB to debug and visualize the memory, arguments, and syscalls your shell performs.

```bash
as -g -o shell.o shell.s
ld -o shell shell.o
./shell
```

Try entering:

```
ls
```

It should list files by launching `/usr/bin/ls`.
