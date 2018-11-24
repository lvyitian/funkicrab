# Funki Crab

Funki Crab is an optimising Brainfuck compiler written in Rust.

**Note: The quality of the code in Funki Crab is, currently, abysmal. If you're looking to use it as a source of information... don't. I'm working on improving this situation.**

## Example

### Brainfuck Input

```bf
++++++++++[>+++++++>++++++++++>+++>+<<<<-]>++.>+.+++++++
..+++.>++.<<+++++++++++++++.>.+++.------.--------.>+.>.
```
### Funki Crab Output (C)

```c
#include <stdio.h>

char outputs[13] = { 72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100, 33, 10 };

int main() {
    fwrite(outputs, sizeof(char), 13, stdout);
	return 0;
}
```

### Execution Result

```
Hello World!
```

## Purpose

I created Funki Crab as an exercise in learning about compiler development, Immediate Representation (IR) techniques and optimisation. Brainfuck struck me as a
sensible language for such a project given its simplicity, Turing-completeness and wealth of potential optimisations.

Funki Crab is an anagram of Brainfuck. 'Crab' is a reference to Ferris, the Rust mascot.

## Targets

Currently, Funki Crab can only transpile Brainfuck to C. In the future, it will be capable of compiling to assembly or even raw binaries.

## Optimisations

### Combining Shifts, Increments And Decrements

Funki Crab can combine consecutive shifts, increments and decrements together into a single IR instruction.

Example:

`++++>>>><<---+<`

Becomes:

```c
*ptr += 4;
ptr += 2;
*ptr -= 2;
--ptr;
```

### Copy-And-Multiply Loop Elision

Funki Crab can unroll loops that count down one cell in order to add-and-multiply other cells into single IR instructions.

Example:

`[->+++>>>--<<<<]`

Becomes:

```c
ptr[1] += *ptr * 3;
ptr[4] -= *ptr * 2;
*ptr = 0;
```

### Zeroing And Set Loop Elision

Funki Crab can transform loops that decrement to zero and add into a single IR instruction.

Example:

`[-]++++`

Becomes:

```c
*ptr = 4;
```

### Shift Elision

Funki Crab can elide shift operations by keeping track of the increment and passing it on to future operations.

Example:

`+>>>+`

Becomes:

```c
++*ptr;
++ptr[3];
```

### Inter-Loop Shift Elision

Funki Crab can elide shifts across loop boundaries.

Example:

`+<<[>]>>`

```c
++*ptr;
while (ptr[-2]) {
	++ptr;
}
```

### Dead Code Elimination

Funki Crab can eliminate many common instances of dead code.

Example:

`+,>[-],`

Becomes:

```c
*ptr = getchar();
++ptr;
*ptr = getchar();
```

### Static Cell Analysis

Funki Crab can analyse the value of cells and operations performed upon cells at compile-time and reason about their effect on later parts of the program.
This means that even more dead code can be eliminated and certain instructions can safely be elided. In addition, Funki Crab can use this analysis information to
test the accessibility of a loop, eliding it if possible.

Example:

`>[.-]`

Becomes:

*Nothing*

### Dead Tail Elimination

Funki Crab can analyse instructions at the tail of a program and eliminate them if it can prove that they have no impact.

Example:

`+.>>+<---+>`

Becomes:

```c
++*ptr;
putchar(ptr[0]);
```

### Compile-Time Execution

Funki Crab is capable of partial execution of the program at compile-time for all instructions other than `.`. For programs that do not rely on input, and finish
execution with fewer than 1 million instructions, it's even possible for Funki Crab to fully execute the entire program and emit only the code necessary to print
the end result.

## Combining Optimisations

Funki Crab is capable of combining the optimisations listed above, and more, through multiple transformations of the IR.

Example:

`+.>>+<---+>[-<+[-][>>++++[.-]][-++]->+-]--`

Becomes:

*Nothing*

## Speedup

Typical speedup with Funki Crab optimisations enabled vs disabled for substantial Brainfuck programs is ~100x.

## Future

I don't plan to maintain Funki Crab as a long-term project. Perhaps I'll add things like better loop unrolling, constant propagation, compile-time execution or
instruction reordering to improve cache coherency in the future.
