# StudentProjects

* ~~T01: Bitvektor nach Integer Transformation (Variante Nutz Transformation)~~
* T02: Invariantensynthese
    * Flexiblere Templates
    * Quantorenelimination
    * (Aspekt Interprozedural, CHC)
* T03: Quantorenelimination für modulo
* T04: Verfahren unseres Unihorn CHC Solvers beschreiben
* T05: Poor-man's tree interpolation (interpolation wrapper)
* ~~T06: TreeAutomizer~~
    * Baumautomatenoperation neu machen
    * Refinement Engine für Tree Automizer
    * Beweisgenerierung
* T07: Faultlocalization
    * Für Programme (schon publiziert)
    * Granularität Squence Points
* T08: Quantorenelimination nichtlineare real Arithmetik
* ~~T09: Constant Propagation Program Transformation~~
* ~~T10: Ultimate Referee um Support für Prozeduren erweitern (Prozedurkontrakte)~~
* T11: Counterexample Validation auf Boogie Ebene
* T12: PolyPac Simplification mit Alex' Congruence Closure Array-Support geben
* ~~T13: Optimierungen für SimplifyDDA~~
* T14: Re-implementation of the control flow graph construction
* T15: Egraphs for simplification an quantifier elimination


### Topic T02: Constraint-based Invariant Synthesis ★★★★★
* Motivation: See Section 19 of the [lecture on program verification](https://swt.informatik.uni-freiburg.de/teaching/SS2022/program-verification). Verification would be so easy if we could just synthesize invariants.
* In fact not only useful for safety invariants but also for ranking functions and danger invariants.
* We already have an implementation but we want our new ideas require a re-implementation.
* New idea: (Step 1) state constraints (Step 2) apply quantifier elimination (Step 3) apply Farkas' Lemma.
* Add support for various templates and an algorithm that iteratively tries different templates.
* Extension: Implement a heuristic that select the template.
* Extension: support for interprocedural safety invariant (i.e., procedure contracts)
* Extension: Heuristic for relaxing constraints in order to boost quantifier elimination

### Topic T03: Quantifier elimination for div and mod
* Motivation: See https://github.com/ultimate-pa/ultimate/wiki/Finished-Project-Topics#quantifier-elimination-for-div-and-mod
* Related to linear arithmetic (on modules instead of vector spaced). especially to Smith normal form. https://en.wikipedia.org/wiki/Smith_normal_form

### Topic T07: Faultlocalization
Motivation: We do not only detect if there is a bug, we also want to pinpoint statement that could be responsible for the bug.
[Poster](http://www2.informatik.uni-freiburg.de/~heizmann/2019TACAS-Christakis,Heizmann,Mansur,Schilling,Wustholz-SemanticFaultLocalizationAndSuspiciousnessRanking.pdf)
Implement the following [paper](https://link.springer.com/chapter/10.1007/978-3-030-17462-0_13) in Ultimate

### Topic T11: Counterexample validation for Boogie.
Motivation: Ultimate Automizer reports counterexample. Internally, the counterexamples undergo some backtranslation(s). The counterexamples look correct at a first glance, but we do not really know if some statements were dropped or added.
* Implement an interpreter for Boogie. The interpreter does not have to be interactive. It just has to process a sequence of states.
* Difficulties: Nondeterministic values. Maybe we need a symbolic representation of values or make benevolent guesses for missing values.
* Extension: Interactive interpreter for Boogie

### Topic T12: Congruence Closure for PolyPacSimplification
The PolyPacSimplification is an inexpensive simplification of logical formulas. (For simplification of logical formulas see T13.) Currently, the PolyPacSimplification utilizes polynomial relations (e.g. formulas of the form `x+y>=0` or `v=w`) for the simplification. In this project, we want to utilize also information about equalities or not equals relations for the simplification. E.g., the context for the simplification is (also) a data structure that learns from `f(x)≠f(y)` that `x` and `y` are different.

### Topic T14: Construction of a control flow graph for Boogie programs
* Motivation: Currently, our construction of a control-flow graph for Boogie programs consists of two steps: First we transform structured Boogie into unstructured Boogie (lots of gotos istead of if-then-else and while), secondly we construct a control-flow graph for the unstructured Boogie code. Working with the resulting control-flow graph is sometimes difficult because we cannot see the connection of the orignal Boogie code and the control flow graph easily.
* In this project we implement a simple construction of the control-flow graph. This construction was presented in Section 11 of the [lecture on program verification](https://swt.informatik.uni-freiburg.de/teaching/SS2022/program-verification).
* Extensions: Implement Large Block Encoding, Formal pesentation of our (interprocedural) control-flow graph.

### Topic T15: Egraphs for simplification and quantifier elimination
* Motivation: In practise (our program verification) we sometimes have formulas where quantified subformulas can be removed by our formula simplification (see T13 for simplification). Our new simplification avoids the costly simplification of quantified subformulas and hence cannot remove these subformulas. We implemented a new quantifier elimination technique (DualJunctionSgi) which addresses these formulas but that works only in a few cases.
* A [recent publication](https://link.springer.com/chapter/10.1007/978-3-031-37703-7_4) with promising ideas might improve our formula simplification.
* Can we compute an egraph for each node (resp the critical contraint of each node), replace subterms by their representative in order to get simpler formulas and in oder to successfully apply DualJunctionSgi more often?




## Finished StudentProjects (not available any more but there we might find related follow-up topics)

### Topic T06: Ultimate TreeAutomizer
* This is a great topic if you love tree automata
* Improve our [Ultimate TreeAutomizer](https://arxiv.org/abs/1907.03998v1) by re-implementing/improving the algorithms of some components.

### Topic T09: Constant Propagation Program Transformation ★★★★★
* Motivation: It is very difficult to determine the value of y after the following loop `y:=0 for (int i=0; i<1000; i++) { y = y + x*x; }`. If we however know that the statement `x=2` occurs before the loop, we can replace x by 2 and the loop is simpler.
* We obtain the information for the constant propagation from our abstract interpretation framework. We do the simplification via our context-aware simplification.
* Part 2: If we Automizer computes a Floyd-Hoare annotation for the transformed program, this is not a Floyd-Hoare annotation for the original program. If we add all information form the constant propagation we have a Floyd-Hoare annotation but it contains unnecessary many information. Ideally, we add only the information that is needed. We will solve this by iteratively using weakest precondition. Maybe this is just one application of a general simplification for Floyd-Hoare annotations.
* Extension: Interprocedural programs.

### Topic T10: Add support for Procedures to Ultimate Referee ★★★★★
* Motivation1:  Ultimate Referee works only for program that have one procedure. We want to support several procedures.
* Motivation2:  Ultimate Referee works only for Boogie programs. We want to support also C programs and use Ultimate Referee as a validator at the prestigious [Competition on Software Verification SV-COMP](https://sv-comp.sosy-lab.org/). 
* Ultimate Referee was introduced in Section 8 of the [lecture on program verification](https://swt.informatik.uni-freiburg.de/teaching/SS2022/program-verification).
* Ultimate Referee is available in a web interface. https://ultimate.informatik.uni-freiburg.de/?ui=tool&tool=referee
* Our model of interprocedural program is closely related to Boogie: Input variable cannot be written. The `old` keyword refers to the value of global variables at the beginning of the procedure. A typical annotation could be `x_out = 2*a_in && g == old(g)+1` https://www.microsoft.com/en-us/research/wp-content/uploads/2016/12/krml178.pdf
* Idea1: Collect traces, do trace checks. Idea2: Collapse CFG by large block encoding, do Hoare triple checks.
* Extension: Define semantics of our CFG formally.
* Extension: Tell the user where his invariants are insufficient.

### Topic T13: Simplification of logical formulas ★★★★★
Motivation: Consider the formula `y>=x*x ∧ y>=1 ∧ x=2`. If we analyze the first conjunct, we can conclude from the third conjunct that x*x can be replaced by 4 afterwards, we can conclude form the second conjunct the the first conjunct can be replaced by `true`.
* Basic idea is published in a [paper](https://theory.stanford.edu/~aiken/publications/papers/sas10.pdf)
* Already implemented in Ultimate, but we want to improve the implementation: (1) Heuristic for order of subformulas (2) omit expensive subformulas (3) re-use context computation (4) improve information in context (5) do constant folding (6) strategy for repetitions.
