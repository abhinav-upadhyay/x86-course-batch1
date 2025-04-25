
## Assembly Exercises on Functions

These exercises are designed to practice calling and writing functions in x86-64 assembly, including passing arguments, returning values, using the stack for local variables, and handling recursion.

---

### 1. Exercise: Find Sum and Count of Positive Numbers in an Array

**Goal:**  
Write a function that scans an integer array and computes:
- the **sum** of all positive numbers
- the **count** of positive numbers

---

#### Function Prototype:

```c
void sum_and_count_positive(int* arr, int size, int* sum_out, int* count_out);
```

- `arr` → pointer to the array
- `size` → number of elements
- `sum_out` → pointer to store the sum
- `count_out` → pointer to store the count

---

#### Instructions:

- Use **local variables** stored on the **stack** to keep track of:
  - Running `sum`
  - Running `count`
  - Loop index `i`
- Initialize `sum` and `count` to `0`.
- Loop through the array:
  - If `arr[i] > 0`, add it to `sum` and increment `count`.
- After the loop finishes, store the final `sum` and `count` at the memory locations pointed to by `sum_out` and `count_out`.

---

#### Boilerplate Code:

```assembly
.data
array:      .long 2, -5, 7, -1, 4
array_size: .long 5
sum_result:   .long 0
count_result: .long 0

.text
.globl _start

_start:
    movq $array, %rdi          # arr pointer
    movl array_size, %esi     # size
    movq $sum_result, %edx     # sum_out pointer
    movq $count_result, %ecx   # count_out pointer
    call sum_and_count_positive

    # Exit
    movl $60, %eax
    xorl %edi, %edi
    syscall
```

---

### 2. Exercise: Tail Recursive Count of Positive Numbers in an Array

**Goal:**  
Write a **tail recursive** function that counts the number of positive numbers in an array.

---

#### Function Prototype:

```c
int count_positive_tail(int* arr, int size, int acc);
```

- `arr` → pointer to array
- `size` → number of elements
- `acc` → accumulator (count of positive numbers so far)

---

#### Instructions:

- If `size == 0`, return `acc`.
- Otherwise:
  - If `arr[0] > 0`, increment `acc`.
  - Move to the next element (`arr + 1`) and decrease `size` by 1.
- Make the recursive call passing the updated arguments.
- Use **tail recursion**: the recursive call must be the **last** operation.

---

#### Boilerplate Code:

```assembly
.data
array:      .long 1, -2, 3, -4, 5
array_size: .long 5
result:     .long 0

.text
.globl _start

_start:
    movq $array, %rdi    # arr
    movl array_size, %esi # size
    xorl %edx, %edx            # acc = 0
    call count_positive_tail

    # Store result into memory
    movl %eax, result

    # Exit
    movl $60, %eax
    xorl %edi, %edi
    syscall
```

---

### Additional Hints

- Remember to properly set up and tear down the function stack frame (`push %rbp`, `mov %rsp, %rbp`, `leave`, `ret`).
- Stack allocate enough space for **local variables**.
- Manage registers carefully: arguments will come in `%rdi`, `%rsi`, `%rdx`, `%rcx`, etc.
- When writing recursive functions, ensure that the **recursive call is the last action** for true tail recursion.

---
