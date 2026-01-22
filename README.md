*This project has been created as part of the 42 curriculum by yelmajdo*

## Description

Push_swap is a sorting algorithm project that uses two stacks and 11 allowed operations to sort a list of integers in ascending order. The goal is to output the minimum number of operations needed.

## Problem Statement

Given a list of unsorted integers, use stack_a and stack_b with only specific operations to sort stack_a in ascending order. The challenge is to minimize the number of operations.

## The 11 Allowed Operations

### Stack A Operations
- **sa** (swap a): Swap the first two elements of stack_a
- **ra** (rotate a): Shift all elements of stack_a up by one; the first element becomes the last
- **rra** (reverse rotate a): Shift all elements of stack_a down by one; the last element becomes the first

### Stack B Operations
- **sb** (swap b): Swap the first two elements of stack_b
- **rb** (rotate b): Shift all elements of stack_b up by one
- **rrb** (reverse rotate b): Shift all elements of stack_b down by one

### Both Stacks
- **ss**: Execute sa and sb simultaneously
- **rr**: Execute ra and rb simultaneously
- **rrr**: Execute rra and rrb simultaneously

### Move Operations
- **pa** (push a): Move the first element of stack_b to the top of stack_a
- **pb** (push b): Move the first element of stack_a to the top of stack_b

## Data Structure

The project uses a **singly-linked list** with the following node structure:

```c
typedef struct s_node
{
    int             value;
    int             index;
    struct s_node   *next;
}   t_node;
```

Each number gets an **index** (0-based) representing its rank among all numbers. This is crucial for the radix sort algorithm.

Example: For input `3 2 1 4 5`
- 1 (smallest) → index 0
- 2 → index 1
- 3 → index 2
- 4 → index 3
- 5 (largest) → index 4

## Algorithm Overview

The sorting strategy depends on the number of elements:

### 1. **Single Element (size = 1)**
Already sorted, no operations needed.

### 2. **Two Elements (size = 2)**
```
If a[0] > a[1]: execute sa (swap)
Otherwise: already sorted
```

### 3. **Three Elements (size = 3)**
Optimized to handle all 6 permutations with a maximum of 2 operations:
1. Find the position of the maximum element
2. If max is at position 0: rotate right (rra)
3. If max is at position 1: rotate left (ra)
4. If max is already at position 2: do nothing
5. After max is at the end, if first two elements are out of order: swap (sa)

### 4. **Four or Five Elements (size = 4-5)**

Strategy: Move smallest elements to stack_b, sort remaining 3, then insert them back correctly.

**Steps:**
1. Identify the 2 smallest elements (indices 0 and 1)
2. Push these 2 smallest elements to stack_b
3. Sort the remaining 3 elements on stack_a (using sort_3 logic)
4. For each element in stack_b:
   - Find the correct insertion position in stack_a
   - Rotate stack_a so the insertion point is at the top
   - Push the element back with pa

**Example: Input `3 2 1 4 5` with indices `2 1 0 3 4`**
- Push index 0 (value 1) to stack_b
- Push index 1 (value 2) to stack_b
- Sort `3 4 5` (indices `2 3 4`) on stack_a
- Insert index 0 at the beginning
- Insert index 1 at position 1
- Result: sorted stack_a = `1 2 3 4 5`

### 5. **Six or More Elements (size >= 6)**

**Radix Sort Algorithm** using binary representation:

#### How Radix Sort Works:

1. **Index Assignment**: Convert each value to its 0-based rank (index)

2. **Calculate Max Bits**: Determine how many bits are needed to represent the largest index
   - For 6 elements: max_index = 5, which needs 3 bits (binary: 101)
   - For 100 elements: max_index = 99, which needs 7 bits

3. **Bit-by-Bit Processing**: Process each bit position from least significant (bit 0) to most significant
   
4. **For Each Bit Position**:
   - Start with all elements in stack_a, stack_b is empty
   - Iterate through all current elements in stack_a (size - elements_pushed times)
   - For each element at the top of stack_a:
     - Check the current bit in its index
     - If bit is 0: push to stack_b (pb)
     - If bit is 1: rotate stack_a to move element to the back (ra)
   - After processing all elements, push everything back from stack_b to stack_a (pa)
   - This leaves stack_a with elements sorted by this bit, stack_b empty

#### Bit Processing Example:

For input `3 2 1 4 5` with indices `[2, 1, 0, 3, 4]`:

**Bit 0 (Least Significant):**
- Index 2 = 010 in binary → bit[0] = 0 → **pb** (push to b)
- Index 1 = 001 in binary → bit[0] = 1 → **ra** (rotate a)
- Index 0 = 000 in binary → bit[0] = 0 → **pb** (push to b)
- Index 3 = 011 in binary → bit[0] = 1 → **ra** (rotate a)
- Index 4 = 100 in binary → bit[0] = 0 → **pb** (push to b)
- Stack_a now has: `[3, 1]` (bits ending in 1)
- Stack_b has: `[4, 0, 2]` (bits ending in 0)
- Push all back: **pa pa pa** → Stack_a = `[2, 0, 4, 3, 1]`

**Bit 1:**
- Repeat for bit position 1 (the middle bit)

**Bit 2 (if needed):**
- Repeat for bit position 2

**Result**: After processing all bits, stack_a is sorted by index value, and since indices are assigned by sorting, stack_a is now sorted by value!

#### Why This Works:

Radix sort works because:
1. Processing least significant bit first ensures stable sorting
2. Each bit divides elements into two groups (bit=0 or bit=1)
3. After processing all bits, elements are sorted by their binary representation
4. Since indices are assigned in sorted order of values, sorting by index = sorting by value

#### Time Complexity:
- **O(b × n)** where b = number of bits, n = number of elements
- For 100 elements (max_index=99, needs 7 bits): 7 × 100 = 700 operations
- For 500 elements (max_index=499, needs 9 bits): 9 × 500 = 4500 operations

## Input Validation

The program validates:
- **No non-numeric arguments** (except signs `+` and `-`)
- **No duplicate numbers** (same number appears multiple times = error)
- **Numbers within int range** (-2,147,483,648 to 2,147,483,647)
- **Handles both formats**:
  - Space-separated: `./push_swap 3 2 1`
  - Quoted string: `./push_swap "3 2 1"`
- **Overflow detection**: Numbers beyond INT_MIN and INT_MAX are rejected

Output on invalid input: `Error` to stderr (exit code 1)


## Operation Count Analysis

**Best Case:**
- Already sorted: 0 operations

**Small Arrays:**
- 2 elements: max 1 operation
- 3 elements: max 2 operations
- 4 elements: typically 3-8 operations
- 5 elements: typically 5-12 operations

**Large Arrays (using Radix Sort):**
- 6 elements: ~18-24 operations
- 100 elements: ~700 operations
- 500 elements: ~4500 operations


## Instructions

### Compilation

Compile the project using:

```bash
make
```

To remove object files:

```bash
make clean
```

To remove object files and the executable:

```bash
make fclean
```

To recompile the project:

```bash
make re
```

### Execution

Run the program with a list of integers as arguments:

```bash
./push_swap 3 2 1
```

To verify the correctness of the output using the checker (if available):

```bash
ARG="4 67 3 87 23"; ./push_swap $ARG | ./checker_OS $ARG
```

## Resources

### References

- 42 Subject PDF — push_swap

- Stack data structure
https://www.w3schools.com/dsa/dsa_data_stacks.php

- Sorting algorithms
https://www.freecodecamp.org/news/sorting-algorithms-explained-with-examples-in-python-java-and-c/

- Big-O notation
https://www.geeksforgeeks.org/dsa/analysis-algorithms-big-o-analysis/

- Radix sort
https://www.geeksforgeeks.org/dsa/radix-sort/


### Use of Artificial Intelligence

#### Artificial Intelligence tools were used as a support resource to better understand algorithmic concepts and project requirements. AI assistance was limited to:
- Helping analyze complexity and optimization strategies
- Assisting with debugging logic and edge case reasoning
- Improving documentation clarity
