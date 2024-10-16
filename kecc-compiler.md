---
layout: default
---

## The Kecc compiler.
This is my second post on compilers. 
KECC is C compiler developped at [link](KAIST) lab using the Rust programming language. There are few reasons that made me got into another compiler project 
* I recently learned Rust and wanted to used it in a real life project for more understanding.
* Unlike [link](COOL) which is developped in a monolithic way, KECC adopts the very recents compiler development technics which made it very modular and closed to what is used in real compilers structures like [link] LLVM.
* This is my occasion to implement real life optimizations used in real life compilers
* the KECC compiler is provided with a fair amount skeleton code which is most like 
contributing to a open Source project. For somebody aspiring to have a fair
amount of contribution to big open source projects like [link](Pytorch) and [link](LLVM),
this is a perfect way to my maiden voyage into open source contribution.

### KECC organization:
Unlike the [link](COOL Compiler) emphasizes more on the backend part or would I say emphasizes less on the front part 
of the compiler (lexer parser) which are handled by an external crate lang_c which parses the C
language into an AST. The main part of the project consists of translating the 
provided AST into an [link] SSA-based [link] intermediate representation (IR) 
then applied some optimizations upon the generated IR (Gloval Value Numbering, constant propagation) 
and finish by emitting a [RISC-V] assembly program.

### parser
### IR generation
### CFG simplification
### Deadcode Elimination
### Memory to Register (mem2reg) 

## Conclusion



[back](./)
