# Hintergrundinfos

## SMT

### Was ist ein SMT-Solver?
Ein SMT-Solver (Satisfiability Modulo Theories) ist ein Werkzeug, 
das automatisch entscheidet, ob eine logische Formel erfüllbar ist – 
also ob es eine Belegung der Variablen gibt, die die Formel wahr macht. 
Im Vergleich zu klassischen SAT-Solvern, die nur Boolesche Logik verstehen, 
können SMT-Solver zusätzlich mit reichhaltigeren Datentypen umgehen: 
ganze Zahlen, reelle Zahlen, Arrays und einige mehr.

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

### Das Framework

Das Ultimate Program Analysis framework besteht aus einigen Tools. 
https://ultimate-pa.org/

Eines davon ist der Software-Verifizierer Ultimate Automizer, der automatisch beweist, ob ein C-Programm eine gegebene Spezifikation erfüllt.
https://ultimate-pa.org/automizer/

### Vom C-Programm zur SMT-Formel 

Ultimate Automizer übersetzt ein C-Programm zunächst in ein Boogie-Programm, das anschließend in einen Kontrollflussgraphen (CFG) überführt wird. Die Kanten des CFGs sind mit SMT-Formeln beschriftet – so wird etwa die Zuweisung x := x + 1 zur Formel (= x' (+ x 1)) in SMT-LIB-Syntax.

### Zwei Übersetzungsvarianten

Die Übersetzung von C-Programmen in einen CFG existiert in zwei Varianten, die sich in der Wahl der SMT-Theorie unterscheiden.
Das Integer-Modell verwendet mathematische Integer. Da C-Integer-Typen jedoch eine feste Wortbreite haben, muss nach Rechenoperationen oft ein Modulo (z.B. 2³²) ergänzt werden, um die C-Semantik korrekt abzubilden. Trotz dieser zusätzlichen Operationen ist die Verifikation mit diesem Modell durchschnittlich schneller. Ein wesentlicher Nachteil ist jedoch, dass bitweise Operationen nicht exakt dargestellt werden können und überapproximiert werden müssen – was dazu führt, dass die Verifikation häufig unknown zurückgibt.
Das Bitvektormodell bildet die C-Semantik exakt ab, inklusive bitweiser Operationen und Überlaufverhalten. Die Verifikation ist dadurch präziser, aber deutlich teurer – Timeouts sind häufig.


### Formelsimplifizierung
Während des Verifikationsprozesses werden kontinuierlich neue SMT-Formeln erzeugt, indem bestehende Formeln durch Konjunktion, Disjunktion oder Substitution kombiniert werden. Dabei entstehen leicht Terme, die unnötig groß sind, obwohl logisch äquivalente, kleinere Terme existieren. Simplifizierungsalgorithmen spielen daher eine zentrale Rolle im Framework: Schnelle Algorithmen werden nach jeder Formelkonstruktion aufgerufen, während aufwändigere Verfahren – die selbst viele Solver-Aufrufe benötigen – gezielt eingesetzt werden.

# Idee 1: Simplifizierung von Bitvektortermen mit Rewrite-Regeln
Bei der Simplifizierung von Termen der Integer-Theorie ist Ultimate bereits gut aufgestellt. Für die Bitvektortheorie hingegen gibt es noch erhebliches Verbesserungspotenzial: Viele Rewrite-Regeln, die semantisch äquivalente aber strukturell kleinere Terme erzeugen, sind bisher nicht implementiert. Ziel dieses Projekts ist es, solche Vereinfachungsschritte zu identifizieren, korrekt zu implementieren und in Ultimate zu integrieren.

Ein paar KI-generierte Beispiele für solche Rewrite-Regeln (manche davon sind vielleicht schon implementiert).

* Absorption: (bvand x (bvor x y)) → x
* Idempotenz: (bvand x x) → x
* Neutrales Element: (bvor x #b000...0) → x
* Annihilator: (bvand x #b111...1) → x
* De Morgan: (bvnot (bvand x y)) → (bvor (bvnot x) (bvnot y))
* Doppelte Negation: (bvnot (bvnot x)) → x
* Shift um 0: (bvshl x #b000...0) → x
* Konstantenfaltung: (bvadd #b0101 #b0011) → #b1000

### Literatur
* Frank hat das folgende schon für die Übersetzung von C nach Boogie implementiert, da kann man wohl was übernehmen.
https://link.springer.com/chapter/10.1007/978-3-030-89051-3_16
Frank hat das für die Übersetzung relevante hier schön zusammengefasst.
https://link.springer.com/chapter/10.1007/978-3-031-57256-2_31

* Bitvektor Rewrite Regeln von CVC5
https://github.com/cvc5/cvc5/blob/main/src/theory/bv/rewrites

* Vielleicht mal reinschauen
[Syntax-Guided Rewrite Rule Enumeration for SMT Solvers" (SAT 2019)](http://theory.stanford.edu/~barrett/pubs/NRB+19.pdf)

* Abschnitt Rewrite Interessant
https://cs.stanford.edu/~niemetz/publications/2023/NiemetzP-CAV23.pdf

* Natürlich gerne weitere suchen

### Sinnvolle Regeln auswählen
Problem: Zu jeder Formel gibt es unendlich viele größere Formeln die dazu logisch äquivalent sind. Wir können also leicht beliebig viele Simplifizierungsregeln finden. Jede Regel man Ultimate minimal langsamer könnte aber ein großer Gewinn sein. Wir wollen irgendwie systematisch vielversprechendes implementieren.
Ideen:
* Einfache Regeln kosten wenig performance und sind vielleicht oft anwendbar.
* Regeln die anderen geholfen haben, helfen uns vielleicht auch.
* Einfach mal blind Regeln implementieren und auf Benchmarks evaluieren ob sie jemals/manchmal/oft angewendet werden. (Doof weil dann macht man ja unnötige arbeit...)
* Ein Schritt eines Verifikationsalgorithmus von Ultimate Automizer arbeitet mit ziemlich vielen (bereits simplifizierten) Formeln und prüft diese auf Äquivalenz. Wir könnten äquivalente Formelpaare mit erheblichem Größenunterschied auf die Festplatte schreiben und schauen ob dies Ideen für Simplifizierungsregeln liefern. Diese wären dann ja automatisch praxisrelevant. (Doof ist dass diese Formeln manchmal Seitenlang sind und von Menschen nicht mehr verstanden werden können.)
* ...

### Wo in Ultimate würde man das implementieren?
Startpunkt für die Rewrite-basierte Simplifizierung ist die `unfterm` Methode in `SmtUtils`. Viele Methoden in `SmtUtils` implementieren solche Rewrite-Regeln.
https://github.com/ultimate-pa/ultimate/blob/f5c6788a5bc52bcf59a9f46511f7a9039935daa6/trunk/source/Library-SmtLibUtils/src/de/uni_freiburg/informatik/ultimate/lib/smtlibutils/SmtUtils.java#L1470

### Wo in Ultimate implementiert man Tests?
Wichtig: Wir brauchen Tests für jede Regel.
Klassen mit Tests zur Simplifizierung: SmtUtilsTest , UltimateNormalFormTest, SimplificationTest, PolynomialRelationTest, PolynomialRelationTestModBasedSimplification.
Vielleicht müssen wir da mal Aufräumen. Aber vermutlich wollen wir analog zu einer der anderen Klassen eine eigene mit Bitvektorsimplifizierungstests und existierende Bitvektorsimplifizierungstests aus existierenden Klassen verschieben.


# Idee 2: Simplifizierung von Bitvektorungleichungen.

### Ausgangssituation
Für Gleichungen von Bitvektoren, Integern und Reals, sowie für Ungleichungen von Integern und Reals haben wir die sehr hilfreiche Klasse PolynomialRelation.
https://github.com/ultimate-pa/ultimate/blob/f5c6788a5bc52bcf59a9f46511f7a9039935daa6/trunk/source/Library-SmtLibUtils/src/de/uni_freiburg/informatik/ultimate/lib/smtlibutils/polynomials/PolynomialRelation.java#L81
Leider funktioniert diese Datenstuktur (verwende nur ein Polynom und mache dadurch implizit schon einige Simplifizierungsschritte) nicht für die acht(!) Ungleichungsoperatoren von Bitvektortermen (ult, slt, ule, sle, ugt, sgt, uge, sge).

### Ziel
Baue ähnliche Datenstruktur mit zwei Polynomen (linke Seite, rechte Seite).
Wir sollten aber vorher nochmal gut überlegen ob es sich lohnt. Vielleicht sind diese Ungleichungen so fies, dass man zu wenig simplifizieren kann und ein milderes Mittel zur Simplifizierung ausreicht.

# Idee 3: Rückübersetzung von Bitvektortermen (und Integertermen)
Im Erfolgsfall generiert Ultimate Automizer Invarianten und Funktionskontrakte die die Korrektheit Beweisen. Diese Beweise werden für den mit SMT-Termen beschrifteten CFG generiert. Diese Terme müssen in C-Expression zurückübersetzt werden damit der Benutzer diese verstehen kann. Unsere naive Rückübersetzung ist konzeptuell inkorrekt. Wir brauchen eine Übersetzung die Bitlängen berücksichtigt und bedenkt dass Operatoren in SMT-LIB und C eine leicht andere Semantik haben.
Mehr dazu via E-Mail.
