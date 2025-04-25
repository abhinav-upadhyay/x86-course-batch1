# Assignment: Build a Tiny In-Memory Key-Value Store (Redis CLI)

## Objective

Build a simple **key-value store** in x86-64 assembly, similar to a very basic **Redis** server.

You will support two commands:

- `put <key> <value>` — stores the key-value pair into memory
- `get <key>` — retrieves and prints the value associated with a key

All data is stored **dynamically in memory** using your own `allocate` function (based on `brk`).

You are provided with a basic hash function to assist in bucket placement.

---

## Functional Requirements

- Keys are treated as **strings** (even if they look numeric).
- Values are also **strings**.
- A **hash table** of fixed size (64 buckets) is used to organize the data.
- **Separate chaining** (linked list of nodes) is used to handle hash collisions.
- **No deletion** is required.
- **No resizing** of the hash table is required.
- Dynamic memory is allocated using your `allocate` function.

---

## System Overview

### Structures:

1. **Hash Table:**
   - An array of 64 pointers (buckets).
   - Each bucket points to a linked list of nodes.

2. **Key-Value Node:**
   - 8 bytes: Pointer to key string
   - 8 bytes: Pointer to value string
   - 8 bytes: Pointer to next node (for collisions)

Total size per node = **24 bytes**.

### Memory Rules:

- Each key and value string must be separately allocated using `allocate`.
- Key and value are copied from user input into dynamically allocated space.
- Only `put` and `get` operations need to be supported.
- No real `free` is required — all memory grows upward.

---

## Source File Organization
- Split it into multiple modules
  - hash_table.s: implements a hash table
  - linkedlist.s: implements a linked list and used by hash table for chaining
  - memory.s: implements memory allocation functions: allocate/free
  - main.s: implements the cli
---

## Boilerplate Provided

You can use the following simple hash function for your hash table:

1. A basic hash function:

```assembly
.globl hash
# Input: rdi = pointer to key (string)
# Output: rax = hash bucket index (0-63)
hash:
    pushq %rbp
    movq %rsp, %rbp

    xorl %eax, %eax        # clear eax (hash accumulator)

.hash_loop:
    movzbq (%rdi), %rcx    # load byte, zero-extend
    testb %cl, %cl         # check for null terminator
    je .hash_done

    addq %rcx, %rax        # add character value
    incq %rdi              # move to next character
    jmp .hash_loop

.hash_done:
    andq $63, %rax         # modulo 64 buckets
    popq %rbp
    ret
```

2. A partially filled `.data` section:

```assembly
.data
buckets:
    .space 512          # 64 buckets * 8 bytes each

newline:
    .byte 10

input_buffer:
    .space 128          # For reading user input
```

---

## Tasks to Implement

### Functions in your main file:

1. **put**  
   Insert or update a key-value pair in the hash table.
   - Hash the key.
   - Traverse the bucket:
     - If key exists, update value.
     - Otherwise, create new node:
       - Allocate memory for node.
       - Allocate memory for key string.
       - Allocate memory for value string.
       - Copy strings into memory.
       - Insert node at the head of the bucket's list.

2. **get**  
   Lookup a key in the hash table and print its value.
   - Hash the key.
   - Traverse the linked list in the bucket:
     - If found, print the value.
     - If not found, print a "Key not found" message.

3. **Simple CLI loop**
   - Read a line from user input into `input_buffer`.
   - Parse the command:
     - If it starts with "put", extract key and value and call `put`.
     - If it starts with "get", extract key and call `get`.
   - Repeat until manually exited (you can use Ctrl+C to terminate).

---

### Notes:

- Assume maximum key size = 32 bytes.
- Assume maximum value size = 32 bytes.
- Simplify parsing: you can split based on spaces manually.
- Your allocate function is **grow-only** using `brk`. No real `free` is needed.
- Dummy `free` function can be provided but it will do nothing.

---

## About `allocate` and `free`

You must implement:

- `allocate` (already described separately): allocates dynamic memory using `brk`.
- `free` must exist, but it can simply be a dummy function (empty body), because individual freeing is not supported with simple `brk`-based allocation.

Implement `allocate` and `free` in a **separate source file** (e.g., `memory.s`).

You must **assemble and link** both your main file and the memory file:

```bash
as -o main.o main.s
as -o memory.o memory.s
as -o hash_table.o hash_table.s
as -o linkedlist.o linkedlist.s
ld -o rediscli main.o memory.o hash_table.o linkedlist.o
```

---

## Hints

- Carefully manage pointers and memory addresses.
- You can use `movsb` or manual byte-by-byte copying to copy strings into allocated memory.
- Always null-terminate copied strings.
- When printing strings, loop until you reach the null byte (`\0`).

---

## Example Interaction

```
> put foo bar
> put hello world
> get foo
bar
> get hello
world
> get unknown
Key not found
```

---

## Key Learning Outcomes

- Dynamic memory management using `brk`
- Implementing and using hash tables
- Building real dynamic structures (key-value stores)
- Basic parsing and string manipulation in assembly
- Handling system I/O and memory management manually

