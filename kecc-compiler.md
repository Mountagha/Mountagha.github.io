---
layout: default
---

## The Kecc compiler.
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
The intermediate representation generation part is pretty challenging as it is the first time I was experiencing it. What makes 
### CFG simplification
#### constant propagation
### Deadcode Elimination
### Memory to Register (mem2reg) 

## Conclusion



[back](./)
