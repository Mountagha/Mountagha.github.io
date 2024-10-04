---
layout: default
---

## The Cool compiler.

COOL (Classroom Oriented Object Language) is a small programming language created at Stanford solely for academic purposes. Although relatively small compared to mainstream programming languages, it still retains many of the features of modern programming languages, including the Class/Object paradigm, inheritance, static typing, and automatic memory management.
COOL programs are sets of classes. A class encapsulates the variables and procedures of a data type. Instances of a class are objects. COOL is an expression language, meaning most COOL constructs are expressions, and every expression has a value and a type.
Below is an example of the computation of Fibonacci in COOL:
[Snippet of Rust code Fibonacci]
Cool Compiler Organization
If you take a look at the GitHub repository of the project, you’ll see that the repo is organized in a C/C++ style, with the most important directories being:
includes: Contains the headers of the implementations.
src: Contains the actual source code implementations.
examples: Contains COOL source code files, which we test our compiler with.
lib: Contains an important file representing the runtime system of the COOL language. It includes:
The startup code, which invokes the main method of the main program.
Code for methods of predefined classes (Object, IO, String).
Special procedures needed by COOL programs to test objects for equality and handle runtime errors.
The Garbage collector.
Structurally, the COOL compiler follows the traditional monolithic approach to compiler development. In this method, the Abstract Syntax Tree (AST), produced by the parser, is the central component, with all other parts of the compiler depending on it. Here’s the general flow:
The lexer provides input to the parser, which parses the program using grammar rules to produce an AST.
The semantic analysis phase then uses the AST to perform type checking and ensure the program’s conformity, leaving the AST enriched with additional information like type annotations.
Finally, the code generation phase walks through the AST to generate the corresponding machine code for our target architecture, in this case, MIPS.

The Lexer
The Scanner (lexer) is the most straightforward and easiest part of the compiler. It’s a simple, handcrafted component that processes the source code one character at a time and outputs a vector of tokens. A token represents components of the source language such as identifiers, type identifiers, keywords, etc.
[Image class Token]
No magical algorithm is used to scan the source code. Instead, a giant switch-case handles each character or group of characters through pattern recognition, as shown in the snippet below.
The result of the lexer is a vector containing all the tokens encountered in the provided code.

The Parser
The parser involves more logic than the scanner. It takes the vector of tokens and, by following the grammar of the language, constructs an Abstract Syntax Tree (AST) that connects the basic components of the language according to the grammar.
Here’s an excerpt of the COOL grammar, followed by the corresponding parsing code: [Snippet of grammar] [Snippet of Parsing code]
The parser and the following parts of the compiler use the visitor pattern, a common construct used in these scenarios. The visitor pattern allows us to add behavior to existing classes without modifying them directly—only by "visiting" them. If you’d like to dive deeper into this pattern, I suggest this blog by Nystrom, which even provides a Java implementation. If you’re not fond of Java like me, I’ve provided an equivalent implementation in Python. Feel free to read, use, or give feedback.
It’s a very elegant and intuitive pattern to use in these circumstances. After parsing, we’re left with a tree structure with "Program" at the root. The nodes represent multiple "classes," and each class is a root of its own containing "attributes" or "procedures."
There’s also an AST printer provided [here] (for debugging purposes).

The Semantic Analyzer
The semantic analyzer is the final part of the compiler’s front-end and catches the remaining errors in the source code. It’s implemented similarly to the parser, using the visitor pattern. The most important part of the semantic analyzer is type checking. For example, it ensures that:
The arguments of a procedure are compatible with the types of the parameters.
The type of the last expression in a procedure is compatible with the return type.
A value assigned to a variable has a compatible type.
It also checks that variables are defined before use, the inheritance graph is well-formed, and that dispatching selects the correct procedure depending on the object type and arguments (polymorphism).
The output is still the AST but now includes type annotations for all the nodes.

The Code Generator
The code generator is the fourth and final part of the compiler. Its sole role is to generate assembly code—in our case, MIPS—that can run on a machine capable of executing MIPS instructions. This phase generates all constants used in the program, defines the global data, and traverses the AST to generate assembly code for each corresponding node.
[Snippet of visitClassStmt]
This part of the process taught me the most about how programs work under the hood. To get the code to run, I had to debug the generated assembly by hand due to issues like misaligned offsets, corrupted registers, or malformed stack calls. It was all worth it in the end, though, as the compiler successfully compiled and executed all 20 or so source files in the examples directory (some of which are non-trivial).

Conclusion
Writing a compiler is a rewarding experience, at least for me. It’s similar to writing a mini operating system, a database, or any other low-level system. It’s demanding, but the reward at the end makes it worthwhile.
There are some improvements I’d like to add to COOL (probably in the near future) to enhance and make it more robust:
Better Error Handling: This would help programmers with more explicit error messages (like in Rust), providing a better experience.
An Integrated Garbage Collector: Currently, the garbage collector is provided in the runtime system, hard-coded in assembly. Ideally, it should be implemented alongside the compiler.
Basic Optimizations: The current code generation is straightforward but could be improved with optimizations like constant propagation, dead code elimination, or loop unrolling.
If you’re interested in reading the code, the GitHub repository can be found here. Feel free to use, copy, or leave comments.
Thanks for reading!

[back](./)
