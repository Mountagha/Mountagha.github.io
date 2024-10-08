---
layout: default
---

## My Journey to Compilers

Ever since I started programming seriously at university, compiler technology has always fascinated me. The process of taking a textual form of some unreadable gibberish and turning it into something that actually executes and produces meaningful output was nothing short of amazing to me.

Over the past years, I’ve tried multiple times to build a compiler that would go beyond what is taught at school. In high school, all my compiler classes were purely theoretical, focusing on state machines and how to turn non-deterministic automata into deterministic ones. None of that really made me understand what a compiler was. I just knew that it was this magic piece of software that could transform text from an editor into something understandable by the computer hardware.

After graduating and starting a full-time job, I figured I could finally write a compiler and quench my curiosity. It was during this time, after extensive research, that I stumbled upon the [COOL compiler](https://web.stanford.edu/class/cs143/)—an academic compiler whose purpose was to teach compiler technology in a relatively manageable time frame, meaning not too long. The entire source code wouldn’t exceed roughly 5,000 lines of code. The course was well-organized, breaking the content into multiple video capsules, each smoothly introducing a concept by the brilliant teacher [Alex Aiken](https://profiles.stanford.edu/alex-aiken)...

I started motivated and went through all the necessary theory to implement the lexer. But when I actually began writing it, I disappointingly found that the lexer was handled by the [YACC suite](https://en.wikipedia.org/wiki/Yacc), which did not give me full credit for the work. I convinced myself that the lexer would be simple enough to understand, and if I wanted to build it from scratch later, I could. At least that’s what I told myself to avoid quitting. I continued and finished the lexer, then moved on to the parser. After watching all the theory videos, I began coding, only to discover that another popular tool, [Bison](https://en.wikipedia.org/wiki/GNU_Bison), was used for the parser. My frustration grew. I realized that even if I finished the compiler, it wouldn’t completely satisfy my curiosity. Like any quitter, I gave up. I let the project fade away and returned to my day job.

It wasn’t until a year or so later, when I stumbled upon [Robert Nystrom](https://x.com/munificentbob)’s book (the online version), that everything finally clicked. If there’s one technical book I’ve enjoyed the most, it would be [Crafting Interpreters](https://craftinginterpreters.com/) of the same author. I enjoyed his writing so much that I bought the hard copy, even though the online version would have been sufficient. The book is thick—around 700 pages—well-written, and easy to read, with bits of humor here and there. Although the book teaches how to write an interpreter, the concepts are almost always the same as those needed to write a compiler. The book is divided into two main parts. The first part implements an AST traversal interpreter for a small, well-crafted language. The second part (my favorite) implements a virtual machine interpreter, which is like a real compiler, only it simulates the CPU on which the bytecode is executed.

Needless to say, I read the book thoroughly, back and forth, and implemented both versions of the interpreters. (See my GitHub [here](https://github.com/Mountagha/crafting-interpreter) and [here](https://github.com/Mountagha/crafting-interpreter) for the AST-based and bytecode-based versions, respectively.) What I found really interesting was Nystrom vertical approach to writing, instead of the traditional horizontal approach. You don’t have to implement the whole lexer, for example, before moving on to the parser. His approach was more iterative: build a lexer for a small subset of the language’s syntax, then parse that part and interpret it to see the result. Then, move on to another language feature, iterating over and over until all the features/constructs of the language are supported.

After implementing my first interpreter, I felt more confident but was still left with some frustration. The mini-language for which the interpreter is written is relatively simple. It’s dynamically typed, which simplified things since I didn’t have to handle type checking, a major part of a compiler for typed languages.

I then decided to return to my old friend, [the COOL compiler](https://web.stanford.edu/class/cs143/), which I had abandoned a year or two earlier. But this time, I resolved to write my own lexer and parser from scratch, armed with the skills I had learned from Nystrom's book.

The lexing and parsing went relatively well. It became more challenging when I reached the semantic analysis stage. There were many logical checks, each requiring two or three programming concepts or data structures to overcome, which were challenging in their own right. Type inference, inheritance checking, type conformity... But that, I discovered, is the beauty of compilers. They require almost all of the data structure techniques you learn in a typical computer science curriculum. 

However, the real challenge—the one that took the longest to get right—was the code generation phase. Debugging and getting the compiler to correctly process all the COOL compiler source files took a tremendous amount of effort. This was expected, though, as it was new territory for me, far removed from the smooth tutorial on interpreters I had learned from Nystrom’s book. Here, I was wrestling with octets in memory and stack offsets (yes, I often debugged the generated assembly by hand, computing memory addresses manually). But the reward I felt when all the source files were compiled and executed was far greater than I had expected. Plus, I learned a ton about how a program is executed at the lowest level (well, not the *lowest* level—I wasn’t reading bits flowing through circuits, haha).

If you’re interested in my side projects, particularly the technical aspects, please have a look at this [page](./cool-compiler.md) where I discuss the COOL compiler and [here](./kecc-compiler.md) which is a more sophiticated compiler using modern compiler construction techniques; the same approaches used in a compiler structrure like [LLVM](https://llvm.org/). Or, even better, check out my GitHub repository where you can review the code and provide feedback.

Thanks for reading.



[back](./)
