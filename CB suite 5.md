# 5  Computing with Register Machines

> My aim is to show that the heavenly machine is not a kind of divine, live being, but a kind of clockwork (and he who believes that a clock has soul attributes the maker's glory to the work), insofar as nearly all the manifold motions are caused by a most simple and material force, just as all motions of the clock are caused by a single weight.
> 
> - Johannes Kepler (letter to Herwart von Hohenburg, 1605)

[](https://sourceacademy.org/sicpjs/5#p1)

We began this book by studying processes and by describing processes in terms of functions written in JavaScript. To explain the meanings of these functions, we used a succession of models of evaluation: the substitution model of chapter [1](https://sourceacademy.org/sicpjs/1), the environment model of chapter [3](https://sourceacademy.org/sicpjs/3), and the metacircular evaluator of chapter [4](https://sourceacademy.org/sicpjs/4). Our examination of the metacircular evaluator, in particular, dispelled much of the mystery of how JavaScript-like languages are interpreted. But even the metacircular evaluator leaves important questions unanswered, because it fails to elucidate the mechanisms of control in a JavaScript system. For instance, the evaluator does not explain how the evaluation of a subexpression manages to return a value to the expression that uses this value. Also, the evaluator does not explain how some recursive functions can generate iterative processes (that is, be evaluated using constant space) whereas other recursive functions will generate recursive processes.[1](https://sourceacademy.org/sicpjs/5#footnote-1) This chapter addresses both of these issues.

[](https://sourceacademy.org/sicpjs/5#p2)

We will describe processes in terms of the step-by-step operation of a traditional computer. Such a computer, or _register machine_, sequentially executes _instructions_ that manipulate the contents of a fixed set of storage elements called _registers_. A typical register-machine instruction applies a primitive operation to the contents of some registers and assigns the result to another register. Our descriptions of processes executed by register machines will look very much like "machine-language" programs for traditional computers. However, instead of focusing on the machine language of any particular computer, we will examine several JavaScriptfunctions and design a specific register machine to execute each function. Thus, we will approach our task from the perspective of a hardware architect rather than that of a machine-language computer programmer. In designing register machines, we will develop mechanisms for implementing important programming constructs such as recursion. We will also present a language for describing designs for register machines. In section [5.2](https://sourceacademy.org/sicpjs/5.2) we will implement a JavaScript program that uses these descriptions to simulate the machines we design.

[](https://sourceacademy.org/sicpjs/5#p3)

Most of the primitive operations of our register machines are very simple. For example, an operation might add the numbers fetched from two registers, producing a result to be stored into a third register. Such an operation can be performed by easily described hardware. In order to deal with list structure, however, we will also use the memory operations `head`,`tail`, and `pair`, which require an elaborate storage-allocation mechanism. In section [5.3](https://sourceacademy.org/sicpjs/5.3) we study their implementation in terms of more elementary operations.

[](https://sourceacademy.org/sicpjs/5#p4)

In section [5.4](https://sourceacademy.org/sicpjs/5.4), after we have accumulated experience formulating simple functions as register machines, we will design a machine that carries out the algorithm described by the metacircular evaluator of section [4.1](https://sourceacademy.org/sicpjs/4.1). This will fill in the gap in our understanding of how JavaScript programs are interpreted, by providing an explicit model for the mechanisms of control in the evaluator. In section [5.5](https://sourceacademy.org/sicpjs/5.5) we will study a simple compiler that translates JavaScript programs into sequences of instructions that can be executed directly with the registers and operations of the evaluator register machine.

---

[[1]](https://sourceacademy.org/sicpjs/5#footnote-link-1) With our metacircular evaluator, a recursive function always gives rise to a recursive process, even when the process should be iterative according to the distinction of section [1.2.1](https://sourceacademy.org/sicpjs/1.2.1). See footnote [2](https://sourceacademy.org/sicpjs/4.1.1#footnote-2) in section [4.1.1](https://sourceacademy.org/sicpjs/4.1.1).
# 5.1  Designing Register Machines

[](https://sourceacademy.org/sicpjs/5.1#p1)

To design a register machine, we must design its _data paths_ (registers and operations) and the _controller_ that sequences these operations. To illustrate the design of a simple register machine, let us examine Euclid's Algorithm, which is used to compute the greatest common divisor (GCD) of two integers. As we saw in section [1.2.5](https://sourceacademy.org/sicpjs/1.2.5), Euclid's Algorithm can be carried out by an iterative process, as specified by the following function:

```javascript
function gcd(a, b) {
    return b === 0 ? a : gcd(b, a % b);
} 
```

[](https://sourceacademy.org/sicpjs/5.1#p2)

A machine to carry out this algorithm must keep track of two numbers, aa and bb, so let us assume that these numbers are stored in two registers with those names. The basic operations required are testing whether the contents of register `b` is zero and computing the remainder of the contents of register `a` divided by the contents of register `b`. The remainder operation is a complex process, but assume for the moment that we have a primitive device that computes remainders. On each cycle of the GCD algorithm, the contents of register `a` must be replaced by the contents of register `b`, and the contents of `b` must be replaced by the remainder of the old contents of `a` divided by the old contents of `b`. It would be convenient if these replacements could be done simultaneously, but in our model of register machines we will assume that only one register can be assigned a new value at each step. To accomplish the replacements, our machine will use a third "temporary" register, which we call `t`. (First the remainder will be placed in `t`, then the contents of `b` will be placed in `a`, and finally the remainder stored in `t` will be placed in `b`.)

[](https://sourceacademy.org/sicpjs/5.1#p3)

We can illustrate the registers and operations required for this machine by using the data-path diagram shown in figure [5.1](https://sourceacademy.org/sicpjs/5.1#fig-5.1). In this diagram, the registers (`a`, `b`, and `t`) are represented by rectangles. Each way to assign a value to a register is indicated by an arrow with a button—drawn as ⊗⊗— behind the head, pointing from the source of data to the register. When pushed, the button allows the value at the source to "flow" into the designated register. The label next to each button is the name we will use to refer to the button. The names are arbitrary, and can be chosen to have mnemonic value (for example, `a<-b` denotes pushing the button that assigns the contents of register `b` to register `a`). The source of data for a register can be another register (as in the `a<-b` assignment), an operation result (as in the `t<-r` assignment), or a constant (a built-in value that cannot be changed, represented in a data-path diagram by a triangle containing the constant).

[](https://sourceacademy.org/sicpjs/5.1#p4)

An operation that computes a value from constants and the contents of registers is represented in a data-path diagram by a trapezoid containing a name for the operation. For example, the box marked `rem` in figure [5.1](https://sourceacademy.org/sicpjs/5.1#fig-5.1) represents an operation that computes the remainder of the contents of the registers `a` and `b` to which it is attached. Arrows (without buttons) point from the input registers and constants to the box, and arrows connect the operation's output value to registers. A test is represented by a circle containing a name for the test. For example, our GCD machine has an operation that tests whether the contents of register `b` is zero. A test also has arrows from its input registers and constants, but it has no output arrows; its value is used by the controller rather than by the data paths. Overall, the data-path diagram shows the registers and operations that are required for the machine and how they must be connected. If we view the arrows as wires and the ⊗⊗ buttons as switches, the data-path diagram is very like the wiring diagram for a machine that could be constructed from electrical components.

[](https://sourceacademy.org/sicpjs/5.1#fig-5.1)

![#fig-5.1](https://sicp.sourceacademy.org/img_original/Fig5.1a.std.svg)

##### Figure 5.1 Data paths for a GCD machine.

[](https://sourceacademy.org/sicpjs/5.1#p5)

In order for the data paths to actually compute GCDs, the buttons must be pushed in the correct sequence. We will describe this sequence in terms of a controller diagram, as illustrated in figure [5.2](https://sourceacademy.org/sicpjs/5.1#fig-5.2). The elements of the controller diagram indicate how the data-path components should be operated. The rectangular boxes in the controller diagram identify data-path buttons to be pushed, and the arrows describe the sequencing from one step to the next. The diamond in the diagram represents a decision. One of the two sequencing arrows will be followed, depending on the value of the data-path test identified in the diamond. We can interpret the controller in terms of a physical analogy: Think of the diagram as a maze in which a marble is rolling. When the marble rolls into a box, it pushes the data-path button that is named by the box. When the marble rolls into a decision node (such as the test for `b` =0=0), it leaves the node on the path determined by the result of the indicated test. Taken together, the data paths and the controller completely describe a machine for computing GCDs. We start the controller (the rolling marble) at the place marked `start`, after placing numbers in registers `a` and `b`. When the controller reaches `done`, we will find the value of the GCD in register `a`.

[](https://sourceacademy.org/sicpjs/5.1#fig-5.2)

![#fig-5.2](https://sicp.sourceacademy.org/img_original/Fig5.2.std.svg)

##### Figure 5.2 Controller for a GCD machine.

[](https://sourceacademy.org/sicpjs/5.1#ex-5.1)

**Exercise 5.1**

Design a register machine to compute factorials using the iterative algorithm specified by the following function. Draw data-path and controller diagrams for this machine.

```javascript
function factorial(n) {
    function iter(product, counter) {
        return counter > n 
               ? product
               : iter(counter * product,
                      counter + 1);
   }
   return iter(1, 1);
} 
```

Show Solution

# 5.1.1   A Language for Describing Register Machines

[](https://sourceacademy.org/sicpjs/5.1.1#p1)

Data-path and controller diagrams are adequate for representing simple machines such as GCD, but they are unwieldy for describing large machines such as a JavaScript interpreter. To make it possible to deal with complex machines, we will create a language that presents, in textual form, all the information given by the data-path and controller diagrams. We will start with a notation that directly mirrors the diagrams.

[](https://sourceacademy.org/sicpjs/5.1.1#p2)

We define the data paths of a machine by describing the registers and the operations. To describe a register, we give it a name and specify the buttons that control assignment to it. We give each of these buttons a name and specify the source of the data that enters the register under the button's control. (The source is a register, a constant, or an operation.) To describe an operation, we give it a name and specify its inputs (registers or constants).

[](https://sourceacademy.org/sicpjs/5.1.1#p3)

We define the controller of a machine as a sequence of _instructions_ together with _labels_ that identify _entry points_ in the sequence. An instruction is one of the following:

- The name of a data-path button to push to assign a value to a register. (This corresponds to a box in the controller diagram.)
- A `test` instruction, which performs a specified test.
- A conditional branch (`branch` instruction) to a location indicated by a controller label, based on the result of the previous test. (The test and branch together correspond to a diamond in the controller diagram.) If the test is false, the controller should continue with the next instruction in the sequence. Otherwise, the controller should continue with the instruction after the label.
- An unconditional branch (`go_to` instruction) naming a controller label at which to continue execution.

The machine starts at the beginning of the controller instruction sequence and stops when execution reaches the end of the sequence. Except when a branch changes the flow of control, instructions are executed in the order in which they are listed.

data_paths(
  registers(
    list(
      pair(name("a"),
           buttons(name("a<-b"), source(register("b")))),
      pair(name("b"),
           buttons(name("b<-t"), source(register("t")))),
      pair(name("t"),
           buttons(name("t<-r"), source(operation("rem")))))),
  operations(
    list(
      pair(name("rem"),
           inputs(register("a"), register("b"))),
      pair(name("="),
           inputs(register("b"), constant(0))))));

controller(
  list(
    "test_b",                     // label
      test("="),                  // test
      branch(label("gcd_done")),  // conditional branch
      "t<-r",                     // button push
      "a<-b",                     // button push
      "b<-t",                     // button push
      go_to(label("test_b")),     // unconditional branch
    "gcd_done"));                 // label

##### Figure 5.3 A specification of the GCD machine.

[](https://sourceacademy.org/sicpjs/5.1.1#p4)

Figure [5.3](https://sourceacademy.org/sicpjs/5.1.1#fig-5.3) shows the GCD machine described in this way. This example only hints at the generality of these descriptions, since the GCD machine is a very simple case: Each register has only one button, and each button and test is used only once in the controller.

[](https://sourceacademy.org/sicpjs/5.1.1#p5)

Unfortunately, it is difficult to read such a description. In order to understand the controller instructions we must constantly refer back to the definitions of the button names and the operation names, and to understand what the buttons do we may have to refer to the definitions of the operation names. We will thus transform our notation to combine the information from the data-path and controller descriptions so that we see it all together.

[](https://sourceacademy.org/sicpjs/5.1.1#p6)

To obtain this form of description, we will replace the arbitrary button and operation names by the definitions of their behavior. That is, instead of saying (in the controller) "Push button `t<-r`" and separately saying (in the data paths) "Button `t<-r` assigns the value of the `rem` operation to register `t`" and "The `rem` operation's inputs are the contents of registers `a` and `b`," we will say (in the controller) "Push the button that assigns to register `t` the value of the `rem` operation on the contents of registers `a` and `b`." Similarly, instead of saying (in the controller) "Perform the `=` test" and separately saying (in the data paths) "The `=` test operates on the contents of register `b` and the constant 0," we will say "Perform the `=` test on the contents of register `b` and the constant 0." We will omit the data-path description, leaving only the controller sequence. Thus, the GCD machine is described as follows:

```javascript
controller(
  list(
    "test_b",
      test(list(op("="), reg("b"), constant(0))),
      branch(label("gcd_done")),
      assign("t", list(op("rem"), reg("a"), reg("b"))),
      assign("a", reg("b")),
      assign("b", reg("t")),
      go_to(label("test_b")),
    "gcd_done")) 
```

[](https://sourceacademy.org/sicpjs/5.1.1#p7)

This form of description is easier to read than the kind illustrated in figure [5.3](https://sourceacademy.org/sicpjs/5.1.1#fig-5.3), but it also has disadvantages:

- It is more verbose for large machines, because complete descriptions of the data-path elements are repeated whenever the elements are mentioned in the controller instruction sequence. (This is not a problem in the GCD example, because each operation and button is used only once.) Moreover, repeating the data-path descriptions obscures the actual data-path structure of the machine; it is not obvious for a large machine how many registers, operations, and buttons there are and how they are interconnected.
- Because the controller instructions in a machine definition look like JavaScript expressions, it is easy to forget that they are not arbitrary JavaScript expressions. They can notate only legal machine operations. For example, operations can operate directly only on constants and the contents of registers, not on the results of other operations.

In spite of these disadvantages, we will use this register-machine language throughout this chapter, because we will be more concerned with understanding controllers than with understanding the elements and connections in data paths. We should keep in mind, however, that data-path design is crucial in designing real machines.

[](https://sourceacademy.org/sicpjs/5.1.1#ex-5.2)

**Exercise 5.2**

Use the register-machine language to describe the iterative factorial machine of exercise [5.1](https://sourceacademy.org/sicpjs/5.1#ex-5.1).

Show Solution

[](https://sourceacademy.org/sicpjs/5.1.1#h1)

## Actions

[](https://sourceacademy.org/sicpjs/5.1.1#p8)

Let us modify the GCD machine so that we can type in the numbers whose GCD we want and get the answer printed. We will not discuss how to make a machine that can read and print, but will assume (as we do when we use `prompt` and `display` in JavaScript) that they are available as primitive operations.[1](https://sourceacademy.org/sicpjs/5.1.1#footnote-1)

[](https://sourceacademy.org/sicpjs/5.1.1#p9)

The operation `prompt` is like the operations we have been using in that it produces a value that can be stored in a register. But `prompt` does not take inputs from any registers; its value depends on something that happens outside the parts of the machine we are designing. We will allow our machine's operations to have such behavior, and thus will draw and notate the use of `prompt` just as we do any other operation that computes a value.

[](https://sourceacademy.org/sicpjs/5.1.1#p10)

The operation `display`, on the other hand, differs from the operations we have been using in a fundamental way: It does not produce an output value to be stored in a register. Though it has an effect, this effect is not on a part of the machine we are designing. We will refer to this kind of operation as an _action_. We will represent an action in a data-path diagram just as we represent an operation that computes a value—as a trapezoid that contains the name of the action. Arrows point to the action box from any inputs (registers or constants). We also associate a button with the action. Pushing the button makes the action happen. To make a controller push an action button we use a new kind of instruction called `perform`. Thus, the action of printing the contents of register `a` is represented in a controller sequence by the instruction

perform(list(op("display"), reg("a")))

[](https://sourceacademy.org/sicpjs/5.1.1#p11)

Figure [5.4](https://sourceacademy.org/sicpjs/5.1.1#fig-5.4) shows the data paths and controller for the new GCD machine. Instead of having the machine stop after printing the answer, we have made it start over, so that it repeatedly reads a pair of numbers, computes their GCD, and prints the result. This structure is like the driver loops we used in the interpreters of chapter [4](https://sourceacademy.org/sicpjs/4).

[](https://sourceacademy.org/sicpjs/5.1.1#fig-5.4)

![#fig-5.4](https://sicp.sourceacademy.org/img_javascript/Fig5.4c.std.svg)

```javascript
controller(
  list(      
    "gcd_loop",
      assign("a", list(op("prompt"))),
      assign("b", list(op("prompt"))),
    "test_b",
      test(list(op("="), reg("b"), constant(0))),
      branch(label("gcd_done")),
      assign("t", list(op("rem"), reg("a"), reg("b"))),
      assign("a", reg("b")),
      assign("b", reg("t")),
      go_to(label("test_b")),
    "gcd_done",
      perform(list(op("display"), reg("a"))),
      go_to(label("gcd_loop")))) 
```

##### Figure 5.4 A GCD machine that reads inputs and prints results.

---

[[1]](https://sourceacademy.org/sicpjs/5.1.1#footnote-link-1) This assumption glosses over a great deal of complexity. Implementation of reading and printing requires significant effort, for example to handle character encodings for different languages.

# 5.1.2   Abstraction in Machine Design

[](https://sourceacademy.org/sicpjs/5.1.2#p1)

We will often define a machine to include "primitive" operations that are actually very complex. For example, in sections [5.4](https://sourceacademy.org/sicpjs/5.4) and [5.5](https://sourceacademy.org/sicpjs/5.5) we will treat JavaScript's environment manipulations as primitive. Such abstraction is valuable because it allows us to ignore the details of parts of a machine so that we can concentrate on other aspects of the design. The fact that we have swept a lot of complexity under the rug, however, does not mean that a machine design is unrealistic. We can always replace the complex "primitives" by simpler primitive operations.

[](https://sourceacademy.org/sicpjs/5.1.2#p2)

Consider the GCD machine. The machine has an instruction that computes the remainder of the contents of registers `a` and `b` and assigns the result to register `t`. If we want to construct the GCD machine without using a primitive remainder operation, we must specify how to compute remainders in terms of simpler operations, such as subtraction. Indeed, we can write a JavaScript function that finds remainders in this way:

```javascript
function remainder(n, d) {
    return n < d
           ? n
           : remainder(n - d, d);
} 
```

We can thus replace the remainder operation in the GCD machine's data paths with a subtraction operation and a comparison test. Figure [5.5](https://sourceacademy.org/sicpjs/5.1.2#fig-5.5) shows the data paths and controller for the elaborated machine. The instruction

[](https://sourceacademy.org/sicpjs/5.1.2#fig-5.5)

![#fig-5.5](https://sicp.sourceacademy.org/img_original/Fig5.5b.std.svg)

##### Figure 5.5 Data paths and controller for the elaborated GCD machine.

assign("t", list(op("rem"), reg("a"), reg("b")))

in the GCD controller definition is replaced by a sequence of instructions that contains a loop, as shown in figure [5.6](https://sourceacademy.org/sicpjs/5.1.2#fig-5.6).

```javascript
controller(
  list(
    "test_b",
      test(list(op("="), reg("b"), constant(0))),
      branch(label("gcd_done")),
      assign("t", reg("a")),
    "rem_loop",
      test(list(op("<"), reg("t"), reg("b"))),
      branch(label("rem_done")),
      assign("t", list(op("-"), reg("t"), reg("b"))),
      go_to(label("rem_loop")),
    "rem_done",
      assign("a", reg("b")),
      assign("b", reg("t")),
      go_to(label("test_b")),
    "gcd_done")) 
```

##### Figure 5.6 Controller instruction sequence for the GCD machine in figure [5.5](https://sourceacademy.org/sicpjs/5.1.2#fig-5.5).

[](https://sourceacademy.org/sicpjs/5.1.2#ex-5.3)

**Exercise 5.3**

Design a machine to compute square roots using Newton's method, as described in section [1.1.7](https://sourceacademy.org/sicpjs/1.1.7) and implemented with the following code in section [1.1.8](https://sourceacademy.org/sicpjs/1.1.8):

```javascript
function sqrt(x) {
    function is_good_enough(guess) {
        return math_abs(square(guess) - x) < 0.001;
    }
    function improve(guess) {
        return average(guess, x / guess);
    }
    function sqrt_iter(guess) {
        return is_good_enough(guess)
               ? guess
               : sqrt_iter(improve(guess));
    }
    return  sqrt_iter(1);
} 
```

Begin by assuming that `is_good_enough` and `improve` operations are available as primitives. Then show how to expand these in terms of arithmetic operations. Describe each version of the `sqrt` machine design by drawing a data-path diagram and writing a controller definition in the register-machine language.
# 5.1.3   Subroutines

[](https://sourceacademy.org/sicpjs/5.1.3#p1)

When designing a machine to perform a computation, we would often prefer to arrange for components to be shared by different parts of the computation rather than duplicate the components. Consider a machine that includes two GCD computations—one that finds the GCD of the contents of registers `a` and `b` and one that finds the GCD of the contents of registers `c` and `d`. We might start by assuming we have a primitive `gcd` operation, then expand the two instances of `gcd` in terms of more primitive operations. Figure [5.7](https://sourceacademy.org/sicpjs/5.1.3#fig-5.7) shows just the GCD portions of the resulting machine's data paths, without showing how they connect to the rest of the machine. The figure also shows the corresponding portions of the machine's controller sequence.

[](https://sourceacademy.org/sicpjs/5.1.3#fig-5.7)

![#fig-5.7](https://sicp.sourceacademy.org/img_javascript/Fig5.7b.std.svg)

##### Figure 5.7 Portions of the data paths and controller sequence for a machine with two GCD computations.

[](https://sourceacademy.org/sicpjs/5.1.3#p2)

This machine has two remainder operation boxes and two boxes for testing equality. If the duplicated components are complicated, as is the remainder box, this will not be an economical way to build the machine. We can avoid duplicating the data-path components by using the same components for both GCD computations, provided that doing so will not affect the rest of the larger machine's computation. If the values in registers `a` and `b` are not needed by the time the controller gets to `gcd_2` (or if these values can be moved to other registers for safekeeping), we can change the machine so that it uses registers `a` and `b`, rather than registers `c` and `d`, in computing the second GCD as well as the first. If we do this, we obtain the controller sequence shown in figure [5.8](https://sourceacademy.org/sicpjs/5.1.3#fig-5.8).

[](https://sourceacademy.org/sicpjs/5.1.3#p3)

We have removed the duplicate data-path components (so that the data paths are again as in figure [5.1](https://sourceacademy.org/sicpjs/5.1#fig-5.1)), but the controller now has two GCD sequences that differ only in their entry-point labels. It would be better to replace these two sequences by branches to a single sequence—a `gcd`_subroutine_—at the end of which we branch back to the correct place in the main instruction sequence. We can accomplish this as follows: Before branching to `gcd`, we place a distinguishing value (such as 0 or 1) into a special register, `continue`. At the end of the `gcd` subroutine we return either to `after_gcd_1` or to `after_gcd_2`, depending on the value of the `continue` register. Figure [5.9](https://sourceacademy.org/sicpjs/5.1.3#fig-5.9) shows the relevant portion of the resulting controller sequence, which includes only a single copy of the `gcd` instructions.

"gcd_1",
  test(list(op("="), reg("b"), constant(0))),
  branch(label("after_gcd_1")),
  assign("t", list(op("rem"), reg("a"), reg("b"))),
  assign("a", reg("b")),
  assign("b", reg("t")),
  go_to(label("gcd_1")),
"after_gcd_1",
  ⋮
"gcd_2",
  test(list(op("="), reg("b"), constant(0))),
  branch(label("after_gcd_2")),
  assign("t", list(op("rem"), reg("a"), reg("b"))),
  assign("a", reg("b")),
  assign("b", reg("t")),
  go_to(label("gcd_2")),
"after_gcd_2"
	

##### Figure 5.8 Portions of the controller sequence for a machine that uses the same data-path components for two different GCD computations.

"gcd",
  test(list(op("="), reg("b"), constant(0))),
  branch(label("gcd_done")),
  assign("t", list(op("rem"), reg("a"), reg("b"))),
  assign("a", reg("b")),
  assign("b", reg("t")),
  go_to(label("gcd")),
"gcd_done",
  test(list(op("="), reg("continue"), constant(0))),
  branch(label("after_gcd_1")),
  go_to(label("after_gcd_2")),
  ⋮
  // Before branching to gcdgcd from the first place where
  // it is needed, we place 0 in the continuecontinue register
  assign("continue", constant(0)),
  go_to(label("gcd")),
"after_gcd_1",
  ⋮
  // Before the second use of gcdgcd, we place 1 in the continuecontinue register
  assign("continue", constant(1)),
  go_to(label("gcd")),
"after_gcd_2"
	

##### Figure 5.9 Using a `continue` register to avoid the duplicate controller sequence in figure [5.8](https://sourceacademy.org/sicpjs/5.1.3#fig-5.8).

[](https://sourceacademy.org/sicpjs/5.1.3#p4)

This is a reasonable approach for handling small problems, but it would be awkward if there were many instances of GCD computations in the controller sequence. To decide where to continue executing after the `gcd` subroutine, we would need tests in the data paths and branch instructions in the controller for all the places that use `gcd`. A more powerful method for implementing subroutines is to have the `continue` register hold the label of the entry point in the controller sequence at which execution should continue when the subroutine is finished. Implementing this strategy requires a new kind of connection between the data paths and the controller of a register machine: There must be a way to assign to a register a label in the controller sequence in such a way that this value can be fetched from the register and used to continue execution at the designated entry point.

[](https://sourceacademy.org/sicpjs/5.1.3#p5)

To reflect this ability, we will extend the `assign` instruction of the register-machine language to allow a register to be assigned as value a label from the controller sequence (as a special kind of constant). We will also extend the `go_to` instruction to allow execution to continue at the entry point described by the contents of a register rather than only at an entry point described by a constant label. Using these new constructs we can terminate the `gcd` subroutine with a branch to the location stored in the `continue` register. This leads to the controller sequence shown in figure [5.10](https://sourceacademy.org/sicpjs/5.1.3#fig-5.10).

"gcd",
  test(list(op("="), reg("b"), constant(0))),
  branch(label("gcd_done")),
  assign("t", list(op("rem"), reg("a"), reg("b"))),
  assign("a", reg("b")),
  assign("b", reg("t")),
  go_to(label("gcd")),
"gcd_done",
  go_to(reg("continue")),
  ⋮
  // Before calling gcdgcd, we assign to continuecontinue
  // the label to which gcdgcd should return.
  assign("continue", label("after_gcd_1"))),
  go_to(label("gcd")),
"after_gcd_1",
  ⋮
  // Here is the second call to gcdgcd, with a different continuation.
  assign("continue", label("after_gcd_2")),
  go_to(label("gcd")),
"after_gcd_2"
	

##### Figure 5.10 Assigning labels to the `continue` register simplifies and generalizes the strategy shown in figure [5.9](https://sourceacademy.org/sicpjs/5.1.3#fig-5.9).

[](https://sourceacademy.org/sicpjs/5.1.3#p6)

A machine with more than one subroutine could use multiple continuation registers (e.g., `gcd_continue`, `factorial_continue`) or we could have all subroutines share a single `continue` register. Sharing is more economical, but we must be careful if we have a subroutine (`sub1`) that calls another subroutine (`sub2`). Unless `sub1` saves the contents of `continue` in some other register before setting up `continue` for the call to `sub2`, `sub1` will not know where to go when it is finished. The mechanism developed in the next section to handle recursion also provides a better solution to this problem of nested subroutine calls.
# 5.1.4   Using a Stack to Implement Recursion

[](https://sourceacademy.org/sicpjs/5.1.4#p1)

With the ideas illustrated so far, we can implement any iterative process by specifying a register machine that has a register corresponding to each state variable of the process. The machine repeatedly executes a controller loop, changing the contents of the registers, until some termination condition is satisfied. At each point in the controller sequence, the state of the machine (representing the state of the iterative process) is completely determined by the contents of the registers (the values of the state variables).

[](https://sourceacademy.org/sicpjs/5.1.4#p2)

Implementing recursive processes, however, requires an additional mechanism. Consider the following recursive method for computing factorials, which we first examined in section [1.2.1](https://sourceacademy.org/sicpjs/1.2.1):

```javascript
function factorial(n) {
    return n === 1 
           ? 1
           : n * factorial(n - 1);
} 
```

As we see from the function, computing n!n! requires computing (n−1)!(n−1)!. Our GCD machine, modeled on the function

```javascript
function gcd(a, b) {
    return b === 0 ? a : gcd(b, a % b);
} 
```

similarly had to compute another GCD. But there is an important difference between the `gcd`function, which reduces the original computation to a new GCD computation, and `factorial`, which requires computing another factorial as a subproblem. In GCD, the answer to the new GCD computation is the answer to the original problem. To compute the next GCD, we simply place the new arguments in the input registers of the GCD machine and reuse the machine's data paths by executing the same controller sequence. When the machine is finished solving the final GCD problem, it has completed the entire computation.

[](https://sourceacademy.org/sicpjs/5.1.4#p3)

In the case of factorial (or any recursive process) the answer to the new factorial subproblem is not the answer to the original problem. The value obtained for (n−1)!(n−1)! must be multiplied by nn to get the final answer. If we try to imitate the GCD design, and solve the factorial subproblem by decrementing the `n` register and rerunning the factorial machine, we will no longer have available the old value of `n` by which to multiply the result. We thus need a second factorial machine to work on the subproblem. This second factorial computation itself has a factorial subproblem, which requires a third factorial machine, and so on. Since each factorial machine contains another factorial machine within it, the total machine contains an infinite nest of similar machines and hence cannot be constructed from a fixed, finite number of parts.

[](https://sourceacademy.org/sicpjs/5.1.4#p4)

Nevertheless, we can implement the factorial process as a register machine if we can arrange to use the same components for each nested instance of the machine. Specifically, the machine that computes n!n! should use the same components to work on the subproblem of computing (n−1)!(n−1)!, on the subproblem for (n−2)!(n−2)!, and so on. This is plausible because, although the factorial process dictates that an unbounded number of copies of the same machine are needed to perform a computation, only one of these copies needs to be active at any given time. When the machine encounters a recursive subproblem, it can suspend work on the main problem, reuse the same physical parts to work on the subproblem, then continue the suspended computation.

[](https://sourceacademy.org/sicpjs/5.1.4#p5)

In the subproblem, the contents of the registers will be different than they were in the main problem. (In this case the `n` register is decremented.) In order to be able to continue the suspended computation, the machine must save the contents of any registers that will be needed after the subproblem is solved so that these can be restored to continue the suspended computation. In the case of factorial, we will save the old value of `n`, to be restored when we are finished computing the factorial of the decremented `n` register.[1](https://sourceacademy.org/sicpjs/5.1.4#footnote-1)

[](https://sourceacademy.org/sicpjs/5.1.4#p6)

Since there is no a priori limit on the depth of nested recursive calls, we may need to save an arbitrary number of register values. These values must be restored in the reverse of the order in which they were saved, since in a nest of recursions the last subproblem to be entered is the first to be finished. This dictates the use of a _stack_, or "last in, first out" data structure, to save register values. We can extend the register-machine language to include a stack by adding two kinds of instructions: Values are placed on the stack using a `save` instruction and restored from the stack using a `restore` instruction. After a sequence of values has been `save`d on the stack, a sequence of `restore`s will retrieve these values in reverse order.[2](https://sourceacademy.org/sicpjs/5.1.4#footnote-2)

[](https://sourceacademy.org/sicpjs/5.1.4#p7)

With the aid of the stack, we can reuse a single copy of the factorial machine's data paths for each factorial subproblem. There is a similar design issue in reusing the controller sequence that operates the data paths. To reexecute the factorial computation, the controller cannot simply loop back to the beginning, as with an iterative process, because after solving the (n−1)!(n−1)! subproblem the machine must still multiply the result by nn. The controller must suspend its computation of n!n!, solve the (n−1)!(n−1)! subproblem, then continue its computation of n!n!. This view of the factorial computation suggests the use of the subroutine mechanism described in section [5.1.3](https://sourceacademy.org/sicpjs/5.1.3), which has the controller use a `continue` register to transfer to the part of the sequence that solves a subproblem and then continue where it left off on the main problem. We can thus make a factorial subroutine that returns to the entry point stored in the `continue` register. Around each subroutine call, we save and restore `continue` just as we do the `n` register, since each "level" of the factorial computation will use the same `continue` register. That is, the factorial subroutine must put a new value in `continue` when it calls itself for a subproblem, but it will need the old value in order to return to the place that called it to solve a subproblem.

[](https://sourceacademy.org/sicpjs/5.1.4#fig-5.11)

![#fig-5.11](https://sicp.sourceacademy.org/img_javascript/Fig5.11b.std.svg)

controller(
  list(      
      assign("continue", label("fact_done")), // set up final return address
    "fact_loop",
      test(list(op("="), reg("n"), constant(1))),
      branch(label("base_case")),
      // Set up for recursive call by saving nn and continuecontinue.
      // Set up continuecontinue so that the computation will continue
      // at after_factafter_fact when the subroutine returns.
      save("continue"),
      save("n"),
      assign("n", list(op("-"), reg("n"), constant(1))),
      assign("continue", label("after_fact")),
      go_to(label("fact_loop")),
    "after_fact",
      restore("n"),
      restore("continue"),
      assign("val",                   // valval now contains n(n−1)!n(n−1)!
             list(op("*"), reg("n"), reg("val"))),  
      go_to(reg("continue")),         // return to caller
    "base_case",
      assign("val", constant(1)),     // base case: 1! = 1
      go_to(reg("continue")),         // return to caller
    "fact_done"))
            

##### Figure 5.11 A recursive factorial machine.

[](https://sourceacademy.org/sicpjs/5.1.4#p8)

Figure [5.11](https://sourceacademy.org/sicpjs/5.1.4#fig-5.11) shows the data paths and controller for a machine that implements the recursive `factorial`function. The machine has a stack and three registers, called `n`, `val`, and `continue`. To simplify the data-path diagram, we have not named the register-assignment buttons, only the stack-operation buttons (`sc` and `sn` to save registers, `rc` and `rn` to restore registers). To operate the machine, we put in register `n` the number whose factorial we wish to compute and start the machine. When the machine reaches `fact_done`, the computation is finished and the answer will be found in the `val` register. In the controller sequence, `n` and `continue` are saved before each recursive call and restored upon return from the call. Returning from a call is accomplished by branching to the location stored in `continue`. The register `continue` is initialized when the machine starts so that the last return will go to `fact_done`. The `val` register, which holds the result of the factorial computation, is not saved before the recursive call, because the old contents of `val` is not useful after the subroutine returns. Only the new value, which is the value produced by the subcomputation, is needed.

[](https://sourceacademy.org/sicpjs/5.1.4#p9)

Although in principle the factorial computation requires an infinite machine, the machine in figure [5.11](https://sourceacademy.org/sicpjs/5.1.4#fig-5.11) is actually finite except for the stack, which is potentially unbounded. Any particular physical implementation of a stack, however, will be of finite size, and this will limit the depth of recursive calls that can be handled by the machine. This implementation of factorial illustrates the general strategy for realizing recursive algorithms as ordinary register machines augmented by stacks. When a recursive subproblem is encountered, we save on the stack the registers whose current values will be required after the subproblem is solved, solve the recursive subproblem, then restore the saved registers and continue execution on the main problem. The `continue` register must always be saved. Whether there are other registers that need to be saved depends on the particular machine, since not all recursive computations need the original values of registers that are modified during solution of the subproblem (see exercise [5.4](https://sourceacademy.org/sicpjs/5.1.4#ex-5.4)).

[](https://sourceacademy.org/sicpjs/5.1.4#h1)

## A double recursion

[](https://sourceacademy.org/sicpjs/5.1.4#p10)

Let us examine a more complex recursive process, the tree-recursive computation of the Fibonacci numbers, which we introduced in section [1.2.2](https://sourceacademy.org/sicpjs/1.2.2):

```javascript
function fib(n) {
    return n === 0
           ? 0
           : n === 1
           ? 1
           : fib(n - 1) + fib(n - 2);
} 
```

Just as with factorial, we can implement the recursive Fibonacci computation as a register machine with registers `n`, `val`, and `continue`. The machine is more complex than the one for factorial, because there are two places in the controller sequence where we need to perform recursive calls—once to compute Fib(n−1)(n−1) and once to compute Fib(n−2)(n−2). To set up for each of these calls, we save the registers whose values will be needed later, set the `n` register to the number whose Fib we need to compute recursively (n−1n−1 or n−2n−2), and assign to `continue` the entry point in the main sequence to which to return (`afterfib_n_1` or `afterfib_n_2`, respectively). We then go to `fib_loop`. When we return from the recursive call, the answer is in `val`. Figure [5.12](https://sourceacademy.org/sicpjs/5.1.4#fig-5.12) shows the controller sequence for this machine.

controller(
  list(      
    assign("continue", label("fib_done")),
  "fib_loop",
    test(list(op("<"), reg("n"), constant(2))),
    branch(label("immediate_answer")),
    // set up to compute Fib(n−1)Fib(n−1)
    save("continue"),
    assign("continue", label("afterfib_n_1")),
    save("n"),                     // save old value of nn
    assign("n", list(op("-"), reg("n"), constant(1))), // clobber nn to n−1n−1
    go_to(label("fib_loop")),      // perform recursive call
  "afterfib_n_1",                  // upon return, valval contains Fib(n−1)Fib(n−1)
    restore("n"),
    restore("continue"),
    // set up to compute Fib(n−2)Fib(n−2)
    assign("n", list(op("-"), reg("n"), constant(2))),
    save("continue"),
    assign("continue", label("afterfib_n_2")),
    save("val"),                   // save Fib(n−1)Fib(n−1)
    go_to(label("fib_loop")),
  "afterfib_n_2",                  // upon return, valval contains Fib(n−2)Fib(n−2)
    assign("n", reg("val")),       // nn now contains Fib(n−2)Fib(n−2)
    restore("val"),                // valval now contains Fib(n−1)Fib(n−1)
    restore("continue"),
    assign("val",                  // Fib(n−1)+Fib(n−2)Fib(n−1)+Fib(n−2)
      list(op("+"), reg("val"), reg("n"))),
    go_to(reg("continue")),        // return to caller, answer in valval
  "immediate_answer",
    assign("val", reg("n")),       // base case: Fib(n)=nFib(n)=n
    go_to(reg("continue")),
  "fib_done"))
	

##### Figure 5.12 Controller for a machine to compute Fibonacci numbers.

[](https://sourceacademy.org/sicpjs/5.1.4#ex-5.4)

**Exercise 5.4**

Specify register machines that implement each of the following functions. For each machine, write a controller instruction sequence and draw a diagram showing the data paths.

1. Recursive exponentiation:
    
    ```javascript
    function expt(b, n) {
        return n === 0
               ? 1
               : b * expt(b, n - 1);
    } 
    ```
    
2. Iterative exponentiation:
    
    ```javascript
    function expt(b, n) {	  
        function expt_iter(counter, product) {
            return counter === 0
                   ? product
                   : expt_iter(counter - 1, b * product);
        }
        return expt_iter(n, 1);
    } 
    ```
    

Show Solution

[](https://sourceacademy.org/sicpjs/5.1.4#ex-5.5)

**Exercise 5.5**

Hand-simulate the factorial and Fibonacci machines, using some nontrivial input (requiring execution of at least one recursive call). Show the contents of the stack at each significant point in the execution.

Show Solution

[](https://sourceacademy.org/sicpjs/5.1.4#ex-5.6)

**Exercise 5.6**

Ben Bitdiddle observes that the Fibonacci machine's controller sequence has an extra `save` and an extra `restore`, which can be removed to make a faster machine. Where are these instructions?

Show Solution

---

[[1]](https://sourceacademy.org/sicpjs/5.1.4#footnote-link-1) One might argue that we don't need to save the old `n`; after we decrement it and solve the subproblem, we could simply increment it to recover the old value. Although this strategy works for factorial, it cannot work in general, since the old value of a register cannot always be computed from the new one.

[[2]](https://sourceacademy.org/sicpjs/5.1.4#footnote-link-2) In section [5.3](https://sourceacademy.org/sicpjs/5.3) we will see how to implement a stack in terms of more primitive operations.
# 5.1.5   Instruction Summary

[](https://sourceacademy.org/sicpjs/5.1.5#p1)

A controller instruction in our register-machine language has one of the following forms, where each _input_ii​ is `reg(`_register-name_`)` or `constant(`_constant-value_`)`.

[](https://sourceacademy.org/sicpjs/5.1.5#p2)

These instructions were introduced in section [5.1.1](https://sourceacademy.org/sicpjs/5.1.1):

assign(registerregister-namename, reg(registerregister-namename))

assign(registerregister-namename, constant(constantconstant-valuevalue))

assign(registerregister-namename, list(op(operationoperation-namename), inputinput11​, ……, inputinputnn​))

perform(list(op(operationoperation-namename), inputinput11​, ……, inputinputnn​))

test(list(op(operationoperation-namename), inputinput11​, ……, inputinputnn​))

branch(label(labellabel-namename))

go_to(label(labellabel-namename))
      

[](https://sourceacademy.org/sicpjs/5.1.5#p3)

The use of registers to hold labels was introduced in section [5.1.3](https://sourceacademy.org/sicpjs/5.1.3):

assign(registerregister-namename, label(labellabel-namename))

go_to(reg(registerregister-namename))
      

[](https://sourceacademy.org/sicpjs/5.1.5#p4)

Instructions to use the stack were introduced in section [5.1.4](https://sourceacademy.org/sicpjs/5.1.4):

save(registerregister-namename)

restore(registerregister-namename)
      

[](https://sourceacademy.org/sicpjs/5.1.5#p5)

The only kind of _constant-value_ we have seen so far is a number, but later we will also use strings and lists. For example, `constant("abc")` is the string `"abc"`, `constant(null)` is the empty list, and `constant(list("a", "b", "c"))` is the list `list("a", "b", "c")`.
# 5.2  A Register-Machine Simulator

[](https://sourceacademy.org/sicpjs/5.2#p1)

In order to gain a good understanding of the design of register machines, we must test the machines we design to see if they perform as expected. One way to test a design is to hand-simulate the operation of the controller, as in exercise [5.5](https://sourceacademy.org/sicpjs/5.1.4#ex-5.5). But this is extremely tedious for all but the simplest machines. In this section we construct a simulator for machines described in the register-machine language. The simulator is a JavaScript program with four interface functions. The first uses a description of a register machine to construct a model of the machine (a data structure whose parts correspond to the parts of the machine to be simulated), and the other three allow us to simulate the machine by manipulating the model:

- `make_machine(`_register-names_`,` _operations_`,` _controller_`)`  
    constructs and returns a model of the machine with the given registers, operations, and controller.
- `set_register_contents(`_machine-model_`,` _register-name_`,` _value_`)`  
    stores a value in a simulated register in the given machine.
- `get_register_contents(`_machine-model_`,` _register-name_`)`  
    returns the contents of a simulated register in the given machine.
- `start(`_machine-model_`)`  
    simulates the execution of the given machine, starting from the beginning of the controller sequence and stopping when it reaches the end of the sequence.

[](https://sourceacademy.org/sicpjs/5.2#p2)

As an example of how these functions are used, we can define `gcd_machine` to be a model of the GCD machine of section [5.1.1](https://sourceacademy.org/sicpjs/5.1.1) as follows:

```javascript
const gcd_machine =
    make_machine(
        list("a", "b", "t"),
        list(list("rem", (a, b) => a % b),
             list("=", (a, b) => a === b)),
        list(
          "test_b",
            test(list(op("="), reg("b"), constant(0))),
            branch(label("gcd_done")),
            assign("t", list(op("rem"), reg("a"), reg("b"))),
            assign("a", reg("b")),
            assign("b", reg("t")),
            go_to(label("test_b")),
          "gcd_done")); 
```

The first argument to `make_machine` is a list of register names. The next argument is a table (a list of two-element lists) that pairs each operation name with a JavaScript function that implements the operation (that is, produces the same output value given the same input values). The last argument specifies the controller as a list of labels and machine instructions, as in section [5.1](https://sourceacademy.org/sicpjs/5.1).

[](https://sourceacademy.org/sicpjs/5.2#p3)

To compute GCDs with this machine, we set the input registers, start the machine, and examine the result when the simulation terminates:

```javascript
set_register_contents(gcd_machine, "a", 206); 
```

_"done"_

```javascript
set_register_contents(gcd_machine, "b", 40); 
```

_"done"_

```javascript
start(gcd_machine); 
```

_"done"_

```javascript
get_register_contents(gcd_machine, "a"); 
```

_2_

This computation will run much more slowly than a `gcd`function written in JavaScript, because we will simulate low-level machine instructions, such as `assign`, by much more complex operations.

[](https://sourceacademy.org/sicpjs/5.2#ex-5.7)

**Exercise 5.7**

Use the simulator to test the machines you designed in exercise [5.4](https://sourceacademy.org/sicpjs/5.1.4#ex-5.4).
# 5.2.1   The Machine Model

[](https://sourceacademy.org/sicpjs/5.2.1#p1)

The machine model generated by `make_machine` is represented as a function with local state using the message-passing techniques developed in chapter [3](https://sourceacademy.org/sicpjs/3). To build this model, `make_machine` begins by calling the function`make_new_machine` to construct the parts of the machine model that are common to all register machines. This basic machine model constructed by `make_new_machine` is essentially a container for some registers and a stack, together with an execution mechanism that processes the controller instructions one by one.

[](https://sourceacademy.org/sicpjs/5.2.1#p2)

The function `make_machine` then extends this basic model (by sending it messages) to include the registers, operations, and controller of the particular machine being defined. First it allocates a register in the new machine for each of the supplied register names and installs the designated operations in the machine. Then it uses an _assembler_ (described below in section [5.2.2](https://sourceacademy.org/sicpjs/5.2.2)) to transform the controller list into instructions for the new machine and installs these as the machine's instruction sequence. The function `make_machine` returns as its value the modified machine model.

```javascript
function make_machine(register_names, ops, controller) {
    const machine = make_new_machine();
    for_each(register_name => 
               machine("allocate_register")(register_name), 
             register_names);
    machine("install_operations")(ops);
    machine("install_instruction_sequence")
           (assemble(controller, machine));
    return machine;
} 
```

[](https://sourceacademy.org/sicpjs/5.2.1#h1)

## Registers

[](https://sourceacademy.org/sicpjs/5.2.1#p3)

We will represent a register as a function with local state, as in chapter [3](https://sourceacademy.org/sicpjs/3). The function`make_register` creates a register that holds a value that can be accessed or changed:

```javascript
function make_register(name) {
    let contents = "*unassigned*";
    function dispatch(message) {
        return message === "get"
               ? contents
               : message === "set"
               ? value => { contents = value; }
               : error(message, "unknown request -- make_register");
    }
    return dispatch;
} 
```

The following functions are used to access registers:

```javascript
function get_contents(register) {
    return register("get");
}
function set_contents(register, value) {
    return register("set")(value);
} 
```

[](https://sourceacademy.org/sicpjs/5.2.1#h2)

## The stack

[](https://sourceacademy.org/sicpjs/5.2.1#p4)

We can also represent a stack as a function with local state. The function`make_stack` creates a stack whose local state consists of a list of the items on the stack. A stack accepts requests to `push` an item onto the stack, to `pop` the top item off the stack and return it, and to `initialize` the stack to empty.

```javascript
function make_stack() {
    let stack = null;
    function push(x) { 
        stack = pair(x, stack); 
        return "done";
    }
    function pop() {
        if (is_null(stack)) {
            error("empty stack -- pop");
        } else {
            const top = head(stack);
            stack = tail(stack);
            return top;
        }
    }
    function initialize() {
        stack = null;
        return "done";
    }
    function dispatch(message) {
        return message === "push"
               ? push
               : message === "pop"
               ? pop()
               : message === "initialize"
               ? initialize()
               : error(message, "unknown request -- stack");
    }
    return dispatch;
} 
```

The following functions are used to access stacks:

```javascript
function pop(stack) {
    return stack("pop");
}
function push(stack, value) {
    return stack("push")(value);
} 
```

[](https://sourceacademy.org/sicpjs/5.2.1#h3)

## The basic machine

[](https://sourceacademy.org/sicpjs/5.2.1#p5)

The `make_new_machine`function, shown in figure [5.13](https://sourceacademy.org/sicpjs/5.2.1#fig-5.13), constructs an object whose local state consists of a stack, an initially empty instruction sequence, a list of operations that initially contains an operation to initialize the stack, and a _register table_ that initially contains two registers, named `flag` and `pc` (for "program counter"). The internal function`allocate_register` adds new entries to the register table, and the internal function`lookup_register` looks up registers in the table.

```javascript
function make_new_machine() {
    const pc = make_register("pc");
    const flag = make_register("flag");
    const stack = make_stack();
    let the_instruction_sequence = null;
    let the_ops = list(list("initialize_stack", () => stack("initialize")));
    let register_table = list(list("pc", pc), list("flag", flag));
    function allocate_register(name) {
        if (is_undefined(assoc(name, register_table))) {
            register_table = pair(list(name, make_register(name)),
                                  register_table);
        } else {
            error(name, "multiply defined register");
        }
        return "register allocated";
    }
    function lookup_register(name) {
        const val = assoc(name, register_table);
        return is_undefined(val)
               ? error(name, "unknown register")
               : head(tail(val));
    }
    function execute() {
        const insts = get_contents(pc);
        if (is_null(insts)) {
            return "done";
        } else {
            inst_execution_fun(head(insts))();
            return execute();
        }
    }
    function dispatch(message) {
        function start() {
            set_contents(pc, the_instruction_sequence);
            return execute();
        }
        return message === "start"
               ? start()
               : message === "install_instruction_sequence"
               ? seq => { the_instruction_sequence = seq; }
               : message === "allocate_register"
               ? allocate_register
               : message === "get_register"
               ? lookup_register
               : message === "install_operations"
               ? ops => { the_ops = append(the_ops, ops); }
               : message === "stack"
               ? stack
               : message === "operations"
               ? the_ops
               : error(message, "unknown request -- machine");
    }
    return dispatch;
} 
```

##### Figure 5.13 The `make_new_machine` function implements the basic machine model.

[](https://sourceacademy.org/sicpjs/5.2.1#p6)

The `flag` register is used to control branching in the simulated machine. Our `test` instructions set the contents of `flag` to the result of the test (true or false). Our `branch` instructions decide whether or not to branch by examining the contents of `flag`.

[](https://sourceacademy.org/sicpjs/5.2.1#p7)

The `pc` register determines the sequencing of instructions as the machine runs. This sequencing is implemented by the internal function`execute`. In the simulation model, each machine instruction is a data structure that includes a function of no arguments, called the _instruction execution function_, such that calling this function simulates executing the instruction. As the simulation runs, `pc` points to the place in the instruction sequence beginning with the next instruction to be executed. The function `execute` gets that instruction, executes it by calling the instruction execution function, and repeats this cycle until there are no more instructions to execute (i.e., until `pc` points to the end of the instruction sequence).

[](https://sourceacademy.org/sicpjs/5.2.1#p8)

As part of its operation, each instruction execution function modifies `pc` to indicate the next instruction to be executed. The instructions `branch` and `go_to` change `pc` to point to the new destination. All other instructions simply advance `pc`, making it point to the next instruction in the sequence. Observe that each call to `execute` calls `execute` again, but this does not produce an infinite loop because running the instruction execution function changes the contents of `pc`.

[](https://sourceacademy.org/sicpjs/5.2.1#p9)

The function `make_new_machine` returns a dispatchfunction that implements message-passing access to the internal state. Notice that starting the machine is accomplished by setting `pc` to the beginning of the instruction sequence and calling `execute`.

[](https://sourceacademy.org/sicpjs/5.2.1#p10)

For convenience, we provide an alternate interface to a machine's `start` operation, as well as functions to set and examine register contents, as specified at the beginning of section [5.2](https://sourceacademy.org/sicpjs/5.2):

```javascript
function start(machine) {
    return machine("start");
}
function get_register_contents(machine, register_name) {
    return get_contents(get_register(machine, register_name));
}
function set_register_contents(machine, register_name, value) {
    set_contents(get_register(machine, register_name), value);
    return "done";
} 
```

These functions (and many functions in sections [5.2.2](https://sourceacademy.org/sicpjs/5.2.2) and [5.2.3](https://sourceacademy.org/sicpjs/5.2.3)) use the following to look up the register with a given name in a given machine:

```javascript
function get_register(machine, reg_name) {
    return machine("get_register")(reg_name);
} 
```
# 5.2.2   The Assembler

The assembler transforms the sequence of controller instructions for a machine into a corresponding list of machine instructions, each with its execution function. Overall, the assembler is much like the evaluators we studied in chapter [4](https://sourceacademy.org/sicpjs/4)—there is an input language (in this case, the register-machine language) and we must perform an appropriate action for each type of component in the language.

[](https://sourceacademy.org/sicpjs/5.2.2#p1)

The technique of producing an execution function for each instruction is just what we used in section [4.1.7](https://sourceacademy.org/sicpjs/4.1.7) to speed up the evaluator by separating analysis from runtime execution. As we saw in chapter [4](https://sourceacademy.org/sicpjs/4), much useful analysis of JavaScript expressions could be performed without knowing the actual values of names. Here, analogously, much useful analysis of register-machine-language expressions can be performed without knowing the actual contents of machine registers. For example, we can replace references to registers by pointers to the register objects, and we can replace references to labels by pointers to the place in the instruction sequence that the label designates.

[](https://sourceacademy.org/sicpjs/5.2.2#p2)

Before it can generate the instruction execution functions, the assembler must know what all the labels refer to, so it begins by scanning the controller sequence to separate the labels from the instructions. As it scans the controller, it constructs both a list of instructions and a table that associates each label with a pointer into that list. Then the assembler augments the instruction list by inserting the execution function for each instruction.

[](https://sourceacademy.org/sicpjs/5.2.2#p3)

The `assemble`function is the main entry to the assembler. It takes the controller sequence and the machine model as arguments and returns the instruction sequence to be stored in the model. The function `assemble` calls `extract_labels` to build the initial instruction list and label table from the supplied controller. The second argument to `extract_labels` is a function to be called to process these results: This function uses `update_insts` to generate the instruction execution functions and insert them into the instruction list, and returns the modified list.

```javascript
function assemble(controller, machine) {
    return extract_labels(controller,
                          (insts, labels) => {
                              update_insts(insts, labels, machine);
                              return insts;
                          });
} 
```

[](https://sourceacademy.org/sicpjs/5.2.2#p4)

The function `extract_labels` takes a list `controller` and a function `receive` as arguments. The function `receive` will be called with two values: (1) a list `insts` of instruction data structures, each containing an instruction from `controller`; and (2) a table called `labels`, which associates each label from `controller` with the position in the list `insts` that the label designates.

```javascript
function extract_labels(controller, receive) {
    return is_null(controller)
           ? receive(null, null)
           : extract_labels(
                 tail(controller),
                 (insts, labels) => {
                   const next_element = head(controller);
                   return is_string(next_element)
                          ? receive(insts,
                                    pair(make_label_entry(next_element,
                                                          insts),
                                         labels))
                          : receive(pair(make_inst(next_element),
                                         insts),
                                    labels);
                 });
} 
```

The function `extract_labels` works by sequentially scanning the elements of the `controller` and accumulating the `insts` and the `labels`. If an element is a string (and thus a label) an appropriate entry is added to the `labels` table. Otherwise the element is accumulated onto the `insts` list.[1](https://sourceacademy.org/sicpjs/5.2.2#footnote-1)

[](https://sourceacademy.org/sicpjs/5.2.2#p5)

The function `update_insts` modifies the instruction list, which initially contains only the controller instructions, to include the corresponding execution functions:

```javascript
function update_insts(insts, labels, machine) {
    const pc = get_register(machine, "pc");
    const flag = get_register(machine, "flag");
    const stack = machine("stack");
    const ops = machine("operations");
    return for_each(inst => set_inst_execution_fun(
                                inst,
                                make_execution_function(
                                    inst_controller_instruction(inst),
                                    labels, machine, pc,
                                    flag, stack, ops)),
                    insts);
} 
```

[](https://sourceacademy.org/sicpjs/5.2.2#p6)

The machine instruction data structure simply pairs the controller instruction with the corresponding execution function. The execution function is not yet available when `extract_labels` constructs the instruction, and is inserted later by `update_insts`.

```javascript
function make_inst(inst_controller_instruction) {
    return pair(inst_controller_instruction, null);
}
function inst_controller_instruction(inst) {
    return head(inst);
}
function inst_execution_fun(inst) {
    return tail(inst);
}
function set_inst_execution_fun(inst, fun) {
    set_tail(inst, fun);
} 
```

The controller instruction is not used by our simulator, but is handy to keep around for debugging (see exercise [5.15](https://sourceacademy.org/sicpjs/5.2.4#ex-5.15)).

[](https://sourceacademy.org/sicpjs/5.2.2#p7)

Elements of the label table are pairs:

```javascript
function make_label_entry(label_name, insts) {
    return pair(label_name, insts);
} 
```

Entries will be looked up in the table with

```javascript
function lookup_label(labels, label_name) {
    const val = assoc(label_name, labels);
    return is_undefined(val)
           ? error(label_name, "undefined label -- assemble")
           : tail(val);
} 
```

[](https://sourceacademy.org/sicpjs/5.2.2#ex-5.8)

**Exercise 5.8**

The following register-machine code is ambiguous, because the label `here` is defined more than once:

"start",
  go_to(label("here")),
"here",
  assign("a", constant(3)),
  go_to(label("there")),
"here",
  assign("a", constant(4)),
  go_to(label("there")),
"there",

With the simulator as written, what will the contents of register `a` be when control reaches `there`? Modify the `extract_labels`function so that the assembler will signal an error if the same label name is used to indicate two different locations.

Show Solution

---

[[1]](https://sourceacademy.org/sicpjs/5.2.2#footnote-link-1) Using the `receive`function here is a way to get `extract_labels` to effectively return two values—`labels` and `insts`—without explicitly making a compound data structure to hold them. An alternative implementation, which returns an explicit pair of values, is

function extract_labels(controller) { 
    if (is_null(controller)) {
        return pair(null, null);
    } else {
        const result = extract_labels(tail(controller));
        const insts = head(result);
        const labels = tail(result);
        const next_element = head(controller);
        return is_string(next_element)
               ? pair(insts, 
                      pair(make_label_entry(next_element, insts), labels))
               : pair(pair(make_inst(next_element), insts),
                      labels);
    }
}

which would be called by `assemble` as follows:

function assemble(controller, machine) {
    const result = extract_labels(controller);
    const insts = head(result);
    const labels = tail(result);
    update_insts(insts, labels, machine);
    return insts;
}

You can consider our use of `receive` as demonstrating an elegant way to return multiple values, or simply an excuse to show off a programming trick. An argument like `receive` that is the next function to be invoked is called a "continuation." Recall that we also used continuations to implement the backtracking control structure in the `amb` evaluator in section [4.3.3](https://sourceacademy.org/sicpjs/4.3.3).

# 5.2.3   Instructions and Their Execution Functions

[](https://sourceacademy.org/sicpjs/5.2.3#p1)

The assembler calls `make_execution_function` to generate the execution function for a controller instruction. Like the `analyze`function in the evaluator of section [4.1.7](https://sourceacademy.org/sicpjs/4.1.7), this dispatches on the type of instruction to generate the appropriate execution function. The details of these execution functions determine the meaning of the individual instructions in the register-machine language.

```javascript
function make_execution_function(inst, labels, machine, 
                                 pc, flag, stack, ops) {
    const inst_type = type(inst);
    return inst_type === "assign"
           ? make_assign_ef(inst, machine, labels, ops, pc)
           : inst_type === "test"
           ? make_test_ef(inst, machine, labels, ops, flag, pc)
           : inst_type === "branch"
           ? make_branch_ef(inst, machine, labels, flag, pc)
           : inst_type === "go_to"
           ? make_go_to_ef(inst, machine, labels, pc)
           : inst_type === "save"
           ? make_save_ef(inst, machine, stack, pc)
           : inst_type === "restore"
           ? make_restore_ef(inst, machine, stack, pc)
           : inst_type === "perform"
           ? make_perform_ef(inst, machine, labels, ops, pc)
           : error(inst, "unknown instruction type -- assemble");
} 
```

The elements of the `controller` sequence received by `make_machine` and passed to `assemble` are strings (for labels) and tagged lists (for instructions). The tag in an instruction is a string that identifies the instruction type, such as `"go_to"`, and the remaining elements of the list contains the arguments, such as the destination of the `go_to`. The dispatch in `make_execution_function` uses

```javascript
function type(instruction) { return head(instruction); } 
```

[](https://sourceacademy.org/sicpjs/5.2.3#p2)

The tagged lists are constructed when the `list` expression that is the third argument to `make_machine` is evaluated. Each argument to that `list` is either a string (which evaluates to itself) or a call to a constructor for an instruction tagged list. For example, `assign("b", reg("t"))` calls the constructor `assign` with arguments `"b"` and the result of calling the constructor `reg` with the argument `"t"`. The constructors and their arguments determine the syntax of the individual instructions in the register-machine language. The instruction constructors and selectors are shown below, along with the execution-function generators that use the selectors.

[](https://sourceacademy.org/sicpjs/5.2.3#h1)

## The instruction `assign`

[](https://sourceacademy.org/sicpjs/5.2.3#p3)

The `make_assign_ef`functionmakes execution functions for `assign` instructions:

```javascript
function make_assign_ef(inst, machine, labels, operations, pc) {
    const target = get_register(machine, assign_reg_name(inst));
    const value_exp = assign_value_exp(inst);
    const value_fun =
        is_operation_exp(value_exp)
        ? make_operation_exp_ef(value_exp, machine, labels, operations)
        : make_primitive_exp_ef(value_exp, machine, labels);
    return () => {
               set_contents(target, value_fun());
               advance_pc(pc); 
           };
} 
```

The function `assign` constructs `assign` instructions. The selectors `assign_reg_name` and `assign_value_exp` extract the register name and value expression from an `assign` instruction.

```javascript
function assign(register_name, source) {
    return list("assign", register_name, source);
}
function assign_reg_name(assign_instruction) {
    return head(tail(assign_instruction));
}
function assign_value_exp(assign_instruction) { 
    return head(tail(tail(assign_instruction)));
} 
```

The function `make_assign_ef` looks up the register name with `get_register` to produce the target register object. The value expression is passed to `make_operation_exp_ef` if the value is the result of an operation, and it is passed to `make_primitive_exp_ef` otherwise. These functions (shown below) analyze the value expression and produce an execution function for the value. This is a function of no arguments, called `value_fun`, which will be evaluated during the simulation to produce the actual value to be assigned to the register. Notice that the work of looking up the register name and analyzing the value expression is performed just once, at assembly time, not every time the instruction is simulated. This saving of work is the reason we use execution functions, and corresponds directly to the saving in work we obtained by separating program analysis from execution in the evaluator of section [4.1.7](https://sourceacademy.org/sicpjs/4.1.7).

[](https://sourceacademy.org/sicpjs/5.2.3#p4)

The result returned by `make_assign_ef` is the execution function for the `assign` instruction. When this function is called (by the machine model's `execute`function), it sets the contents of the target register to the result obtained by executing `value_fun`. Then it advances the `pc` to the next instruction by running the function

```javascript
function advance_pc(pc) {
    set_contents(pc, tail(get_contents(pc))); 
} 
```

The function `advance_pc` is the normal termination for all instructions except `branch` and `go_to`.

[](https://sourceacademy.org/sicpjs/5.2.3#h2)

## The instructions `test`, `branch`, and `go_to`

[](https://sourceacademy.org/sicpjs/5.2.3#p5)

The function `make_test_ef` handles `test` instructions in a similar way. It extracts the expression that specifies the condition to be tested and generates an execution function for it. At simulation time, the function for the condition is called, the result is assigned to the `flag` register, and the `pc` is advanced:

```javascript
function make_test_ef(inst, machine, labels, operations, flag, pc) {
    const condition = test_condition(inst);
    if (is_operation_exp(condition)) {
        const condition_fun = make_operation_exp_ef(
                                  condition, machine, 
                                  labels, operations);
        return () => {
                   set_contents(flag, condition_fun());
                   advance_pc(pc); 
               };
    } else {
        error(inst, "bad test instruction -- assemble");
    }
} 
```

The function `test` constructs `test` instructions. The selector `test_condition` extracts the condition from a test.

```javascript
function test(condition) { return list("test", condition); }

function test_condition(test_instruction) {
    return head(tail(test_instruction)); 
} 
```

[](https://sourceacademy.org/sicpjs/5.2.3#p6)

The execution function for a `branch` instruction checks the contents of the `flag` register and either sets the contents of the `pc` to the branch destination (if the branch is taken) or else just advances the `pc` (if the branch is not taken). Notice that the indicated destination in a `branch` instruction must be a label, and the `make_branch_ef`function enforces this. Notice also that the label is looked up at assembly time, not each time the `branch` instruction is simulated.

```javascript
function make_branch_ef(inst, machine, labels, flag, pc) {
    const dest = branch_dest(inst);
    if (is_label_exp(dest)) {
        const insts = lookup_label(labels, label_exp_label(dest));
        return () => {
                   if (get_contents(flag)) {
                       set_contents(pc, insts);
                   } else {
                       advance_pc(pc);
                   }
               };
    } else {
        error(inst, "bad branch instruction -- assemble");
    }
} 
```

The function `branch` constructs `branch` instructions. The selector `branch_dest` extracts the destination from a branch.

```javascript
function branch(label) { return list("branch", label); }

function branch_dest(branch_instruction) {
    return head(tail(branch_instruction)); 
} 
```

[](https://sourceacademy.org/sicpjs/5.2.3#p7)

A `go_to` instruction is similar to a branch, except that the destination may be specified either as a label or as a register, and there is no condition to check—the `pc` is always set to the new destination.

```javascript
function make_go_to_ef(inst, machine, labels, pc) {
    const dest = go_to_dest(inst);
    if (is_label_exp(dest)) {
        const insts = lookup_label(labels, label_exp_label(dest));
        return () => set_contents(pc, insts);
    } else if (is_register_exp(dest)) {
        const reg = get_register(machine, register_exp_reg(dest));
        return () => set_contents(pc, get_contents(reg));
    } else {
        error(inst, "bad go_to instruction -- assemble");
    }
} 
```

The function `go_to` constructs `go_to` instructions. The selector `go_to_dest` extracts the destination from a `go_to` instruction.

```javascript
function go_to(label) { return list("go_to", label); }

function go_to_dest(go_to_instruction) { 
    return head(tail(go_to_instruction)); 
} 
```

[](https://sourceacademy.org/sicpjs/5.2.3#h3)

## Other instructions

[](https://sourceacademy.org/sicpjs/5.2.3#p8)

The stack instructions `save` and `restore` simply use the stack with the designated register and advance the `pc`:

```javascript
function make_save_ef(inst, machine, stack, pc) {
    const reg = get_register(machine, stack_inst_reg_name(inst));
    return () => {
               push(stack, get_contents(reg));
               advance_pc(pc);
           };
}
function make_restore_ef(inst, machine, stack, pc) {
    const reg = get_register(machine, stack_inst_reg_name(inst));
    return () => {
               set_contents(reg, pop(stack));
               advance_pc(pc); 
           };
} 
```

The functions `save` and `restore` construct `save` and `restore` instructions. The selector `stack_inst_reg_name` extracts the register name from such instructions.

```javascript
function save(reg) { return list("save", reg); }

function restore(reg) { return list("restore", reg); }

function stack_inst_reg_name(stack_instruction) {
    return head(tail(stack_instruction));
} 
```

[](https://sourceacademy.org/sicpjs/5.2.3#p9)

The final instruction type, handled by `make_perform_ef`, generates an execution function for the action to be performed. At simulation time, the action function is executed and the `pc` advanced.

```javascript
function make_perform_ef(inst, machine, labels, operations, pc) {
    const action = perform_action(inst);
    if (is_operation_exp(action)) {
        const action_fun = make_operation_exp_ef(action, machine,
                                                 labels, operations);
        return () => { 
                   action_fun(); 
                   advance_pc(pc); 
               };
    } else {
        error(inst, "bad perform instruction -- assemble");
    }
} 
```

The function `perform` constructs `perform` instructions. The selector `perform_action` extracts the action from a `perform` instruction.

```javascript
function perform(action) { return list("perform", action); }

function perform_action(perform_instruction) {
    return head(tail(perform_instruction));
} 
```

[](https://sourceacademy.org/sicpjs/5.2.3#h4)

## Execution functions for subexpressions

[](https://sourceacademy.org/sicpjs/5.2.3#p10)

The value of a `reg`, `label`, or `constant` expression may be needed for assignment to a register (`make_assign_ef`, above) or for input to an operation (`make_operation_exp_ef`, below). The following function generates execution functions to produce values for these expressions during the simulation:

```javascript
function make_primitive_exp_ef(exp, machine, labels) {
    if (is_constant_exp(exp)) {
        const c = constant_exp_value(exp);
        return () => c;
    } else if (is_label_exp(exp)) {
        const insts = lookup_label(labels, label_exp_label(exp));
        return () => insts;
    } else if (is_register_exp(exp)) {
        const r = get_register(machine, register_exp_reg(exp));
        return () => get_contents(r); 
    } else {
        error(exp, "unknown expression type -- assemble");
    }
} 
```

The syntax of `reg`, `label`, and `constant` expressions is determined by the following constructor functions, along with corresponding predicates and selectors.

```javascript
function reg(name) { return list("reg", name); }

function is_register_exp(exp) { return is_tagged_list(exp, "reg"); }

function register_exp_reg(exp) { return head(tail(exp)); }

function constant(value) { return list("constant", value); }

function is_constant_exp(exp) {
    return is_tagged_list(exp, "constant");
}

function constant_exp_value(exp) { return head(tail(exp)); }

function label(name) { return list("label", name); }

function is_label_exp(exp) { return is_tagged_list(exp, "label"); }

function label_exp_label(exp) { return head(tail(exp)); } 
```

[](https://sourceacademy.org/sicpjs/5.2.3#p11)

The instructions `assign`, `perform`, and `test` may include the application of a machine operation (specified by an `op` expression) to some operands (specified by `reg` and `constant` expressions). The following function produces an execution function for an "operation expression"—a list containing the operation and operand expressions from the instruction:

```javascript
function make_operation_exp_ef(exp, machine, labels, operations) {
    const op = lookup_prim(operation_exp_op(exp), operations);
    const afuns = map(e => make_primitive_exp_ef(e, machine, labels),
                      operation_exp_operands(exp));
    return () => apply_in_underlying_javascript(
                     op, map(f => f(), afuns));
} 
```

The syntax of operation expressions is determined by

```javascript
function op(name) { return list("op", name); }

function is_operation_exp(exp) {
    return is_pair(exp) && is_tagged_list(head(exp), "op");
}

function operation_exp_op(op_exp) { return head(tail(head(op_exp))); }

function operation_exp_operands(op_exp) { return tail(op_exp); } 
```

Observe that the treatment of operation expressions is very much like the treatment of function applications by the `analyze_application`function in the evaluator of section [4.1.7](https://sourceacademy.org/sicpjs/4.1.7) in that we generate an execution function for each operand. At simulation time, we call the operand functions and apply the JavaScript function that simulates the operation to the resulting values. We make use of the function `apply_in_underlying_javascript`, as we did in `apply_primitive_function` in section [4.1.4](https://sourceacademy.org/sicpjs/4.1.4). This is needed to apply `op` to all elements of the argument list `afuns` produced by the first `map`, as if they were separate arguments to `op`. Without this, `op` would have been restricted to be a unary function.

[](https://sourceacademy.org/sicpjs/5.2.3#p12)

The simulation function is found by looking up the operation name in the operation table for the machine:

```javascript
function lookup_prim(symbol, operations) {
    const val = assoc(symbol, operations);
    return is_undefined(val)
           ? error(symbol, "unknown operation -- assemble")
           : head(tail(val));
} 
```

[](https://sourceacademy.org/sicpjs/5.2.3#ex-5.9)

**Exercise 5.9**

The treatment of machine operations above permits them to operate on labels as well as on constants and the contents of registers. Modify the expression-processing functions to enforce the condition that operations can be used only with registers and constants.

Show Solution

[](https://sourceacademy.org/sicpjs/5.2.3#ex-5.10)

**Exercise 5.10**

When we introduced `save` and restore in section [5.1.4](https://sourceacademy.org/sicpjs/5.1.4), we didn't specify what would happen if you tried to restore a register that was not the last one saved, as in the sequence

save(y);
save(x);
restore(y);

There are several reasonable possibilities for the meaning of `restore`:

1. `restore(y)` puts into `y` the last value saved on the stack, regardless of what register that value came from. This is the way our simulator behaves. Show how to take advantage of this behavior to eliminate one instruction from the Fibonacci machine of section [5.1.4](https://sourceacademy.org/sicpjs/5.1.4) (figure [5.12](https://sourceacademy.org/sicpjs/5.1.4#fig-5.12)).
2. `restore(y)` puts into `y` the last value saved on the stack, but only if that value was saved from `y`; otherwise, it signals an error. Modify the simulator to behave this way. You will have to change `save` to put the register name on the stack along with the value.
3. `restore(y)` puts into `y` the last value saved from `y` regardless of what other registers were saved after `y` and not restored. Modify the simulator to behave this way. You will have to associate a separate stack with each register. You should make the `initialize_stack` operation initialize all the register stacks.

Show Solution

[](https://sourceacademy.org/sicpjs/5.2.3#ex-5.11)

**Exercise 5.11**

The simulator can be used to help determine the data paths required for implementing a machine with a given controller. Extend the assembler to store the following information in the machine model:

- a list of all instructions, with duplicates removed, sorted by instruction type (`assign`, `go_to`, and so on);
- a list (without duplicates) of the registers used to hold entry points (these are the registers referenced by `go_to` instructions);
- a list (without duplicates) of the registers that are `save`d or `restore`d;
- for each register, a list (without duplicates) of the sources from which it is assigned (for example, the sources for register `val` in the factorial machine of figure [5.11](https://sourceacademy.org/sicpjs/5.1.4#fig-5.11) are `constant(1)` and `list(op("*"), reg("n"), reg("val"))`).

Extend the message-passing interface to the machine to provide access to this new information. To test your analyzer, define the Fibonacci machine from figure [5.12](https://sourceacademy.org/sicpjs/5.1.4#fig-5.12) and examine the lists you constructed.

Show Solution

[](https://sourceacademy.org/sicpjs/5.2.3#ex-5.12)

**Exercise 5.12**

Modify the simulator so that it uses the controller sequence to determine what registers the machine has rather than requiring a list of registers as an argument to `make_machine`. Instead of preallocating the registers in `make_machine`, you can allocate them one at a time when they are first seen during assembly of the instructions.

# 5.2.4   Monitoring Machine Performance

[](https://sourceacademy.org/sicpjs/5.2.4#p1)

Simulation is useful not only for verifying the correctness of a proposed machine design but also for measuring the machine's performance. For example, we can install in our simulation program a "meter" that measures the number of stack operations used in a computation. To do this, we modify our simulated stack to keep track of the number of times registers are saved on the stack and the maximum depth reached by the stack, and add a message to the stack's interface that prints the statistics, as shown below. We also add an operation to the basic machine model to print the stack statistics, by initializing `the_ops` in `make_new_machine` to

list(list("initialize_stack", 
          () => stack("initialize")),
     list("print_stack_statistics", 
          () => stack("print_statistics")));

Here is the new version of`make_stack`:

```javascript
function make_stack() { 
    let stack = null;
    let number_pushes = 0;
    let max_depth = 0;
    let current_depth = 0;
    function push(x) {
        stack = pair(x, stack);
        number_pushes = number_pushes + 1;
        current_depth = current_depth + 1;
        max_depth = math_max(current_depth, max_depth);
        return "done";
    }
    function pop() {
        if (is_null(stack)) {
            error("empty stack -- pop");
        } else {
            const top = head(stack);
            stack = tail(stack);
            current_depth = current_depth - 1;
            return top;
        }
    }
    function initialize() {
        stack = null;
        number_pushes = 0;
        max_depth = 0;
        current_depth = 0;
        return "done";
    }
    function print_statistics() {
        display("total pushes = " + stringify(number_pushes));
        display("maximum depth = " + stringify(max_depth));
    }
    function dispatch(message) {
        return message === "push"
               ? push
               : message === "pop"
               ? pop()
               : message === "initialize"
               ? initialize()
               : message === "print_statistics"
               ? print_statistics()
               : error(message, "unknown request -- stack");
    }
    return dispatch;
} 
```

[](https://sourceacademy.org/sicpjs/5.2.4#p2)

Exercises [5.14](https://sourceacademy.org/sicpjs/5.2.4#ex-5.14) through [5.18](https://sourceacademy.org/sicpjs/5.2.4#ex-5.18) describe other useful monitoring and debugging features that can be added to the register-machine simulator.

[](https://sourceacademy.org/sicpjs/5.2.4#ex-5.13)

**Exercise 5.13**

Measure the number of pushes and the maximum stack depth required to compute n!n! for various small values of nn using the factorial machine shown in Figure [5.11](https://sourceacademy.org/sicpjs/5.1.4#fig-5.11). From your data determine formulas in terms of nn for the total number of push operations and the maximum stack depth used in computing n!n! for any n>1n>1. Note that each of these is a linear function of nn and is thus determined by two constants. In order to get the statistics printed, you will have to augment the factorial machine with instructions to initialize the stack and print the statistics. You may want to also modify the machine so that it repeatedly reads a value for nn, computes the factorial, and prints the result (as we did for the GCD machine in figure [5.4](https://sourceacademy.org/sicpjs/5.1.1#fig-5.4)), so that you will not have to repeatedly invoke `get_register_contents`, `set_register_contents`, and `start`.

Show Solution

[](https://sourceacademy.org/sicpjs/5.2.4#ex-5.14)

**Exercise 5.14**

Add _instruction counting_ to the register machine simulation. That is, have the machine model keep track of the number of instructions executed. Extend the machine model's interface to accept a new message that prints the value of the instruction count and resets the count to zero.

Show Solution

[](https://sourceacademy.org/sicpjs/5.2.4#ex-5.15)

**Exercise 5.15**

Augment the simulator to provide for _instruction tracing_. That is, before each instruction is executed, the simulator should print the instruction. Make the machine model accept `trace_on` and `trace_off` messages to turn tracing on and off.

Show Solution

[](https://sourceacademy.org/sicpjs/5.2.4#ex-5.16)

**Exercise 5.16**

Extend the instruction tracing of exercise [5.15](https://sourceacademy.org/sicpjs/5.2.4#ex-5.15) so that before printing an instruction, the simulator prints any labels that immediately precede that instruction in the controller sequence. Be careful to do this in a way that does not interfere with instruction counting (exercise [5.14](https://sourceacademy.org/sicpjs/5.2.4#ex-5.14)). You will have to make the simulator retain the necessary label information.

Show Solution

[](https://sourceacademy.org/sicpjs/5.2.4#ex-5.17)

**Exercise 5.17**

Modify the `make_register`function of section [5.2.1](https://sourceacademy.org/sicpjs/5.2.1) so that registers can be traced. Registers should accept messages that turn tracing on and off. When a register is traced, assigning a value to the register should print the name of the register, the old contents of the register, and the new contents being assigned. Extend the interface to the machine model to permit you to turn tracing on and off for designated machine registers.

Show Solution

[](https://sourceacademy.org/sicpjs/5.2.4#ex-5.18)

**Exercise 5.18**

Alyssa P. Hacker wants a _breakpoint_ feature in the simulator to help her debug her machine designs. You have been hired to install this feature for her. She wants to be able to specify a place in the controller sequence where the simulator will stop and allow her to examine the state of the machine. You are to implement a function

set_breakpoint(machinemachine, labellabel, nn)
      

that sets a breakpoint just before the nnth instruction after the given label. For example,

set_breakpoint(gcd_machine, "test_b", 4)

installs a breakpoint in `gcd_machine` just before the assignment to register `a`. When the simulator reaches the breakpoint it should print the label and the offset of the breakpoint and stop executing instructions. Alyssa can then use `get_register_contents` and `set_register_contents` to manipulate the state of the simulated machine. She should then be able to continue execution by saying

proceed_machine(machinemachine)
      

She should also be able to remove a specific breakpoint by means of

cancel_breakpoint(machinemachine, labellabel, nn)
      

or to remove all breakpoints by means of

cancel_all_breakpoints(machinemachine)

# 5.3  Storage Allocation and Garbage Collection

[](https://sourceacademy.org/sicpjs/5.3#p1)

In section [5.4](https://sourceacademy.org/sicpjs/5.4), we will show how to implement a JavaScript evaluator as a register machine. In order to simplify the discussion, we will assume that our register machines can be equipped with a _list-structured memory_, in which the basic operations for manipulating list-structured data are primitive. Postulating the existence of such a memory is a useful abstraction when one is focusing on the mechanisms of control in an interpreter, but this does not reflect a realistic view of the actual primitive data operations of contemporary computers. To obtain a more complete picture of how systems can support list-structured memory efficiently, we must investigate how list structure can be represented in a way that is compatible with conventional computer memories.

[](https://sourceacademy.org/sicpjs/5.3#p2)

There are two considerations in implementing list structure. The first is purely an issue of representation: how to represent the "box-and-pointer" structure of pairs, using only the storage and addressing capabilities of typical computer memories. The second issue concerns the management of memory as a computation proceeds. The operation of a JavaScript system depends crucially on the ability to continually create new data objects. These include objects that are explicitly created by the JavaScriptfunctions being interpreted as well as structures created by the interpreter itself, such as environments and argument lists. Although the constant creation of new data objects would pose no problem on a computer with an infinite amount of rapidly addressable memory, computer memories are available only in finite sizes (more's the pity). JavaScript thus provide an _automatic storage allocation_ facility to support the illusion of an infinite memory. When a data object is no longer needed, the memory allocated to it is automatically recycled and used to construct new data objects. There are various techniques for providing such automatic storage allocation. The method we shall discuss in this section is called _garbage collection_.
# 5.3.1   Memory as Vectors

[](https://sourceacademy.org/sicpjs/5.3.1#p1)

A conventional computer memory can be thought of as an array of cubbyholes, each of which can contain a piece of information. Each cubbyhole has a unique name, called its _address_ or _location_. Typical memory systems provide two primitive operations: one that fetches the data stored in a specified location and one that assigns new data to a specified location. Memory addresses can be incremented to support sequential access to some set of the cubbyholes. More generally, many important data operations require that memory addresses be treated as data, which can be stored in memory locations and manipulated in machine registers. The representation of list structure is one application of such _address arithmetic_.

[](https://sourceacademy.org/sicpjs/5.3.1#p2)

To model computer memory, we use a new kind of data structure called a _vector_. Abstractly, a vector is a compound data object whose individual elements can be accessed by means of an integer index in an amount of time that is independent of the index.[1](https://sourceacademy.org/sicpjs/5.3.1#footnote-1) In order to describe memory operations, we use two functions for manipulating vectors:[2](https://sourceacademy.org/sicpjs/5.3.1#footnote-2)

- `vector_ref(`_vector_`,` _n_`)` returns the _n_th element of the vector.
- `vector_set(`_vector_`,` _n_`,` _value_`)` sets the nnth element of the vector to the designated value.

For example, if `v` is a vector, then `vector_ref(v, 5)` gets the fifth entry in the vector `v` and `vector_set(v, 5, 7)` changes the value of the fifth entry of the vector `v` to 7.[3](https://sourceacademy.org/sicpjs/5.3.1#footnote-3) For computer memory, this access can be implemented through the use of address arithmetic to combine a _base address_ that specifies the beginning location of a vector in memory with an _index_ that specifies the offset of a particular element of the vector.

[](https://sourceacademy.org/sicpjs/5.3.1#h1)

## Representing data

[](https://sourceacademy.org/sicpjs/5.3.1#p3)

We can use vectors to implement the basic pair structures required for a list-structured memory. Let us imagine that computer memory is divided into two vectors: `the_heads` and `the_tails`. We will represent list structure as follows: A pointer to a pair is an index into the two vectors. The `head` of the pair is the entry in `the_heads` with the designated index, and the tail of the pair is the entry in `the_tails` with the designated index. We also need a representation for objects other than pairs (such as numbers and strings) and a way to distinguish one kind of data from another. There are many methods of accomplishing this, but they all reduce to using _typed pointers_, that is, to extending the notion of "pointer" to include information on data type.[4](https://sourceacademy.org/sicpjs/5.3.1#footnote-4) The data type enables the system to distinguish a pointer to a pair (which consists of the "pair" data type and an index into the memory vectors) from pointers to other kinds of data (which consist of some other data type and whatever is being used to represent data of that type). Two data objects are considered to be the same (`===`) if their pointers are identical. Figure [5.14](https://sourceacademy.org/sicpjs/5.3.1#fig-5.14) illustrates the use of this method to represent `list(list(1, 2), 3, 4)`, whose box-and-pointer diagram is also shown. We use letter prefixes to denote the data-type information. Thus, a pointer to the pair with index 5 is denoted `p5`, the empty list is denoted by the pointer `e0`, and a pointer to the number 4 is denoted `n4`. In the box-and-pointer diagram, we have indicated at the lower left of each pair the vector index that specifies where the `head` and `tail` of the pair are stored. The blank locations in `the_heads` and `the_tails` may contain parts of other list structures (not of interest here).

[](https://sourceacademy.org/sicpjs/5.3.1#fig-5.14)

![#fig-5.14](https://sicp.sourceacademy.org/img_javascript/Fig5.14b.std.svg)

##### Figure 5.14 Box-and-pointer and memory-vector representations of the list `list(list(1, 2), 3, 4)`.

[](https://sourceacademy.org/sicpjs/5.3.1#p4)

A pointer to a number, such as `n4`, might consist of a type indicating numeric data together with the actual representation of the number 4.[5](https://sourceacademy.org/sicpjs/5.3.1#footnote-5) To deal with numbers that are too large to be represented in the fixed amount of space allocated for a single pointer, we could use a distinct _bignum_ data type, for which the pointer designates a list in which the parts of the number are stored.[6](https://sourceacademy.org/sicpjs/5.3.1#footnote-6)

[](https://sourceacademy.org/sicpjs/5.3.1#p5)

A string might be represented as a typed pointer that designates a sequence of the characters that form the string's printed representation. The parser constructs such a sequence when it encounters a string literal, and the string-concatenation operator `+` and string-producing primitive functions such as `stringify` construct such a sequence. Since we want two instances of a string to be recognized as the "same" string by `===` and we want `===` to be a simple test for equality of pointers, we must ensure that if the system sees the same string twice, it will use the same pointer (to the same sequence of characters) to represent both occurrences. To accomplish this, the system maintains a table, called the _string pool_, of all the strings it has ever encountered. When the system is about to construct a string, it checks the string pool to see if it has ever before seen the same string. If it has not, it constructs a new string (a typed pointer to a new character sequence) and enters this pointer in the string pool. If the system has seen the string before, it returns the string pointer stored in the string pool. This process of replacing strings by unique pointers is called _string interning_.

[](https://sourceacademy.org/sicpjs/5.3.1#h2)

## Implementing the primitive list operations

[](https://sourceacademy.org/sicpjs/5.3.1#p6)

Given the above representation scheme, we can replace each "primitive" list operation of a register machine with one or more primitive vector operations. We will use two registers, `the_heads` and `the_tails`, to identify the memory vectors, and will assume that `vector_ref` and `vector_set` are available as primitive operations. We also assume that numeric operations on pointers (such as incrementing a pointer, using a pair pointer to index a vector, or adding two numbers) use only the index portion of the typed pointer.

[](https://sourceacademy.org/sicpjs/5.3.1#p7)

For example, we can make a register machine support the instructions

assign(regreg11​, list(op("head"), reg(regreg22​)))

assign(regreg11​, list(op("tail"), reg(regreg22​)))
      

if we implement these, respectively, as

assign(regreg11​, list(op("vector_ref"), reg("the_heads"), reg(regreg22​)))

assign(regreg11​, list(op("vector_ref"), reg("the_tails"), reg(regreg22​)))
      

The instructions

perform(list(op("set_head"), reg(regreg11​), reg(regreg22​)))

perform(list(op("set_tail"), reg(regreg11​), reg(regreg22​)))
      

are implemented as

perform(list(op("vector_set"), reg("the_heads"), reg(regreg11​), reg(regreg22​)))

perform(list(op("vector_set"), reg("the_tails"), reg(regreg11​), reg(regreg22​)))
      

[](https://sourceacademy.org/sicpjs/5.3.1#p8)

The operation `pair` is performed by allocating an unused index and storing the arguments to `pair` in `the_heads` and `the_tails` at that indexed vector position. We presume that there is a special register, `free`, that always holds a pair pointer containing the next available index, and that we can increment the index part of that pointer to find the next free location.[7](https://sourceacademy.org/sicpjs/5.3.1#footnote-7) For example, the instruction

assign(regreg11​, list(op("pair"), reg(regreg22​), reg(regreg33​)))
      

is implemented as the following sequence of vector operations:[8](https://sourceacademy.org/sicpjs/5.3.1#footnote-8)

perform(list(op("vector_set"),
             reg("the_heads"), reg("free"), reg(regreg22​))),
perform(list(op("vector_set"),
             reg("the_tails"), reg("free"), reg(regreg33​))),
assign(regreg11​, reg("free")),
assign("free", list(op("+"), reg("free"), constant(1)))
      

[](https://sourceacademy.org/sicpjs/5.3.1#p9)

The `===` operation

list(op("==="), reg(regreg11​), reg(regreg22​))
      

simply tests the equality of all fields in the registers, and predicates such as `is_pair`,`is_null`,`is_string`, and `is_number` need only check the type field.

[](https://sourceacademy.org/sicpjs/5.3.1#h3)

## Implementing stacks

[](https://sourceacademy.org/sicpjs/5.3.1#p10)

Although our register machines use stacks, we need do nothing special here, since stacks can be modeled in terms of lists. The stack can be a list of the saved values, pointed to by a special register `the_stack`. Thus, `save(`_reg_`)` can be implemented as

assign("the_stack", list(op("pair"), reg(regreg), reg("the_stack")))
      

Similarly, `restore(`_reg_`)` can be implemented as

assign(regreg, list(op("head"), reg("the_stack")))
assign("the_stack", list(op("tail"), reg("the_stack")))
      

and `perform(list(op("initialize_stack")))` can be implemented as

assign("the_stack", constant(null))
      

These operations can be further expanded in terms of the vector operations given above. In conventional computer architectures, however, it is usually advantageous to allocate the stack as a separate vector. Then pushing and popping the stack can be accomplished by incrementing or decrementing an index into that vector.

[](https://sourceacademy.org/sicpjs/5.3.1#ex-5.19)

**Exercise 5.19**

Draw the box-and-pointer representation and the memory-vector representation (as in figure [5.14](https://sourceacademy.org/sicpjs/5.3.1#fig-5.14)) of the list structure produced by

const x = pair(1, 2);
const y = list(x, x);

with the `free` pointer initially `p1`. What is the final value of `free` ? What pointers represent the values of `x` and `y`?

Show Solution

[](https://sourceacademy.org/sicpjs/5.3.1#ex-5.20)

**Exercise 5.20**

Implement register machines for the following functions. Assume that the list-structure memory operations are available as machine primitives.

1. Recursive `count_leaves`:
    
    ```javascript
    function count_leaves(tree) {
        return is_null(tree)
               ? 0
               : ! is_pair(tree)
               ? 1
               : count_leaves(head(tree)) +
                 count_leaves(tail(tree));
    } 
    ```
    
2. Recursive `count_leaves` with explicit counter:
    
    ```javascript
    function count_leaves(tree) {
        function count_iter(tree, n) {
            return is_null(tree)
                   ? n
                   : ! is_pair(tree) 
                   ? n + 1
                   : count_iter(tail(tree),
                                count_iter(head(tree), n));
        }
        return count_iter(tree, 0);
    } 
    ```
    

Show Solution

[](https://sourceacademy.org/sicpjs/5.3.1#ex-5.21)

**Exercise 5.21**

Exercise [3.12](https://sourceacademy.org/sicpjs/3.3.1#ex-3.12) of section [3.3.1](https://sourceacademy.org/sicpjs/3.3.1) presented an `append`function that appends two lists to form a new list and an `append_mutator` function that splices two lists together. Design a register machine to implement each of these functions. Assume that the list-structure memory operations are available as primitive operations.

Show Solution

---

[[1]](https://sourceacademy.org/sicpjs/5.3.1#footnote-link-1) We could represent memory as lists of items. However, the access time would then not be independent of the index, since accessing the nnth element of a list requires n−1n−1`tail` operations.

[[2]](https://sourceacademy.org/sicpjs/5.3.1#footnote-link-2) As mentioned in section [4.1.4](https://sourceacademy.org/sicpjs/4.1.4) (footnote 2), JavaScript supports vectors as data structures and calls them "arrays." We use the term _vector_ in this book, as it is the more common terminology. The vector functions above are easily implemented using JavaScript's primitive array support.

[[3]](https://sourceacademy.org/sicpjs/5.3.1#footnote-link-3) For completeness, we should specify a `make_vector` operation that constructs vectors. However, in the present application we will use vectors only to model fixed divisions of the computer memory.

[[4]](https://sourceacademy.org/sicpjs/5.3.1#footnote-link-4) This is precisely the same "tagged data" idea we introduced in chapter [2](https://sourceacademy.org/sicpjs/2) for dealing with generic operations. Here, however, the data types are included at the primitive machine level rather than constructed through the use of lists.

[](https://sourceacademy.org/sicpjs/5.3.1#p11)

Type information may be encoded in a variety of ways, depending on the details of the machine on which the JavaScript system is to be implemented. The execution efficiency of JavaScript programs will be strongly dependent on how cleverly this choice is made, but it is difficult to formulate general design rules for good choices. The most straightforward way to implement typed pointers is to allocate a fixed set of bits in each pointer to be a _type field_ that encodes the data type. Important questions to be addressed in designing such a representation include the following: How many type bits are required? How large must the vector indices be? How efficiently can the primitive machine instructions be used to manipulate the type fields of pointers? Machines that include special hardware for the efficient handling of type fields are said to have _tagged architectures_.

[[5]](https://sourceacademy.org/sicpjs/5.3.1#footnote-link-5) This decision on the representation of numbers determines whether `===`, which tests equality of pointers, can be used to test for equality of numbers. If the pointer contains the number itself, then equal numbers will have the same pointer. But if the pointer contains the index of a location where the number is stored, equal numbers will be guaranteed to have equal pointers only if we are careful never to store the same number in more than one location.

[[6]](https://sourceacademy.org/sicpjs/5.3.1#footnote-link-6) This is just like writing a number as a sequence of digits, except that each "digit" is a number between 0 and the largest number that can be stored in a single pointer.

[[7]](https://sourceacademy.org/sicpjs/5.3.1#footnote-link-7) There are other ways of finding free storage. For example, we could link together all the unused pairs into a _free list_. Our free locations are consecutive (and hence can be accessed by incrementing a pointer) because we are using a compacting garbage collector, as we will see in section [5.3.2](https://sourceacademy.org/sicpjs/5.3.2).

[[8]](https://sourceacademy.org/sicpjs/5.3.1#footnote-link-8) This is essentially the implementation of `pair` in terms of `set_head` and `set_tail`, as described in section [3.3.1](https://sourceacademy.org/sicpjs/3.3.1). The operation `get_new_pair` used in that implementation is realized here by the `free` pointer.
# 5.3.2   Maintaining the Illusion of Infinite Memory

[](https://sourceacademy.org/sicpjs/5.3.2#p1)

The representation method outlined in section [5.3.1](https://sourceacademy.org/sicpjs/5.3.1) solves the problem of implementing list structure, provided that we have an infinite amount of memory. With a real computer we will eventually run out of free space in which to construct new pairs.[1](https://sourceacademy.org/sicpjs/5.3.2#footnote-1) However, most of the pairs generated in a typical computation are used only to hold intermediate results. After these results are accessed, the pairs are no longer needed—they are _garbage_. For instance, the computation

accumulate((x, y) => x + y,
           0,
           filter(is_odd, enumerate_interval(0, n)))

constructs two lists: the enumeration and the result of filtering the enumeration. When the accumulation is complete, these lists are no longer needed, and the allocated memory can be reclaimed. If we can arrange to collect all the garbage periodically, and if this turns out to recycle memory at about the same rate at which we construct new pairs, we will have preserved the illusion that there is an infinite amount of memory.

[](https://sourceacademy.org/sicpjs/5.3.2#p2)

In order to recycle pairs, we must have a way to determine which allocated pairs are not needed (in the sense that their contents can no longer influence the future of the computation). The method we shall examine for accomplishing this is known as _garbage collection_. Garbage collection is based on the observation that, at any moment in an interpretation based on list-structured memory, the only objects that can affect the future of the computation are those that can be reached by some succession of `head` and `tail` operations starting from the pointers that are currently in the machine registers.[2](https://sourceacademy.org/sicpjs/5.3.2#footnote-2) Any memory cell that is not so accessible may be recycled.

[](https://sourceacademy.org/sicpjs/5.3.2#p3)

There are many ways to perform garbage collection. The method we shall examine here is called _stop-and-copy_. The basic idea is to divide memory into two halves: "working memory" and "free memory." When `pair` constructs pairs, it allocates these in working memory. When working memory is full, we perform garbage collection by locating all the useful pairs in working memory and copying these into consecutive locations in free memory. (The useful pairs are located by tracing all the `head` and `tail` pointers, starting with the machine registers.) Since we do not copy the garbage, there will presumably be additional free memory that we can use to allocate new pairs. In addition, nothing in the working memory is needed, since all the useful pairs in it have been copied. Thus, if we interchange the roles of working memory and free memory, we can continue processing; new pairs will be allocated in the new working memory (which was the old free memory). When this is full, we can copy the useful pairs into the new free memory (which was the old working memory).[3](https://sourceacademy.org/sicpjs/5.3.2#footnote-3)

[](https://sourceacademy.org/sicpjs/5.3.2#h1)

## Implementation of a stop-and-copy garbage collector

[](https://sourceacademy.org/sicpjs/5.3.2#p4)

We now use our register-machine language to describe the stop-and-copy algorithm in more detail. We will assume that there is a register called `root` that contains a pointer to a structure that eventually points at all accessible data. This can be arranged by storing the contents of all the machine registers in a preallocated list pointed at by `root` just before starting garbage collection.[4](https://sourceacademy.org/sicpjs/5.3.2#footnote-4) We also assume that, in addition to the current working memory, there is free memory available into which we can copy the useful data. The current working memory consists of vectors whose base addresses are in registers called `the_heads` and `the_tails`, and the free memory is in registers called `new_heads` and `new_tails`.

[](https://sourceacademy.org/sicpjs/5.3.2#p5)

Garbage collection is triggered when we exhaust the free cells in the current working memory, that is, when a `pair` operation attempts to increment the `free` pointer beyond the end of the memory vector. When the garbage-collection process is complete, the `root` pointer will point into the new memory, all objects accessible from the `root` will have been moved to the new memory, and the `free` pointer will indicate the next place in the new memory where a new pair can be allocated. In addition, the roles of working memory and new memory will have been interchanged—new pairs will be constructed in the new memory, beginning at the place indicated by `free`, and the (previous) working memory will be available as the new memory for the next garbage collection. Figure [5.15](https://sourceacademy.org/sicpjs/5.3.2#fig-5.15) shows the arrangement of memory just before and just after garbage collection.

[](https://sourceacademy.org/sicpjs/5.3.2#fig-5.15)

![#fig-5.15](https://sicp.sourceacademy.org/img_javascript/Fig5.15c.std.svg)

##### Figure 5.15 Reconfiguration of memory by the garbage-collection process.

[](https://sourceacademy.org/sicpjs/5.3.2#p6)

The state of the garbage-collection process is controlled by maintaining two pointers: `free` and `scan`. These are initialized to point to the beginning of the new memory. The algorithm begins by relocating the pair pointed at by `root` to the beginning of the new memory. The pair is copied, the `root` pointer is adjusted to point to the new location, and the `free` pointer is incremented. In addition, the old location of the pair is marked to show that its contents have been moved. This marking is done as follows: In the `head` position, we place a special tag that signals that this is an already-moved object. (Such an object is traditionally called a _broken heart_.)[5](https://sourceacademy.org/sicpjs/5.3.2#footnote-5) In the `tail` position we place a _forwarding address_ that points at the location to which the object has been moved.

[](https://sourceacademy.org/sicpjs/5.3.2#p7)

After relocating the root, the garbage collector enters its basic cycle. At each step in the algorithm, the `scan` pointer (initially pointing at the relocated root) points at a pair that has been moved to the new memory but whose `head` and `tail` pointers still refer to objects in the old memory. These objects are each relocated, and the `scan` pointer is incremented. To relocate an object (for example, the object indicated by the `head` pointer of the pair we are scanning) we check to see if the object has already been moved (as indicated by the presence of a broken-heart tag in the `head` position of the object). If the object has not already been moved, we copy it to the place indicated by `free`, update `free`, set up a broken heart at the object's old location, and update the pointer to the object (in this example, the `head` pointer of the pair we are scanning) to point to the new location. If the object has already been moved, its forwarding address (found in the `tail` position of the broken heart) is substituted for the pointer in the pair being scanned. Eventually, all accessible objects will have been moved and scanned, at which point the `scan` pointer will overtake the `free` pointer and the process will terminate.

[](https://sourceacademy.org/sicpjs/5.3.2#p8)

We can specify the stop-and-copy algorithm as a sequence of instructions for a register machine. The basic step of relocating an object is accomplished by a subroutine called `relocate_old_result_in_new`. This subroutine gets its argument, a pointer to the object to be relocated, from a register named `old`. It relocates the designated object (incrementing `free` in the process), puts a pointer to the relocated object into a register called `new`, and returns by branching to the entry point stored in the register `relocate_continue`. To begin garbage collection, we invoke this subroutine to relocate the `root` pointer, after initializing `free` and `scan`. When the relocation of `root` has been accomplished, we install the new pointer as the new `root` and enter the main loop of the garbage collector.

"begin_garbage_collection",
  assign("free", constant(0)),
  assign("scan", constant(0)),
  assign("old", reg("root")),
  assign("relocate_continue", label("reassign_root")),
  go_to(label("relocate_old_result_in_new")),
"reassign_root",
  assign("root", reg("new")),
  go_to(label("gc_loop")),

[](https://sourceacademy.org/sicpjs/5.3.2#p9)

In the main loop of the garbage collector we must determine whether there are any more objects to be scanned. We do this by testing whether the `scan` pointer is coincident with the `free` pointer. If the pointers are equal, then all accessible objects have been relocated, and we branch to `gc_flip`, which cleans things up so that we can continue the interrupted computation. If there are still pairs to be scanned, we call the relocate subroutine to relocate the `head` of the next pair (by placing the `head` pointer in `old`). The `relocate_continue` register is set up so that the subroutine will return to update the `head` pointer.

"gc_loop",
  test(list(op("==="), reg("scan"), reg("free"))),
  branch(label("gc_flip")),
  assign("old", list(op("vector_ref"), reg("new_heads"), reg("scan"))),
  assign("relocate_continue", label("update_head")),
  go_to(label("relocate_old_result_in_new")),

[](https://sourceacademy.org/sicpjs/5.3.2#p10)

At `update_head`, we modify the `head` pointer of the pair being scanned, then proceed to relocate the `tail` of the pair. We return to `update_tail` when that relocation has been accomplished. After relocating and updating the `tail`, we are finished scanning that pair, so we continue with the main loop.

"update_head",
  perform(list(op("vector_set"), 
               reg("new_heads"), reg("scan"), reg("new"))),
  assign("old", list(op("vector_ref"), 
                     reg("new_tails"), reg("scan"))),
  assign("relocate_continue", label("update_tail")),
  go_to(label("relocate_old_result_in_new")),

"update_tail",
  perform(list(op("vector_set"), 
               reg("new_tails"), reg("scan"), reg("new"))),
  assign("scan", list(op("+"), reg("scan"), constant(1))),
  go_to(label("gc_loop")),

[](https://sourceacademy.org/sicpjs/5.3.2#p11)

The subroutine `relocate_old_result_in_new` relocates objects as follows: If the object to be relocated (pointed at by `old`) is not a pair, then we return the same pointer to the object unchanged (in `new`). (For example, we may be scanning a pair whose `head` is the number 4. If we represent the `head` by `n4`, as described in section [5.3.1](https://sourceacademy.org/sicpjs/5.3.1), then we want the "relocated"`head` pointer to still be `n4`.) Otherwise, we must perform the relocation. If the `head` position of the pair to be relocated contains a broken-heart tag, then the pair has in fact already been moved, so we retrieve the forwarding address (from the `tail` position of the broken heart) and return this in `new`. If the pointer in `old` points at a yet-unmoved pair, then we move the pair to the first free cell in new memory (pointed at by `free`) and set up the broken heart by storing a broken-heart tag and forwarding address at the old location. The subroutine `relocate_old_result_in_new` uses a register `oldht` to hold the `head` or the `tail` of the object pointed at by `old`.[6](https://sourceacademy.org/sicpjs/5.3.2#footnote-6)

"relocate_old_result_in_new",
  test(list(op("is_pointer_to_pair"), reg("old"))),
  branch(label("pair")),
  assign("new", reg("old")),
  go_to(reg("relocate_continue")),
"pair",
  assign("oldht", list(op("vector_ref"), 
                       reg("the_heads"), reg("old"))),
  test(list(op("is_broken_heart"), reg("oldht"))),
  branch(label("already_moved")),
  assign("new", reg("free")),     // new location for pair
  // Update freefree pointer
  assign("free", list(op("+"), reg("free"), constant(1))),
  // Copy the head and tail to new memory
  perform(list(op("vector_set"),
               reg("new_heads"), reg("new"),
               reg("oldht"))),
  assign("oldht", list(op("vector_ref"), 
                       reg("the_tails"), reg("old"))),
  perform(list(op("vector_set"),
               reg("new_tails"), reg("new"),
               reg("oldht"))),
  // Construct the broken heart
  perform(list(op("vector_set"),
               reg("the_heads"), reg("old"),
               constant("broken_heart"))),
  perform(list(op("vector_set"),
               reg("the_tails"), reg("old"),
               reg("new"))),
  go_to(reg("relocate_continue")),
"already_moved",
  assign("new", list(op("vector_ref"), 
                     reg("the_tails"), reg("old"))),
  go_to(reg("relocate_continue")),
      

[](https://sourceacademy.org/sicpjs/5.3.2#p12)

At the very end of the garbage collection process, we interchange the role of old and new memories by interchanging pointers: interchanging `the_heads` with `new_heads`, and `the_tails` with `new_tails`. We will then be ready to perform another garbage collection the next time memory runs out.

```javascript
"gc_flip",
  assign("temp", reg("the_tails")),
  assign("the_tails", reg("new_tails")),
  assign("new_tails", reg("temp")),
  assign("temp", reg("the_heads")),
  assign("the_heads", reg("new_heads")),
  assign("new_heads", reg("temp")) 
```

---

[[1]](https://sourceacademy.org/sicpjs/5.3.2#footnote-link-1) This may not be true eventually, because memories may get large enough so that it would be impossible to run out of free memory in the lifetime of the computer. For example, there are about 3×10163×1016 nanoseconds in a year, so if we were to `pair` once per nanosecond we would need about 10181018 cells of memory to build a machine that could operate for 30 years without running out of memory. That much memory seems absurdly large by today's standards, but it is not physically impossible. On the other hand, processors are getting faster and modern computers have increasingly large numbers of processors operating in parallel on a single memory, so it may be possible to use up memory much faster than we have postulated.

[[2]](https://sourceacademy.org/sicpjs/5.3.2#footnote-link-2) We assume here that the stack is represented as a list as described in section [5.3.1](https://sourceacademy.org/sicpjs/5.3.1), so that items on the stack are accessible via the pointer in the stack register.

[[3]](https://sourceacademy.org/sicpjs/5.3.2#footnote-link-3) This idea was invented and first implemented by Minsky, as part of the implementation of Lisp for the PDP-1 at the MIT Research Laboratory of Electronics. It was further developed by Fenichel and Yochelson (1969) for use in the Lisp implementation for the Multics time-sharing system. Later, Baker (1978) developed a "real-time" version of the method, which does not require the computation to stop during garbage collection. Baker's idea was extended by Hewitt, Lieberman, and Moon (see Lieberman and Hewitt 1983) to take advantage of the fact that some structure is more volatile and other structure is more permanent. An alternative commonly used garbage-collection technique is the _mark-sweep_ method. This consists of tracing all the structure accessible from the machine registers and marking each pair we reach. We then scan all of memory, and any location that is unmarked is "swept up" as garbage and made available for reuse. A full discussion of the mark-sweep method can be found in Allen 1978. The Minsky-Fenichel-Yochelson algorithm is the dominant algorithm in use for large-memory systems because it examines only the useful part of memory. This is in contrast to mark-sweep, in which the sweep phase must check all of memory. A second advantage of stop-and-copy is that it is a _compacting_ garbage collector. That is, at the end of the garbage-collection phase the useful data will have been moved to consecutive memory locations, with all garbage pairs compressed out. This can be an extremely important performance consideration in machines with virtual memory, in which accesses to widely separated memory addresses may require extra paging operations.

[[4]](https://sourceacademy.org/sicpjs/5.3.2#footnote-link-4) This list of registers does not include the registers used by the storage-allocation system: `root`, `the_heads`,`the_tails`, and the other registers that will be introduced in this section.

[[5]](https://sourceacademy.org/sicpjs/5.3.2#footnote-link-5) The term _broken heart_ was coined by David Cressey, who wrote a garbage collector for MDL, a dialect of Lisp developed at MIT during the early 1970s.

[[6]](https://sourceacademy.org/sicpjs/5.3.2#footnote-link-6) The garbage collector uses the low-level predicate `is_pointer_to_pair` instead of the list-structure `is_pair` operation because in a real system there might be various things that are treated as pairs for garbage-collection purposes. For example, a function object may be implemented as a special kind of "pair" that doesn't satisfy the `is_pair` predicate. For simulation purposes, `is_pointer_to_pair` can be implemented as `is_pair`.

# 5.4  The Explicit-Control Evaluator

[](https://sourceacademy.org/sicpjs/5.4#p1)

In section [5.1](https://sourceacademy.org/sicpjs/5.1) we saw how to transform simple JavaScript programs into descriptions of register machines. We will now perform this transformation on a more complex program, the metacircular evaluator of sections [4.1.1](https://sourceacademy.org/sicpjs/4.1.1)–[4.1.4](https://sourceacademy.org/sicpjs/4.1.4), which shows how the behavior of a JavaScript interpreter can be described in terms of the functions `evaluate` and `apply`. The _explicit-control evaluator_ that we develop in this section shows how the underlying function-calling and argument-passing mechanisms used in the evaluation process can be described in terms of operations on registers and stacks. In addition, the explicit-control evaluator can serve as an implementation of a JavaScript interpreter, written in a language that is very similar to the native machine language of conventional computers. The evaluator can be executed by the register-machine simulator of section [5.2](https://sourceacademy.org/sicpjs/5.2). Alternatively, it can be used as a starting point for building a machine-language implementation of a JavaScript evaluator, or even a special-purpose machine for evaluating JavaScript programs. Figure [5.16](https://sourceacademy.org/sicpjs/5.4#fig-5.16) shows such a hardware implementation: a silicon chip that acts as an evaluator for Scheme, the language used in place of JavaScript in the original edition of this book. The chip designers started with the data-path and controller specifications for a register machine similar to the evaluator described in this section and used design automation programs to construct the integrated-circuit layout.[1](https://sourceacademy.org/sicpjs/5.4#footnote-1)

[](https://sourceacademy.org/sicpjs/5.4#fig-5.16)

![#fig-5.16](https://sicp.sourceacademy.org/img_original/chip.png)

##### Figure 5.16 A silicon-chip implementation of an evaluator for Scheme.

[](https://sourceacademy.org/sicpjs/5.4#h1)

## Registers and operations

[](https://sourceacademy.org/sicpjs/5.4#p2)

In designing the explicit-control evaluator, we must specify the operations to be used in our register machine. We described the metacircular evaluator in terms of abstract syntax, using functions such as `is_literal` and `make_function`. In implementing the register machine, we could expand these functions into sequences of elementary list-structure memory operations, and implement these operations on our register machine. However, this would make our evaluator very long, obscuring the basic structure with details. To clarify the presentation, we will include as primitive operations of the register machine the syntax functions given in section [4.1.2](https://sourceacademy.org/sicpjs/4.1.2) and the functions for representing environments and other runtime data given in sections [4.1.3](https://sourceacademy.org/sicpjs/4.1.3) and [4.1.4](https://sourceacademy.org/sicpjs/4.1.4). In order to completely specify an evaluator that could be programmed in a low-level machine language or implemented in hardware, we would replace these operations by more elementary operations, using the list-structure implementation we described in section [5.3](https://sourceacademy.org/sicpjs/5.3).

[](https://sourceacademy.org/sicpjs/5.4#p3)

Our JavaScript evaluator register machine includes a stack and seven registers: `comp`, `env`, `val`, `continue`, `fun`, `argl`, and `unev`. The `comp` register is used to hold the component to be evaluated, and `env` contains the environment in which the evaluation is to be performed. At the end of an evaluation, `val` contains the value obtained by evaluating the component in the designated environment. The `continue` register is used to implement recursion, as explained in section [5.1.4](https://sourceacademy.org/sicpjs/5.1.4). (The evaluator needs to call itself recursively, since evaluating a component requires evaluating its subcomponents.) The registers `fun`, `argl`, and `unev` are used in evaluating function applications.

[](https://sourceacademy.org/sicpjs/5.4#p4)

We will not provide a data-path diagram to show how the registers and operations of the evaluator are connected, nor will we give the complete list of machine operations. These are implicit in the evaluator's controller, which will be presented in detail.

---

[[1]](https://sourceacademy.org/sicpjs/5.4#footnote-link-1) See Batali et al. 1982 for more information on the chip and the method by which it was designed.
# 5.4.1   The Dispatcher and Basic Evaluation

[](https://sourceacademy.org/sicpjs/5.4.1#p1)

The central element in the evaluator is the sequence of instructions beginning at `eval_dispatch`. This corresponds to the `evaluate`function of the metacircular evaluator described in section [4.1.1](https://sourceacademy.org/sicpjs/4.1.1). When the controller starts at `eval_dispatch`, it evaluates the component specified by `comp` in the environment specified by `env`. When evaluation is complete, the controller will go to the entry point stored in `continue`, and the `val` register will hold the value of the component. As with the metacircular `evaluate`, the structure of `eval_dispatch` is a case analysis on the syntactic type of the component to be evaluated.[1](https://sourceacademy.org/sicpjs/5.4.1#footnote-1)

"eval_dispatch",
  test(list(op("is_literal"), reg("comp"))),
  branch(label("ev_literal")),
  test(list(op("is_name"), reg("comp"))),
  branch(label("ev_name")),
  test(list(op("is_application"), reg("comp"))),
  branch(label("ev_application")),
  test(list(op("is_operator_combination"), reg("comp"))),
  branch(label("ev_operator_combination")),
  test(list(op("is_conditional"), reg("comp"))),
  branch(label("ev_conditional")),
  test(list(op("is_lambda_expression"), reg("comp"))),
  branch(label("ev_lambda")),
  test(list(op("is_sequence"), reg("comp"))),
  branch(label("ev_sequence")),
  test(list(op("is_block"), reg("comp"))),
  branch(label("ev_block")),
  test(list(op("is_return_statement"), reg("comp"))),
  branch(label("ev_return")),
  test(list(op("is_function_declaration"), reg("comp"))),
  branch(label("ev_function_declaration")),
  test(list(op("is_declaration"), reg("comp"))),
  branch(label("ev_declaration")),
  test(list(op("is_assignment"), reg("comp"))),
  branch(label("ev_assignment")),
  go_to(label("unknown_component_type")),

[](https://sourceacademy.org/sicpjs/5.4.1#h1)

## Evaluating simple expressions

[](https://sourceacademy.org/sicpjs/5.4.1#p2)

Numbers and strings, names, and lambda expressions have no subexpressions to be evaluated. For these, the evaluator simply places the correct value in the `val` register and continues execution at the entry point specified by `continue`. Evaluation of simple expressions is performed by the following controller code:

"ev_literal",
  assign("val", list(op("literal_value"), reg("comp"))),
  go_to(reg("continue")),

"ev_name",
  assign("val", list(op("symbol_of_name"), reg("comp"), reg("env"))),
  assign("val", list(op("lookup_symbol_value"),
                     reg("val"), reg("env"))),
  go_to(reg("continue")),

"ev_lambda",
  assign("unev", list(op("lambda_parameter_symbols"), reg("comp"))),
  assign("comp", list(op("lambda_body"), reg("comp"))),
  assign("val", list(op("make_function"),
                     reg("unev"), reg("comp"), reg("env"))),
  go_to(reg("continue")),

Observe how `ev_lambda` uses the `unev` and `comp` registers to hold the parameters and body of the lambda expression so that they can be passed to the `make_function` operation, along with the environment in `env`.

[](https://sourceacademy.org/sicpjs/5.4.1#h2)

## Conditionals

[](https://sourceacademy.org/sicpjs/5.4.1#p3)

As with the metacircular evaluator, syntactic forms are handled by selectively evaluating fragments of the component. For a conditional, we must evaluate the predicate and decide, based on the value of predicate, whether to evaluate the consequent or the alternative.

[](https://sourceacademy.org/sicpjs/5.4.1#p4)

Before evaluating the predicate, we save the conditional itself, which is in `comp`, so that we can later extract the consequent or alternative. To evaluate the predicate expression, we move it to the `comp` register and go to `eval_dispatch`. The environment in the `env` register is already the correct one in which to evaluate the predicate. However, we save `env` because we will need it later to evaluate the consequent or the alternative. We set up `continue` so that evaluation will resume at `ev_conditional_decide` after the predicate has been evaluated. First, however, we save the old value of `continue`, which we will need later in order to return to the evaluation of the statement that is waiting for the value of the conditional.

"ev_conditional",
  save("comp"), // save conditional for later
  save("env"),
  save("continue"),
  assign("continue", label("ev_conditional_decide")),
  assign("comp", list(op("conditional_predicate"), reg("comp"))),
  go_to(label("eval_dispatch")), // evaluate the predicate

[](https://sourceacademy.org/sicpjs/5.4.1#p5)

When we resume at `ev_conditional_decide` after evaluating the predicate, we test whether it was true or false and, depending on the result, place either the consequent or the alternative in `comp` before going to `eval_dispatch`.[2](https://sourceacademy.org/sicpjs/5.4.1#footnote-2) Notice that restoring `env` and `continue` here sets up `eval_dispatch` to have the correct environment and to continue at the right place to receive the value of the conditional.

"ev_conditional_decide",
  restore("continue"),
  restore("env"),
  restore("comp"),
  test(list(op("is_falsy"), reg("val"))),
  branch(label("ev_conditional_alternative")),
"ev_conditional_consequent",
  assign("comp", list(op("conditional_consequent"), reg("comp"))),
  go_to(label("eval_dispatch")),
"ev_conditional_alternative",
  assign("comp", list(op("conditional_alternative"), reg("comp"))),
  go_to(label("eval_dispatch")),

[](https://sourceacademy.org/sicpjs/5.4.1#h3)

## Sequence Evaluation

[](https://sourceacademy.org/sicpjs/5.4.1#p6)

The portion of the explicit-control evaluator beginning at `ev_sequence`, which handles sequences of statements, is analogous to the metacircular evaluator's `eval_sequence` function.

[](https://sourceacademy.org/sicpjs/5.4.1#p7)

The entries at `ev_sequence_next` and `ev_sequence_continue` form a loop that successively evaluates each statement in a sequence. The list of unevaluated statements is kept in `unev`. At `ev_sequence` we place the sequence of statements to be evaluated in `unev`. If the sequence is empty, we set `val` to `undefined` and jump to `continue` via `ev_sequence_empty`. Otherwise we start the sequence-evaluation loop, first saving the value of `continue` on the stack, because the `continue` register will be used for local flow of control in the loop, and the original value is needed for continuing after the statement sequence. Before evaluating each statement, we check to see if there are additional statements to be evaluated in the sequence. If so, we save the rest of the unevaluated statements (held in `unev`) and the environment in which these must be evaluated (held in `env`) and call `eval_dispatch` to evaluate the statement, which has been placed in `comp`. The two saved registers are restored after this evaluation, at `ev_sequence_continue`.

[](https://sourceacademy.org/sicpjs/5.4.1#p8)

The final statement in the sequence is handled differently, at the entry point `ev_sequence_last_statement`. Since there are no more statements to be evaluated after this one, we need not save `unev` or `env` before going to `eval_dispatch`. The value of the whole sequence is the value of the last statement, so after the evaluation of the last statement there is nothing left to do except continue at the entry point that was saved at `ev_sequence`. Rather than setting up `continue` to arrange for `eval_dispatch` to return here and then restoring `continue` from the stack and continuing at that entry point, we restore `continue` from the stack before going to `eval_dispatch`, so that `eval_dispatch` will continue at that entry point after evaluating the statement.

"ev_sequence",
  assign("unev", list(op("sequence_statements"), reg("comp"))),
  test(list(op("is_empty_sequence"), reg("unev"))), 
  branch(label("ev_sequence_empty")),
  save("continue"),
"ev_sequence_next",
  assign("comp", list(op("first_statement"), reg("unev"))),
  test(list(op("is_last_statement"), reg("unev"))),
  branch(label("ev_sequence_last_statement")),
  save("unev"),
  save("env"),
  assign("continue", label("ev_sequence_continue")),
  go_to(label("eval_dispatch")),
"ev_sequence_continue",
  restore("env"),
  restore("unev"),
  assign("unev", list(op("rest_statements"), reg("unev"))),
  go_to(label("ev_sequence_next")),
"ev_sequence_last_statement",
  restore("continue"),
  go_to(label("eval_dispatch")),

"ev_sequence_empty",
  assign("val", constant(undefined)),
  go_to(reg("continue")),

[](https://sourceacademy.org/sicpjs/5.4.1#p9)

Unlike `eval_sequence` in the metacircular evaluator, `ev_sequence` does not need to check whether a return statement was evaluated so as to terminate the sequence evaluation. The "explicit control" in this evaluator allows a return statement to jump directly to the continuation of the current function application without resuming the sequence evaluation. Thus sequence evaluation does not need to be concerned with returns, or even be aware of the existence of return statements in the language. Because a return statement jumps out of the sequence-evaluation code, the restores of saved registers at `ev_sequence_continue` won't be executed. We will see later how the return statement removes these values from the stack.

---

[[1]](https://sourceacademy.org/sicpjs/5.4.1#footnote-link-1) In our controller, the dispatch is written as a sequence of `test` and `branch` instructions. Alternatively, it could have been written in a data-directed style, which avoids the need to perform sequential tests and facilitates the definition of new component types.

[[2]](https://sourceacademy.org/sicpjs/5.4.1#footnote-link-2) In this chapter, we will use the function `is_falsy` to test the value of the predicate. This allows us to write the consequent and alternative branches in the same order as in a conditional, and simply fall through to the consequent branch when the predicate holds. The function `is_falsy` is declared as the opposite of the `is_truthy` function used to test predicates of conditionals in section [4.1.1](https://sourceacademy.org/sicpjs/4.1.1).
# 5.4.2   Evaluating Function Applications

[](https://sourceacademy.org/sicpjs/5.4.2#p1)

A function application is specified by a combination containing a function expression and argument expressions. The function expression is a subexpression whose value is a function, and the argument expressions are subexpressions whose values are the arguments to which the function should be applied. The metacircular `evaluate` handles applications by calling itself recursively to evaluate each element of the combination, and then passing the results to `apply`, which performs the actual function application. The explicit-control evaluator does the same thing; these recursive calls are implemented by `go_to` instructions, together with use of the stack to save registers that will be restored after the recursive call returns. Before each call we will be careful to identify which registers must be saved (because their values will be needed later).[1](https://sourceacademy.org/sicpjs/5.4.2#footnote-1)

[](https://sourceacademy.org/sicpjs/5.4.2#p2)

As in the metacircular evaluator, operator combinations are transformed into applications of primitive functions corresponding to the operators. This takes place at `ev_operator_combination`, which performs this transformation in place in `comp` and falls through to `ev_application`.[2](https://sourceacademy.org/sicpjs/5.4.2#footnote-2)

[](https://sourceacademy.org/sicpjs/5.4.2#p3)

We begin the evaluation of an application by evaluating the function expression to produce a function, which will later be applied to the evaluated argument expressions. To evaluate the function expression, we move it to the `comp` register and go to `eval_dispatch`. The environment in the `env` register is already the correct one in which to evaluate the function expression. However, we save `env` because we will need it later to evaluate the argument expressions. We also extract the argument expressions into `unev` and save this on the stack. We set up `continue` so that `eval_dispatch` will resume at `ev_appl_did_function_expression` after the function expression has been evaluated. First, however, we save the old value of `continue`, which tells the controller where to continue after the application.

"ev_operator_combination",
  assign("comp", list(op("operator_combination_to_application"),
                      reg("comp"), reg("env"))),
"ev_application",
  save("continue"),
  save("env"),
  assign("unev", list(op("arg_expressions"), reg("comp"))),
  save("unev"),
  assign("comp", list(op("function_expression"), reg("comp"))),
  assign("continue", label("ev_appl_did_function_expression")),
  go_to(label("eval_dispatch")),

[](https://sourceacademy.org/sicpjs/5.4.2#p4)

Upon returning from evaluating the function expression, we proceed to evaluate the argument expressions of the application and to accumulate the resulting arguments in a list, held in `argl`. (This is like the evaluation of a sequence of statements, except that we collect the values.) First we restore the unevaluated argument expressions and the environment. We initialize `argl` to an empty list. Then we assign to the `fun` register the function that was produced by evaluating the function expression. If there are no argument expressions, we go directly to `apply_dispatch`. Otherwise we save `fun` on the stack and start the argument-evaluation loop:[3](https://sourceacademy.org/sicpjs/5.4.2#footnote-3)

"ev_appl_did_function_expression",
  restore("unev"), // the argument expressions
  restore("env"),
  assign("argl", list(op("empty_arglist"))),
  assign("fun", reg("val")), // the function
  test(list(op("is_null"), reg("unev"))),
  branch(label("apply_dispatch")),
  save("fun"),

[](https://sourceacademy.org/sicpjs/5.4.2#p5)

Each cycle of the argument-evaluation loop evaluates an argument expression from the list in `unev` and accumulates the result into `argl`. To evaluate an argument expression, we place it in the `comp` register and go to `eval_dispatch`, after setting `continue` so that execution will resume with the argument-accumulation phase. But first we save the arguments accumulated so far (held in `argl`), the environment (held in `env`), and the remaining argument expressions to be evaluated (held in `unev`). A special case is made for the evaluation of the last argument expression, which is handled at `ev_appl_last_arg`.

"ev_appl_argument_expression_loop",
  save("argl"),
  assign("comp", list(op("head"), reg("unev"))),
  test(list(op("is_last_argument_expression"), reg("unev"))),
  branch(label("ev_appl_last_arg")),
  save("env"),
  save("unev"),
  assign("continue", label("ev_appl_accumulate_arg")),
  go_to(label("eval_dispatch")),

[](https://sourceacademy.org/sicpjs/5.4.2#p6)

When an argument expression has been evaluated, the value is accumulated into the list held in `argl`. The argument expression is then removed from the list of unevaluated argument expressions in `unev`, and the argument-evaluation loop continues.

"ev_appl_accumulate_arg",
  restore("unev"),
  restore("env"),
  restore("argl"),
  assign("argl", list(op("adjoin_arg"), reg("val"), reg("argl"))),
  assign("unev", list(op("tail"), reg("unev"))),
  go_to(label("ev_appl_argument_expression_loop")),

[](https://sourceacademy.org/sicpjs/5.4.2#p7)

Evaluation of the last argument expression is handled differently, as is the last statement in a sequence. There is no need to save the environment or the list of unevaluated argument expressions before going to `eval_dispatch`, since they will not be required after the last argument expression is evaluated. Thus, we return from the evaluation to a special entry point `ev_appl_accum_last_arg`, which restores the argument list, accumulates the new argument, restores the saved function, and goes off to perform the application.[4](https://sourceacademy.org/sicpjs/5.4.2#footnote-4)

"ev_appl_last_arg",
  assign("continue", label("ev_appl_accum_last_arg")),
  go_to(label("eval_dispatch")),
"ev_appl_accum_last_arg",
  restore("argl"),
  assign("argl", list(op("adjoin_arg"), reg("val"), reg("argl"))),
  restore("fun"),
  go_to(label("apply_dispatch")),

[](https://sourceacademy.org/sicpjs/5.4.2#p8)

The details of the argument-evaluation loop determine the order in which the interpreter evaluates the argument expressions of a combination (e.g., left to right or right to left—see exercise ). This order is not determined by the metacircular evaluator, which inherits its control structure from the underlying JavaScript in which it is implemented.[5](https://sourceacademy.org/sicpjs/5.4.2#footnote-5) Because we use `head` in `ev_appl_argument_expression_loop` to extract successive argument expressions from `unev` and `tail` at `ev_appl_accumulate_arg` to extract the rest of the argument expressions, the explicit-control evaluator will evaluate the argument expressions of a combination in left-to-right order, as required by the ECMAScript specification.

[](https://sourceacademy.org/sicpjs/5.4.2#h1)

## Function Application

[](https://sourceacademy.org/sicpjs/5.4.2#p9)

The entry point `apply_dispatch` corresponds to the `apply` function of the metacircular evaluator. By the time we get to `apply_dispatch`, the `fun` register contains the function to apply and `argl` contains the list of evaluated arguments to which it must be applied. The saved value of `continue` (originally passed to `eval_dispatch` and saved at `ev_application`), which tells where to return with the result of the function application, is on the stack. When the application is complete, the controller transfers to the entry point specified by the saved `continue`, with the result of the application in `val`. As with the metacircular `apply`, there are two cases to consider. Either the function to be applied is a primitive or it is a compound function.

"apply_dispatch",
  test(list(op("is_primitive_function"), reg("fun"))),
  branch(label("primitive_apply")),
  test(list(op("is_compound_function"), reg("fun"))),
  branch(label("compound_apply")),
  go_to(label("unknown_function_type")),

[](https://sourceacademy.org/sicpjs/5.4.2#p10)

We assume that each primitive is implemented so as to obtain its arguments from `argl` and place its result in `val`. To specify how the machine handles primitives, we would have to provide a sequence of controller instructions to implement each primitive and arrange for `primitive_apply` to dispatch to the instructions for the primitive identified by the contents of `fun`. Since we are interested in the structure of the evaluation process rather than the details of the primitives, we will instead just use an `apply_primitive_function` operation that applies the function in fun to the arguments in `argl`. For the purpose of simulating the evaluator with the simulator of section [5.2](https://sourceacademy.org/sicpjs/5.2) we use the function `apply_primitive_function`, which calls on the underlying JavaScript system to perform the application, just as we did for the metacircular evaluator in section [4.1.1](https://sourceacademy.org/sicpjs/4.1.1). After computing the value of the primitive application, we restore `continue` and go to the designated entry point.

"primitive_apply",
  assign("val", list(op("apply_primitive_function"),
                     reg("fun"), reg("argl"))),
  restore("continue"),
  go_to(reg("continue")),

[](https://sourceacademy.org/sicpjs/5.4.2#p11)

The sequence of instructions labeled `compound_apply` specifies the application of compound functions. To apply a compound function, we proceed in a way similar to what we did in the metacircular evaluator. We construct a frame that binds the function's parameters to the arguments, use this frame to extend the environment carried by the function, and evaluate in this extended environment the body of the function.

[](https://sourceacademy.org/sicpjs/5.4.2#p12)

At this point the compound function is in register `fun` and its arguments are in `argl`. We extract the function's parameters into `unev` and its environment into `env`. We then replace the environment in `env` with the environment constructed by extending it with bindings of the parameters to the given arguments. We then extract the body of the function into `comp`. The natural next step would be to restore the saved `continue` and proceed to `eval_dispatch` to evaluate the body and go to the restored continuation with the result in `val`, as is done for the last statement of a sequence. But there is a complication!

[](https://sourceacademy.org/sicpjs/5.4.2#p13)

The complication has two aspects. One is that at any point in the evaluation of the body, a return statement may require the function to return the value of the return expression as the value of the body. But a return statement may be nested arbitrarily deeply in the body; so the stack at the moment the return statement is encountered is not necessarily the stack that is needed for a return from the function. One way to make it possible to adjust the stack for the return is to put a \emph{marker} on the stack that can be found by the return code. This is implemented by the `push_marker_to_stack` instruction. The return code can then use the `revert_stack_to_marker` instruction to restore the stack to the place indicated by the marker before evaluating the return expression.[6](https://sourceacademy.org/sicpjs/5.4.2#footnote-6)

[](https://sourceacademy.org/sicpjs/5.4.2#p14)

The other aspect of the complication is that if the evaluation of the body terminates without executing a return statement, the value of the body must be `undefined`. To handle this, we set up the `continue` register to point to the entry point `return_undefined` before going off to `eval_dispatch` to evaluate the body. If a return statement is not encountered during evaluation of the body, evaluation of the body will continue at `return_undefined`.

"compound_apply",
  assign("unev", list(op("function_parameters"), reg("fun"))),
  assign("env", list(op("function_environment"), reg("fun"))),
  assign("env", list(op("extend_environment"), 
                     reg("unev"), reg("argl"), reg("env"))),
  assign("comp", list(op("function_body"), reg("fun"))),
  push_marker_to_stack(),
  assign("continue", label("return_undefined")),
  go_to(label("eval_dispatch")),

[](https://sourceacademy.org/sicpjs/5.4.2#p15)

The only places in the interpreter where the `env` register is assigned a new value are `compound_apply` and `ev_block` (section [5.4.3](https://sourceacademy.org/sicpjs/5.4.3)). Just as in the metacircular evaluator, the new environment for evaluation of a function body is constructed from the environment carried by the function, together with the argument list and the corresponding list of names to be bound.

[](https://sourceacademy.org/sicpjs/5.4.2#p16)

When a return statement is evaluated at `ev_return`, we use the `revert_stack_to_marker` instruction to restore the stack to its state at the beginning of the function call by removing all values from the stack down to and including the marker. As a consequence, `restore("continue")` will restore the continuation of the function call, which was saved at `ev_application`. We then proceed to evaluate the return expression, whose result will be placed in `val` and thus be the value returned from the function when we continue after the evaluation of the return expression.

"ev_return",
  revert_stack_to_marker(),
  restore("continue"),
  assign("comp", list(op("return_expression"), reg("comp"))),
  go_to(label("eval_dispatch")),

[](https://sourceacademy.org/sicpjs/5.4.2#p17)

If no return statement is encountered during evaluation of the function body, evaluation continues at `return_undefined`, the continuation that was set up at `compound_apply`. To return `undefined` from the function, we put `undefined` into `val` and go to the entry point that was put onto the stack at `ev_application`. Before we can restore that continuation from the stack, however, we must remove the marker that was saved at `compound_apply`.

"return_undefined",
  revert_stack_to_marker(),
  restore("continue"),
  assign("val", constant(undefined)),
  go_to(reg("continue")),

[](https://sourceacademy.org/sicpjs/5.4.2#h2)

## Return Statements and Tail Recursion

[](https://sourceacademy.org/sicpjs/5.4.2#p18)

In chapter [1](https://sourceacademy.org/sicpjs/1) we said that the process described by a function such as

function sqrt_iter(guess, x) {
    return is_good_enough(guess, x)
           ? guess
           : sqrt_iter(improve(guess, x), x);
}

is an iterative process. Even though the function is syntactically recursive (defined in terms of itself), it is not logically necessary for an evaluator to save information in passing from one call to `sqrt_iter` to the next.[7](https://sourceacademy.org/sicpjs/5.4.2#footnote-7) An evaluator that can execute a function such as `sqrt_iter` without requiring increasing storage as the function continues to call itself is called a _tail-recursive_ evaluator.

[](https://sourceacademy.org/sicpjs/5.4.2#p19)

The metacircular implementation of the evaluator in chapter [4](https://sourceacademy.org/sicpjs/4) isn't tail-recursive. It implements a return statement as a constructor of a return value object containing the value to be returned and inspects the result of a function call to see whether it is such an object. If the evaluation of a function body produces a return value object, the return value of the function is the contents of that object; otherwise, the return value is `undefined`. Both the construction of the return value object and the eventual inspection of the result of the function call are deferred operations, which lead to an accumulation of information on the stack.

[](https://sourceacademy.org/sicpjs/5.4.2#p20)

Our explicit-control evaluator _is_ tail-recursive, because it does not need to wrap up return values for inspection and thus avoids the buildup of stack from deferred operations. At `ev_return`, in order to evaluate the expression that computes the return value of a function, we transfer directly to `eval_dispatch` with nothing more on the stack than right before the function call. We accomplish this by undoing any saves to the stack by the function (which are useless because we are returning) using `revert_stack_to_marker`. Then, rather than arranging for `eval_dispatch` to come back here and _then_ restoring `continue` from the stack and continuing at that entry point, we restore `continue` from the stack _before_ going to `eval_dispatch` so that `eval_dispatch` will continue at that entry point after evaluating the expression. Finally, we transfer to `eval_dispatch` without saving any information on the stack. Thus, when we proceed to evaluate a return expression, the stack is the same as just before the call to the function whose return value we are about to compute. Hence, evaluating a return expression—even if it is a function call (as in `sqrt_iter`, where the conditional expression reduces to a call to `sqrt_iter`)—will not cause any information to accumulate on the stack.[8](https://sourceacademy.org/sicpjs/5.4.2#footnote-8)

[](https://sourceacademy.org/sicpjs/5.4.2#p21)

If we did not think to take advantage of the fact that it is unnecessary to hold on to the useless information on the stack while evaluating a return expression, we might have taken the straightforward approach of evaluating the return expression, coming back to restore the stack, and finally continuing at the entry point that is waiting for the result of the function call:

"ev_return",  // alternative implementation: not tail-recursive
  assign("comp", list(op("return_expression"), reg("comp"))),
  assign("continue", label("ev_restore_stack")),
  go_to(label("eval_dispatch")),
"ev_restore_stack",
  revert_stack_to_marker(),    // undo saves in current function
  restore("continue"),         // undo save at ev_applicationev_application
  go_to(reg("continue")),
          

[](https://sourceacademy.org/sicpjs/5.4.2#p22)

This may seem like a minor change to our previous code for evaluation of return statements: The only difference is that we delay undoing any register saves to the stack until after the evaluation of the return expression. The interpreter will still give the same value for any expression. But this change is fatal to the tail-recursive implementation, because we must now come back after evaluating the return expression in order to undo the (useless) register saves. These extra saves will accumulate during a nest of function calls. Consequently, processes such as `sqrt_iter` will require space proportional to the number of iterations rather than requiring constant space. This difference can be significant. For example, with tail recursion, an infinite loop can be expressed using only the function-call and return mechanisms:

function count(n) {
    display(n);
    return count(n + 1);
}

Without tail recursion, such a function would eventually run out of stack space, and expressing a true iteration would require some control mechanism other than function call.

[](https://sourceacademy.org/sicpjs/5.4.2#p23)

Note that our JavaScript implementation requires the use of `return` in order to be tail-recursive. Because the undoing of the register saves takes place at `ev_return`, removing `return` from the `count` function above will cause it to eventually run out of stack space. This explains the use of `return` in the infinite driver loops in chapter [4](https://sourceacademy.org/sicpjs/4).

[](https://sourceacademy.org/sicpjs/5.4.2#ex-5.22)

**Exercise 5.22**

Explain how the stack builds up if `return` is removed from `count`:

function count(n) {
    display(n);
    count(n + 1);
}

Show Solution

[](https://sourceacademy.org/sicpjs/5.4.2#ex-5.23)

**Exercise 5.23**

Implement the equivalent of `push_marker_to_stack` by using `save` at `compound_apply` to store a special marker value on the stack. Implement the equivalent of `revert_stack_to_marker` at `ev_return` and `return_undefined` as a loop that repeatedly performs a `restore` until it hits the marker. Note that this will require restoring a value to a register other than the one it was saved from. (Although we are careful to avoid that in our evaluator, our stack implementation actually allows it. See exercise [5.10](https://sourceacademy.org/sicpjs/5.2.3#ex-5.10).) This is necessary because the only way to pop from the stack is by restoring to a register. Hint: You will need to create a unique constant to serve as the marker, for example with `const marker = list("marker")`. Because `list` creates a new pair, it cannot be `===` to anything else on the stack.

Show Solution

[](https://sourceacademy.org/sicpjs/5.4.2#ex-5.24)

**Exercise 5.24**

Implement `push_marker_to_stack` and `revert_stack_to_marker` as register-machine instructions, following the implementation of `save` and `restore` in section [5.2.3](https://sourceacademy.org/sicpjs/5.2.3). Add functions `push_marker` and `pop_marker` to access stacks, mirroring the implementation of `push` and `pop` in section [5.2.1](https://sourceacademy.org/sicpjs/5.2.1). Note that you do not need to actually insert a marker into the stack. Instead, you can add a local state variable to the stack model to keep track of the position of the last `save` before each `push_marker_to_stack`. If you choose to put a marker on the stack, see the hint in exercise [5.23](https://sourceacademy.org/sicpjs/5.4.2#ex-5.23).

Show Solution

---

[[1]](https://sourceacademy.org/sicpjs/5.4.2#footnote-link-1) This is an important but subtle point in translating algorithms from a procedural language, such as JavaScript, to a register-machine language. As an alternative to saving only what is needed, we could save all the registers (except `val`) before each recursive call. This is called a _framed-stack_ discipline. This would work but might save more registers than necessary; this could be an important consideration in a system where stack operations are expensive. Saving registers whose contents will not be needed later may also hold on to useless data that could otherwise be garbage-collected, freeing space to be reused.

[[2]](https://sourceacademy.org/sicpjs/5.4.2#footnote-link-2) We assume that the syntax transformer `operator_combination_to_application` is available as a machine operation. In an actual implementation built from scratch, we would use our explicit-control evaluator to interpret a JavaScript program that performs source-level transformations like this one and `function_decl_to_constant_decl` in a syntax phase that runs before execution.

[[3]](https://sourceacademy.org/sicpjs/5.4.2#footnote-link-3) We add to the evaluator data-structure functions in section [4.1.3](https://sourceacademy.org/sicpjs/4.1.3) the following two functions for manipulating argument lists:

function empty_arglist() { return null; }

function adjoin_arg(arg, arglist) {
    return append(arglist, list(arg));
}

We also make use of an additional syntax function to test for the last argument expression in an application:

function is_last_argument_expression(arg_expression) {
    return is_null(tail(arg_expression));
}

[[4]](https://sourceacademy.org/sicpjs/5.4.2#footnote-link-4) The optimization of treating the last argument expression specially is known as _evlis tail recursion_ (see Wand 1980). We could be somewhat more efficient in the argument evaluation loop if we made evaluation of the first argument expression a special case too. This would permit us to postpone initializing `argl` until after evaluating the first argument expression, so as to avoid saving `argl` in this case. The compiler in section [5.5](https://sourceacademy.org/sicpjs/5.5) performs this optimization. (Compare the `construct_arglist` function of section [5.5.3](https://sourceacademy.org/sicpjs/5.5.3).)

[[5]](https://sourceacademy.org/sicpjs/5.4.2#footnote-link-5) The order of argument-expression evaluation by the function `list_of_values` in the metacircular evaluator is determined by the order of evaluation of the arguments to `pair`, which is used to construct the argument list. The version of `list_of_values` in footnote [3](https://sourceacademy.org/sicpjs/4.1.1#footnote-3) of section [4.1](https://sourceacademy.org/sicpjs/4.1) calls `pair` directly; the version in the text uses `map`, which calls `pair`. (See exercise [4.1](https://sourceacademy.org/sicpjs/4.1.1#ex-4.1).)

[[6]](https://sourceacademy.org/sicpjs/5.4.2#footnote-link-6) The special instructions `push_marker_to_stack` and `revert_stack_to_marker` are not strictly necessary and could be implemented by explicitly pushing and popping a marker value onto and off the stack. Anything that could not be confused with a value in the program can be used as a marker. See exercise [5.23](https://sourceacademy.org/sicpjs/5.4.2#ex-5.23).

[[7]](https://sourceacademy.org/sicpjs/5.4.2#footnote-link-7) We saw in section [5.1](https://sourceacademy.org/sicpjs/5.1) how to implement such a process with a register machine that had no stack; the state of the process was stored in a fixed set of registers.

[[8]](https://sourceacademy.org/sicpjs/5.4.2#footnote-link-8) This implementation of tail recursion is one variety of a well-known optimization technique used by many compilers. In compiling a function that ends with a function call, one can replace the call by a jump to the called function's entry point. Building this strategy into the interpreter, as we have done in this section, provides the optimization uniformly throughout the language.
# 5.4.3   Blocks, Assignments, and Declarations

[](https://sourceacademy.org/sicpjs/5.4.3#h1)

## Blocks

[](https://sourceacademy.org/sicpjs/5.4.3#p1)

The body of a block is evaluated with respect to the current environment extended by a frame that binds all local names to the value `"*unassigned*"`. We temporarily make use of the `val` register to hold the list of all variables declared in the block, which is obtained by `scan_out_declarations` from section [4.1.1](https://sourceacademy.org/sicpjs/4.1.1). The functions `scan_out_declarations` and `list_of_unassigned` are assumed to be available as machine operations.[1](https://sourceacademy.org/sicpjs/5.4.3#footnote-1)

"ev_block",
  assign("comp", list(op("block_body"), reg("comp"))),
  assign("val", list(op("scan_out_declarations"), reg("comp"))),

  save("comp"),    // so we can use it to temporarily hold *unassigned**unassigned* values
  assign("comp", list(op("list_of_unassigned"), reg("val"))),
  assign("env", list(op("extend_environment"),
                     reg("val"), reg("comp"), reg("env"))),
  restore("comp"), // the block body
  go_to(label("eval_dispatch")),
	  

[](https://sourceacademy.org/sicpjs/5.4.3#h2)

## Assignments and declarations

[](https://sourceacademy.org/sicpjs/5.4.3#p2)

Assignments are handled by `ev_assignment`, reached from `eval_dispatch` with the assignment expression in `comp`. The code at `ev_assignment` first evaluates the value part of the expression and then installs the new value in the environment. The function `assign_symbol_value` is assumed to be available as a machine operation.

"ev_assignment",
  assign("unev", list(op("assignment_symbol"), reg("comp"))),
  save("unev"), // save variable for later
  assign("comp", list(op("assignment_value_expression"), reg("comp"))),
  save("env"),
  save("continue"),
  assign("continue", label("ev_assignment_install")),
  go_to(label("eval_dispatch")), // evaluate assignment value
"ev_assignment_install",
  restore("continue"),
  restore("env"),
  restore("unev"),
  perform(list(op("assign_symbol_value"),
               reg("unev"), reg("val"), reg("env"))),
  go_to(reg("continue")),

[](https://sourceacademy.org/sicpjs/5.4.3#p3)

Declarations of variables and constants are handled in a similar way. Note that whereas the value of an assignment is the value that was assigned, the value of a declaration is `undefined`. This is handled by setting `val` to `undefined` before continuing. As in the metacircular evaluator, we transform a function declaration into a constant declaration whose value expression is a lambda expression. This happens at `ev_function_declaration`, which makes the transformation in place in `comp` and falls through to `ev_declaration`.

"ev_function_declaration",
  assign("comp", 
         list(op("function_decl_to_constant_decl"), reg("comp"))),
"ev_declaration",
  assign("unev", list(op("declaration_symbol"), reg("comp"))),
  save("unev"), // save declared name
  assign("comp",
         list(op("declaration_value_expression"), reg("comp"))),
  save("env"),
  save("continue"),
  assign("continue", label("ev_declaration_assign")),
  go_to(label("eval_dispatch")), // evaluate declaration value
"ev_declaration_assign",
  restore("continue"),
  restore("env"),
  restore("unev"),
  perform(list(op("assign_symbol_value"),
               reg("unev"), reg("val"), reg("env"))),
  assign("val", constant(undefined)),
  go_to(reg("continue")),

[](https://sourceacademy.org/sicpjs/5.4.3#ex-5.25)

**Exercise 5.25**

Extend the evaluator to handle while loops, by translating them to applications of a function `while_loop`, as shown in exercise [4.7](https://sourceacademy.org/sicpjs/4.1.2#ex-4.7). You can paste the declaration of the function `while_loop` in front of user programs. You may "cheat" by assuming that the syntax transformer `while_to_application` is available as a machine operation. Refer to exercise [4.7](https://sourceacademy.org/sicpjs/4.1.2#ex-4.7) to discuss whether this approach works if return, break, and continue statements are allowed inside the while loop. If not, how can you modify the explicit-control evaluator to run programs with while loops that include these statements?

Show Solution

[](https://sourceacademy.org/sicpjs/5.4.3#ex-5.26)

**Exercise 5.26**

Modify the evaluator so that it uses normal-order evaluation, based on the lazy evaluator of section [4.2](https://sourceacademy.org/sicpjs/4.2).

Show Solution

---

[[1]](https://sourceacademy.org/sicpjs/5.4.3#footnote-link-1) Footnote [2](https://sourceacademy.org/sicpjs/5.4.2#footnote-2) suggests that an actual implementation would perform syntax transformations before program execution. In the same vein, names declared in blocks should be scanned out in a preprocessing step rather than each time a block is evaluated.

# 5.4.4   Running the Evaluator

[](https://sourceacademy.org/sicpjs/5.4.4#p1)

With the implementation of the explicit-control evaluator we come to the end of a development, begun in chapter [1](https://sourceacademy.org/sicpjs/1), in which we have explored successively more precise models of the evaluation process. We started with the relatively informal substitution model, then extended this in chapter [3](https://sourceacademy.org/sicpjs/3) to the environment model, which enabled us to deal with state and change. In the metacircular evaluator of chapter [4](https://sourceacademy.org/sicpjs/4), we used JavaScript itself as a language for making more explicit the environment structure constructed during evaluation of an component. Now, with register machines, we have taken a close look at the evaluator's mechanisms for storage management, argument passing, and control. At each new level of description, we have had to raise issues and resolve ambiguities that were not apparent at the previous, less precise treatment of evaluation. To understand the behavior of the explicit-control evaluator, we can simulate it and monitor its performance.

[](https://sourceacademy.org/sicpjs/5.4.4#p2)

We will install a driver loop in our evaluator machine. This plays the role of the `driver_loop`function of section [4.1.4](https://sourceacademy.org/sicpjs/4.1.4). The evaluator will repeatedly print a prompt, read a program, evaluate the program by going to `eval_dispatch`, and print the result. If nothing is entered at the prompt, we jump to the label `evaluator_done`, which is the last entry point in the controller. The following instructions form the beginning of the explicit-control evaluator's controller sequence:[1](https://sourceacademy.org/sicpjs/5.4.4#footnote-1)

"read_evaluate_print_loop",
  perform(list(op("initialize_stack"))),
  assign("comp", list(op("user_read"),
                      constant("EC-evaluate input:"))),
  assign("comp", list(op("parse"), reg("comp"))),
  test(list(op("is_null"), reg("comp"))),
  branch(label("evaluator_done")),
  assign("env", list(op("get_current_environment"))),
  assign("val", list(op("scan_out_declarations"), reg("comp"))),
  save("comp"),    // so we can use it to temporarily hold *unassigned**unassigned* values
  assign("comp", list(op("list_of_unassigned"), reg("val"))),
  assign("env", list(op("extend_environment"), 
                     reg("val"), reg("comp"), reg("env"))),
  perform(list(op("set_current_environment"), reg("env"))),
  restore("comp"), // the program 
  assign("continue", label("print_result")),
  go_to(label("eval_dispatch")),
"print_result",
  perform(list(op("user_print"),
               constant("EC-evaluate value:"), reg("val"))),
  go_to(label("read_evaluate_print_loop")), 
  

We store the current environment, initially the global environment, in the variable `current_environment` and update it each time around the loop to remember past declarations. The operations `get_current_environment` and `set_current_environment` simply get and set this variable.

let current_environment = the_global_environment;

function get_current_environment() {
    return current_environment;
}

function set_current_environment(env) {
    current_environment = env;
}

[](https://sourceacademy.org/sicpjs/5.4.4#p3)

When we encounter an error in a function (such as the "unknown function type" error indicated at `apply_dispatch`), we print an error message and return to the driver loop.[2](https://sourceacademy.org/sicpjs/5.4.4#footnote-2)

"unknown_component_type",
  assign("val", constant("unknown syntax")),
  go_to(label("signal_error")),
      
"unknown_function_type",
  restore("continue"), // clean up stack (from apply_dispatchapply_dispatch)
  assign("val", constant("unknown function type")),
  go_to(label("signal_error")),
      
"signal_error",
  perform(list(op("user_print"),
               constant("EC-evaluator error:"), reg("val"))),
  go_to(label("read_evaluate_print_loop")),
      

[](https://sourceacademy.org/sicpjs/5.4.4#p4)

For the purposes of the simulation, we initialize the stack each time through the driver loop, since it might not be empty after an error (such as an undeclared name) interrupts an evaluation.[3](https://sourceacademy.org/sicpjs/5.4.4#footnote-3)

[](https://sourceacademy.org/sicpjs/5.4.4#p5)

If we combine all the code fragments presented in sections [5.4.1](https://sourceacademy.org/sicpjs/5.4.1)–[5.4.4](https://sourceacademy.org/sicpjs/5.4.4), we can create an evaluator machine model that we can run using the register-machine simulator of section [5.2](https://sourceacademy.org/sicpjs/5.2).

const eceval = make_machine(list("comp", "env", "val", "fun",
                                 "argl", "continue", "unev"),
                            eceval_operations,
                            list("read_evaluate_print_loop",
                                 ⟨⟨entire machine controller as given above⟩⟩
                                 "evaluator_done"));
      

We must define JavaScript functions to simulate the operations used as primitives by the evaluator. These are the same functions we used for the metacircular evaluator in section [4.1](https://sourceacademy.org/sicpjs/4.1), together with the few additional ones defined in footnotes throughout section [5.4](https://sourceacademy.org/sicpjs/5.4).

const eceval_operations = list(list("is_literal", is_literal),
                               ⟨complete   list   of  operations  for   eceval   machine⟩⟨completelistofoperationsforecevalmachine⟩);
      

[](https://sourceacademy.org/sicpjs/5.4.4#p6)

Finally, we can initialize the global environment and run the evaluator:

```javascript
const the_global_environment = setup_environment();
start(eceval); 
```

function append(x, y) {
    return is_null(x)	
           ? y
           : pair(head(x), append(tail(x), y));
}

_EC-evaluate value:
undefined_

append(list("a", "b", "c"), list("d", "e", "f"));

_EC-evaluate value:
["a", ["b", ["c", ["d", ["e", ["f", null]]]]]]_

[](https://sourceacademy.org/sicpjs/5.4.4#p7)

Of course, evaluating programs in this way will take much longer than if we had directly typed them into JavaScript, because of the multiple levels of simulation involved. Our programs are evaluated by the explicit-control-evaluator machine, which is being simulated by a JavaScript program, which is itself being evaluated by the JavaScript interpreter.

[](https://sourceacademy.org/sicpjs/5.4.4#h1)

## Monitoring the performance of the evaluator

[](https://sourceacademy.org/sicpjs/5.4.4#p8)

Simulation can be a powerful tool to guide the implementation of evaluators. Simulations make it easy not only to explore variations of the register-machine design but also to monitor the performance of the simulated evaluator. For example, one important factor in performance is how efficiently the evaluator uses the stack. We can observe the number of stack operations required to evaluate various programs by defining the evaluator register machine with the version of the simulator that collects statistics on stack use (section [5.2.4](https://sourceacademy.org/sicpjs/5.2.4)), and adding an instruction at the evaluator's `print_result` entry point to print the statistics:

"print_result",
  perform(list(op("print_stack_statistics"))), // added instruction
  // rest is same as before
  perform(list(op("user_print"),
               constant("EC-evaluate value:"), reg("val"))),
  go_to(label("read_evaluate_print_loop")),
      

Interactions with the evaluator now look like this:

function factorial (n) {
    return n === 1
           ? 1
           : factorial(n - 1) * n;
}

_total pushes = 4 
maximum depth = 3
EC-evaluate value:
undefined_

factorial(5);

_total pushes = 151 
maximum depth = 28
EC-evaluate value:
120_

Note that the driver loop of the evaluator reinitializes the stack at the start of each interaction, so that the statistics printed will refer only to stack operations used to evaluate the previous program.

[](https://sourceacademy.org/sicpjs/5.4.4#ex-5.27)

**Exercise 5.27**

Use the monitored stack to explore the tail-recursive property of the evaluator (section [5.4.2](https://sourceacademy.org/sicpjs/5.4.2)). Start the evaluator and define the iterative `factorial`function from section [1.2.1](https://sourceacademy.org/sicpjs/1.2.1):

function factorial(n) {
    function iter(product, counter) {
        return counter > n
               ? product
               : iter(counter * product,
                      counter + 1);
    }
    return iter(1, 1);
}

Run the function with some small values of nn. Record the maximum stack depth and the number of pushes required to compute n!n! for each of these values.

1. You will find that the maximum depth required to evaluate n!n! is independent of nn. What is that depth?
2. Determine from your data a formula in terms of nn for the total number of push operations used in evaluating n!n! for any n≥1n≥1. Note that the number of operations used is a linear function of nn and is thus determined by two constants.

Show Solution

[](https://sourceacademy.org/sicpjs/5.4.4#ex-5.28)

**Exercise 5.28**

For comparison with exercise [5.27](https://sourceacademy.org/sicpjs/5.4.4#ex-5.27), explore the behavior of the following function for computing factorials recursively:

function factorial(n) {
    return n === 1 
           ? 1
           : factorial(n - 1) * n;
}

By running this function with the monitored stack, determine, as a function of nn, the maximum depth of the stack and the total number of pushes used in evaluating n!n! for n≥1n≥1. (Again, these functions will be linear.) Summarize your experiments by filling in the following table with the appropriate expressions in terms of nn:

[](https://sourceacademy.org/sicpjs/5.4.4#fig-)

![#fig-](https://sicp.sourceacademy.org/img_original/527table.svg)

The maximum depth is a measure of the amount of space used by the evaluator in carrying out the computation, and the number of pushes correlates well with the time required.

Show Solution

[](https://sourceacademy.org/sicpjs/5.4.4#ex-2.89)

**Exercise 2.89**

Modify the definition of the evaluator by changing `ev_return` as described in section [5.4.2](https://sourceacademy.org/sicpjs/5.4.2) so that the evaluator is no longer tail-recursive. Rerun your experiments from exercises [5.27](https://sourceacademy.org/sicpjs/5.4.4#ex-5.27) and [5.28](https://sourceacademy.org/sicpjs/5.4.4#ex-5.28) to demonstrate that both versions of the `factorial`function now require space that grows linearly with their input.

Show Solution

[](https://sourceacademy.org/sicpjs/5.4.4#ex-5.29)

**Exercise 5.29**

Monitor the stack operations in the tree-recursive Fibonacci computation:

function fib(n) {
    return n < 2 ? n : fib(n - 1) + fib(n - 2);
}

1. Give a formula in terms of nn for the maximum depth of the stack required to compute Fib(n)Fib(n) for n≥2n≥2. Hint: In section [1.2.2](https://sourceacademy.org/sicpjs/1.2.2) we argued that the space used by this process grows linearly with nn.
2. Give a formula for the total number of pushes used to compute Fib(n)Fib(n) for n≥2n≥2. You should find that the number of pushes (which correlates well with the time used) grows exponentially with nn. Hint: Let S(n)S(n) be the number of pushes used in computing Fib(n)Fib(n). You should be able to argue that there is a formula that expresses S(n)S(n) in terms of S(n−1)S(n−1), S(n−2)S(n−2), and some fixed "overhead" constant kk that is independent of nn. Give the formula, and say what kk is. Then show that S(n)S(n) can be expressed as aFib(n+1)+baFib(n+1)+b and give the values of aa and bb.

Show Solution

[](https://sourceacademy.org/sicpjs/5.4.4#ex-5.30)

**Exercise 5.30**

Our evaluator currently catches and signals only two kinds of errors—unknown component types and unknown function types. Other errors will take us out of the evaluator read-evaluate-print loop. When we run the evaluator using the register-machine simulator, these errors are caught by the underlying JavaScript system. This is analogous to the computer crashing when a user program makes an error.[4](https://sourceacademy.org/sicpjs/5.4.4#footnote-4) It is a large project to make a real error system work, but it is well worth the effort to understand what is involved here.

1. Errors that occur in the evaluation process, such as an attempt to access an unbound name, could be caught by changing the lookup operation to make it return a distinguished condition code, which cannot be a possible value of any user name. The evaluator can test for this condition code and then do what is necessary to go to `signal_error`. Find all of the places in the evaluator where such a change is necessary and fix them. This is lots of work.
2. Much worse is the problem of handling errors that are signaled by applying primitive functions such as an attempt to divide by zero or an attempt to extract the `head` of a string. In a professionally written high-quality system, each primitive application is checked for safety as part of the primitive. For example, every call to `head` could first check that the argument is a pair. If the argument is not a pair, the application would return a distinguished condition code to the evaluator, which would then report the failure. We could arrange for this in our register-machine simulator by making each primitive function check for applicability and returning an appropriate distinguished condition code on failure. Then the `primitive_apply` code in the evaluator can check for the condition code and go to `signal_error` if necessary. Build this structure and make it work. This is a major project.

Show Solution

---

[[1]](https://sourceacademy.org/sicpjs/5.4.4#footnote-link-1) We assume here that `user_read`, `parse`, and the various printing operations are available as primitive machine operations, which is useful for our simulation, but completely unrealistic in practice. These are actually extremely complex operations. In practice, reading and printing would be implemented using low-level input-output operations such as transferring single characters to and from a device.

[[2]](https://sourceacademy.org/sicpjs/5.4.4#footnote-link-2) There are other errors that we would like the interpreter to handle, but these are not so simple. See exercise [5.30](https://sourceacademy.org/sicpjs/5.4.4#ex-5.30).

[[3]](https://sourceacademy.org/sicpjs/5.4.4#footnote-link-3) We could perform the stack initialization only after errors, but doing it in the driver loop will be convenient for monitoring the evaluator's performance, as described below.

[[4]](https://sourceacademy.org/sicpjs/5.4.4#footnote-link-4) This manifests itself as, for example, a "kernel panic" or a "blue screen of death" or even a reboot. Automatic rebooting is an approach typically used on phones and tablets. Most modern operating systems do a decent job of preventing user programs from causing an entire machine to crash.
# 5.5  Compilation

[](https://sourceacademy.org/sicpjs/5.5#p1)

The explicit-control evaluator of section [5.4](https://sourceacademy.org/sicpjs/5.4) is a register machine whose controller interprets JavaScript programs. In this section we will see how to run JavaScript programs on a register machine whose controller is not a JavaScript interpreter.

[](https://sourceacademy.org/sicpjs/5.5#p2)

The explicit-control evaluator machine is universal—it can carry out any computational process that can be described in JavaScript. The evaluator's controller orchestrates the use of its data paths to perform the desired computation. Thus, the evaluator's data paths are universal: They are sufficient to perform any computation we desire, given an appropriate controller.[1](https://sourceacademy.org/sicpjs/5.5#footnote-1)

[](https://sourceacademy.org/sicpjs/5.5#p3)

Commercial general-purpose computers are register machines organized around a collection of registers and operations that constitute an efficient and convenient universal set of data paths. The controller for a general-purpose machine is an interpreter for a register-machine language like the one we have been using. This language is called the _native language_ of the machine, or simply _machine language_. Programs written in machine language are sequences of instructions that use the machine's data paths. For example, the explicit-control evaluator's instruction sequence can be thought of as a machine-language program for a general-purpose computer rather than as the controller for a specialized interpreter machine.

[](https://sourceacademy.org/sicpjs/5.5#p4)

There are two common strategies for bridging the gap between higher-level languages and register-machine languages. The explicit-control evaluator illustrates the strategy of interpretation. An interpreter written in the native language of a machine configures the machine to execute programs written in a language (called the _source language_) that may differ from the native language of the machine performing the evaluation. The primitive functions of the source language are implemented as a library of subroutines written in the native language of the given machine. A program to be interpreted (called the _source program_) is represented as a data structure. The interpreter traverses this data structure, analyzing the source program. As it does so, it simulates the intended behavior of the source program by calling appropriate primitive subroutines from the library.

[](https://sourceacademy.org/sicpjs/5.5#p5)

In this section, we explore the alternative strategy of _compilation_. A compiler for a given source language and machine translates a source program into an equivalent program (called the _object program_) written in the machine's native language. The compiler that we implement in this section translates programs written in JavaScript into sequences of instructions to be executed using the explicit-control evaluator machine's data paths.[2](https://sourceacademy.org/sicpjs/5.5#footnote-2)

[](https://sourceacademy.org/sicpjs/5.5#p6)

Compared with interpretation, compilation can provide a great increase in the efficiency of program execution, as we will explain below in the overview of the compiler. On the other hand, an interpreter provides a more powerful environment for interactive program development and debugging, because the source program being executed is available at run time to be examined and modified. In addition, because the entire library of primitives is present, new programs can be constructed and added to the system during debugging.

[](https://sourceacademy.org/sicpjs/5.5#p7)

In view of the complementary advantages of compilation and interpretation, modern program-development environments pursue a mixed strategy. These systems are generally organized so that interpreted functions and compiled functions can call each other. This enables a programmer to compile those parts of a program that are assumed to be debugged, thus gaining the efficiency advantage of compilation, while retaining the interpretive mode of execution for those parts of the program that are in the flux of interactive development and debugging.[3](https://sourceacademy.org/sicpjs/5.5#footnote-3) In section [5.5.7](https://sourceacademy.org/sicpjs/5.5.7), after we have implemented the compiler, we will show how to interface it with our interpreter to produce an integrated interpreter-compiler system.

[](https://sourceacademy.org/sicpjs/5.5#h1)

## An overview of the compiler

[](https://sourceacademy.org/sicpjs/5.5#p8)

Our compiler is much like our interpreter, both in its structure and in the function it performs. Accordingly, the mechanisms used by the compiler for analyzing components will be similar to those used by the interpreter. Moreover, to make it easy to interface compiled and interpreted code, we will design the compiler to generate code that obeys the same conventions of register usage as the interpreter: The environment will be kept in the `env` register, argument lists will be accumulated in `argl`, a function to be applied will be in `fun`, functions will return their answers in `val`, and the location to which a function should return will be kept in `continue`. In general, the compiler translates a source program into an object program that performs essentially the same register operations as would the interpreter in evaluating the same source program.

[](https://sourceacademy.org/sicpjs/5.5#p9)

This description suggests a strategy for implementing a rudimentary compiler: We traverse the component in the same way the interpreter does. When we encounter a register instruction that the interpreter would perform in evaluating the component, we do not execute the instruction but instead accumulate it into a sequence. The resulting sequence of instructions will be the object code. Observe the efficiency advantage of compilation over interpretation. Each time the interpreter evaluates a component—for example, `f(96, 22)`—it performs the work of classifying the component (discovering that this is a function application) and testing for the end of the list of argument expressions (discovering that there are two argument expressions). With a compiler, the component is analyzed only once, when the instruction sequence is generated at compile time. The object code produced by the compiler contains only the instructions that evaluate the function expression and the two argument expressions, assemble the argument list, and apply the function (in `fun`) to the arguments (in `argl`).

[](https://sourceacademy.org/sicpjs/5.5#p10)

This is the same kind of optimization we implemented in the analyzing evaluator of section [4.1.7](https://sourceacademy.org/sicpjs/4.1.7). But there are further opportunities to gain efficiency in compiled code. As the interpreter runs, it follows a process that must be applicable to any component in the language. In contrast, a given segment of compiled code is meant to execute some particular component. This can make a big difference, for example in the use of the stack to save registers. When the interpreter evaluates a component, it must be prepared for any contingency. Before evaluating a subcomponent, the interpreter saves all registers that will be needed later, because the subcomponent might require an arbitrary evaluation. A compiler, on the other hand, can exploit the structure of the particular component it is processing to generate code that avoids unnecessary stack operations.

[](https://sourceacademy.org/sicpjs/5.5#p11)

As a case in point, consider the application `f(96, 22)`. Before the interpreter evaluates the function expression of the application, it prepares for this evaluation by saving the registers containing the argument expressions and the environment, whose values will be needed later. The interpreter then evaluates the function expression to obtain the result in `val`, restores the saved registers, and finally moves the result from `val` to `fun`. However, in the particular expression we are dealing with, the function expression is the name `f`, whose evaluation is accomplished by the machine operation `lookup_symbol_value`, which does not alter any registers. The compiler that we implement in this section will take advantage of this fact and generate code that evaluates the function expression using the instruction

assign("fun", 
       list(op("lookup_symbol_value"), constant("f"), reg("env")))

where the argument to `lookup_symbol_value` is extracted at compile time from the parser's representation of `f(96, 22)`. This code not only avoids the unnecessary saves and restores but also assigns the value of the lookup directly to `fun`, whereas the interpreter would obtain the result in `val` and then move this to `fun`.

[](https://sourceacademy.org/sicpjs/5.5#p12)

A compiler can also optimize access to the environment. Having analyzed the code, the compiler can know in which frame the value of a particular name will be located and access that frame directly, rather than performing the `lookup_symbol_value` search. We will discuss how to implement such lexical addressing in section [5.5.6](https://sourceacademy.org/sicpjs/5.5.6). Until then, however, we will focus on the kind of register and stack optimizations described above. There are many other optimizations that can be performed by a compiler, such as coding primitive operations "in line" instead of using a general `apply` mechanism (see exercise [5.40](https://sourceacademy.org/sicpjs/5.5.5#ex-5.40)); but we will not emphasize these here. Our main goal in this section is to illustrate the compilation process in a simplified (but still interesting) context.

---

[[1]](https://sourceacademy.org/sicpjs/5.5#footnote-link-1) This is a theoretical statement. We are not claiming that the evaluator's data paths are a particularly convenient or efficient set of data paths for a general-purpose computer. For example, they are not very good for implementing high-performance floating-point calculations or calculations that intensively manipulate bit vectors.

[[2]](https://sourceacademy.org/sicpjs/5.5#footnote-link-2) Actually, the machine that runs compiled code can be simpler than the interpreter machine, because we won't use the `comp` and `unev` registers. The interpreter used these to hold pieces of unevaluated components. With the compiler, however, these components get built into the compiled code that the register machine will run. For the same reason, we don't need the machine operations that deal with component syntax. But compiled code will use a few additional machine operations (to represent compiled function objects) that didn't appear in the explicit-control evaluator machine.

[[3]](https://sourceacademy.org/sicpjs/5.5#footnote-link-3) Language implementations often delay the compilation of program parts even when they are assumed to be debugged, until there is enough evidence that compiling them would lead to an overall efficiency advantage. The evidence is obtained at run time by monitoring the number of times the program parts are being interpreted. This technique is called _just-in-time compilation_.
# 5.5.1   Structure of the Compiler

[](https://sourceacademy.org/sicpjs/5.5.1#p1)

In section [4.1.7](https://sourceacademy.org/sicpjs/4.1.7) we modified our original metacircular interpreter to separate analysis from execution. We analyzed each component to produce an execution function that took an environment as argument and performed the required operations. In our compiler, we will do essentially the same analysis. Instead of producing execution functions, however, we will generate sequences of instructions to be run by our register machine.

[](https://sourceacademy.org/sicpjs/5.5.1#p2)

The function`compile` is the top-level dispatch in the compiler. It corresponds to the `evaluate`function of section [4.1.1](https://sourceacademy.org/sicpjs/4.1.1), the `analyze`function of section [4.1.7](https://sourceacademy.org/sicpjs/4.1.7), and the `eval_dispatch` entry point of the explicit-control-evaluator in section [5.4.1](https://sourceacademy.org/sicpjs/5.4.1). The compiler, like the interpreters, uses the component-syntax functions defined in section [4.1.2](https://sourceacademy.org/sicpjs/4.1.2).[1](https://sourceacademy.org/sicpjs/5.5.1#footnote-1)The function `compile` performs a case analysis on the syntactic type of the component to be compiled. For each type of component, it dispatches to a specialized _code generator_:

function compile(component, target, linkage) {
    return is_literal(component)
           ? compile_literal(component, target, linkage)
           : is_name(component)
           ? compile_name(component, target, linkage)
           : is_application(component)
           ? compile_application(component, target, linkage)
           : is_operator_combination(component)
           ? compile(operator_combination_to_application(component),
                     target, linkage)
           : is_conditional(component)
           ? compile_conditional(component, target, linkage)
           : is_lambda_expression(component)
           ? compile_lambda_expression(component, target, linkage)
           : is_sequence(component)
           ? compile_sequence(sequence_statements(component),
                              target, linkage)
           : is_block(component)
           ? compile_block(component, target, linkage)
           : is_return_statement(component)
           ? compile_return_statement(component, target, linkage)
           : is_function_declaration(component)
           ? compile(function_decl_to_constant_decl(component),
                     target, linkage)
           : is_declaration(component)
           ? compile_declaration(component, target, linkage)
           : is_assignment(component)
           ? compile_assignment(component, target, linkage)
           : error(component, "unknown component type -- compile");
}

[](https://sourceacademy.org/sicpjs/5.5.1#h1)

## Targets and linkages

[](https://sourceacademy.org/sicpjs/5.5.1#p3)

The function `compile` and the code generators that it calls take two arguments in addition to the component to compile. There is a _target_, which specifies the register in which the compiled code is to return the value of the component. There is also a _linkage descriptor_, which describes how the code resulting from the compilation of the component should proceed when it has finished its execution. The linkage descriptor can require the code to do one of the following three things:

- proceed to the next instruction in sequence (this is specified by the linkage descriptor `"next"`),
- jump to the current value of the `continue` register as part of returning from a function call (this is specified by the linkage descriptor `"return"`), or
- jump to a named entry point (this is specified by using the designated label as the linkage descriptor).

[](https://sourceacademy.org/sicpjs/5.5.1#p4)

For example, compiling the literal `5` with a target of the `val` register and a linkage of `"next"` should produce the instruction

assign("val", constant(5))

Compiling the same expression with a linkage of `"return"` should produce the instructions

assign("val", constant(5)),
go_to(reg("continue"))

In the first case, execution will continue with the next instruction in the sequence. In the second case, we will jump to whatever entry point is stored in the `continue` register. In both cases, the value of the expression will be placed into the target `val` register. Our compiler uses the `"return"` linkage when compiling the return expression of a return statement. Just as in the explicit-control evaluator, returning from a function call happens in three steps:

1. reverting the stack to the marker and restoring `continue` (which holds a continuation set up at the beginning of the function call)
2. computing the return value and placing it in `val`
3. jumping to the entry point in `continue`

Compilation of a return statement explicitly generates code for reverting the stack and restoring `continue`. The return expression is compiled with target `val` and linkage `"return"` so that the generated code for computing the return value places the return value in `val` and ends by jumping to `continue`.

[](https://sourceacademy.org/sicpjs/5.5.1#h2)

## Instruction sequences and stack usage

[](https://sourceacademy.org/sicpjs/5.5.1#p5)

Each code generator returns an _instruction sequence_ containing the object code it has generated for the component. Code generation for a compound component is accomplished by combining the output from simpler code generators for subcomponents, just as evaluation of a compound component is accomplished by evaluating the subcomponents.

[](https://sourceacademy.org/sicpjs/5.5.1#p6)

The simplest method for combining instruction sequences is a function called `append_instruction_sequences`, which takes as arguments two instruction sequences that are to be executed sequentially. It appends them and returns the combined sequence. That is, if seq1seq1​ and seq2seq2​ are sequences of instructions, then evaluating

      append_instruction_sequences(seqseq11​, seqseq22​)
      

produces the sequence

seqseq11​
seqseq22​
      

[](https://sourceacademy.org/sicpjs/5.5.1#p7)

Whenever registers might need to be saved, the compiler's code generators use `preserving`, which is a more subtle method for combining instruction sequences. The function `preserving` takes three arguments: a set of registers and two instruction sequences that are to be executed sequentially. It appends the sequences in such a way that the contents of each register in the set is preserved over the execution of the first sequence, if this is needed for the execution of the second sequence. That is, if the first sequence modifies the register and the second sequence actually needs the register's original contents, then `preserving` wraps a `save` and a `restore` of the register around the first sequence before appending the sequences. Otherwise, `preserving` simply returns the appended instruction sequences. Thus, for example,

      preserving(list(regreg11​, regreg22​), seqseq11​, seqseq22​)
      

produces one of the following four sequences of instructions, depending on how _seq_11​ and _seq_22​ use _reg_11​ and _reg_22​:

seq1save(reg1),save(reg2),save(reg2),seq2seq1seq1save(reg1),restore(reg1),restore(reg2),seq1seq2seq2restore(reg1),restore(reg2),seq2seq1​seq2​​save(reg1​),seq1​restore(reg1​),seq2​​save(reg2​),seq1​restore(reg2​),seq2​​save(reg2​),save(reg1​),seq1​restore(reg1​),restore(reg2​),seq2​​

[](https://sourceacademy.org/sicpjs/5.5.1#p8)

By using `preserving` to combine instruction sequences the compiler avoids unnecessary stack operations. This also isolates the details of whether or not to generate `save` and `restore` instructions within the `preserving`function, separating them from the concerns that arise in writing each of the individual code generators. In fact no `save` or `restore` instructions are explicitly produced by the code generators, except that the code for calling a function saves `continue` and the code for returning from a function restores it: These corresponding `save` and `restore` instructions are explicitly generated by different calls to `compile`, not as a matched pair by `preserving` (as we will see in section [5.5.3](https://sourceacademy.org/sicpjs/5.5.3)).

[](https://sourceacademy.org/sicpjs/5.5.1#p9)

In principle, we could represent an instruction sequence simply as a list of instructions. The function `append_instruction_sequences` could then combine instruction sequences by performing an ordinary list `append`. However, `preserving` would then be a complex operation, because it would have to analyze each instruction sequence to determine how the sequence uses its registers. The function `preserving` would be inefficient as well as complex, because it would have to analyze each of its instruction sequence arguments, even though these sequences might themselves have been constructed by calls to `preserving`, in which case their parts would have already been analyzed. To avoid such repetitious analysis we will associate with each instruction sequence some information about its register use. When we construct a basic instruction sequence we will provide this information explicitly, and the functions that combine instruction sequences will derive register-use information for the combined sequence from the information associated with the sequences being combined.

[](https://sourceacademy.org/sicpjs/5.5.1#p10)

An instruction sequence will contain three pieces of information:

- the set of registers that must be initialized before the instructions in the sequence are executed (these registers are said to be _needed_ by the sequence),
- the set of registers whose values are modified by the instructions in the sequence, and
- the actual instructions in the sequence.

We will represent an instruction sequence as a list of its three parts. The constructor for instruction sequences is thus

function make_instruction_sequence(needs, modifies, instructions) {
    return list(needs, modifies, instructions);
}

[](https://sourceacademy.org/sicpjs/5.5.1#p11)

For example, the two-instruction sequence that looks up the value of the symbol `"x"` in the current environment, assigns the result to `val`, and then proceeds to the continuation, requires registers `env` and `continue` to have been initialized, and modifies register `val`. This sequence would therefore be constructed as

make_instruction_sequence(list("env", "continue"), list("val"),
    list(assign("val",
                list(op("lookup_symbol_value"), constant("x"),
                     reg("env"))),
         go_to(reg("continue"))));

[](https://sourceacademy.org/sicpjs/5.5.1#p12)

The functions for combining instruction sequences are shown in section [5.5.4](https://sourceacademy.org/sicpjs/5.5.4).

[](https://sourceacademy.org/sicpjs/5.5.1#ex-5.31)

**Exercise 5.31**

In evaluating a function application, the explicit-control evaluator always saves and restores the `env` register around the evaluation of the function expression, saves and restores `env` around the evaluation of each argument expression (except the final one), saves and restores `argl` around the evaluation of each argument expression, and saves and restores `fun` around the evaluation of the argument-expression sequence. For each of the following applications, say which of these `save` and `restore` operations are superfluous and thus could be eliminated by the compiler's `preserving` mechanism:

f("x", "y")

f()("x", "y")

f(g("x"), y)

f(g("x"), "y")

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.1#ex-5.32)

**Exercise 5.32**

Using the `preserving` mechanism, the compiler will avoid saving and restoring `env` around the evaluation of the function expression of an application in the case where the function expression is a name. We could also build such optimizations into the evaluator. Indeed, the explicit-control evaluator of section [5.4](https://sourceacademy.org/sicpjs/5.4) already performs a similar optimization, by treating applications with no arguments as a special case.

1. Extend the explicit-control evaluator to recognize as a separate class of components applications whose function expression is a name, and to take advantage of this fact in evaluating such components.
2. Alyssa P. Hacker suggests that by extending the evaluator to recognize more and more special cases we could incorporate all the compiler's optimizations, and that this would eliminate the advantage of compilation altogether. What do you think of this idea?

Show Solution

---

[[1]](https://sourceacademy.org/sicpjs/5.5.1#footnote-link-1) Notice, however, that our compiler is a JavaScript program, and the syntax functions that it uses to manipulate expressions are the actual JavaScript functions used with the metacircular evaluator. For the explicit-control evaluator, in contrast, we assumed that equivalent syntax operations were available as operations for the register machine. (Of course, when we simulated the register machine in JavaScript, we used the actual JavaScript functions in our register machine simulation.)
# 5.5.2   Compiling Components

[](https://sourceacademy.org/sicpjs/5.5.2#p1)

In this section and the next we implement the code generators to which the `compile`function dispatches.

[](https://sourceacademy.org/sicpjs/5.5.2#h1)

## Compiling linkage code

[](https://sourceacademy.org/sicpjs/5.5.2#p2)

In general, the output of each code generator will end with instructions—generated by the function`compile_linkage`—that implement the required linkage. If the linkage is `"return"` then we must generate the instruction `go_to(reg("continue"))`. This needs the `continue` register and does not modify any registers. If the linkage is `"next"`, then we needn't include any additional instructions. Otherwise, the linkage is a label, and we generate a `go_to` to that label, an instruction that does not need or modify any registers.

function compile_linkage(linkage) {
    return linkage === "return"
           ? make_instruction_sequence(list("continue"), null,
                                       list(go_to(reg("continue"))))
           : linkage === "next"
           ? make_instruction_sequence(null, null, null)
           : make_instruction_sequence(null, null, 
                                       list(go_to(label(linkage))));
}

The linkage code is appended to an instruction sequence by `preserving` the `continue` register, since a `"return"` linkage will require the `continue` register: If the given instruction sequence modifies `continue` and the linkage code needs it, `continue` will be saved and restored.

function end_with_linkage(linkage, instruction_sequence) {
    return preserving(list("continue"),
                      instruction_sequence,
                      compile_linkage(linkage));
}

[](https://sourceacademy.org/sicpjs/5.5.2#h2)

## Compiling simple components

[](https://sourceacademy.org/sicpjs/5.5.2#p3)

The code generators for literal expressions and names construct instruction sequences that assign the required value to the target register and then proceed as specified by the linkage descriptor.

[](https://sourceacademy.org/sicpjs/5.5.2#p4)

The literal value is extracted at compile time from the component being compiled and put into the constant part of the `assign` instruction. For a name, an instruction is generated to use the `lookup_symbol_value` operation when the compiled program is run, to look up the value associated with a symbol in the current environment. Like a literal value, the symbol is extracted at compile time from the component being compiled. Thus `symbol_of_name(component)` is executed only once, when the program is being compiled, and the symbol appears as a constant in the `assign` instruction.

[](https://sourceacademy.org/sicpjs/5.5.2#p5)

function compile_literal(component, target, linkage) {
    const literal = literal_value(component);
    return end_with_linkage(linkage,
               make_instruction_sequence(null, list(target),
                   list(assign(target, constant(literal)))));
}
function compile_name(component, target, linkage) {
    const symbol = symbol_of_name(component);
    return end_with_linkage(linkage,
               make_instruction_sequence(list("env"), list(target),
                   list(assign(target,
                               list(op("lookup_symbol_value"),
                                    constant(symbol),
                                    reg("env"))))));
}

These assignment instructions modify the target register, and the one that looks up a symbol needs the `env` register.

[](https://sourceacademy.org/sicpjs/5.5.2#p6)

Assignments and declarations are handled much as they are in the interpreter. The function `compile_assignment_declaration` recursively generates code that computes the value to be associated with the symbol and appends to it a two-instruction sequence that updates the value associated with the symbol in the environment and assigns the value of the whole component (the assigned value for an assignment or `undefined` for a declaration) to the target register. The recursive compilation has target `val` and linkage `"next"` so that the code will put its result into `val` and continue with the code that is appended after it. The appending is done preserving `env`, since the environment is needed for updating the symbol–value association and the code for computing the value could be the compilation of a complex expression that might modify the registers in arbitrary ways.

function compile_assignment(component, target, linkage) {
    return compile_assignment_declaration(
               assignment_symbol(component),
               assignment_value_expression(component),
               reg("val"),
               target, linkage);
}

function compile_declaration(component, target, linkage) {
    return compile_assignment_declaration(
               declaration_symbol(component),
               declaration_value_expression(component),
               constant(undefined),
               target, linkage);
}
function compile_assignment_declaration(
             symbol, value_expression, final_value,
             target, linkage) {
    const get_value_code = compile(value_expression, "val", "next");
    return end_with_linkage(linkage,
               preserving(list("env"),
                   get_value_code,
                   make_instruction_sequence(list("env", "val"),
                                             list(target),
                       list(perform(list(op("assign_symbol_value"),
                                         constant(symbol),
                                         reg("val"),
                                         reg("env"))),
                            assign(target, final_value)))));
}

The appended two-instruction sequence requires `env` and `val` and modifies the target. Note that although we preserve `env` for this sequence, we do not preserve `val`, because the `get_value_code` is designed to explicitly place its result in `val` for use by this sequence. (In fact, if we did preserve `val`, we would have a bug, because this would cause the previous contents of `val` to be restored right after the `get_value_code` is run.)

[](https://sourceacademy.org/sicpjs/5.5.2#h3)

## Compiling conditionals

[](https://sourceacademy.org/sicpjs/5.5.2#p7)

The code for a conditional compiled with a given target and linkage has the form

⟨⟨compilation of predicate, target val, linkage "next"⟩⟩
  test(list(op("is_falsy"), reg("val"))),
  branch(label("false_branch")),
"true_branch",
  ⟨⟨compilation of consequent with given target and given linkage or after_cond⟩⟩
"false_branch",
  ⟨⟨compilation of alternative with given target and linkage⟩⟩
"after_cond"
      

[](https://sourceacademy.org/sicpjs/5.5.2#p8)

To generate this code, we compile the predicate, consequent, and alternative, and combine the resulting code with instructions to test the predicate result and with newly generated labels to mark the true and false branches and the end of the conditional.[1](https://sourceacademy.org/sicpjs/5.5.2#footnote-1) In this arrangement of code, we must branch around the true branch if the test is false. The only slight complication is in how the linkage for the true branch should be handled. If the linkage for the conditional is `"return"` or a label, then the true and false branches will both use this same linkage. If the linkage is `"next"`, the true branch ends with a jump around the code for the false branch to the label at the end of the conditional.

function compile_conditional(component, target, linkage) {
    const t_branch = make_label("true_branch");
    const f_branch = make_label("false_branch");
    const after_cond = make_label("after_cond");
    const consequent_linkage =
            linkage === "next" ? after_cond : linkage;
    const p_code = compile(conditional_predicate(component),
                           "val", "next");
    const c_code = compile(conditional_consequent(component),
                           target, consequent_linkage);
    const a_code = compile(conditional_alternative(component),
                           target, linkage);
    return preserving(list("env", "continue"),
             p_code,
             append_instruction_sequences(
               make_instruction_sequence(list("val"), null,
                 list(test(list(op("is_falsy"), reg("val"))),
                      branch(label(f_branch)))),
               append_instruction_sequences(
                 parallel_instruction_sequences(
                   append_instruction_sequences(t_branch, c_code),
                   append_instruction_sequences(f_branch, a_code)),
                 after_cond)));
}

The `env` register is preserved around the predicate code because it could be needed by the true and false branches, and `continue` is preserved because it could be needed by the linkage code in those branches. The code for the true and false branches (which are not executed sequentially) is appended using a special combiner `parallel_instruction_sequences` described in section [5.5.4](https://sourceacademy.org/sicpjs/5.5.4).

[](https://sourceacademy.org/sicpjs/5.5.2#h4)

## Compiling sequences

[](https://sourceacademy.org/sicpjs/5.5.2#p9)

The compilation of statement sequences parallels their evaluation in the explicit-control evaluator with one exception: If a return statement appears anywhere in a sequence, we treat it as if it were the last statement. Each statement of the sequence is compiled—the last statement (or a return statement) with the linkage specified for the sequence, and the other statements with linkage `"next"` (to execute the rest of the sequence). The instruction sequences for the individual statements are appended to form a single instruction sequence, such that `env` (needed for the rest of the sequence) and `continue` (possibly needed for the linkage at the end of the sequence) are preserved.[2](https://sourceacademy.org/sicpjs/5.5.2#footnote-2)

function compile_sequence(seq, target, linkage) {
    return is_empty_sequence(seq)
           ? compile_literal(make_literal(undefined), target, linkage)
           : is_last_statement(seq) ||
                 is_return_statement(first_statement(seq))
           ? compile(first_statement(seq), target, linkage)
           : preserving(list("env", "continue"),
                 compile(first_statement(seq), target, "next"),
                 compile_sequence(rest_statements(seq), 
                                  target, linkage));
}

Treating a return statement as if it were the last statement in a sequence avoids compiling any "dead code" after the return statement that can never be executed. Removing the `is_return_statement` check does not change the behavior of the object program; however, there are many reasons not to compile dead code, which are beyond the scope of this book (security, compilation time, size of the object code, etc.), and many compilers give warnings for dead code.[3](https://sourceacademy.org/sicpjs/5.5.2#footnote-3)

[](https://sourceacademy.org/sicpjs/5.5.2#h5)

## Compiling blocks

[](https://sourceacademy.org/sicpjs/5.5.2#p10)

A block is compiled by prepending an `assign` instruction to the compiled body of the block. The assignment extends the current environment by a frame that binds the names declared in the block to the value `"*unassigned*"`. This operation both needs and modifies the `env` register.

function compile_block(stmt, target, linkage) {
    const body = block_body(stmt);
    const locals = scan_out_declarations(body);
    const unassigneds = list_of_unassigned(locals);
    return append_instruction_sequences(
               make_instruction_sequence(list("env"), list("env"),
                   list(assign("env", list(op("extend_environment"),
                                           constant(locals),
                                           constant(unassigneds),
                                           reg("env"))))),
               compile(body, target, linkage));
}

[](https://sourceacademy.org/sicpjs/5.5.2#h6)

## Compiling lambda expressions

[](https://sourceacademy.org/sicpjs/5.5.2#p11)

Lambda expressions construct functions. The object code for a lambda expression must have the form

⟨⟨construct function object and assign it to target register⟩⟩
⟨⟨linkage⟩⟩
	  

When we compile the lambda expression, we also generate the code for the function body. Although the body won't be executed at the time of function construction, it is convenient to insert it into the object code right after the code for the lambda expression. If the linkage for the lambda expression is a label or `"return"`, this is fine. But if the linkage is `"next"`, we will need to skip around the code for the function body by using a linkage that jumps to a label that is inserted after the body. The object code thus has the form

⟨⟨construct function object and assign it to target register⟩⟩
⟨⟨code for given linkage⟩⟩ oror go_to(label("after_lambda"))
⟨⟨compilation of function body⟩⟩
"after_lambda"
	  

[](https://sourceacademy.org/sicpjs/5.5.2#p12)

The function `compile_lambda_expression` generates the code for constructing the function object followed by the code for the function body. The function object will be constructed at run time by combining the current environment (the environment at the point of declaration) with the entry point to the compiled function body (a newly generated label).[4](https://sourceacademy.org/sicpjs/5.5.2#footnote-4)

function compile_lambda_expression(exp, target, linkage) {
    const fun_entry = make_label("entry");
    const after_lambda = make_label("after_lambda");
    const lambda_linkage =
            linkage === "next" ? after_lambda : linkage;
    return append_instruction_sequences(
               tack_on_instruction_sequence(
                   end_with_linkage(lambda_linkage,
                       make_instruction_sequence(list("env"),
                                                 list(target),
                           list(assign(target,
                                    list(op("make_compiled_function"),
                                         label(fun_entry),
                                         reg("env")))))),
                   compile_lambda_body(exp, fun_entry)),
               after_lambda);
}

The function `compile_lambda_expression` uses the special combiner `tack_on_instruction_sequence` (from section [5.5.4](https://sourceacademy.org/sicpjs/5.5.4)) rather than `append_instruction_sequences` to append the function body to the lambda expression code, because the body is not part of the sequence of instructions that will be executed when the combined sequence is entered; rather, it is in the sequence only because that was a convenient place to put it.

[](https://sourceacademy.org/sicpjs/5.5.2#p13)

The function `compile_lambda_body` constructs the code for the body of the function. This code begins with a label for the entry point. Next come instructions that will cause the runtime evaluation environment to switch to the correct environment for evaluating the function body—namely, the environment of the function, extended to include the bindings of the parameters to the arguments with which the function is called. After this comes the code for the function body, augmented to ensure that it ends with a return statement. The augmented body is compiled with target `val` so that its return value will be placed in `val`. The linkage descriptor passed to this compilation is irrelevant, as it will be ignored.[5](https://sourceacademy.org/sicpjs/5.5.2#footnote-5) Since a linkage argument is required, we arbitrarily pick `"next"`.

function compile_lambda_body(exp, fun_entry) {
    const params  = lambda_parameter_symbols(exp);
    return append_instruction_sequences(
        make_instruction_sequence(list("env", "fun", "argl"),
                                  list("env"),
            list(fun_entry,
                 assign("env",
                        list(op("compiled_function_env"),
                             reg("fun"))),
                 assign("env",
                        list(op("extend_environment"),
                             constant(params),
                             reg("argl"),
                             reg("env"))))),
        compile(append_return_undefined(lambda_body(exp)),
                "val", "next"));
}

[](https://sourceacademy.org/sicpjs/5.5.2#p14)

To ensure that all functions end by executing a return statement, `compile_lambda_body` appends to the lambda body a return statement whose return expression is the literal `undefined`. To do so, it uses the function `append_return_undefined`, which constructs the parser's tagged-list representation (from section [4.1.2](https://sourceacademy.org/sicpjs/4.1.2)) of a sequence consisting of the body and a `return undefined;` statement.

function append_return_undefined(body) {
    return list("sequence", list(body,
                                 list("return_statement",
                                      list("literal", undefined))));
}

This simple transformation of lambda bodies is a third way to ensure that a function that does not return explicitly has the return value `undefined`. In the metacircular evaluator, we used a return-value object, which also played a role in stopping a sequence evaluation. In the explicit-control evaluator, functions that did not return explicitly continued to an entry point that stored `undefined` in `val`. See exercise [5.34](https://sourceacademy.org/sicpjs/5.5.2#ex-5.34) for a more elegant way to handle insertion of return statements.

[](https://sourceacademy.org/sicpjs/5.5.2#ex-5.33)

**Exercise 5.33**

Footnote [3](https://sourceacademy.org/sicpjs/5.5.2#footnote-3) pointed out that the compiler does not identify all instances of dead code. What would be required of a compiler to detect all instances of dead code?

[](https://sourceacademy.org/sicpjs/5.5.2#p15)

Hint: The answer depends on how we define dead code. One possible (and useful) definition is "code following a return statement in a sequence"—but what about code in the consequent branch of `if (false)` …… or code following a call to `run_forever()` in exercise [4.15](https://sourceacademy.org/sicpjs/4.1.5#ex-4.15)?

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.2#ex-5.34)

**Exercise 5.34**

The current design of `append_return_undefined` is a bit crude: It always appends a `return undefined;` to a lambda body, even if there is already a return statement in every execution path of the body. Rewrite `append_return_undefined` so that it inserts `return undefined;` at the end of only those paths that do not contain a return statement. Test your solution on the functions below, substituting any expressions for e1e1​ and e2e2​ and any (non-return) statements for s1s1​ and s2s2​. In `t`, a return statement should be added either at both `(*)`'s or just at `(**)`. In `w` and `h`, a return statement should be added at one of the `(*)`'s. In `m`, no return statement should be added.

function t(b) {   function w(b) {    function m(b) {    function h(b1, b2) {
   if (b) {          if (b) {           if (b) {           if (b1) {
      s1  s1​                return e1; e1​;          return e1; e1​;          return e1e1​;
      (*)                                                  } else {
   } else {          } else {           } else {              if (b2) {
      s2  s2​                 s1  s1​                 return e2e2​;             s1s1​
      (*)               (*)                                      (*)
   }                 }                  }                     } else {
   (**)              (*)                                         return e2e2​;
}                 }                  }                        }
                                                              (*)
                                                           }
                                                           (*)
                                                        }
	  

Show Solution

---

[[1]](https://sourceacademy.org/sicpjs/5.5.2#footnote-link-1) We can't just use the labels `true_branch`,`false_branch`, and `after_cond` as shown above, because there might be more than one conditional in the program. The compiler uses the function`make_label` to generate labels. The function `make_label` takes a string as argument and returns a new string that begins with the given string. For example, successive calls to `make_label("a")` would return `"a1"`, `"a2"`, and so on. The function `make_label` can be implemented similarly to the generation of unique variable names in the query language, as follows:

let label_counter = 0;

function new_label_number() {
    label_counter = label_counter + 1;
    return label_counter;
}
function make_label(string) {
    return string + stringify(new_label_number());
}

[[2]](https://sourceacademy.org/sicpjs/5.5.2#footnote-link-2) The `continue` register would be needed for a `"return"` linkage, which can result from a compilation by `compile_and_go` (section [5.5.7](https://sourceacademy.org/sicpjs/5.5.7)).

[[3]](https://sourceacademy.org/sicpjs/5.5.2#footnote-link-3) Our compiler does not detect all dead code. For example, a conditional statement whose consequent and alternative branches both end in a return statement will not stop subsequent statements from being compiled. See exercises [5.33](https://sourceacademy.org/sicpjs/5.5.2#ex-5.33) and [5.34](https://sourceacademy.org/sicpjs/5.5.2#ex-5.34).

[[4]](https://sourceacademy.org/sicpjs/5.5.2#footnote-link-4) We need machine operations to implement a data structure for representing compiled functions, analogous to the structure for compound functions described in section [4.1.3](https://sourceacademy.org/sicpjs/4.1.3):

function make_compiled_function(entry, env) {
    return list("compiled_function", entry, env);
}
function is_compiled_function(fun) {
    return is_tagged_list(fun, "compiled_function");
}
function compiled_function_entry(c_fun) {
    return head(tail(c_fun));
}
function compiled_function_env(c_fun) {
    return head(tail(tail(c_fun)));
}

[[5]](https://sourceacademy.org/sicpjs/5.5.2#footnote-link-5) The augmented function body is a sequence ending with a return statement. Compilation of a sequence of statements uses the linkage `"next"` for all its component statements except the last, for which it uses the given linkage. In this case, the last statement is a return statement, and as we will see in section [5.5.3](https://sourceacademy.org/sicpjs/5.5.3), a return statement always uses the `"return"` linkage descriptor for its return expression. Thus all function bodies will end with a `"return"` linkage, not the `"next"` we pass as the linkage argument to `compile` in `compile_lambda_body`.
# 5.5.3   Compiling Applications and Return Statements

[](https://sourceacademy.org/sicpjs/5.5.3#p1)

The essence of the compilation process is the compilation of function applications. The code for an application compiled with a given target and linkage has the form

⟨⟨compilation of function expression, target fun, linkage "next"⟩⟩
⟨⟨evaluate argument expressions and construct argument list in argl⟩⟩
⟨⟨compilation of function call with given target and linkage⟩⟩
      

The registers `env`, `fun`, and `argl` may have to be saved and restored during evaluation of the function and argument expressions. Note that this is the only place in the compiler where a target other than `val` is specified.

[](https://sourceacademy.org/sicpjs/5.5.3#p2)

The required code is generated by `compile_application`. This recursively compiles the function expression, to produce code that puts the function to be applied into `fun`, and compiles the argument expressions, to produce code that evaluates the individual argument expressions of the application. The instruction sequences for the argument expressions are combined (by `construct_arglist`) with code that constructs the list of arguments in `argl`, and the resulting argument-list code is combined with the function code and the code that performs the function call (produced by `compile_function_call`). In appending the code sequences, the `env` register must be preserved around the evaluation of the function expression (since evaluating the function expression might modify `env`, which will be needed to evaluate the argument expressions), and the `fun` register must be preserved around the construction of the argument list (since evaluating the argument expressions might modify `fun`, which will be needed for the actual function application). The `continue` register must also be preserved throughout, since it is needed for the linkage in the function call.

function compile_application(exp, target, linkage) {
    const fun_code = compile(function_expression(exp), "fun", "next");
    const argument_codes = map(arg => compile(arg, "val", "next"),
                               arg_expressions(exp));
    return preserving(list("env", "continue"),
                      fun_code,
                      preserving(list("fun", "continue"),
                          construct_arglist(argument_codes),
                          compile_function_call(target, linkage)));
}

[](https://sourceacademy.org/sicpjs/5.5.3#p3)

The code to construct the argument list will evaluate each argument expression into `val` and then combine that value with the argument list being accumulated in `argl` using `pair`. Since we adjoin the arguments to the front of `argl` in sequence, we must start with the last argument and end with the first, so that the arguments will appear in order from first to last in the resulting list. Rather than waste an instruction by initializing `argl` to the empty list to set up for this sequence of evaluations, we make the first code sequence construct the initial `argl`. The general form of the argument-list construction is thus as follows:

⟨⟨compilation of last argument, targeted to val⟩⟩
assign("argl", list(op("list"), reg("val"))),
⟨⟨compilation of next argument, targeted to val⟩⟩
assign("argl", list(op("pair"), reg("val"), reg("argl"))),
……
⟨⟨compilation of first argument, targeted to val⟩⟩
assign("argl", list(op("pair"), reg("val"), reg("argl"))),
      

The `argl` register must be preserved around each argument evaluation except the first (so that arguments accumulated so far won't be lost), and `env` must be preserved around each argument evaluation except the last (for use by subsequent argument evaluations).

[](https://sourceacademy.org/sicpjs/5.5.3#p4)

Compiling this argument code is a bit tricky, because of the special treatment of the first argument expression to be evaluated and the need to preserve `argl` and `env` in different places. The `construct_arglist`function takes as arguments the code that evaluates the individual argument expressions. If there are no argument expressions at all, it simply emits the instruction

assign(argl, constant(null))

Otherwise, `construct_arglist` creates code that initializes `argl` with the last argument, and appends code that evaluates the rest of the arguments and adjoins them to `argl` in succession. In order to process the arguments from last to first, we must reverse the list of argument code sequences from the order supplied by `compile_application`.

function construct_arglist(arg_codes) {
    if (is_null(arg_codes)) {
        return make_instruction_sequence(null, list("argl"),
                   list(assign("argl", constant(null))));
    } else {
        const rev_arg_codes = reverse(arg_codes);
        const code_to_get_last_arg =
            append_instruction_sequences(
                head(rev_arg_codes),
                make_instruction_sequence(list("val"), list("argl"),
                    list(assign("argl", 
                                list(op("list"), reg("val"))))));
        return is_null(tail(rev_arg_codes))
               ? code_to_get_last_arg
               : preserving(list("env"),
                     code_to_get_last_arg,
                     code_to_get_rest_args(tail(rev_arg_codes)));
    }
}
function code_to_get_rest_args(arg_codes) {
    const code_for_next_arg =
        preserving(list("argl"),
            head(arg_codes),
            make_instruction_sequence(list("val", "argl"), list("argl"),
                list(assign("argl", list(op("pair"),
                                         reg("val"), reg("argl"))))));
    return is_null(tail(arg_codes))
           ? code_for_next_arg
           : preserving(list("env"),
                        code_for_next_arg,
                        code_to_get_rest_args(tail(arg_codes)));
}

[](https://sourceacademy.org/sicpjs/5.5.3#h1)

## Applying functions

[](https://sourceacademy.org/sicpjs/5.5.3#p5)

After evaluating the elements of a function application, the compiled code must apply the function in `fun` to the arguments in `argl`. The code performs essentially the same dispatch as the `apply`function in the metacircular evaluator of section [4.1.1](https://sourceacademy.org/sicpjs/4.1.1) or the `apply_dispatch` entry point in the explicit-control evaluator of section [5.4.2](https://sourceacademy.org/sicpjs/5.4.2). It checks whether the function to be applied is a primitive function or a compiled function. For a primitive function, it uses `apply_primitive_function`; we will see shortly how it handles compiled functions. The function-application code has the following form:

Observe that the compiled branch must skip around the primitive branch. Therefore, if the linkage for the original function call was `"next"`, the compound branch must use a linkage that jumps to a label that is inserted after the primitive branch. (This is similar to the linkage used for the true branch in `compile_conditional`.)

function compile_function_call(target, linkage) {
    const primitive_branch = make_label("primitive_branch");
    const compiled_branch = make_label("compiled_branch");
    const after_call = make_label("after_call");
    const compiled_linkage = linkage === "next" ? after_call : linkage;
    return append_instruction_sequences(
        make_instruction_sequence(list("fun"), null,
            list(test(list(op("is_primitive_function"), reg("fun"))),
                 branch(label(primitive_branch)))),
        append_instruction_sequences(
            parallel_instruction_sequences(
                append_instruction_sequences(
                    compiled_branch,
                    compile_fun_appl(target, compiled_linkage)),
                append_instruction_sequences(
                    primitive_branch,
                    end_with_linkage(linkage,
                        make_instruction_sequence(list("fun", "argl"),
                                                  list(target),
                            list(assign(
                                   target,
                                   list(op("apply_primitive_function"),
                                        reg("fun"), reg("argl")))))))),
            after_call));
}

The primitive and compound branches, like the true and false branches in `compile_conditional`, are appended using `parallel_instruction_sequences` rather than the ordinary `append_instruction_sequences`, because they will not be executed sequentially.

[](https://sourceacademy.org/sicpjs/5.5.3#h2)

## Applying compiled functions

[](https://sourceacademy.org/sicpjs/5.5.3#p6)

The handling of function application and return is the most subtle part of the compiler. A compiled function (as constructed by `compile_lambda_expression`) has an entry point, which is a label that designates where the code for the function starts. The code at this entry point computes a result in `val`and ends by executing the instructions from a compiled return statement.

[](https://sourceacademy.org/sicpjs/5.5.3#p7)

The code for a compiled-function application uses the stack in the same way as the explicit-control evaluator (section [5.4.2](https://sourceacademy.org/sicpjs/5.4.2)): before jumping to the compiled function's entry point, it saves the continuation of the function call to the stack, followed by a mark that allows reverting the stack to the state right before the call with the continuation on top.

    // set up for return from function
  save("continue"),
  push_marker_to_stack(),
  // jump to the function's entry point
  assign("val", list(op("compiled_function_entry"), reg("fun"))),
  go_to(reg("val")),
        

Compiling a return statement (with `compile_return_statement`) generates corresponding code for reverting the stack and restoring and jumping to `continue`.

    revert_stack_to_marker(),
  restore("continue"),
  ⟨⟨evaluate the return expression and store the result in val⟩⟩
  go_to(reg("continue")), // "return""return"-linkage code
         

Unless a function enters an infinite loop, it will end by executing the above return code, resulting from either a return statement in the program or one inserted by `compile_lambda_body` to return `undefined`.[1](https://sourceacademy.org/sicpjs/5.5.3#footnote-1)

[](https://sourceacademy.org/sicpjs/5.5.3#p8)

Straightforward code for a compiled-function application with a given target and linkage would set up `continue` to make the function return to a local label instead of to the final linkage, to copy the function value from `val` to the target register if necessary. It would look like this if the linkage is a label:

    assign("continue", label("fun_return")), // where function should return to
  save("continue"),       // will be restored by the function
  push_marker_to_stack(), // allows the function to revert stack to find fun_returnfun_return
  assign("val", list(op("compiled_function_entry"), reg("fun"))),
  go_to(reg("val")),    // eventually reverts stack, restores and jumps to continuecontinue
"fun_return",             // the function returns to here
  assign(targettarget, reg("val")), // included if target is not valval
  go_to(label(linkagelinkage)),   // linkage code
      

or like this—saving the caller's continuation at the start in order to restore and go to it at the end—if the linkage is `"return"` (that is, if the application is in a return statement and its value is the result to be returned):

    save("continue"),       // save the caller's continuation
  assign("continue", label("fun_return")), // where function should return to
  save("continue"),       // will be restored by the function
  push_marker_to_stack(), // allows the function to revert stack to find fun_returnfun_return
  assign("val", list(op("compiled_function_entry"), reg("fun"))),
  go_to(reg("val")),    // eventually reverts stack, restores and jumps to continuecontinue
"fun_return",             // the function returns to here
  assign(targettarget, reg("val")), // included if target is not valval
  restore("continue"),    // restore the caller's continuation
  go_to(reg("continue")), // linkage code
      

This code sets up `continue` so that the function will return to a label `fun_return` and jumps to the function's entry point. The code at `fun_return` transfers the function's result from `val` to the target register (if necessary) and then jumps to the location specified by the linkage. (The linkage is always `"return"` or a label, because `compile_function_call` replaces a `"next"` linkage for the compound-function branch by an `after_call` label.) Before jumping to the function's entry point, we save `continue` and execute `push_marker_to_stack()` to enable the function to return to the intended location in the program with the expected stack. Matching `revert_stack_to_marker()` and `restore("continue")` instructions are generated by `compile_return_statement` for each return statement in the body of the function.[2](https://sourceacademy.org/sicpjs/5.5.3#footnote-2)

[](https://sourceacademy.org/sicpjs/5.5.3#p9)

In fact, if the target is not `val`, the above is exactly the code our compiler will generate.[3](https://sourceacademy.org/sicpjs/5.5.3#footnote-3) Usually, however, the target is `val` (the only time the compiler specifies a different register is when targeting the evaluation of a function expression to `fun`), so the function result is put directly into the target register and there is no need to jump to a special location that copies it. Instead we simplify the code by setting up `continue` so that the called function will "return" directly to the place specified by the caller's linkage:

⟨⟨set up continue for linkage and push the marker⟩⟩
assign("val", list(op("compiled_function_entry"), reg("fun"))),
go_to(reg("val")),
      

If the linkage is a label, we set up `continue` so that the function will continue at that label. (That is, the `go_to(reg("continue"))` the called function ends with becomes equivalent to the `go_to(label(`_linkage_`))` at `fun_return` above.)

assign("continue", label(linkagelinkage)),
save("continue"),
push_marker_to_stack(),
assign("val", list(op("compiled_function_entry"), reg("fun"))),
go_to(reg("val")),
      

If the linkage is `"return"`, we don't need to assign `continue`: It already holds the desired location. (That is, the `go_to(reg("continue"))` the called function ends with goes directly to the place where the `go_to(reg("continue"))` at `fun_`{\hspace{0pt}}`return`  would have gone.)

save("continue"),
push_marker_to_stack(),
assign("val", list(op("compiled_function_entry"), reg("fun"))),
go_to(reg("val")),

With this implementation of the `"return"` linkage, the compiler generates tail-recursive code. A function call in a return statement whose value is the result to be returned does a direct transfer, without saving unnecessary information on the stack.

[](https://sourceacademy.org/sicpjs/5.5.3#p10)

Suppose instead that we had handled the case of a function call with a linkage of `"return"` and a target of `val` in the same way as for a non-`val` target. This would destroy tail recursion. Our system would still return the same value for any function call. But each time we called a function, we would save `continue` and return after the call to undo the (useless) save. These extra saves would accumulate during a nest of function calls.[4](https://sourceacademy.org/sicpjs/5.5.3#footnote-4)

[](https://sourceacademy.org/sicpjs/5.5.3#p11)

The function `compile_fun_appl` generates the above function-application code by considering four cases, depending on whether the target for the call is `val` and whether the linkage is `"return"`. Observe that the instruction sequences are declared to modify all the registers, since executing the function body can change the registers in arbitrary ways.[5](https://sourceacademy.org/sicpjs/5.5.3#footnote-5)

function compile_fun_appl(target, linkage) {
    const fun_return = make_label("fun_return");
    return target === "val" && linkage !== "return"
           ? make_instruction_sequence(list("fun"), all_regs,
                 list(assign("continue", label(linkage)),
                      save("continue"),
                      push_marker_to_stack(),
                      assign("val", list(op("compiled_function_entry"),
                                         reg("fun"))),
                      go_to(reg("val"))))
           : target !== "val" && linkage !== "return"
           ? make_instruction_sequence(list("fun"), all_regs,
                 list(assign("continue", label(fun_return)),
                      save("continue"),
                      push_marker_to_stack(),
                      assign("val", list(op("compiled_function_entry"),
                                         reg("fun"))),
                      go_to(reg("val")),
                      fun_return,
                      assign(target, reg("val")),
                      go_to(label(linkage))))
           : target === "val" && linkage === "return"
           ? make_instruction_sequence(list("fun", "continue"),
                                       all_regs,
                 list(save("continue"),
                      push_marker_to_stack(),
                      assign("val", list(op("compiled_function_entry"),
                                         reg("fun"))),
                      go_to(reg("val"))))
           : // target !== "val" && linkage === "return"target !== "val" && linkage === "return"
             error(target, "return linkage, target not val -- compile");
}
      

[](https://sourceacademy.org/sicpjs/5.5.3#p12)

We have shown how to generate tail-recursive linkage code for a function application when the linkage is `"return"`—that is, when the application is in a return statement and its value is the result to be returned. Similarly, as explained in section [5.4.2](https://sourceacademy.org/sicpjs/5.4.2), the stack-marker mechanism used here (and in the explicit-control evaluator) for the call and return produces tail-recursive behavior only in that situation. These two aspects of the code generated for function application combine to ensure that when a function ends by returning the value of a function call, no stack accumulates.

[](https://sourceacademy.org/sicpjs/5.5.3#h3)

## Compiling return statements

[](https://sourceacademy.org/sicpjs/5.5.3#p13)

The code for a return statement takes the following form, regardless of the given linkage and target:

revert_stack_to_marker(),
restore("continue"),   // saved by compile_fun_applcompile_fun_appl
⟨⟨evaluate the return expression and store the result in val⟩⟩
go_to(reg("continue")) // "return""return"-linkage code
          

The instructions to revert the stack using the marker and then restore `continue` correspond to the instructions generated by `compile_fun_appl` to save `continue` and mark the stack. The final jump to `continue` is generated by the use of the `"return"` linkage when compiling the return expression. The function `compile_return_statement` is different from all other code generators in that it ignores the target and linkage arguments—it always compiles the return expression with target `val` and linkage `"return"`.

function compile_return_statement(stmt, target, linkage) {
    return append_instruction_sequences(
               make_instruction_sequence(null, list("continue"),
                   list(revert_stack_to_marker(),
                        restore("continue"))),
               compile(return_expression(stmt), "val", "return"));
}

---

[[1]](https://sourceacademy.org/sicpjs/5.5.3#footnote-link-1) Because the execution of a function body always ends with a return, there is no need here for a mechanism like the `return_undefined` entry point from section [5.4.2](https://sourceacademy.org/sicpjs/5.4.2).

[[2]](https://sourceacademy.org/sicpjs/5.5.3#footnote-link-2) Elsewhere in the compiler, all saves and restores of registers are generated by `preserving` to preserve a register's value across a sequence of instructions by saving it before those instructions and restoring it after—for example over the evaluation of the predicate of a conditional. But this mechanism cannot generate instructions to save and restore `continue` for a function application and the corresponding return, because these are compiled separately and are not contiguous. Instead, these saves and restores must be explicitly generated by `compile_fun_appl` and `compile_return_statement`.

[[3]](https://sourceacademy.org/sicpjs/5.5.3#footnote-link-3) Actually, we signal an error when the target is not `val` and the linkage is `"return"`, since the only place we request a `"return"` linkage is in compiling return expressions, and our convention is that functions return their values in `val`.

[[4]](https://sourceacademy.org/sicpjs/5.5.3#footnote-link-4) Making a compiler generate tail-recursive code is desirable, especially in the functional paradigm. However, compilers for common languages, including C and C++, do not always do this, and therefore these languages cannot represent iterative processes in terms of function call alone. The difficulty with tail recursion in these languages is that their implementations use the stack to store function arguments and local names as well as return addresses. The JavaScript implementations described in this book store arguments and names in memory to be garbage-collected. The reason for using the stack for names and arguments is that it avoids the need for garbage collection in languages that would not otherwise require it, and is generally believed to be more efficient. Sophisticated compilers can, in fact, use the stack for arguments without destroying tail recursion. (See Hanson 1990 for a description.) There is also some debate about whether stack allocation is actually more efficient than garbage collection in the first place, but the details seem to hinge on fine points of computer architecture. (See Appel 1987 and Miller and Rozas 1994 for opposing views on this issue.)

[[5]](https://sourceacademy.org/sicpjs/5.5.3#footnote-link-5) The constant`all_regs` is bound to the list of names of all the registers:

const all_regs = list("env", "fun", "val", "argl", "continue");
# 5.5.4   Combining Instruction Sequences

[](https://sourceacademy.org/sicpjs/5.5.4#p1)

This section describes the details on how instruction sequences are represented and combined. Recall from section [5.5.1](https://sourceacademy.org/sicpjs/5.5.1) that an instruction sequence is represented as a list of the registers needed, the registers modified, and the actual instructions. We will also consider a label (string) to be a degenerate case of an instruction sequence, which doesn't need or modify any registers. So to determine the registers needed and modified by instruction sequences we use the selectors

function registers_needed(s) {
    return is_string(s) ? null : head(s);
}
function registers_modified(s) {
    return is_string(s) ? null : head(tail(s));
}
function instructions(s) {
    return is_string(s) ? list(s) : head(tail(tail(s)));
}

and to determine whether a given sequence needs or modifies a given register we use the predicates

function needs_register(seq, reg) {
    return ! is_null(member(reg, registers_needed(seq)));
}
function modifies_register(seq, reg) {
    return ! is_null(member(reg, registers_modified(seq)));
}

In terms of these predicates and selectors, we can implement the various instruction sequence combiners used throughout the compiler.

[](https://sourceacademy.org/sicpjs/5.5.4#p2)

The basic combiner is `append_instruction_sequences`. This takes as arguments two instruction sequences that are to be executed sequentially and returns an instruction sequence whose statements are the statements of the two sequences appended together. The subtle point is to determine the registers that are needed and modified by the resulting sequence. It modifies those registers that are modified by either sequence; it needs those registers that must be initialized before the first sequence can be run (the registers needed by the first sequence), together with those registers needed by the second sequence that are not initialized (modified) by the first sequence.

[](https://sourceacademy.org/sicpjs/5.5.4#p3)

The function `append_instruction_sequences` is given two instruction sequences `seq1` and `seq2` and returns the instruction sequence whose instructions are the instructions of `seq1` followed by the instructions of `seq2`, whose modified registers are those registers that are modified by either `seq1` or `seq2`, and whose needed registers are the registers needed by `seq1` together with those registers needed by `seq2` that are not modified by `seq1`. (In terms of set operations, the new set of needed registers is the union of the set of registers needed by `seq1` with the set difference of the registers needed by `seq2` and the registers modified by `seq1`.) Thus, `append_instruction_sequences` is implemented as follows:

function append_instruction_sequences(seq1, seq2) {
    return make_instruction_sequence(
               list_union(registers_needed(seq1),
                          list_difference(registers_needed(seq2),
                                          registers_modified(seq1))),
               list_union(registers_modified(seq1),
                          registers_modified(seq2)),
               append(instructions(seq1), instructions(seq2)));
}

[](https://sourceacademy.org/sicpjs/5.5.4#p4)

This function uses some simple operations for manipulating sets represented as lists, similar to the (unordered) set representation described in section [2.3.3](https://sourceacademy.org/sicpjs/2.3.3):

function list_union(s1, s2) {
    return is_null(s1)
           ? s2
           : is_null(member(head(s1), s2))
           ? pair(head(s1), list_union(tail(s1), s2))
           : list_union(tail(s1), s2);
}
function list_difference(s1, s2) {
    return is_null(s1)
           ? null
           : is_null(member(head(s1), s2))
           ? pair(head(s1), list_difference(tail(s1), s2))
           : list_difference(tail(s1), s2);
}

[](https://sourceacademy.org/sicpjs/5.5.4#p5)

The function `preserving`, the second major instruction sequence combiner, takes a list of registers `regs` and two instruction sequences `seq1` and `seq2` that are to be executed sequentially. It returns an instruction sequence whose instructions are the instructions of `seq1` followed by the instructions of `seq2`, with appropriate `save` and `restore` instructions around `seq1` to protect the registers in `regs` that are modified by `seq1` but needed by `seq2`. To accomplish this, `preserving` first creates a sequence that has the required `save`s followed by the instructions of `seq1` followed by the required `restore`s. This sequence needs the registers being saved and restored in addition to the registers needed by `seq1`, and modifies the registers modified by `seq1` except for the ones being saved and restored. This augmented sequence and `seq2` are then appended in the usual way. The following function implements this strategy recursively, walking down the list of registers to be preserved:

function preserving(regs, seq1, seq2) {
    if (is_null(regs)) {
        return append_instruction_sequences(seq1, seq2);
    } else {
        const first_reg = head(regs);
        return needs_register(seq2, first_reg) &&
               modifies_register(seq1, first_reg)
               ? preserving(tail(regs),
                     make_instruction_sequence(
                         list_union(list(first_reg),
                                    registers_needed(seq1)),
                         list_difference(registers_modified(seq1),
                                         list(first_reg)),
                         append(list(save(first_reg)),
                                append(instructions(seq1),
                                       list(restore(first_reg))))),
                     seq2)
               : preserving(tail(regs), seq1, seq2);
    }
}

[](https://sourceacademy.org/sicpjs/5.5.4#p6)

Another sequence combiner, `tack_on_instruction_sequence`, is used by `compile_lambda_expression` to append a function body to another sequence. Because the function body is not "in line" to be executed as part of the combined sequence, its register use has no impact on the register use of the sequence in which it is embedded. We thus ignore the function body's sets of needed and modified registers when we tack it onto the other sequence.

function tack_on_instruction_sequence(seq, body_seq) {
    return make_instruction_sequence(
               registers_needed(seq),
               registers_modified(seq),
               append(instructions(seq), instructions(body_seq)));
}

[](https://sourceacademy.org/sicpjs/5.5.4#p7)

The functions `compile_conditional` and `compile_function_call` use a special combiner called `parallel_instruction_sequences` to append the two alternative branches that follow a test. The two branches will never be executed sequentially; for any particular evaluation of the test, one branch or the other will be entered. Because of this, the registers needed by the second branch are still needed by the combined sequence, even if these are modified by the first branch.

function parallel_instruction_sequences(seq1, seq2) {
    return make_instruction_sequence(
               list_union(registers_needed(seq1),
                          registers_needed(seq2)),
               list_union(registers_modified(seq1),
                          registers_modified(seq2)),
               append(instructions(seq1), instructions(seq2)));
}

# 5.5.5   An Example of Compiled Code

[](https://sourceacademy.org/sicpjs/5.5.5#p1)

Now that we have seen all the elements of the compiler, let us examine an example of compiled code to see how things fit together. We will compile the declaration of a recursive `factorial`function by passing as first argument to `compile` the result of applying `parse` to a string representation of the program (here using back quotes `` ` ``……`` ` ``, which work like single and double quotation marks but allow the string to span multiple lines):

compile(parse(`
function factorial(n) {
    return n === 1
           ? 1
           : factorial(n - 1) * n;
}
              `),
        "val",
        "next");

We have specified that the value of the declaration should be placed in the `val` register. We don't care what the compiled code does after executing the declaration, so our choice of `"next"` as the linkage descriptor is arbitrary.

[](https://sourceacademy.org/sicpjs/5.5.5#p2)

The function `compile` determines that it was given a function declaration, so it transforms it to a constant declaration and then calls `compile_declaration`. This compiles code to compute the value to be assigned (targeted to `val`), followed by code to install the declaration, followed by code to put the value of the declaration (which is the value `undefined`) into the target register, followed finally by the linkage code. The `env` register is preserved around the computation of the value, because it is needed in order to install the declaration. Because the linkage is `"next"`, there is no linkage code in this case. The skeleton of the compiled code is thus

⟨⟨save env if modified by code to compute value⟩⟩
⟨⟨compilation of declaration value, target val, linkage "next"⟩⟩
⟨⟨restore env if saved above⟩⟩
perform(list(op("assign_symbol_value"),
             constant("factorial"),
             reg("val"),
             reg("env"))),
assign("val", constant(undefined))
      

[](https://sourceacademy.org/sicpjs/5.5.5#p3)

The expression that is compiled to produce the value for the name`factorial` is a lambda expression whose value is the function that computes factorials. The function `compile` handles this by calling `compile_lambda_expression`, which compiles the function body, labels it as a new entry point, and generates the instruction that will combine the function body at the new entry point with the runtime environment and assign the result to `val`. The sequence then skips around the compiled function code, which is inserted at this point. The function code itself begins by extending the function's declaration environment by a frame that binds the parameter `n` to the function argument. Then comes the actual function body. Since this code for the value of the name doesn't modify the `env` register, the optional `save` and `restore` shown above aren't generated. (The function code at `entry1` isn't executed at this point, so its use of `env` is irrelevant.) Therefore, the skeleton for the compiled code becomes

    assign("val", list(op("make_compiled_function"), 
                     label("entry1"), 
                     reg("env"))),
  go_to(label("after_lambda2")),
"entry1",
  assign("env", list(op("compiled_function_env"), reg("fun"))),
  assign("env", list(op("extend_environment"),
                     constant(list("n")), 
                     reg("argl"), 
                     reg("env"))),
  ⟨⟨compilation of function body⟩⟩
"after_lambda2",
  perform(list(op("assign_symbol_value"), 
               constant("factorial"), 
               reg("val"), 
               reg("env"))),
  assign("val", constant(undefined))
      

[](https://sourceacademy.org/sicpjs/5.5.5#p4)

A function body is always compiled (by `compile_lambda_body`) with target `val` and linkage `"next"`. The body in this case consists of a single return statement:[1](https://sourceacademy.org/sicpjs/5.5.5#footnote-1)

return n === 1
       ? 1
       : factorial(n - 1) * n;

The function `compile_return_statement` generates code to revert the stack using the marker and to restore the `continue` register, and then compiles the return expression with target `val` and linkage `"return"`, because its value is to be returned from the function. The return expression is a conditional expression, for which `compile_conditional` generates code that first computes the predicate (targeted to `val`), then checks the result and branches around the true branch if the predicate is false. Registers `env` and `continue` are preserved around the predicate code, since they may be needed for the rest of the conditional expression. The true and false branches are both compiled with target `val` and linkage `"return"`. (That is, the value of the conditional, which is the value computed by either of its branches, is the value of the function.)

[](https://sourceacademy.org/sicpjs/5.5.5#p5)

The predicate `n === 1` is a function application (after transformation of the operator combination). This looks up the function expression (the symbol `"==="`) and places this value in `fun`. It then assembles the arguments `1` and the value of `n` into `argl`. Then it tests whether `fun` contains a primitive or a compound function, and dispatches to a primitive branch or a compound branch accordingly. Both branches resume at the `after_call` label. The compound branch must set up `continue` to jump past the primitive branch and push a marker to the stack to match the revert operation in the compiled return statement of the function. The requirements to preserve registers around the evaluation of the function and argument expressions don't result in any saving of registers, because in this case those evaluations don't modify the registers in question.

[](https://sourceacademy.org/sicpjs/5.5.5#p6)

The true branch, which is the constant 1, compiles (with target `val` and linkage `"return"`) to

    assign("val", constant(1)),
  go_to(reg("continue")),
      

The code for the false branch is another function call, where the function is the value of the symbol `"*"`, and the arguments are `n` and the result of another function call (a call to `factorial`). Each of these calls sets up `fun` and `argl` and its own primitive and compound branches. Figure [5.17](https://sourceacademy.org/sicpjs/5.5.5#fig-5.17) shows the complete compilation of the declaration of the `factorial`function. Notice that the possible `save` and `restore` of `continue` and `env` around the predicate, shown above, are in fact generated, because these registers are modified by the function call in the predicate and needed for the function call and the `"return"` linkage in the branches.

// construct the function and skip over the code for the function body
  assign("val", list(op("make_compiled_function"), 
                     label("entry1"), reg("env"))),
  go_to(label("after_lambda2")),
"entry1",                           // calls to factorialfactorial will enter here
  assign("env", list(op("compiled_function_env"), reg("fun"))),
  assign("env", list(op("extend_environment"), constant(list("n")), 
                     reg("argl"), reg("env"))),
// begin actual function body
  revert_stack_to_marker(),         // starts with a return statement
  restore("continue"),
  save("continue"),                 // preserve registers across predicate
  save("env"),
// compute n === 1n === 1
  assign("fun", list(op("lookup_symbol_value"), constant("==="), reg("env"))),
  assign("val", constant(1)),
  assign("argl", list(op("list"), reg("val"))),
  assign("val", list(op("lookup_symbol_value"), constant("n"), reg("env"))),
  assign("argl", list(op("pair"), reg("val"), reg("argl"))),
  test(list(op("is_primitive_function"), reg("fun"))),
  branch(label("primitive_branch6")),
"compiled_branch7",
  assign("continue", label("after_call8")),
  save("continue"),
  push_marker_to_stack(),
  assign("val", list(op("compiled_function_entry"), reg("fun"))),
  go_to(reg("val")),
"primitive_branch6",
  assign("val", list(op("apply_primitive_function"), reg("fun"), reg("argl"))),
"after_call8",                      // valval now contains result of n === 1n === 1
  restore("env"),
  restore("continue"),
  test(list(op("is_falsy"), reg("val"))),
  branch(label("false_branch4")),
"true_branch3",                     // return 1
  assign("val", constant(1)),
  go_to(reg("continue")),
"false_branch4",
// compute and return factorial(n - 1) * nfactorial(n - 1) * n
  assign("fun", list(op("lookup_symbol_value"), constant("*"), reg("env"))),
  save("continue"),
  save("fun"),                      // save ** function
  assign("val", list(op("lookup_symbol_value"), constant("n"), reg("env"))),
  assign("argl", list(op("list"), reg("val"))),
  save("argl"),                     // save partial argument list for **
// compute factorial(n - 1)factorial(n - 1) which is the other argument for **
  assign("fun", list(op("lookup_symbol_value"), 
                     constant("factorial"), reg("env"))),
  save("fun"),                      // save factorialfactorial function
    

##### Figure 5.17 Compilation of the declaration of the `factorial`function (continued on next page).

// compute n - 1n - 1 which is the argument for factorialfactorial
  assign("fun", list(op("lookup_symbol_value"), constant("-"), reg("env"))),
  assign("val", constant(1)),
  assign("argl", list(op("list"), reg("val"))),
  assign("val", list(op("lookup_symbol_value"), constant("n"), reg("env"))),
  assign("argl", list(op("pair"), reg("val"), reg("argl"))),
  test(list(op("is_primitive_function"), reg("fun"))),
  branch(label("primitive_branch10")),
"compiled_branch11",
  assign("continue", label("after_call12")),
  save("continue"),
  push_marker_to_stack(),
  assign("val", list(op("compiled_function_entry"), reg("fun"))),
  go_to(reg("val")),
"primitive_branch10",
  assign("val", list(op("apply_primitive_function"), reg("fun"), reg("argl"))),
"after_call12",                     // valval now contains result of n - 1n - 1
  assign("argl", list(op("list"), reg("val"))),
  restore("fun"),                   // restore factorialfactorial
// apply factorialfactorial
  test(list(op("is_primitive_function"), reg("fun"))),
  branch(label("primitive_branch14")),
"compiled_branch15",
  assign("continue", label("after_call16")),
  save("continue"),                 // set up for compiled function −−
  push_marker_to_stack(),           //   return in function will restore stack
  assign("val", list(op("compiled_function_entry"), reg("fun"))),
  go_to(reg("val")),
"primitive_branch14",
  assign("val", list(op("apply_primitive_function"), reg("fun"), reg("argl"))),
"after_call16",                     // valval now contains result of factorial(n - 1)factorial(n - 1)
  restore("argl"),                  // restore partial argument list for **
  assign("argl", list(op("pair"), reg("val"), reg("argl"))),
  restore("fun"),                   // restore **
  restore("continue"),
// apply ** and return its value
  test(list(op("is_primitive_function"), reg("fun"))),
  branch(label("primitive_branch18")),
"compiled_branch19", // note that a compound function here is called tail-recursively
  save("continue"),
  push_marker_to_stack(),
  assign("val", list(op("compiled_function_entry"), reg("fun"))),
  go_to(reg("val")),
"primitive_branch18",
  assign("val", list(op("apply_primitive_function"), reg("fun"), reg("argl"))),
  go_to(reg("continue")),
"after_call20",
"after_cond5",
"after_lambda2",
// assign the function to the name factorialfactorial
  perform(list(op("assign_symbol_value"), 
               constant("factorial"), reg("val"), reg("env"))),
  assign("val", constant(undefined))
    

##### Figure 5.18 (continued)

[](https://sourceacademy.org/sicpjs/5.5.5#ex-5.35)

**Exercise 5.35**

Consider the following declaration of a factorial function, which is slightly different from the one given above:

function factorial_alt(n) {
    return n === 1
           ? 1
           : n * factorial_alt(n - 1);
}

Compile this function and compare the resulting code with that produced for `factorial`. Explain any differences you find. Does either program execute more efficiently than the other?

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.5#ex-5.36)

**Exercise 5.36**

Compile the iterative factorial function

function factorial(n) {
    function iter(product, counter) {
        return counter > n
               ? product
               : iter(product * counter, counter + 1);
    }
    return iter(1, 1);
}

Annotate the resulting code, showing the essential difference between the code for iterative and recursive versions of `factorial` that makes one process build up stack space and the other run in constant stack space.

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.5#ex-5.37)

**Exercise 5.37**

What program was compiled to produce the code shown in figure [5.19](https://sourceacademy.org/sicpjs/5.5.5#fig-5.19)?

Show Solution

##### Figure 5.19 An example of compiler output (continued on next page). See exercise [5.37](https://sourceacademy.org/sicpjs/5.5.5#ex-5.37).

##### Figure 5.20 (continued)

[](https://sourceacademy.org/sicpjs/5.5.5#ex-5.38)

**Exercise 5.38**

What order of evaluation does our compiler produce for arguments of an application? Is it left-to-right (as mandated by the ECMAScript specification), right-to-left, or some other order? Where in the compiler is this order determined? Modify the compiler so that it produces some other order of evaluation. (See the discussion of order of evaluation for the explicit-control evaluator in section [5.4.1](https://sourceacademy.org/sicpjs/5.4.1).) How does changing the order of argument evaluation affect the efficiency of the code that constructs the argument list?

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.5#ex-5.39)

**Exercise 5.39**

One way to understand the compiler's `preserving` mechanism for optimizing stack usage is to see what extra operations would be generated if we did not use this idea. Modify `preserving` so that it always generates the `save` and `restore` operations. Compile some simple expressions and identify the unnecessary stack operations that are generated. Compare the code to that generated with the `preserving` mechanism intact.

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.5#ex-5.40)

**Exercise 5.40**

Our compiler is clever about avoiding unnecessary stack operations, but it is not clever at all when it comes to compiling calls to the primitive functions of the language in terms of the primitive operations supplied by the machine. For example, consider how much code is compiled to compute `a + 1`: The code sets up an argument list in `argl`, puts the primitive addition function (which it finds by looking up the symbol `"+"` in the environment) into `fun`, and tests whether the function is primitive or compound. The compiler always generates code to perform the test, as well as code for primitive and compound branches (only one of which will be executed). We have not shown the part of the controller that implements primitives, but we presume that these instructions make use of primitive arithmetic operations in the machine's data paths. Consider how much less code would be generated if the compiler could _open-code_ primitives—that is, if it could generate code to directly use these primitive machine operations. The expression `a + 1` might be compiled into something as simple as[2](https://sourceacademy.org/sicpjs/5.5.5#footnote-2)

assign("val", list(op("lookup_symbol_value"), constant("a"), reg("env"))),
assign("val", list(op("+"), reg("val"), constant(1)))

In this exercise we will extend our compiler to support open coding of selected primitives. Special-purpose code will be generated for calls to these primitive functions instead of the general function-application code. In order to support this, we will augment our machine with special argument registers `arg1` and `arg2`. The primitive arithmetic operations of the machine will take their inputs from `arg1` and `arg2`. The results may be put into `val`, `arg1`, or `arg2`. The compiler must be able to recognize the application of an open-coded primitive in the source program. We will augment the dispatch in the `compile`function to recognize the names of these primitives in addition to the syntactic forms it currently recognizes. For each syntactic form our compiler has a code generator. In this exercise we will construct a family of code generators for the open-coded primitives.

1. The open-coded primitives, unlike the syntactic forms, all need their argument expressions evaluated. Write a code generator `spread_arguments` for use by all the open-coding code generators. The function `spread_arguments` should take a list of argument expressions and compile the given argument expressions targeted to successive argument registers. Note that an argument expression may contain a call to an open-coded primitive, so argument registers will have to be preserved during argument-expression evaluation.
2. The JavaScript operators `===`, `*`, `-`, and `+`, among others, are implemented in the register machine as primitive functions and are referred to in the global environment with the symbols `"==="`, `"*"`, `"-"`, and `"+"`. In JavaScript, it is not possible to redeclare these names, because they do not meet the syntactic restrictions for names. This means it is safe to open-code them. For each of the primitive functions`===`,`*`, `-`, and `+`, write a code generator that takes an application with a function expression that names that function, together with a target and a linkage descriptor, and produces code to spread the arguments into the registers and then perform the operation targeted to the given target with the given linkage. Make `compile` dispatch to these code generators.
3. Try your new compiler on the `factorial` example. Compare the resulting code with the result produced without open coding.

Show Solution

---

[[1]](https://sourceacademy.org/sicpjs/5.5.5#footnote-link-1) Because of the `append_return_undefined` in `compile_lambda_body`, the body actually consists of a sequence with two return statements. However, the dead-code check in `compile_sequence` will stop after the compilation of the first return statement, so the body effectively consists of only a single return statement.

[[2]](https://sourceacademy.org/sicpjs/5.5.5#footnote-link-2) We have used the same symbol `+` here to denote both the source-language function and the machine operation. In general there will not be a one-to-one correspondence between primitives of the source language and primitives of the machine.
# 5.5.6   Lexical Addressing

[](https://sourceacademy.org/sicpjs/5.5.6#p1)

One of the most common optimizations performed by compilers is the optimization of name lookup. Our compiler, as we have implemented it so far, generates code that uses the `lookup_symbol_value` operation of the evaluator machine. This searches for a name by comparing it with each name that is currently bound, working frame by frame outward through the runtime environment. This search can be expensive if the frames are deeply nested or if there are many names. For example, consider the problem of looking up the value of `x` while evaluating the expression `x * y * z` in an application of the function of five arguments that is returned by

```javascript
((x, y) =>
   (a, b, c, d, e) =>
     ((y, z) => x * y * z)(a * b * x, c + d + x))(3, 4) 
```

Each time `lookup_symbol_value` searches for `x`, it must determine that the symbol `"x"` is not equal to `"y"` or `"z"` (in the first frame), nor to `"a"`, `"b"`, `"c"`, `"d"`, or `"e"` (in the second frame). Because our language is lexically scoped, the runtime environment for any component will have a structure that parallels the lexical structure of the program in which the component appears. Thus, the compiler can know, when it analyzes the above expression, that each time the function is applied the binding for `x` in `x * y * z` will be found two frames out from the current frame and will be the first binding in that frame.

[](https://sourceacademy.org/sicpjs/5.5.6#p2)

We can exploit this fact by inventing a new kind of name-lookup operation, `lexical_address_lookup`, that takes as arguments an environment and a _lexical address_ that consists of two numbers: a _frame number_, which specifies how many frames to pass over, and a _displacement number_, which specifies how many bindings to pass over in that frame. The operation `lexical_address_lookup` will produce the value of the name stored at that lexical address relative to the current environment. If we add the `lexical_address_lookup` operation to our machine, we can make the compiler generate code that references names using this operation, rather than `lookup_symbol_value`. Similarly, our compiled code can use a new `lexical_address_assign` operation instead of `assign_symbol_value`. With lexical addressing, there is no need to include any symbolic references to names in the object code, and frames do not need to include symbols at run time.

[](https://sourceacademy.org/sicpjs/5.5.6#p3)

In order to generate such code, the compiler must be able to determine the lexical address of a name it is about to compile a reference to. The lexical address of a name in a program depends on where one is in the code. For example, in the following program, the address of `x` in expression e1e1​ is (2,0)—two frames back and the first name in the frame. At that point `y` is at address (0,0) and `c` is at address (1,2). In expression e2e2​, `x` is at (1,0), `y` is at (1,1), and `c` is at (0,2).

((x, y) =>
   (a, b, c, d, e) =>
     ((y, z) => e1e1​)(e2e2​, c + d + x))(3, 4);
      

[](https://sourceacademy.org/sicpjs/5.5.6#p4)

One way for the compiler to produce code that uses lexical addressing is to maintain a data structure called a _compile-time environment_. This keeps track of which bindings will be at which positions in which frames in the runtime environment when a particular name-access operation is executed. The compile-time environment is a list of frames, each containing a list of symbols. There will be no values associated with the symbols, since values are not computed at compile time. (Exercise [5.46](https://sourceacademy.org/sicpjs/5.5.6#ex-5.46) will change this, as an optimization for constants.) The compile-time environment becomes an additional argument to `compile` and is passed along to each code generator. The top-level call to `compile` uses a compile-time-environment that includes the names of all primitive functions and primitive values. When the body of a lambda expression is compiled, `compile_lambda_body` extends the compile-time environment by a frame containing the function's parameters, so that the body is compiled with that extended environment. Similarly, when the body of a block is compiled, `compile_block` extends the compile-time environment by a frame containing the scanned-out local names of the body. At each point in the compilation, `compile_name` and `compile_assignment_declaration` use the compile-time environment in order to generate the appropriate lexical addresses.

[](https://sourceacademy.org/sicpjs/5.5.6#p5)

Exercises [5.41](https://sourceacademy.org/sicpjs/5.5.6#ex-5.41) through [5.44](https://sourceacademy.org/sicpjs/5.5.6#ex-5.44) describe how to complete this sketch of the lexical-addressing strategy in order to incorporate lexical lookup into the compiler. Exercises [5.45](https://sourceacademy.org/sicpjs/5.5.6#ex-5.45) and [5.46](https://sourceacademy.org/sicpjs/5.5.6#ex-5.46) describe other uses for the compile-time environment.

[](https://sourceacademy.org/sicpjs/5.5.6#ex-5.41)

**Exercise 5.41**

Write a function`lexical_address_lookup` that implements the new lookup operation. It should take two arguments—a lexical address and a runtime environment—and return the value of the name stored at the specified lexical address. The function `lexical_address_lookup` should signal an error if the value of the name is the string `"*unassigned*"`. Also write a function`lexical_address_assign` that implements the operation that changes the value of the name at a specified lexical address.

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.6#ex-5.42)

**Exercise 5.42**

Modify the compiler to maintain the compile-time environment as described above. That is, add a compile-time-environment argument to `compile` and the various code generators, and extend it in `compile_lambda_body` and `compile_block`.

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.6#ex-5.43)

**Exercise 5.43**

Write a function`find_symbol` that takes as arguments a symbol and a compile-time environment and returns the lexical address of the symbol with respect to that environment. For example, in the program fragment that is shown above, the compile-time environment during the compilation of expression e1e1​ is

```javascript
list(list("y", "z"),
     list("a", "b", "c", "d", "e"),
     list("x", "y")) 
```

The function `find_symbol` should produce

```javascript
find_symbol("c", list(list("y", "z"),
                      list("a", "b", "c", "d", "e"),
                      list("x", "y"))); 
```

_list(1, 2)_

```javascript
find_symbol("x", list(list("y", "z"),
                      list("a", "b", "c", "d", "e"),
                      list("x", "y"))); 
```

_list(2, 0)_

```javascript
find_symbol("w", list(list("y", "z"),
                      list("a", "b", "c", "d", "e"),
                      list("x", "y"))); 
```

_"not found"_

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.6#ex-5.44)

**Exercise 5.44**

Using `find_symbol` from exercise [5.43](https://sourceacademy.org/sicpjs/5.5.6#ex-5.43), rewrite `compile_assignment_declaration` and `compile_name` to output lexical-address instructions. In cases where `find_symbol` returns `"not found"` (that is, where the name is not in the compile-time environment), you should report a compile-time error. Test the modified compiler on a few simple cases, such as the nested lambda combination at the beginning of this section.

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.6#ex-5.45)

**Exercise 5.45**

In JavaScript, an attempt to assign a new value to a name that is declared as a constant leads to an error. Exercise [4.11](https://sourceacademy.org/sicpjs/4.1.3#ex-4.11) shows how to detect such errors at run time. With the techniques presented in this section, we can detect attempts to assign a new value to a constant _at compile time_. For this purpose, extend the functions `compile_lambda_body` and `compile_block` to record in the compile-time environment whether a name is declared as a variable (using `let` or as a parameter), or as a constant (using `const` or `function`). Modify `compile_assignment` to report an appropriate error when it detects an assignment to a constant.

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.6#ex-5.46)

**Exercise 5.46**

Knowledge about constants at compile time opens the door to many optimizations that allow us to generate more efficient object code. In addition to the extension of the compile-time environment in exercise [5.45](https://sourceacademy.org/sicpjs/5.5.6#ex-5.45) to indicate names declared as constants, we may store the value of a constant if it is known at compile time, or other information that can help us optimize the code.

1. A constant declaration such as `const` _name_ `=` _literal_`;` allows us to replace all occurrences of _name_ within the scope of the declaration by _literal_ so that _name_ doesn't have to be looked up in the runtime environment. This optimization is called _constant propagation_. Use an extended compile-time environment to store literal constants, and modify `compile_name` to use the stored constant in the generated `assign` instruction instead of the `lookup_symbol_value` operation.
2. Function declaration is a derived component that expands to constant declaration. Let us assume that the names of primitive functions in the global environment are also considered constants. If we further extend our compile-time environment to keep track of which names refer to compiled functions and which ones to primitive functions, we can move the test that checks whether a function is compiled or primitive from run time to compile time. This makes the object code more efficient because it replaces a test that must be performed once per function application in the generated code by one that is performed by the compiler. Using such an extended compile-time environment, modify `compile_function_call` so that if it can be determined at compile time whether the called function is compiled or primitive, only the instructions in the `compiled_branch` or the `primitive_branch` are generated.
3. Replacing constant names with their literal values as in part (a) paves the way for another optimization, namely replacing applications of primitive functions to literal values with the compile-time computed result. This optimization, called _constant folding_, replaces expressions such as `40 + 2` by `42` by performing the addition in the compiler. Extend the compiler to perform constant folding for arithmetic operations on numbers and for string concatenation.

# 5.5.7   Interfacing Compiled Code to the Evaluator

[](https://sourceacademy.org/sicpjs/5.5.7#p1)

We have not yet explained how to load compiled code into the evaluator machine or how to run it. We will assume that the explicit-control-evaluator machine has been defined as in section [5.4.4](https://sourceacademy.org/sicpjs/5.4.4), with the additional operations specified in footnote [4](https://sourceacademy.org/sicpjs/5.5.2#footnote-4) (section [5.5.2](https://sourceacademy.org/sicpjs/5.5.2)). We will implement a function`compile_and_go` that compiles a JavaScript program, loads the resulting object code into the evaluator machine, and causes the machine to run the code in the evaluator global environment, print the result, and enter the evaluator's driver loop. We will also modify the evaluator so that interpreted components can call compiled functions as well as interpreted ones. We can then put a compiled function into the machine and use the evaluator to call it:

```javascript
compile_and_go(parse(`
function factorial(n) {
    return n === 1
           ? 1
           : factorial(n - 1) * n;
}
                     `)); 
```

_EC-evaluate value:
undefined_

factorial(5);

_EC-evaluate value:
120_

[](https://sourceacademy.org/sicpjs/5.5.7#p2)

To allow the evaluator to handle compiled functions (for example, to evaluate the call to `factorial` above), we need to change the code at `apply_dispatch` (section [5.4.2](https://sourceacademy.org/sicpjs/5.4.2)) so that it recognizes compiled functions (as distinct from compound or primitive functions) and transfers control directly to the entry point of the compiled code:[1](https://sourceacademy.org/sicpjs/5.5.7#footnote-1)

"apply_dispatch",
  test(list(op("is_primitive_function"), reg("fun"))),
  branch(label("primitive_apply")),
  test(list(op("is_compound_function"), reg("fun"))),
  branch(label("compound_apply")),
  test(list(op("is_compiled_function"), reg("fun"))),
  branch(label("compiled_apply")),
  go_to(label("unknown_function_type")),

"compiled_apply",
  push_marker_to_stack(),
  assign("val", list(op("compiled_function_entry"), reg("fun"))),
  go_to(reg("val")),

At `compiled_apply`, as at `compound_apply`, we push a marker to the stack so that a return statement in the compiled function can revert the stack to this state. Note that there is no save of `continue` at `compiled_apply` before the marking of the stack, because the evaluator was arranged so that at `apply_dispatch`, the continuation would be at the top of the stack.

[](https://sourceacademy.org/sicpjs/5.5.7#p3)

To enable us to run some compiled code when we start the evaluator machine, we add a `branch` instruction at the beginning of the evaluator machine, which causes the machine to go to a new entry point if the `flag` register is set.[2](https://sourceacademy.org/sicpjs/5.5.7#footnote-2)

    branch(label("external_entry")), // branches if flag is set    
"read_evaluate_print_loop",
  perform(list(op("initialize_stack"))),
  ……
      

The code at `external_entry` assumes that the machine is started with `val` containing the location of an instruction sequence that puts a result into `val` and ends with `go_to(reg("continue"))`. Starting at this entry point jumps to the location designated by `val`, but first assigns `continue` so that execution will return to `print_result`, which prints the value in `val` and then goes to the beginning of the evaluator's read-evaluate-print loop.[3](https://sourceacademy.org/sicpjs/5.5.7#footnote-3)

"external_entry",
  perform(list(op("initialize_stack"))),
  assign("env", list(op("get_current_environment"))),
  assign("continue", label("print_result")),
  go_to(reg("val")),

[](https://sourceacademy.org/sicpjs/5.5.7#p4)

Now we can use the following function to compile a function declaration, execute the compiled code, and run the read-evaluate-print loop so we can try the function. Because we want the compiled code to proceed to the location in `continue` with its result in `val`, we compile the program with a target of `val` and a linkage of `"return"`. In order to transform the object code produced by the compiler into executable instructions for the evaluator register machine, we use the function`assemble` from the register-machine simulator (section [5.2.2](https://sourceacademy.org/sicpjs/5.2.2)). For the interpreted program to refer to the names that are declared at top level in the compiled program, we scan out the top-level names and extend the global environment by binding these names to `"*unassigned*"`, knowing that the compiled code will assign them the correct values. We then initialize the `val` register to point to the list of instructions, set the `flag` so that the evaluator will go to `external_entry`, and start the evaluator.

```javascript
function compile_and_go(program) {
    const instrs = assemble(instructions(compile(program,
                                                 "val", "return")),
                            eceval);
    const toplevel_names = scan_out_declarations(program);
    const unassigneds = list_of_unassigned(toplevel_names);
    set_current_environment(extend_environment(
                               toplevel_names,
                               unassigneds, 
                               the_global_environment));
    set_register_contents(eceval, "val", instrs);
    set_register_contents(eceval, "flag", true);
    return start(eceval);
} 
```

[](https://sourceacademy.org/sicpjs/5.5.7#p5)

If we have set up stack monitoring, as at the end of section [5.4.4](https://sourceacademy.org/sicpjs/5.4.4), we can examine the stack usage of compiled code:

compile_and_go(parse(`
function factorial(n) {
    return n === 1
           ? 1
           : factorial(n - 1) * n;
}
                     `));

_total pushes = 0 
maximum depth = 0
EC-evaluate value:
undefined_

factorial(5);

_total pushes = 36 
maximum depth = 14
EC-evaluate value:
120_

Compare this example with the evaluation of `factorial(5)` using the interpreted version of the same function, shown at the end of section [5.4.4](https://sourceacademy.org/sicpjs/5.4.4). The interpreted version required 151 pushes and a maximum stack depth of 28. This illustrates the optimization that results from our compilation strategy.

[](https://sourceacademy.org/sicpjs/5.5.7#h1)

## Interpretation and compilation

[](https://sourceacademy.org/sicpjs/5.5.7#p6)

With the programs in this section, we can now experiment with the alternative execution strategies of interpretation and compilation.[4](https://sourceacademy.org/sicpjs/5.5.7#footnote-4) An interpreter raises the machine to the level of the user program; a compiler lowers the user program to the level of the machine language. We can regard the JavaScript language (or any programming language) as a coherent family of abstractions erected on the machine language. Interpreters are good for interactive program development and debugging because the steps of program execution are organized in terms of these abstractions, and are therefore more intelligible to the programmer. Compiled code can execute faster, because the steps of program execution are organized in terms of the machine language, and the compiler is free to make optimizations that cut across the higher-level abstractions.[5](https://sourceacademy.org/sicpjs/5.5.7#footnote-5)

[](https://sourceacademy.org/sicpjs/5.5.7#p7)

The alternatives of interpretation and compilation also lead to different strategies for porting languages to new computers. Suppose that we wish to implement JavaScript for a new machine. One strategy is to begin with the explicit-control evaluator of section [5.4](https://sourceacademy.org/sicpjs/5.4) and translate its instructions to instructions for the new machine. A different strategy is to begin with the compiler and change the code generators so that they generate code for the new machine. The second strategy allows us to run any JavaScript program on the new machine by first compiling it with the compiler running on our original JavaScript system, and linking it with a compiled version of the runtime library.[6](https://sourceacademy.org/sicpjs/5.5.7#footnote-6) Better yet, we can compile the compiler itself, and run this on the new machine to compile other JavaScript programs.[7](https://sourceacademy.org/sicpjs/5.5.7#footnote-7) Or we can compile one of the interpreters of section [4.1](https://sourceacademy.org/sicpjs/4.1) to produce an interpreter that runs on the new machine.

[](https://sourceacademy.org/sicpjs/5.5.7#ex-5.47)

**Exercise 5.47**

By comparing the stack operations used by compiled code to the stack operations used by the evaluator for the same computation, we can determine the extent to which the compiler optimizes use of the stack, both in speed (reducing the total number of stack operations) and in space (reducing the maximum stack depth). Comparing this optimized stack use to the performance of a special-purpose machine for the same computation gives some indication of the quality of the compiler.

1. Exercise [5.28](https://sourceacademy.org/sicpjs/5.4.4#ex-5.28) asked you to determine, as a function of nn, the number of pushes and the maximum stack depth needed by the evaluator to compute n!n! using the recursive factorial function given above. Exercise [5.13](https://sourceacademy.org/sicpjs/5.2.4#ex-5.13) asked you to do the same measurements for the special-purpose factorial machine shown in figure [5.11](https://sourceacademy.org/sicpjs/5.1.4#fig-5.11). Now perform the same analysis using the compiled `factorial`function.
    
    [](https://sourceacademy.org/sicpjs/5.5.7#p8)
    
    Take the ratio of the number of pushes in the compiled version to the number of pushes in the interpreted version, and do the same for the maximum stack depth. Since the number of operations and the stack depth used to compute n!n! are linear in nn, these ratios should approach constants as nn becomes large. What are these constants? Similarly, find the ratios of the stack usage in the special-purpose machine to the usage in the interpreted version.
    
    [](https://sourceacademy.org/sicpjs/5.5.7#p9)
    
    Compare the ratios for special-purpose versus interpreted code to the ratios for compiled versus interpreted code. You should find that the special-purpose machine is much more efficient than the compiled code, since the hand-tailored controller code should be much better than what is produced by our rudimentary general-purpose compiler.
    
2. Can you suggest improvements to the compiler that would help it generate code that would come closer in performance to the hand-tailored version?

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.7#ex-5.48)

**Exercise 5.48**

Carry out an analysis like the one in exercise [5.47](https://sourceacademy.org/sicpjs/5.5.7#ex-5.47) to determine the effectiveness of compiling the tree-recursive Fibonacci function

```javascript
function fib(n) { 
    return n < 2 ? n : fib(n - 1) + fib(n - 2); 
} 
```

compared to the effectiveness of using the special-purpose Fibonacci machine of figure [5.12](https://sourceacademy.org/sicpjs/5.1.4#fig-5.12). (For measurement of the interpreted performance, see exercise [5.29](https://sourceacademy.org/sicpjs/5.4.4#ex-5.29).) For Fibonacci, the time resource used is not linear in nn; hence the ratios of stack operations will not approach a limiting value that is independent of nn.

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.7#ex-5.49)

**Exercise 5.49**

This section described how to modify the explicit-control evaluator so that interpreted code can call compiled functions. Show how to modify the compiler so that compiled functions can call not only primitive functions and compiled functions, but interpreted functions as well. This requires modifying `compile_function_call` to handle the case of compound (interpreted) functions. Be sure to handle all the same `target` and `linkage` combinations as in `compile_fun_appl`. To do the actual function application, the code needs to jump to the evaluator's `compound_apply` entry point. This label cannot be directly referenced in object code (since the assembler requires that all labels referenced by the code it is assembling be defined there), so we will add a register called `compapp` to the evaluator machine to hold this entry point, and add an instruction to initialize it:

To test your code, start by declaring a function `f` that calls a function `g`. Use `compile_and_go` to compile the declaration of `f` and start the evaluator. Now, typing at the evaluator, declare `g` and try to call `f`.

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.7#ex-5.50)

**Exercise 5.50**

The `compile_and_go` interface implemented in this section is awkward, since the compiler can be called only once (when the evaluator machine is started). Augment the compiler–interpreter interface by providing a `compile_and_run` primitive that can be called from within the explicit-control evaluator as follows:

compile_and_run(parse(`
function factorial(n) {
    return n === 1
           ? 1
           : factorial(n - 1) * n;
}
                      `));

_EC-evaluate value:
undefined_

```javascript
factorial(5) 
```

_EC-Eval value:
120_

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.7#ex-5.51)

**Exercise 5.51**

As an alternative to using the explicit-control evaluator's read-evaluate-print loop, design a register machine that performs a read-compile-execute-print loop. That is, the machine should run a loop that reads a program, compiles it, assembles and executes the resulting code, and prints the result. This is easy to run in our simulated setup, since we can arrange to call the functions`compile` and `assemble` as "register-machine operations."

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.7#ex-5.52)

**Exercise 5.52**

Use the compiler to compile the metacircular evaluator of section [4.1](https://sourceacademy.org/sicpjs/4.1) and run this program using the register-machine simulator. Because the parser takes a string as input, you will need to convert the program into a string. The simplest way to do this is to use the back quotes (`` ` ``), as we have done for the example inputs to `compile_and_go` and `compile_and_run`. The resulting interpreter will run very slowly because of the multiple levels of interpretation, but getting all the details to work is an instructive exercise.

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.7#ex-5.53)

**Exercise 5.53**

Develop a rudimentary implementation of JavaScript in C (or some other low-level language of your choice) by translating the explicit-control evaluator of section [5.4](https://sourceacademy.org/sicpjs/5.4) into C. In order to run this code you will need to also provide appropriate storage-allocation routines and other runtime support.

Show Solution

[](https://sourceacademy.org/sicpjs/5.5.7#ex-5.54)

**Exercise 5.54**

As a counterpoint to exercise [5.53](https://sourceacademy.org/sicpjs/5.5.7#ex-5.53), modify the compiler so that it compiles JavaScript functions into sequences of C instructions. Compile the metacircular evaluator of section [4.1](https://sourceacademy.org/sicpjs/4.1) to produce a JavaScript interpreter written in C.

Show Solution

---

[[1]](https://sourceacademy.org/sicpjs/5.5.7#footnote-link-1) Of course, compiled functions as well as interpreted functions are compound (nonprimitive). For compatibility with the terminology used in the explicit-control evaluator, in this section we will use "compound" to mean interpreted (as opposed to compiled).

[[2]](https://sourceacademy.org/sicpjs/5.5.7#footnote-link-2) Now that the evaluator machine starts with a `branch`, we must always initialize the `flag` register before starting the evaluator machine. To start the machine at its ordinary read-evaluate-print loop, we could use

```javascript
function start_eceval() {
    set_register_contents(eceval, "flag", false);
    return start(eceval);
} 
```

[[3]](https://sourceacademy.org/sicpjs/5.5.7#footnote-link-3) Since a compiled function is an object that the system may try to print, we also modify the system print operation `user_print` (from section [4.1.4](https://sourceacademy.org/sicpjs/4.1.4)) so that it will not attempt to print the components of a compiled function:

function user_print(string, object) {
    function prepare(object) {
        return is_compound_function(object)
               ? "< compound function >"
               : is_primitive_function(object)
               ? "< primitive function >"
               : is_compiled_function(object)
               ? "< compiled function >"
               : is_pair(object)
               ? pair(prepare(head(object)),
                      prepare(tail(object)))
               : object;
    }
    display(string + " " + stringify(prepare(object)));
}

[[4]](https://sourceacademy.org/sicpjs/5.5.7#footnote-link-4) We can do even better by extending the compiler to allow compiled code to call interpreted functions. See exercise [5.49](https://sourceacademy.org/sicpjs/5.5.7#ex-5.49).

[[5]](https://sourceacademy.org/sicpjs/5.5.7#footnote-link-5) Independent of the strategy of execution, we incur significant overhead if we insist that errors encountered in execution of a user program be detected and signaled, rather than being allowed to kill the system or produce wrong answers. For example, an out-of-bounds array reference can be detected by checking the validity of the reference before performing it. The overhead of checking, however, can be many times the cost of the array reference itself, and a programmer should weigh speed against safety in determining whether such a check is desirable. A good compiler should be able to produce code with such checks, should avoid redundant checks, and should allow programmers to control the extent and type of error checking in the compiled code. Compilers for popular languages, such as C and C++, put hardly any error-checking operations into running code, so as to make things run as fast as possible. As a result, it falls to programmers to explicitly provide error checking. Unfortunately, people often neglect to do this, even in critical applications where speed is not a constraint. Their programs lead fast and dangerous lives. For example, the notorious "Worm" that paralyzed the Internet in 1988 exploited the UNIXTMTM operating system's failure to check whether the input buffer has overflowed in the finger daemon. (See Spafford 1989.)

[[6]](https://sourceacademy.org/sicpjs/5.5.7#footnote-link-6) Of course, with either the interpretation or the compilation strategy we must also implement for the new machine storage allocation, input and output, and all the various operations that we took as "primitive" in our discussion of the evaluator and compiler. One strategy for minimizing work here is to write as many of these operations as possible in JavaScript and then compile them for the new machine. Ultimately, everything reduces to a small kernel (such as garbage collection and the mechanism for applying actual machine primitives) that is hand-coded for the new machine.

[[7]](https://sourceacademy.org/sicpjs/5.5.7#footnote-link-7) This strategy leads to amusing tests of correctness of the compiler, such as checking whether the compilation of a program on the new machine, using the compiled compiler, is identical with the compilation of the program on the original JavaScript system. Tracking down the source of differences is fun but often frustrating, because the results are extremely sensitive to minuscule details.

# References

Abelson, Harold, Andrew Berlin, Jacob Katzenelson, William McAllister, Guillermo Rozas, Gerald Jay Sussman, and Jack Wisdom. 1992. The Supercomputer Toolkit: A general framework for special-purpose computing. _International Journal of High-Speed Electronics_ 3(3):337–361.

Allen, John. 1978. _Anatomy of Lisp._ New York: McGraw-Hill.

Appel, Andrew W. 1987. Garbage collection can be faster than stack allocation. _Information Processing Letters_ 25(4):275–279.

Backus, John. 1978. Can programming be liberated from the von Neumann style? _Communications of the ACM_ 21(8):613–641.

Baker, Henry G., Jr. 1978. List processing in real time on a serial computer. _Communications of the ACM_ 21(4):280–293.

Batali, John, Neil Mayle, Howard Shrobe, Gerald Jay Sussman, and Daniel Weise. 1982. The Scheme-81 architecture—System and chip. In _Proceedings of the MIT Conference on Advanced Research in VLSI,_ edited by Paul Penfield, Jr. Dedham, MA: Artech House.

Borning, Alan. 1977. ThingLab—An object-oriented system for building simulations using constraints. In _Proceedings of the 5th International Joint Conference on Artificial Intelligence._

Borodin, Alan, and Ian Munro. 1975. _The Computational Complexity of Algebraic and Numeric Problems._ New York: American Elsevier.

Chaitin, Gregory J. 1975. Randomness and mathematical proof. _Scientific American_ 232(5):47–52.

Church, Alonzo. 1941. _The Calculi of Lambda-Conversion._ Princeton, N.J.: Princeton University Press.

Clark, Keith L. 1978. Negation as failure. In _Logic and Data Bases._ New York: Plenum Press, pp. 293–322.

Clinger, William. 1982. Nondeterministic call by need is neither lazy nor by name. In _Proceedings of the ACM Symposium on Lisp and Functional Programming,_ pp. 226–234.

Colmerauer A., H. Kanoui, R. Pasero, and P. Roussel. 1973. Un système de communication homme-machine en français. Technical report, Groupe d'Intelligence Artificielle, Université d'Aix-Marseille II, Luminy.

Cormen, Thomas H., Charles E. Leiserson, Ronald L. Rivest, and Clifford Stein. 2022. _Introduction to Algorithms._ 4th edition. Cambridge, MA: MIT Press.

Crockford, Douglas. 2008. _JavaScript: The Good Parts._ Sebastopol, CA: O'Reilly Media.

Darlington, John, Peter Henderson, and David Turner. 1982. _Functional Programming and Its Applications._ New York: Cambridge University Press.

Dijkstra, Edsger W. 1968a. The structure of the "THE" multiprogramming system. _Communications of the ACM_ 11(5):341–346.

Dijkstra, Edsger W. 1968b. Cooperating sequential processes. In _Programming Languages_, edited by F. Genuys. New York: Academic Press, pp. 43–112.

Dinesman, Howard P. 1968. _Superior Mathematical Puzzles_. New York: Simon and Schuster.

de Kleer, Johan, Jon Doyle, Guy Steele, and Gerald J. Sussman. 1977. AMORD: Explicit control of reasoning. In _Proceedings of the ACM Symposium on Artificial Intelligence and Programming Languages,_ pp. 116–125.

Doyle, Jon. 1979. A truth maintenance system. _Artificial Intelligence_ 12:231–272.

ECMA. 1997. ECMAScript: A general purpose, cross-platform programming language. 1st edition, edited by Guy L. Steele Jr. _Ecma International._

ECMA. 2015. ECMAScript: A general purpose, cross-platform programming language. 6th edition, edited by Allen Wirfs-Brock. _Ecma International._

ECMA. 2020. ECMAScript: A general purpose, cross-platform programming language. 11th edition, edited by Jordan Harband. _Ecma International._

Edwards, A. W. F. 2019. _Pascal's Arithmetical Triangle_. Mineola, New York: Dover Publications.

Feeley, Marc. 1986. Deux approches à l'implantation du language Scheme. Masters thesis, Université de Montréal.

Feeley, Marc and Guy Lapalme. 1987. Using closures for code generation. _Journal of Computer Languages_ 12(1):47–66.

Feigenbaum, Edward, and Howard Shrobe. 1993. The Japanese National Fifth Generation Project: Introduction, survey, and evaluation. In _Future Generation Computer Systems,_ vol. 9, pp. 105–117.

Feller, William. 1957. _An Introduction to Probability Theory and Its Applications,_ volume 1. New York: John Wiley &amp; Sons.

Fenichel, R., and J. Yochelson. 1969. A Lisp garbage collector for virtual memory computer systems. _Communications of the ACM_ 12(11):611–612.

Floyd, Robert. 1967. Nondeterministic algorithms. _JACM,_ 14(4):636–644.

Forbus, Kenneth D., and Johan de Kleer. 1993. _Building Problem Solvers._ Cambridge, MA: MIT Press.

Friedman, Daniel P., and David S. Wise. 1976. CONS should not evaluate its arguments. In _Automata, Languages, and Programming: Third International Colloquium,_ edited by S. Michaelson and R. Milner, pp. 257–284.

Friedman, Daniel P., Mitchell Wand, and Christopher T. Haynes. 1992. _Essentials of Programming Languages._ Cambridge, MA: MIT Press/McGraw-Hill.

Gabriel, Richard P. 1988. The Why of _Y_. _Lisp Pointers_ 2(2):15–25.

Goldberg, Adele, and David Robson. 1983. _Smalltalk-80: The Language and Its Implementation._ Reading, MA: Addison-Wesley.

Gordon, Michael, Robin Milner, and Christopher Wadsworth. 1979. _Edinburgh LCF._ Lecture Notes in Computer Science, volume 78. New York: Springer-Verlag.

Gray, Jim, and Andreas Reuter. 1993. _Transaction Processing: Concepts and Models._ San Mateo, CA: Morgan-Kaufman.

Green, Cordell. 1969. Application of theorem proving to problem solving. In _Proceedings of the International Joint Conference on Artificial Intelligence,_ pp. 219–240.

Green, Cordell, and Bertram Raphael. 1968. The use of theorem-proving techniques in question-answering systems. In _Proceedings of the ACM National Conference,_ pp. 169–181.

Guttag, John V. 1977. Abstract data types and the development of data structures. _Communications of the ACM_ 20(6):397–404.

Hamming, Richard W. 1980. _Coding and Information Theory._ Englewood Cliffs, N.J.: Prentice-Hall.

Hanson, Christopher P. 1990. Efficient stack allocation for tail-recursive languages. In _Proceedings of ACM Conference on Lisp and Functional Programming,_ pp. 106–118.

Hanson, Christopher P. 1991. A syntactic closures macro facility. _Lisp Pointers,_ 4(4):9–16.

Hardy, Godfrey H. 1921. Srinivasa Ramanujan. _Proceedings of the London Mathematical Society_ XIX(2).

Hardy, Godfrey H., and E. M. Wright. 1960. _An Introduction to the Theory of Numbers._ 4th edition. New York: Oxford University Press.

Havender, J. 1968. Avoiding deadlocks in multi-tasking systems. _IBM Systems Journal_ 7(2):74–84.

Henderson, Peter. 1980. _Functional Programming: Application and Implementation._ Englewood Cliffs, N.J.: Prentice-Hall.

Henderson. Peter. 1982. Functional Geometry. In _Conference Record of the 1982 ACM Symposium on Lisp and Functional Programming,_ pp. 179–187.

Hewitt, Carl E. 1969. PLANNER: A language for proving theorems in robots. In _Proceedings of the International Joint Conference on Artificial Intelligence,_ pp. 295–301.

Hewitt, Carl E. 1977. Viewing control structures as patterns of passing messages. _Journal of Artificial Intelligence_ 8(3):323–364.

Hoare, C. A. R. 1972. Proof of correctness of data representations. _Acta Informatica_ 1(1):271–281.

Hodges, Andrew. 1983. _Alan Turing: The Enigma._ New York: Simon and Schuster.

Hofstadter, Douglas R. 1979. _Gödel, Escher, Bach: An Eternal Golden Braid._ New York: Basic Books.

Hughes, R. J. M. 1990. Why functional programming matters. In _Research Topics in Functional Programming_, edited by David Turner. Reading, MA: Addison-Wesley, pp. 17–42.

IEEE Std 1178-1990. 1990. _IEEE Standard for the Scheme Programming Language._

Ingerman, Peter, Edgar Irons, Kirk Sattley, and Wallace Feurzeig; assisted by M. Lind, Herbert Kanner, and Robert Floyd. 1960. THUNKS: A way of compiling procedure statements, with some comments on procedure declarations. Unpublished manuscript. (Also, private communication from Wallace Feurzeig.)

Jaffar, Joxan, and Peter J. Stuckey. 1986. Semantics of infinite tree logic programming. _Theoretical Computer Science_ 46:141–158.

Kaldewaij, Anne. 1990. _Programming: The Derivation of Algorithms._ New York: Prentice-Hall.

Knuth, Donald E. 1997a. _Fundamental Algorithms._ Volume 1 of _The Art of Computer Programming._ 3rd edition. Reading, MA: Addison-Wesley.

Knuth, Donald E. 1997b. _Seminumerical Algorithms._ Volume 2 of _The Art of Computer Programming._ 3rd edition. Reading, MA: Addison-Wesley.

Konopasek, Milos, and Sundaresan Jayaraman. 1984. _The TK!Solver Book: A Guide to Problem-Solving in Science, Engineering, Business, and Education._ Berkeley, CA: Osborne/McGraw-Hill.

Kowalski, Robert. 1973. Predicate logic as a programming language. Technical report 70, Department of Computational Logic, School of Artificial Intelligence, University of Edinburgh.

Kowalski, Robert. 1979. _Logic for Problem Solving._ New York: North-Holland.

Lamport, Leslie. 1978. Time, clocks, and the ordering of events in a distributed system. _Communications of the ACM_ 21(7):558–565.

Lampson, Butler, J. J. Horning, R. London, J. G. Mitchell, and G. K. Popek. 1981. Report on the programming language Euclid. Technical report, Computer Systems Research Group, University of Toronto.

Landin, Peter. 1965. A correspondence between Algol 60 and Church's lambda notation: Part I. _Communications of the ACM_ 8(2):89–101.

Lieberman, Henry, and Carl E. Hewitt. 1983. A real-time garbage collector based on the lifetimes of objects. _Communications of the ACM_ 26(6):419–429.

Liskov, Barbara H., and Stephen N. Zilles. 1975. Specification techniques for data abstractions. _IEEE Transactions on Software Engineering_ 1(1):7–19.

McAllester, David Allen. 1978. A three-valued truth-maintenance system. Memo 473, MIT Artificial Intelligence Laboratory.

McAllester, David Allen. 1980. An outlook on truth maintenance. Memo 551, MIT Artificial Intelligence Laboratory.

McCarthy, John. 1967. A basis for a mathematical theory of computation. In _Computer Programing and Formal Systems_, edited by P. Braffort and D. Hirschberg. North-Holland, pp. 33–70.

McDermott, Drew, and Gerald Jay Sussman. 1972. Conniver reference manual. Memo 259, MIT Artificial Intelligence Laboratory.

Miller, Gary L. 1976. Riemann's Hypothesis and tests for primality. _Journal of Computer and System Sciences_ 13(3):300–317.

Miller, James S., and Guillermo J. Rozas. 1994. Garbage collection is fast, but a stack is faster. Memo 1462, MIT Artificial Intelligence Laboratory.

Moon, David. 1978. MacLisp reference manual, Version 0. Technical report, MIT Laboratory for Computer Science.

Morris, J. H., Eric Schmidt, and Philip Wadler. 1980. Experience with an applicative string processing language. In _Proceedings of the 7th Annual ACM SIGACT/SIGPLAN Symposium on the Principles of Programming Languages._

Phillips, Hubert. 1934. _The Sphinx Problem Book_. London: Faber and Faber.

Phillips, Hubert. 1961. _My Best Puzzles in Logic and Reasoning_. New York: Dover Publications.

Rabin, Michael O. 1980. Probabilistic algorithm for testing primality. _Journal of Number Theory_ 12:128–138.

Raymond, Eric. 1996. _The New Hacker's Dictionary._ 3rd edition. Cambridge, MA: MIT Press.

Raynal, Michel. 1986. _Algorithms for Mutual Exclusion._ Cambridge, MA: MIT Press.

Rees, Jonathan A., and Norman I. Adams IV. 1982. T: A dialect of Lisp or, lambda: The ultimate software tool. In _Conference Record of the 1982 ACM Symposium on Lisp and Functional Programming,_ pp. 114–122.

Rivest, Ronald L., Adi Shamir, and Leonard M. Adleman. 1978. A method for obtaining digital signatures and public-key cryptosystems. _Communications of the ACM,_ 21(2):120–126.

Robinson, J. A. 1965. A machine-oriented logic based on the resolution principle. _Journal of the ACM_ 12(1):23.

Robinson, J. A. 1983. Logic programming—Past, present, and future. _New Generation Computing_ 1:107–124.

Sagade, Y. 2015. [SICP exercise 1.14](http://www.ysagade.nl/2015/04/12/sicp-change-growth/)

Spafford, Eugene H. 1989. The Internet Worm: Crisis and aftermath. _Communications of the ACM_ 32(6):678–688.

Steele, Guy Lewis, Jr. 1977. Debunking the "expensive procedure call" myth. In _Proceedings of the National Conference of the ACM,_ pp. 153–162.

Steele, Guy Lewis, Jr., and Gerald Jay Sussman. 1975. Scheme: An interpreter for the extended lambda calculus. Memo 349, MIT Artificial Intelligence Laboratory.

Steele, Guy Lewis, Jr., Donald R. Woods, Raphael A. Finkel, Mark R. Crispin, Richard M. Stallman, and Geoffrey S. Goodfellow. 1983. _The Hacker's Dictionary._ New York: Harper &amp; Row.

Stoy, Joseph E. 1977. _Denotational Semantics._ Cambridge, MA: MIT Press.

Sussman, Gerald Jay, and Richard M. Stallman. 1975. Heuristic techniques in computer-aided circuit analysis. _IEEE Transactions on Circuits and Systems_ CAS-22(11):857–865.

Sussman, Gerald Jay, and Guy Lewis Steele Jr. 1980. Constraints—A language for expressing almost-hierarchical descriptions. _AI Journal_ 14:1–39.

Sussman, Gerald Jay, and Jack Wisdom. 1992. Chaotic evolution of the solar system. _Science_ 257:256–262.

Sussman, Gerald Jay, Terry Winograd, and Eugene Charniak. 1971. Microplanner reference manual. Memo 203A, MIT Artificial Intelligence Laboratory.

Sutherland, Ivan E. 1963. SKETCHPAD: A man-machine graphical communication system. Technical report 296, MIT Lincoln Laboratory.

Thatcher, James W., Eric G. Wagner, and Jesse B. Wright. 1978. Data type specification: Parameterization and the power of specification techniques. In _Conference Record of the Tenth Annual ACM Symposium on Theory of Computing_, pp. 119–132.

Turner, David. 1981. The future of applicative languages. In _Proceedings of the 3rd European Conference on Informatics,_ Lecture Notes in Computer Science, volume 123. New York: Springer-Verlag, pp. 334–348.

Wand, Mitchell. 1980. Continuation-based program transformation strategies. _Journal of the ACM_ 27(1):164–180.

Waters, Richard C. 1979. A method for analyzing loop programs. _IEEE Transactions on Software Engineering_ 5(3):237–247.

Winston, Patrick. 1992. _Artificial Intelligence_. 3rd edition. Reading, MA: Addison-Wesley.

Zabih, Ramin, David McAllester, and David Chapman. 1987. Non-deterministic Lisp with dependency-directed backtracking. _AAAI-87_, pp. 59–64.

Zippel, Richard. 1979. Probabilistic algorithms for sparse polynomials. Ph.D. dissertation, Department of Electrical Engineering and Computer Science, MIT.

Zippel, Richard. 1993. _Effective Polynomial Computation._ Boston, MA: Kluwer Academic Publishers.

# Background

[](https://sourceacademy.org/sicpjs/making-of#h1)

## Background

[](https://sourceacademy.org/sicpjs/making-of#p1)

The JavaScript adaptation of SICP is an open-source community effort. The software and data required for making these web pages and the PDF edition are contained in the github repository [source-academy / sicp](https://github.com/source-academy/sicp), and improvements, extensions and discussions are handled in this repository using `git`.

[](https://sourceacademy.org/sicpjs/making-of#p2)

Martin Henz started translating SICP to JavaScript in 2008. He obtained the original LaTeXLATE​X sources of the second edition from Gerald Jay Sussman, and converted them to a custom-built XML format. The original sources are retained in the XML format, which allows for generating the [comparison edition](https://sicp.sourceacademy.org/). A processing system written in XSLT resulted in the first version of the JavaScript adaptation around 2009, covering the first few sections of SICP. The content of SICP JS contained in the XML files are undergoing continuous improvement by the adapters Martin Henz and Tobias Wrigstad, and by the community of SICP JS readers, using the github repository.

[](https://sourceacademy.org/sicpjs/making-of#p3)

In the book, program fragments often require other program fragments. In order to collect and execute the necessary programs, the corresponding `SNIPPET` tags in the xml files include `REQUIRES` tags. The XML processors use these tags in order to assemble the executable programs. The project thus can be seen as a _literate programming system_, custom-made for authoring SICP JS.

[](https://sourceacademy.org/sicpjs/making-of#h2)

## Interactive SICP JS

[](https://sourceacademy.org/sicpjs/making-of#p4)

[Interactive SICP JS](https://sourceacademy.org/sicpjs) was designed and implemented by Samuel Fang in 2021. The XML textbook sources are translated to a JSON format, which are then read and rendered by a dedicated component of the Source Academy.

[](https://sourceacademy.org/sicpjs/making-of#h3)

## Comparison Edition

[](https://sourceacademy.org/sicpjs/making-of#p5)

The precursor of the comparison edition was the mobile-friendly web edition of SICP JS, designed and implemented by Liu Hang in 2017. Feng Piaopiao improved the system in 2018. He Xinyue and Wang Qian developed the [comparison edition](https://sicp.sourceacademy.org/) in 2020. Formulas are retained in the resulting HTML files and are typeset by the reader's browser on the fly, using the MathJax system. The comparison edition is maintained by Martin Henz.

[](https://sourceacademy.org/sicpjs/making-of#h4)

## PDF Edition

[](https://sourceacademy.org/sicpjs/making-of#p6)

The early PDF editions (2010-2018) used XSLT for generating LaTeXLATE​X from the XML sources. The first Node.js version of the PDF edition was designed and implemented by Chan Ger Hean in 2019. Tobias Wrigstad and Martin Henz are the main developers of this system.

[](https://sourceacademy.org/sicpjs/making-of#h5)

## Figures

[](https://sourceacademy.org/sicpjs/making-of#p7)

Most figures are adapted from the [HTML5/EPUB3 version of SICP](https://github.com/sarabander/sicp) by Andres Raba. The figures are licensed under Creative Commons Attribution-ShareAlike 4.0 International License [(cc by-sa)](https://creativecommons.org/licenses/by-sa/4.0). JavaScript adaptations of figures were done by Tobias Wrigstad using Inkscape and gratuitous use of `sed`.