# Hintergrundinfos

## SMT

### Was ist ein SMT-Solver?
Ein SMT-Solver (Satisfiability Modulo Theories) ist ein Werkzeug, 
das automatisch entscheidet, ob eine logische Formel erfüllbar ist – 
also ob es eine Belegung der Variablen gibt, die die Formel wahr macht. 
Im Vergleich zu klassischen SAT-Solvern, die nur Boolesche Logik verstehen, 
können SMT-Solver zusätzlich mit reichhaltigeren Datentypen umgehen: 
ganze Zahlen, reelle Zahlen, Arrays und einige mehr.
Onlinetutorial zu SMT-Solvern
https://microsoft.github.io/z3guide/docs/logic/intro

### Was ist SMT-LIB?
Damit verschiedene SMT-Solver miteinander verglichen und ausgetauscht werden können, 
wurde SMT-LIB als gemeinsamer Standard etabliert – ähnlich wie SQL für Datenbanken. 
SMT-LIB definiert eine einheitliche Eingabesprache, in der Formeln solver-unabhängig 
formuliert werden können, sowie eine wachsende Bibliothek realer Benchmark-Instanzen aus 
Verifikation und Programmanalyse.
Website: https://smt-lib.org/

### Was sind SMT-Theorien?
Die „Theories" in SMT bezeichnen formale Erweiterungen der Aussagenlogik um konkrete Datentypen und deren Operationen. 
Eine Theorie legt fest, welche Symbole (z.B. +, <, =) zur Verfügung stehen und welche Axiome für sie gelten. 
Gängige Theorien sind etwa lineare Arithmetik über ganzen Zahlen, 
die Theorie der Arrays oder die Theorie der Bitvektoren.

### Was ist die Bitvektortheorie?
Die Bitvektortheorie (QF_BV in SMT-LIB) modelliert Variablen fester Wortbreite – also genau so, 
wie Integer-Typen auf realer Hardware funktionieren, inklusive Überlauf. 
Sie stellt bitweise Operationen (and, or, xor, not), Shifts sowie Arithmetik mit Zweierkomplementsemantik zur Verfügung. 
Für die Verifikation von C-Programmen ist sie besonders relevant, 
da sie das Verhalten von int, unsigned, char und ähnlichen Typen präzise abbildet.

### Gutes Onlinetutorial eines SMT-Solvers
https://microsoft.github.io/z3guide/docs/logic/intro

## Verifikation / Ultimate

Das Ultimate Program Analysis framework besteht aus einigen Tools. 
https://ultimate-pa.org/
Eines davon ist der Software Verfier Ultimate Automizer.
https://ultimate-pa.org/automizer/

Ultimate Automizer übersetzt ein C Programm zunächst in ein Boogie Programm. 
Das Boogie Programm wird anschließend in einen Kontrollflussgraphen (CFG) übersetzt dessen Kanten mit SMT-Formeln 
beschriftet sind so wird z.B. x := x + 1 zur formel x' = x + 1, bzw (= x' (+ x 1)) in SMT-LIB Syntax.
Während des komplexen Verifikationprozesses werden viele SMT-Formeln erzeugt und manche davon an 
SMT-Solver gesendet um diese auf Erfüllbarkeit zu überprüfen.
Typisch dabei ist, dass komplexe Algorithmen aus existierenden Formeln mit Hilfe von Konjunktion (and) 
Disjunktion (or) oder Substitution (eine Art subterme durch andere Terme zu ersetzen) neue große Terme erzeugen.
Diese Terme sind oft unnötig groß und es existieren kleine Terme die dazu logisch äquivalent sind.
Wichtige Algorithmus im Ultimate Framework sind daher auch Algorithmen die Formeln simplifizieren.
Wir haben verschiedene Simplifizierungalgorithmen. 
Manche sind sehr schnell, manche sind sehr teuer und verwenden selbst viele SMT-Solver-Aufrufe zu simplifizierung einer einzigen Formel.
Die Simplifizierung ist so wichtig, dass die schnellen Simplifizierungen nach jedem Bauen einer Formel aufgerufen werden.

# Idee Simplifizierung von Bitvektortermen

