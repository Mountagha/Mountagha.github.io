---
layout: default
---

## The Cool compiler.

COOL (Classroom Oriented-Object language) is a small programming language created at Standford solely for academic purposes. Although relatively small compared to main stream programming languages, it still retains many of the features of modern programming languages including the Class/Object paradigm, inheritance, static typing and automatic memory management.
Cool programs are sets of classes. A class encapsulates the variables and procedures of a data type. Instances of a class are objects.
Cool is an (italic: expression language). Most cool constructs are expressions, and every expression has a value and a type.
Below an example of computation of fibonacci in cool

Snippet of Rust code fibonacci

Cool Compiler Organization
The way the cool compiler is organized is the old way compiler are developed i.e the monolithic way. What I mean by that is that the the abstract syntax tree which is the result of the parser is the central piece over with all the other part of the compilers depend. The flow goes like this. The lexer gives its input to the parser which parse the program using the grammar to produce an AST. Then the semantic analysis use the latter to do it analysis mostly type checking the program and ensure conformity of the program and then left the AST with more informations such as type annotations. Then finally the codegen part walk through the AST to generate the corresponding machine code in which we are compiling our source code to; in our case [link]mips.


[back](./)
