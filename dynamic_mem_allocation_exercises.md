# Exercise: Build a Dynamic Linked List Using Your own Allocator

## Objective

Implement a **singly linked list** in x86-64 assembly.  
Each **node** must be **dynamically allocated** using your own `allocate` function, which you will implement in a **separate source file**.

You will insert integers into the list and connect the nodes together using pointers stored in memory.

---

## What You Need to Do

- Implement a function `insert_node(int value)` in the main source file:
  - Allocate 16 bytes for a new node using `allocate`.
  - Store the given integer in the node.
  - Update the "next" pointer to connect the new node into the list.
- Maintain a global variable `head` that points to the start of the linked list.

- Insert a few nodes manually from `_start`, passing values like 10, 20, 30.

---

## Node Structure

Each node must have the following layout in memory:

| Offset | Field        | Size     | Description              |
|:-------|:-------------|:---------|:--------------------------|
| 0      | Value         | 4 bytes  | The integer value         |
| 4–7    | (Padding)     | 4 bytes  | Unused (alignment padding)|
| 8      | Next pointer  | 8 bytes  | Address of the next node  |

Total size per node: **16 bytes** (aligned to 8 bytes).

---

## Boilerplate Setup

```assembly
.data
head: .quad 0         # Address of the first node (initially NULL)

.text
.globl _start

_start:
    # Insert nodes manually

    movl $10, %edi     # Insert value 10
    call insert_node

    movl $20, %edi     # Insert value 20
    call insert_node

    movl $30, %edi     # Insert value 30
    call insert_node

    # Optional: Traverse and verify (not required if short on time)

    # Exit
    movl $60, %eax
    xorl %edi, %edi
    syscall
```

---

## Implementing the `allocate` Function

- Write the `allocate` function in a **separate source file** (e.g., `memory.s`).
- `allocate` must:
  - Take the size to allocate in bytes (`rdi`).
  - Use the `brk` system call to grow the program's heap.
  - Return the address of the allocated memory block in `rax`.

- You can define `allocate` as:

```assembly
.globl allocate

allocate:
    pushq %rbp
    movq %rsp, %rbp

    # Implementation using brk syscall
    # Example:
    #  - Maintain a static pointer for the program break
    #  - Increase it by requested size
    #  - Return old break as allocated address

    popq %rbp
    ret
```


---

## Building and Linking Instructions

Since `allocate` is in a separate file:

1. Assemble both files separately using `as`:

```bash
as -o main.o main.s
as -o memory.o memory.s
```

2. Link them together using `ld`:

```bash
ld -o program main.o memory.o
```

Now you can run `./program` as usual.

---

## Rough Steps Inside `insert_node`

You can guide the students roughly:

- Call `allocate(16)` to get memory for one node.
- Store the given value at offset 0.
- Set the new node's next pointer to the current value of `head`.
- Update `head` to point to the new node.

This builds the list **inserting at the front** every time.

---

## Expected Behavior

After inserting 10, 20, 30:

- `head` points to the node with value 30.
- The nodes are connected as: `30 -> 20 -> 10 -> NULL`

Each node contains:
- The integer value.
- A pointer to the next node.

---

## Bonus (Optional)

If time permits:

- Implement a `traverse_list` function that walks through the list and prints the integer values.
- Count how many nodes exist and store or print the count.

---

## Key Takeaways

- Understanding how **dynamic memory allocation** enables flexible, extensible data structures.
- Practice working with **pointers and memory offsets**.
- Building a **real dynamic system structure** (linked list) instead of static data.

