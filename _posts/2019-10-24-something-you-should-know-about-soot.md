---
layout: post
title: Something you should know about Soot
date: 2019-10-24 00:00:00 -0000
tags: [projects]
image: android-logo.png
---

[Soot][soot_link] started off as a Java optimization framework. By now, researchers and practitioners from around the world use Soot to analyze, instrument, optimize and visualize Java and Android applications.

[soot_link]: https://sable.github.io/soot/

### Who develops and maintains Soot?
Soot was originally developed by the Sable Research Group of McGill University. The [first publication][soot_paper] on Soot appeared at CASCON 1999. The current maintenance is driven by Eric Bodden’s Software Engineering Group at Heinz Nixdorf Institute of Paderborn University.

[soot_paper]: https://dl.acm.org/citation.cfm?id=1925818


### What can Soot do?

Soot can perform program analysis, e.g., parsing methods, building the control-flow graph (CFG), building call graph, and performing points-to analysis, on Android applications and other java artifact. 

### What is CFG?
1. A control flow graph provides finer details into the structure of the program as a whole, and of the subroutines in paricular.
2. In the control-flow graph, the nodes are simplified statements, and the edges are possible control flows between such statements.
3. In general, CFG is an over-approximation. It processes the branches with the condition maybe true and false. 

### What is call graph? 
Call graph is a static abstraction of all method calls that a program may execute at runtime. The call graph is a representation of the calling relationships between subroutines in a program. 


### How does Soot work?

- Execution of Soot is separated into several phases called packs. The first step is to produce Jimple Code to be fed into the various packs. 
- All variables and method calls are explicitly declared, even `this` reference. Special references for `this` and parameters. `this := @this: Main;` will be transformed to `virtualinvoke this.<Main: void bar(Main,int)>(this, i1);`. 
- The most popular IR produced by Soot is Jimple (Java's Simple). Jimple has no stack operations, instead assignments. The statement of Jimple has at most one reference on the left-hand side and at most two references on the right-hand side.
- Soot manages the method as a `Body`. We can use a `Body` to access various information, most notably we can retrieve a Collection, called Chain. The `getLocals()` can get locals decalred, and the `getUnits()` can get the statements.
- Soot manges statement as `Unit`. Through a `Unit` we can retrieve values used `getUseBoxes()`, values defined `getDefBoxes()` or even both `getUseAndDefBoxes()`. We can get at the `Units` jumping to this `Unit` `getBoxesPointingToThis()` and `Units` this `Unit` is jumping to `getUnitBoxes()`.  `Unit` also provides various methods of querying about branching behavior, such as `fallsThrough()` and `branches()`.
- A single datum is represented as a `Value`, e.g., locals, constants, expressions, and many more.
- References are called `Boxes`. `UnitBoxes` refer to `Units`. Used when a single `Unit` can have multiple successors, i.e. when branching. `ValueBoxes` refer to `Values`. As previously described, each `Unit` has a notion of values used and defined in it, this can be very useful for replacing use or def boxes in units, for instance when performing constant folding. 
- All classes can be categorized into argument classes, application classes, and library classes.
  - The argument classes is the classes we specify to Soot.
  - The application classes are the classes that are the classes to be analyzed or transformed by Soot, and turned into output.
  - The library classes are the classes that are referred to by application classes but not application classes. They are used in the analyses and transformations but are not themselves transformed or output.


### Call Soot perform inter-procedural analysis?
Yes! The inter-procdural analysis of Soot is provided by Heros, which implements the IFDS and IDE frameworks.
