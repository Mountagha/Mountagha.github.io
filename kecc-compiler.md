---
layout: default
---

## The Kecc compiler. (In progress.)
This is my second post on compilers. 
KECC is C compiler developped at [KAIST](https://www.kaist.ac.kr/en/) lab using the Rust programming language. There are few reasons that made me got into another compiler project 
* I recently learned Rust and wanted to used it in a real life project for more understanding.
* Unlike [COOL compiler](./cool-compiler.md) which is developped in a monolithic way, KECC adopts the very recent compiler development technics which made it very modular and closed to what is used in real compilers structures like [LLVM](https://llvm.org/).
* This is my occasion to implement real life optimizations used in real life compilers
* the KECC compiler is provided with a fair amount skeleton code which is most like 
contributing to a open Source project. For somebody aspiring to have a fair
amount of contribution to big open source projects like [Pytorch](https://github.com/pytorch/pytorch) and [link](LLVM),
this is a perfect way to my maiden voyage into open source contribution.

### KECC organization:
Unlike the [Cool Compiler](./cool-compiler.md) KECC emphasizes more on the backend part or would I say emphasizes less on the front part 
of the compiler (lexer parser) which are handled by an external crate lang_c which parses the C
language into an AST. The main part of the project consists of translating the 
provided AST into an [SSA-based intermediate representation (IR)](https://en.wikipedia.org/wiki/Static_single-assignment_form) 
then applied some optimizations upon the generated IR (Gloval Value Numbering, constant propagation mem2reg...) 
and finish by emitting a [RISC-V](https://fr.wikipedia.org/wiki/RISC-V) assembly program.

### parser
KAIST parser is entirely handled by [Rust Lang_c crate](https://crates.io/crates/lang-c) which parses the source code into an AST whose root is the Translation unit. The root got external declarations as children which also got their children (You know the music).

```Rust 

// From 6.9 External definitions

/// Entire C source file after preprocessing
///
/// (C11 6.9)
#[derive(Debug, PartialEq, Clone)]
pub struct TranslationUnit(pub Vec<Node<ExternalDeclaration>>);

/// Top-level elements of a C program
///
/// (C11 6.9)
#[derive(Debug, PartialEq, Clone)]
pub enum ExternalDeclaration {
    Declaration(Node<Declaration>),
    StaticAssert(Node<StaticAssert>),
    FunctionDefinition(Node<FunctionDefinition>),
}

/// Function definition
///
/// (C11 6.9.1)
#[derive(Debug, PartialEq, Clone)]
pub struct FunctionDefinition {
    /// Return type of the function, possibly mixed with other specifiers
    pub specifiers: Vec<Node<DeclarationSpecifier>>,
    /// Contains function name and parameter list
    pub declarator: Node<Declarator>,
    /// K&R style parameter type definitions (C11 6.9.1 §6)
    pub declarations: Vec<Node<Declaration>>,
    /// Body of the function.
    pub statement: Node<Statement>,
}

```
In that part of the project, an AST printer was asked to be implemented, going through the AST and then regerenating the C code. The way the parser is tested is pretty clever. The AST printer goes through the AST parser and then generate back the original C source code. To test it, the generated code was parsed again, and the resulting AST was compared with the original to ensure they were identical.

```Rust 

impl WriteLine for TranslationUnit {
    fn write_line(&self, indent: usize, write: &mut dyn Write) -> Result<()> {
        for ext_decl in &self.0 {
            ext_decl.write_line(indent, write)?;
            writeln!(write)?;
        }
        Ok(())
    }
}

impl WriteLine for ExternalDeclaration {
    fn write_line(&self, indent: usize, write: &mut dyn Write) -> Result<()> {
        match self {
            Self::Declaration(decl) => decl.write_line(indent, write),
            Self::StaticAssert(_) => panic!(),
            Self::FunctionDefinition(fdef) => fdef.write_line(indent, write),
        }
    }
}

```

### IR generation
The intermediate representation (IR) generation process has been particularly challenging, as it was my first time working on such a task. What made it even more difficult was the absence of a formal specification for the IR. Instead, I had to rely on a collection of C source files paired with their corresponding IR outputs. It felt somewhat like a reverse engineering exercise to deduce and produce the correct IR.

Like most modern IRs, KECC IR is based on SSA (Static Single Assignment) form. In this representation, each register is assigned a value only once, which simplifies the analysis and facilitates optimizations, as we will explore later.

Below is an example of a Fibonacci program written in C alongside its corresponding generated IR.

```C

int nonce = 1; // For random input

int fibonacci(int n) {
    if (n < 2) {
        return n;
    }

    return fibonacci(n - 2) + fibonacci(n - 1);
}

int main() {
    int number = nonce % 20;
    return fibonacci(number);
}

```
the generated IR.

```
var i32 @nonce = 1

fun i32 @fibonacci (i32) {
init:
  bid: b0
  allocations: 
    %l0:i32:n

block b0:
  %b0:p0:i32:n
  %b0:i0:unit = store %b0:p0:i32 %l0:i32*
  %b0:i1:i32 = load %l0:i32*
  %b0:i2:u1 = cmp lt %b0:i1:i32 2:i32
  br %b0:i2:u1, b1(), b2()

block b1:
  %b1:i0:i32 = load %l0:i32*
  ret %b1:i0:i32

block b2:
  j b3()

block b3:
  %b3:i0:i32 = load %l0:i32*
  %b3:i1:i32 = sub %b3:i0:i32 2:i32
  %b3:i2:i32 = call @fibonacci:[ret:i32 params:(i32)]*(%b3:i1:i32)
  %b3:i3:i32 = load %l0:i32*
  %b3:i4:i32 = sub %b3:i3:i32 1:i32
  %b3:i5:i32 = call @fibonacci:[ret:i32 params:(i32)]*(%b3:i4:i32)
  %b3:i6:i32 = add %b3:i2:i32 %b3:i5:i32
  ret %b3:i6:i32

block b4:
  j b3()

block b5:
  ret undef:i32
}

fun i32 @main () {
init:
  bid: b0
  allocations: 
    %l0:i32:number

block b0:
  %b0:i0:i32 = load @nonce:i32*
  %b0:i1:i32 = mod %b0:i0:i32 20:i32
  %b0:i2:unit = store %b0:i1:i32 %l0:i32*
  %b0:i3:i32 = load %l0:i32*
  %b0:i4:i32 = call @fibonacci:[ret:i32 params:(i32)]*(%b0:i3:i32)
  ret %b0:i4:i32

block b1:
  ret 0:i32
}

```
#### Test Driven Development (TDD) to produce the correct IR.

KECC provides an excellent skeleton code, including an IR interpreter that evaluates the generated IR and produces a result equivalent to executing the original C code. This interpreter made testing the IR generation process straightforward: by interpreting both the provided IR and the IR generated by our program, we could compare their results to validate correctness.

One particularly interesting aspect was how the author suggested developing the IR generation process. KECC comes with a fuzzer, a tool that generates random C files—ranging from simple programs with a few lines to highly complex ones with thousands of lines. The fuzzer tests the IR generation pipeline by feeding these random C files into our IR generation code, interpreting the resulting IR through the KECC IR interpreter, and comparing the output to that produced by GCC or Clang.

This process proved invaluable for uncovering numerous bugs that would have been almost impossible to detect otherwise. The workflow was as follows:

1. Generate a random C file using the fuzzer.
2. Generate the corresponding IR for the C file using our IR generation program.
3. Interpret the generated IR using KECC's IR interpreter.
4. Compile and execute the original C file with GCC or Clang.
5. Compare the results of both executions.

### Optimization

#### CFG simplification
#### constant propagation
#### Deadcode Elimination
#### Memory to Register (mem2reg) 

### Code Generation (RISCV 64)

## Conclusion



[back](./)
