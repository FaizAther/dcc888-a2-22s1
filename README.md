# dcc888-a2-2022
dcc888 @ assn 2

The course has three projects, each one worth 10 points, which gives us
a total of 30 points. Two of the projects use the LLVM compilation
framework. We have an online course about LLVM
[here](https://youtube.com/playlist?list=PLDSTpI7ZVmVnvqtebWnnI8YeB8bJoGOyv).

2.  Second project: dead-code elimination - June 12th - 11:59pm (BRT).

Second Project: Dead-Code Elimination - 10 points
-------------------------------------------------

In this project you will implement a dead-code elimination pass that
uses [Range
Analysis](http://homepages.dcc.ufmg.br/~fernando/classes/dcc888/ementa/slides/RangeAnalysis.pdf)
to cut off useless code. Range analysis, as we have seen in
[class](http://homepages.dcc.ufmg.br/~fernando/classes/dcc888/ementa/slides/RangeAnalysis.pdf),
is a data-flow technique that finds the lower and upper limits that the
integer variables in a program will assume throughout the execution of
that program. You must use, in this homework, the implementation of
range analysis that is available [here (LLVM
3.8)](https://homepages.dcc.ufmg.br/~fernando/classes/dcc888/assignment/src/RangeAnalysis_LLVM_38.zip)
or here [here (LLVM
8.0)](https://homepages.dcc.ufmg.br/~fernando/classes/dcc888/assignment/src/RangeAnalysis_LLVM_80.zip).
For some instructions on how to run it, check
[these](https://homepages.dcc.ufmg.br/~fernando/classes/dcc888/assignment/images/AngelicaTips.png)
hints. The range analysis is also available in a git
[repository](https://github.com/vhscampos/range-analysis). Basically,
you must run the implementation of range analysis on an input program,
and then use the information that it provides to remove code that you
are sure never to be executed. Which kind of code is this? Well, below
we have two programs that have useless code:

    int foo() {
      int i = 0;
      int sum = 0;
      for (i = 0; i < 100; i++) {
        sum += i;
        if (i > 101) {
          break;
        }
      }
      return sum;
    }

In this case, the branch that encloses the break will never be true, as
the variable i is upper limited by 99. Another example of useless code
is seen below:

    int foo(int k) {
      int i = 0;
      if (k % 2) {
        i++;
      }
      return i & 1;
    }

In this example, the return statement could be simplified to `return i`,
because the variable i is within the bounds \[0, 1\]. Your pass must be
able to eliminate obviously redundant code, like the branch in the first
example. Of course, it is difficult to determine everything that must be
eliminated, as a more advanced dead-code elimination pass can do more
than a naive implementation. In order to know if your code is good
enough, we may try it on a few examples, and you must reduce at least
half of them. Your pass must print a few statistics:

-   Number of instructions eliminated
-   Number of basic blocks entirely eliminated

You can obtain them using the LLVM\'s
[stats](http://llvm.org/docs/ProgrammersManual.html#the-statistic-class-stats-option)
option. You will need to add some code like these lines below into your
pass:

    STATISTIC(InstructionsEliminated, "Number of instructions eliminated");
    STATISTIC(BasicBlocksEliminated,  "Number of basic blocks entirely eliminated");

In the end, your pass must be able to convert a bytecode like the one on
the left into something like the bytecode on the right:

  --------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------
  ![Example before dead-code elimination.](https://homepages.dcc.ufmg.br/~fernando/classes/dcc888/assignment/images/originalProg.png)
  ![Example after dead-code elimination](https://homepages.dcc.ufmg.br/~fernando/classes/dcc888/assignment/images/optimizedProg.png)
  --------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------

Feel free to use as much help from LLVM as you want. In particular, you
may find procedures like `RecursivelyDeleteTriviallyDeadInstructions`,
available in `llvm/Transforms/Utils/Local.h`, very useful. Just to help
you a bit more, below you have a piece of code that does what you will
be doing all the time in this assignment:

    bool::RADeadCodeElimination::solveICmpInstruction(ICmpInst* I) {
      InterProceduralRA < Cousot >* ra = &getAnalysis < InterProceduralRA < Cousot > >();
      Range range1 = ra->getRange(I->getOperand(0));
      Range range2 = ra->getRange(I->getOperand(1));
      switch (I->getPredicate()){
        case CmpInst::ICMP_SLT:
        //This code is always true
        if (range1.getUpper().slt(range2.getLower())) {
          ...
        }
        break;
        case ...
      }
      ...
    }

To link your pass with our range analysis, I recommend you to take a
look into the file `ClientRA.cpp`, which is available in our
[distribution](https://homepages.dcc.ufmg.br/~fernando/classes/dcc888/assignment/src/RangeAnalysis_LLVM_38.zip).
For the command line flags used to activate the range analysis, take a
look into the script `run_RA.sh`, also available in that zip file. If
you are using LLVM 8.0 or higher, then make sure you generate bytecodes
with optimizations enabled, e.g.:

    clang -Xclang -disable-O0-optnone ${file}.c -o ${file}.bc -c -emit-llvm

**What must be turned in:** each group must e-mail the instructor a zip
file, containing the pass, plus a README.txt file. The zip file must
contain the directory in which the pass was implemented, including the
Makefile. The instructor must be able to compile the pass by just
unpacking the directory in `/llvm/lib/Transforms`, and then typing
`make`. The README.txt file must contain the command line that should be
used to run the pass.