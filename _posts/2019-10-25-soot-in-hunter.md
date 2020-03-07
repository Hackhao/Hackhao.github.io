---
layout: post
title: 'Use Soot in Project Hunter'
date: 2019-10-25 00:00:00 -0000
tags: [projects]
image: android.png
---

<i class="fa fa-leaf"></i> __Since the work has not been published, it is not convenient to post the details of Project Hunter. But the use of Soot in Project Hunter can be shared with everyone.__

***

### How does Soot build a Call Graph?

__First__, we need to build a call graph of an Android application. I need to evaluate whether Soot can provide the functions we require. The following is the workflow of Soot when building a call graph.

1. Mark all methods of the application classes as reachable methods.
2. Mark all _<span style="color:red">implicit methods</span>_ as the reachable methods, e.g., ___ThreadGroup___ and ___ClassLoader___. That is _<span style="color:green">Soot has taken the thread and reflection into the consideration.</span>_ [Click][entrypoints_site] for details.
3. Call ___OnFlyCallGraphBuilder___ to build the Call Graph with the reachable methods.
  - We have set the _types\_for\_invoke_ and _cg.spark_ options.
4. Distinguish a user class and a JDK class. (Soot uses the package prefix as the heuristic.)
5. For all user's classes, we perform the [type based reflection resolution][tbrr_site].
6. Build a fast hierarchy.
7. Process the methods. 
   1. Handle the reachable methods:
     - If the method contains `<init>`, build an Edge from the `<init>` to the `void finalize()`.
     - If the method belongs to the reflection, it will be added to the RefloectionModel.
   2. Handle the string constants.
   3. Handle the arrays.
   4. Handle the bases.
   5. Handle the receivers. 

After analysis, I think Soot is sufficient to build the Call Graph we need.

[entrypoints_site]: https://github.com/Sable/soot/blob/4a04145d1a4d2713ef2733e33abd5ba77b2fd848/src/main/java/soot/EntryPoints.java
[tbrr_site]: https://github.com/Sable/soot/blob/master/src/main/java/soot/jimple/toolkits/callgraph/OnFlyCallGraphBuilder.java


***

### How does Soot analyze the infoflow?

__Second__, we need to analysis the infoflow of the application to help us track some variables.

#### Background 

__<span style="color:red">Interprocedural Flow-insensitive Summary</span>__

The Interprocedural flow-insensitive summary describes for each external entry, the set of formal parameter variables and global variables that may be used (data read from) and the set of formal parameter variables and global variables that may be modified by an invocation of that entry. 

The terms __flow-insensitive__ and __may__ distinguish the information from __flow-sensitive__ and __must__ information. Intuitively, __must__ facts hold on all execution paths through a subroutine (e.g., __variable X must be modified__) while __may__ facts hold on at least one execution path. 

A problem is __flow-sensitive__ if information about control internal to subroutines are used to compute the final set of data flow facts and a __flow-insensitive__ if this information is ignored. The term __summary__ reflects the use of the information to summarize the behavior of the external routine. 

Flow-insensitive may summary information that can dramatically reduce the number of data dependencies.


__<span style="color:red">The Program Summary Graph</span>__

The program summary graph is an abstraction of a complete program. It summarizes the interprocedural control flow in a way that generalizes the more traditional call graph but is more compact than the program supergraph described by Myers[^1]. The program summary graph has four types of nodes: _entry nodes_, _call nodes_, _exit nodes_, and _return nodes_. There are entry and exit nodes for every formal parameter of every routine. There are call and return nodes for every actual parameter of every call site. These nodes represent the four interprocedural events: procedure entry, procedure invocation, procedure exit, and procedure return. For now, assume that global variables are treated explicitly as formal parameters and actual parameters.

There are edges from call nodes to entry nodes that correspond to the binding of formal parameters to actual parameters. There are also edges from exit nodes to return nodes that correspond to this same binding. These edges depend only on the call structure of the program, not the internals of the subroutine.

There are also edges from entry and return nodes to call and exit nodes. These edges summarize the control flow structure of the subroutine, and their construction begins with solving the standard data flow problem of reaching definitions: a definition of variable __x__ reaches a point __p__ in the subroutine if there is an execution path from the definition to __p__ along which __x__ is not killed. Local to each routine, we can construct reaching information treating each actual parameter at a call site to be a use followed by a definition that kills. Each entry statement defines all parameters and each __RETURN__ statement uses all parameters.

[^1]: A precise interprocedural data flow algorithm.

#### What does Soot actually do?

The following is the introduction of the infoflow analysis in Soot.
> InfoFlowAnalysis written by Richard L. Halpert, 2007-02-24. <br>
> Constructs data flow tables for each method of every application class. Ignores indirect flow.<br>
> These tables conservatively approximate how data flows from parameters, fields, and globals to parameters, fields, globals, and the return value. <br>
> Note that a ref-type parameter (or field or global) might allow access to a large data structure, but that entire structure will be represented only by the parameter's one node in the data flow graph. <br>
> Provides a high level interface to access the data flow information.


__InfoflowAnalysis:__


- __UseFinder:__ Find a list of all uses of fields of each application class within the application classes by looking at every application method. Find a list of all calls to methods of each application class within the application classes by using the call graph.
- 

__(In progress ...)__