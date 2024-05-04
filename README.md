# finalreview
x# Final Review (Spring 2024)

# Format

- Same question format as the midterm
- Open notes: bring as much printed material as you want
- Closed computers / phones / everything else
- You'll be expected to be familiar with:
  - Languages (and IRs) used in our compilers
  - Compiler passes in our assignments
- Should take only 30-40 minutes
- If you successfully completed the homework assignments and exercises, the questions should be easy
- Liberal partial credit applied (especially when you're asked to write code)

# Major topics

- Loops and dataflow analysis
  - While loops & typechecking of loops
    - what do the while loops look like and how do we typecheck them?
  - Cyclic control flow graphs and challenges of liveness analysis
    - We had to update explicate control(the backwards edge)
  - Fixpoint-based dataflow analysis
    - We start out with an assumotion that is wrong and keep updating it iteratively until nothing changes
    - Likely a question that is "do the data flow analysis"
- Heap allocation
  - Tuples
    - How do we do the typechecking for those?
    - We allocate tuples in the heap, tuple value variables are actually pointers to heap locations
  - Representation and tagging
    - When we create a tuple, we have to add a tag so the garbage collector knows more information about it
    - Here is a program, create the tags for the tuples
  - Root stack: garbage collector needs to find all tuple value variables
  - Garbage collection
    - How it works.. you follow pointers from the root stack that has things that are not garbage in the fromspace to things that are garabage in the tospace
- Functions
  - Calling convention
  - Limiting number of arguments
  - Functions as values
- Objects (only conceptual knowledge expected)
  - Method tables
  - Representation with tuples
- Lexically scoped functions (including lambda) (only conceptual knowledge expected)
  - Static/lexical vs dynamic scope
  - Lifting lambdas to top-level functions
  - Closure conversion
- Dynamic typing (only conceptual knowledge expected)
  - Definitions
  - "Any" type and typechecking
  - Inject / project
  - Representation (tagging)
  - Runtime checks & generating x86
- Optimization (only conceptual knowledge expected)
  - Local
    - Partial evaluation
    - Constant propagation
    - Dead code elimination
  - Intraprocedural
    - Register allocation
    - Static typing
    - Move biasing
    - Eliminating unneeded jumps
  - Interprocedural
    - Inlining
- How code gets executed
  - Role of the compiler and linker
  - Object files vs executables
  - Encoding instructions

# Sample Questions

## Cyclic Control Flow Graphs

Consider the following pseudo-x86 program with 4 basic blocks.

    start:
      movq $0, x
      jmp label_1
    label_1:
      cmpq $10, x
      jl label_2
      jmp label_3
    label_2:
      addq $1, x
      jmp label_1
    label_3:
      jmp conclusion
    
1. Draw the control flow graph for this program (without
   instructions - only labels required).

2. In what order should liveness analysis be performed on the basic
   blocks of this program?

   The order does not matter, there is not oder that is going to make it easier.
   The way to do this is to do the data flow analysis approach ehere you start with the assumption that all    the live before sets will at the labels are empty and iterate liveness analysisuntil the live before        sets stop changing. When they have stopped changing, then you have reacvhed the fixed point

   You needs to know reads_of and writes_of before along with the equation.
   1. Look at next instruction
   2. Remove everything that gets written
   3. Add everything that gets read

4. Perform the liveness analysis for this program, filling in the
   live-after sets for each instruction and the live-before sets for
   each basic block.

## Heap allocation

Consider the following program:

    x = (5, 6)
    y = (x, 5, x)
    z = (x, y)

1. Assuming registers are *not* used, which variables will be placed on the stack, and which variables will be placed on the root stack?

   x, y, and z are all tuple-valued variables and will be placed on the root stack. and nothing goes on the regular stack. The garbage collector needs to know where the tuple-valued variables are.


3. Draw the tags for the tuples `x`, `y`, and `z`.

64 bit tag with 1 at least signficant bit, 6 bits for length, 50 bits for pointer mask(0 if element was an integer or boolean) or 1 if it was another tuple

x = all 0s
   
5. Draw the root stack and heap after this program runs, assuming the initial heap size is large enough that no garbage collection is needed.

Consider the following heap state (root stack and from-space):

![](heap-diagram1.png)

4. Complete the diagram by filling the to-space with the new state of the heap *after* garbage collection. Assume Cheney's algorithm for copying collection.

## Functions


Consider the following x86 program.

```
add1_main:
  pushq %rbp
  movq %rsp, %rbp
  pushq %rbx
  pushq %r12
  pushq %r13
  pushq %r14
  subq $0, %rsp

  jmp add1_start
add1_start:
  movq %rdi, %r12
  movq %r12, %r12
  addq $1, %r12
  movq %r12, %rax
  jmp add1_conclusion
add1_conclusion:
  addq $0, %rsp
  popq %r14
  popq %r13
  popq %r12
  popq %rbx
  popq %rbp
  retq

  .globl main
main:
  pushq %rbp
  movq %rsp, %rbp
  pushq %rbx
  pushq %r12
  pushq %r13
  pushq %r14
  subq $0, %rsp
  movq $16384, %rdi
  movq $16, %rsi
  callq initialize
  movq rootstack_begin(%rip), %r15

  jmp main_start
main_start:
  movq $5, %rdi
  callq add1_main
  movq %rax, %r12
  movq %r12, %rdi
  callq print_int
  jmp main_conclusion
main_conclusion:
  movq $0, %rax
  addq $0, %rsp
  popq %r14
  popq %r13
  popq %r12
  popq %rbx
  popq %rbp
  retq
```

7. Write the original Python program for which this x86 program is (roughly) the output of our compiler.


## Anonymous functions (lambda)

8. What are the free variables of the lambda expression `lambda x: -> x + y`?

   y is a free variable, x is a bound variable

Consider the following program:

```
y = 10
f = lambda x: y
y = 20
print(f(5))
```

9. What value is printed under *dynamic scope*?

    20

11. What value is printed program under *lexical scope*?

  10
12. Perform closure conversion to transform this program into a Lfun program (you may omit type annotations). Your transformed code should implement lexical scope.


## Dynamic typing

Consider the following program:

```
if True:
  x = 1 + 2
else:
  x = False + 3
```

12. Does this program have a *static* type, according to our typechecker?
    NO

14. Does running this program result in a *run-time* type error?
    NO, we never run the run time error code

16. Convert this program to one which has a static type, according to the Lany typechecker. Use `inject` and `project`.


## How code gets executed

Consider the following output of `objdump -d adduser.o`:

    adduser.o:    file format Mach-O 64-bit x86-64
    
    Disassembly of section __TEXT,__text:
    _main:
          0:    48 c7 c1 51 00 00 00     movq    $81, %rcx
          7:    48 c7 c2 2d 00 00 00     movq    $45, %rdx
          e:    e8 00 00 00 00     callq    0 <_main+0x13>
         13:    c3     retq

19. What component is responsible for producing the file `adduser.o`?
    The assembler always produces object files

21. What component is responsible for combining `adduser.o` with another file which contains the definition of the `add` function?

    The linker
