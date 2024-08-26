# Writing a C Compiler - Feedback and C++ implementation
  
Hello @romainducrocq,  
  
You can find wheelcc, my C++ implementation of Writing a C Compiler at:  
https://github.com/romainducrocq/wheelcc  
  
## Intro  
  
I’m a young software engineer from France, currently living and working in the Czech Republic, Europe. I have been fascinated by making languages for some while now, ever since I started following the streamer Tsoding and his live-coding series of making a concatenative stack-based language called Porth.  
This gave me the overall picture of how languages worked but found myself to be lost for quite some time in my learning after that. I majored in AI in university without taking any compilers classes at the time and have worked in C/C++ library development ever since, so I was starting literally from the ground up, with no experience or background whatsoever. I tried for some time to get around with some internet tutorials, but the content was too shallow, too limited: I wanted to build a language that could be used to make some serious, real-world development.  
I also tried to go through the infamous Dragon Book (more than once) but could never get past the academic dryness and lack of overall applicability, and it has been sitting on my shelf ever since, dusty, serving no more purpose than to signal that I’m a “serious compiler developer”. Great: I knew how a recursive descent parser was working on paper and vaguely that three-address code was a thing, now how do I translate this into programming a language?  
  
This is when, in yet another session of googling for answers, I stumbled across your blog, which led me to your book, which I purchased right away. I thus got my hands on the **Early Access Edition of October 2023**, and it changed the game for me. I have been working for the past 10 months (on and off, as far as my limited free time was allowing it) on my own implementation of a C compiler and have gotten the project to a state of maturity that I’m satisfied enough with that I want to share it.  
  
## Review
  
I saw recently that the book was published with reviews from the Director of Engineering at Google, as well as professors and authors, but I sincerely think that, as a writer, you would also enjoy the feedback of a target audience reader – an average engineer a few years out of college, with no prior experience in making compilers, but that got fascinated and seriously invested in the project. And even more than that, see how it impacted him, and what he achieved with it.  
  
First and foremost: the book is **FANTASTIC**. It has the perfect hands-on approach fitting for anyone that prefers learning by tinkering, playing with, tweaking and/or bending code, getting their hands dirty with full blown implementations, rather than with theoretical rigour, proofs and descriptive completeness. In other words, while there are plenty of books out there for PhDs, this is the first I’m coming across **FOR ENGINEERS**. I’m one of the many (many) out there that learns by doing, and god damn, I’m happy that this book exists. Here, there is no need to swallow so much info that you’ll become disgusted and saturated by the time you write the first line of code; no, you learn along the way, by seeing the results at each and every step.  
  
On one hand, there is all the information you could possibly need such that you never have to look it up somewhere else, and you can stick with the book the whole way for it is completely sufficient on its own (but the numerous additional resources and links are more than welcome to dig deeper on specific points that would nonetheless scratch your interest). On the other hand, nothing is given to you on a plate, and you still must go through every line with the uttermost attention to make things work, which is absolutely great for understanding and learning. Thus, you can only get a working result if you REALLY understand what is going on.  
  
About the content itself, the features are well chosen and there are enough to get a serious real-world language. If we forget for a second that this is a C implementation, the resulting language could really be used for big enough projects just as it is. And if I had to extend it further, I would personally be mostly interested in implementing enumerations, function pointers and constant types.  
The difficulty curve is also nicely increasing over the course of the project. Once you understand the layout of the first Chapter, as well as how to use the test suite (which took me a few days), the first 8 chapters are going quite smoothly. Chapter 9 and 10 are a bit harder to go through, but still very manageable, and I finished Part 1 in around 3 months. 
Things are getting serious in Part 2 though, where I spent a whole 6 months. The overall most difficult Chapter was the 15th, with the introduction of aggregate types, which had me rewrite many portions of the project to adjust for the added complexity of handling arrays (and structures to come). I also give an honourable mention to the assembly generation of Chapter 13 and 18, which got me some real headaches to put everything together. I spent an ungodly amount of time debugging the assembly code all trough Part 2, but what I could not figure out from the book, I was getting by the many comments in the tests.  
(And as for Part 3, I have yet to do it.)  
  
Thus, with no background nor prior knowledge, I managed to get it all done: **which means that anyone can!** This project also gave me a much deeper understanding of the C language, and made me quite significantly better at my job, which includes a good chunk of C development. For all of that, **thank you**.  
  
## My C++ implementation
  
At this stage, I have fully implemented Part 1 and 2 (Chapters 1 to 18, and all the extra credits).  
  
### Overall implementation
  
When I started the project, I hesitated between Haskell, Rust, C and C++ for the implementation.  
I discarded Haskell as it was too close to OCaml and the functional pseudo-code given in the book, so to avoid falling into the trap of doing a lazy translation without much added value. For Rust, I figured that its fast-growing popularity and native support for pattern matching and algebraic data types would make it the perfect candidate for the project: thus, there would be many other people making way better Rust implementations than I could ever do myself. As for C, the bareness of the standard library would have me spending even more time reimplementing all sorts of features than working on the implementation itself.  
Hence, I decided on building my C compiler in C++, as it is anyway a language I know well and enjoy a lot. Moreover, C++ seemed kinda unfit for the task, without pattern matching nor algebraic data types, and I felt like I was setting myself up for a really fun and exciting time. Despite that, I first implemented Part 1 in Cython to prototype the architecture of the project, and when I was happy enough with it, I rewrote it to C++ before jumping to Part 2.  
  
Yet, as much as I like C++ for its power and control, I am not a big advocate of OOP and other pitfalls of the language. I like to have my C++ close to a C-style like coding design and approach it as procedural C with added sugar. Because of that, I restricted my implementation to a limited subset of C++:  
- smart pointers with reference counting to manage the lifetime of AST nodes,  
- single inheritance and polymorphism to emulate pattern matching on algebraic data types,  
- standard containers, collections, string manipulation and move semantics.  
  
Other than that, the code mostly resembles procedural C, which aims at making it cleaner without adding too much complexity.  
The code is organized by compilation stages: each compilation stage is a single translation unit, with its own data context that is modified through (a lot of) local functions. In that respect, it relies very largely on performing side-effects and functional impurity, and is thus far from your OCaml reference implementation. (But note that I have not read at all the source code of your OCaml compiler until after I finished my own implementation, as to go completely with my way of doing things.)  
  
Because of the very nature of procedural C++ (and my own implementation choices), my implementation has come to diverge greatly from the pseudo-code provided in the book. First, I do not traverse the AST by passing immutable data that is copy-modified in a functional fashion, but I instead pass around pointers to polymorphic data types and modify the referenced content along the way. Second, I have changed multiple algorithms and compilation passes, some of the most noticeable examples being:  
- the whole Semantic Analysis (Identifier Resolution, Loop Labelling and Type Checking) is done in a single traversal of the AST,  
- and the same is done for the Pseudo-register Replacement and the Stack Instruction Fix-up which happen in a single pass as well.  
  
My implementation is targeting Linux, and I have validated it on 7 different distros across:  
- 4 machines: `Debian [11th Gen Intel i7, 16GB Ram]`, `Rocky Linux [11th Gen Intel i5, 16GB Ram]`, `Debian [8th Gen Intel i7, 8GB Ram]` and `Linux Mint [AMD Ryzen 5, 4GB Ram]`,  
- 6 VMs: `Debian`, `Ubuntu`, `OpenSUSE Leap`, `Rocky Linux`, `Arch Linux` and `EndeavourOS`.
  
(Also, I don’t plan to port it to MacOS, as I don’t own nor have access to a MacBook.)  
  
### Build, Run, Test and Performance
  
I have also worked a lot at making the compiler easy to build and use. It works almost out of the box on any Linux, as the only required dependencies for building and runtime are `gcc/g++`, `cmake` and `bash`. It depends only on the C and C++ standard libraries, as well as 2 header-only third-parties already included in the source (boost::regex for the lexer and tinydir for the preprocessor), and POSIX-compliant bash for the driver. Plus, there are scripts to configure, build and install (described in the Readme of the project).  
The compiler is intended to be used with the system-wide command `wheelcc` (which is in fact just a symlink to the driver installed in `/usr/bin/local/`).  
  
I have also added my own scripts for testing, as it makes it easier for me to do a lot of rapid testing, add tests specific to the implementation language or cover additional features outside of the scope of the book (these are also described in the Readme). Otherwise, in order to test with your test suite, you must provide the path to the driver (as the test suite doesn’t handle symlinks as path arguments), like this:  
  
```
$ ./test_compiler ../wheelcc/bin/driver.sh --chapter 18 --extra-credit
----------------------------------------------------------------------
Ran 1379 tests in 34.557s

OK
```
  
Furthermore, I have measured the runtime performance of my implementation against yours. I measured the `real` time (as in, not user or sys), of compiling all the tests of the test suite (minus extra credits) a 100 times, with the `-S` flag (to exclude linking time); for both compilers.  
- i.e.: real time of 100x { `nqcc2.exe -S <file>` for all files in the test suite }, and real time of 100x { `wheelcc -S <file>` for all files in the test suite }.  
  
While there is some significant difference in between machines, I found them to be approximately on par with each other, with my implementation having a small speed advantage, going from a 10% to a 40% faster runtime on different PCs. Still, it is the same order of magnitude, which I was expecting as:  
- OCaml is praised for its fast execution runtime, matching C and C++ in many cases,  
- and I’m just FAR from being the best C++ programmer out there. My whole AST is based on virtual calls and heap allocations, which are not exactly the fastest tools in the box.  
  
Yet, I have extensively profiled my code and it would be hard to improve execution speed any further without falling into micro-optimization, for the simple reason that there is one major and huge bottleneck: on average, 80% of the runtime is spent in the regex during lexing alone. So, at this point, any significant performance increase would necessitate rewriting the lexer without regex.  
  
### Further additions
  
Lastly, I have extended (and still plan to extend) the implementation past the content of the book. Since this post has all been too long already, I will keep this section short and just go over some noteworthy additions:  
- wheelcc has a small built-in preprocessor, that supports includes and comments. Other preprocessor directives, like pragmas, are stripped out and ignored. This means that preprocessing with gcc is only required to support macro expansion but is otherwise not needed,  
- it also has a user-friendly and comprehensive error handler. Error messages include the source line causing the error, and some additional information on variables and types involved. It is still very rudimentary, but it helps a lot while debugging.  
  
As for what is to come: I am now about to take a few weeks of vacations (and a break from the project at the same time), and I plan on starting to implement Part 3 when I come back, circa end of September.  
Once I’ve gone through the whole book, I want to continue adding features and growing my compiler further. I have already in mind to experiment with different things, like:  
- implementing an experimental standard library that will support at the very least the methods used in the test suite (puts, strcmp, malloc, etc.). I want to experiment with syscalls and be able to pass all the tests with -nostdlib,  
- adding an alternative backend for Linux FASM (Intel-based flat assembler). This assembler can turn single-translation-unit programs into very small and fast executables directly, without the need for a linking stage. This assembler is very lightweight and could be embedded directly into a release package of the project, hence allowing to ship a prebuilt completely self-contained compiler for approximately 100KB total without requiring gcc at runtime at all.  
  
    => **Overall, I think that it would be a nice addition to the project to reach a point of development where my implementation could pass the whole test suite without gcc.**  
  
Finally, for the long term of the project, I have another goal in mind (I don’t think this will happen any time before well into 2025): the reason why I started this in the first place, making my own language! I want to create a second, separate project that will use the IR and backend of my C compiler as a target, but provide a new frontend, with a new grammar and semantic that I will design, to create my own “Real-World programming language”.  
  
## Thank you
  
**If you have read this very long message until the end, I’m more than thankful. And more than all, thank you for this amazing book that I have spent way too much time reading and re-reading over the year.**  
  
Romain Ducrocq.  
