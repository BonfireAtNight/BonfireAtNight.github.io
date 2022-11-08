# Beweise und Beweisstrategien
Wie in allen Wissenschaften, so besteht auch in der Mathematik ein Hauptziel darin, *Fragen* zu beantworten.
(Beispiele)

Antworten auf Fragen dieser Art werden durch *Theoreme* gegeben.
Ein Theorem ist eine mathematische Aussage, die Prämissen (Annahmen, Hypothesen) und eine daraus logisch folgende Konklusion benennt. Beispiel: "*Theorem*. Suppose x > 3 and y < 2. Then x2 − 2y > 5."
Um zu zeigen, dass diese Beziehung tatsächlich besteht, muss ein *Beweis* geführt werden.

Bei den Annahmen handelt es sich nicht selten um mathematische Sätze die erst einmal als wahr betrachtet werden.
Natürlich sind Aussagen nur dann
Die Ableitungen werden dabei im Rahmen eines deduktiven Systems vorgenommen (s. XXX), wobei dieses nicht selten implizit bleibt.

## Was ist ein Theorem?
Theoreme sind mathematische Aussagen, die eine logische Konsequenz zum Ausdruck bringen: "if certain assumptions called the *hypotheses* of the theorem are true, then some conclusion must also be true." (Velleman 2006, 85)
Sie werden häufig als Antworten auf mathematische Fragestellungen vorgebracht.
So gibt etwa der *Satz des Euklid* eine positive Antwort auf die Frage, ob es unendlich viele Primzahlen gibt.

Es ist nicht unüblich, den abgeleiteten Satz selbst als Theorem zu bezeichnen.
Dabei ist mutmaßlich der epistemologische Status der Hypothesen entscheidend.
Wenn es sich bei den Hypothesen des Theorems ausschließlich um Axiome des Axiomensystems handelt, im Rahmen dessen das Theorem vorgebracht wird, erscheint die Rede von quasi bedingungslosen Theoremen vertretbar.
Wenn sich unter den Hypothesen eines Theorems Theoreme finden, deren Axiome ausschließlich Axiome sind, dann wird schrittweise ein System gesicherter Sätze aufgebaut, zu denen das abgeleitete Theorem hinzugefügt wird.
Die axiomatisierte Euklidische Geometrie oder die Peano-Arithmetik sind vertraute Beispiele dieser Praxis.
Wenn die Hypothesen aber weniger gesichert sind – etwa weil man sie als Teil einer Aufgabenstellung erhalten hat – dann sollte die Rede von Theoremen immer auch die Hypothesen umfassen.

Zumeist enthalten Theoreme freie Variablen (Velleman 2006, 85).
Wenn diese Variablen mit Werten belegt werden, spricht man von einer *Instanziierung* bzw. einer *Instanz des Theorems*.
Ein korrekter Beweis garantiert, dass bei *allen* Instanzen, bei denen die Hypothesen wahr werden – bei allen *Modellen* der Hypothesen – auch die Konklusion wahr ist.
Für Instanzen, bei denen nicht alle Hypothesen wahr werden, ist das Theorem *nicht anwendbar*, das heißt es sagt nichts über den von den instanziierten Hypothesen beschriebenen Fall.

Wenn ein Fall gefunden werden kann, bei denen die Hypothesen wahr werden, die Konklusion aber nachweislich falsch ist, so ist damit ein *Gegenbeispiel* (counterexample) gefunden.
Damit wäre gezeigt, dass das Theorem *inkorrekt* ist.
Im eigentlichen Sinne handelte es sich gar nicht um ein Theorem.

Nur in den wenigsten Fällen ist der Wertebereich der Variablen so begrenzt, dass die Korrektheit des Theorems durch eine erschöpfende Betrachtung seiner Instanziierungen demonstriert werden könnte.[^1]
Es müssten systematisch alle möglichen Kombinationen von Variablenbelegungen dahingehend geprüft werden, ob die Konklusion in allen Fällen wahr ist, in denen es die Hypothesen sind.
Anders als im negativen Fall kann in der Regel nur durch einen *Beweis* demonstriert werden, dass ein Theorem gilt.

## Was ist ein Beweis?
Ein Beweis eines Arguments ist ein *deduktives Argument*, dessen Prämissen die Hypothesen des Theorems und dessen Konklusion die Konklusion des Theorems sind (Velleman 2006, 86).
Beweise sind also Argumente, die bezüglich des Gegenstandsbereiches der Mathematik definiert sind:
Ein Beweis wird geführt, wenn auf weitgehend *explizite*[^3] und *logisch gültige* Weise über die Eigenschaften *mathematischer* Objekte gesprochen wird, dann werden Beweise geführt.[^2]

Die zu beweisenden Theoreme bringen eine Notwendigkeit zum Ausdruck: Wenn die Hypothesen des Theorems wahr sind, dann *muss* die Konklusion wahr sein.
Dieser Zusammenhang wird in der Logik durch den normativen Begriff[^4] der *Gültigkeit* (logical validity) gefasst. 
Argumente sind gültig, wenn sie gültige Argument*formen* instanziieren (Tomassi, XXX).[^5]
Entscheidend dabei ist die *logische Form* der Aussagen, mit denen das Argument konstruiert wird.
Deshalb ist die logische Form – vor allem der Konklusion des Theorems, aber auch der Hypothesen – auch das Hauptaugenmerk, wenn die Wahl einer erfolgversprechenden Beweisstrategie zu treffen ist.

## Logische Form
Wie eben gesagt ist die *logische Form* der gegebenen und angepeilten Aussagen entscheidend in der Frage, welche Beweisstrategie man verfolgen sollte.
Um ein Theorem zu beweisen, besteht der erste Schritt deshalb darin, die logische Form der involvierten Aussagen zu identifizieren.
Die logische Form liegt nicht offen zutage[^6], deshalb kann es gerade Anfängern helfen, sie zunächst mithilfe von Symbolen zu explizieren.[^7]

An dieser Stelle muss man auf ein nicht unerhebliches technisches Detail hinweisen.
Es wurden sehr viele verschiedene logische Systeme entwickelt, mit deren Hilfe die logische Form von natürlichsprachlichen und mathematischen Aussagen dargestellt werden kann.
Wie der Übergang von der Aussagenlogik zur Prädikatenlogik veranschaulicht, werden neue Systeme häufig mit dem Ziel entwickelt, logische und semantische Sachverhalte auf differenziertere Weise formal darzustellen.
Deshalb wäre es vermutlich irreführend, von *der* logischen Form einer Aussage zu sprechen (Tomassi, XXX).
Welche Darstellungsweise angemessen ist, hängt ab von der Zielsetzung des verfolgten Projekts.
Für die mit Beweisen verfolgten Zwecke sind Aussagen- und Quantorenlogiken bereits hinreichend.[^8]

Noch in einem anderen Sinne kann nicht von *der* logischen Form einer Aussage gesprochen werden.
Selbst im Rahmen *eines* logischen Systems kann ein und dieselbe Aussage durch verschiedene logische Formeln dargestellt werden.
Diesen Umstand sollten wir uns zunutze machen: Häufig empfiehlt es sich, Aussagen anhand äquivalenter logischer Formen umzudeuten
Der Vorteil liegt darin, dass dadurch Beweisstrategien anwendbar werden, die auf den ersten Blick nicht anwendbar erschienen.

## Beweisstrategien
Es ist in der Mathematik und Informatik – und sicher auch in anderen Bereichen – üblich, dass Probleme *transformiert* werden, um sie besser (oder überhaupt) lösen zu können (Velleman 2006, 86-87).
Da es dabei vielfach nicht mehr um die Hypothesen und Konklusion des Theorems geht, wie sie üblicherweise (oder zunächst) vorgebracht werden, sollten diese Wörter bei unseren Überlegungen zu einer möglichen Lösung vielleicht zunächst besser vermieden werden.
Velleman (2006, 88) schlägt vor, vom *Gegebenen* (*the givens*) und dem *Ziel* (*goals*) zu sprechen, den "statements that are known or assumed to be true at some point in the course of figuring out a proof" und dem "statement that remains to be proven at that point".
Sie ergeben sich also nach der Transformation - oder den schrittweise durchgeführten Transformtionen - des ursprünglichen Problems und es ist ihre logische Form, die entscheidend ist.

Zunächst handelt es sich bei den gegebenen Aussagen um die Hypothesen des Theorems, allerdings kommen durch Schlussfolgerungen und Transformationen des Problems (insbesondere durch *Annahmen*) weitere Aussagen hinzu.
Die Zielaussage drückt zunächst die Konklusion des Theorems aus, kann sich jedoch im Zuge unserer Vorabüberlegungen mehrfach ändern.

### Beweis einer Zielaussage der Form: P -> Q

#### Implikationseinführung nach Annahme der Antezedenz

## Anmerkungen
[^1]: Eine bemerkenswerte Ausnahme sind die Theoreme der Aussagenlogik, die häufig mittels Wahrheitstafeln begründet werden können.
[^2]: Wie bei Argumenten und Argumentationen allgemein, so kann man auch bezüglich Beweisen eine Unterscheidung treffen zwischen dem *Akt* des Beweisens und dem, was dabei zum Ausdruck gebracht wird. Anders ausgedrückt, Beweisen als illokutionäre Rolle (*illocutionary force*) einer Äußerung ist abzugrenzen vom Beweis als semantischer Gehalt der Äußerung. Vgl. Brandom ().
[^3]: Es ist unter Mathematikern nicht unüblich, dass als offensichtlich erachtete Beweisschritte in ihren Ausführungen unterschlagen werden (Velleman 2006, 110-111). Der Hauptanspruch an Beweise liegt also darin, dass sie gültigen Schemata folgen; explizit sollen sie nur insoweit sein, dass die intendierte Zielgruppe ihnen folgen kann. Dazu gehören nicht unbedingt Anfänger in einem Mathematik- oder Informatikstudium.
[^4]: Normative Begriffe betreffen ein *Sollen* bzw. eine Form der *Evaluation*. Der Begriff der Gültigkeit benennt eine notwendige Bedingung für *gute* Argumente.
[^5]: Mittels einer Argumentform und seiner Gültigkeit oder Ungültigkeit wird etwas *über* ein logisches System gesagt. Technisch korrekt ist es deshalb, sie mithilfe von *Metavariablen* zu beschreiben und darauf hinzuweisen, dass die Operatoren genau genommen zu unterscheiden sind von denjenigen der Objektsprache selbst. Die Formeln der Objektsprache *instanziieren* die Formeln der Metasprache. So wird die Formel $\phi \lor \psi$ beispielsweise durch die aussagenlogische Formel $P_1 \lor P_2$ instanziiert. Diese technischen Details werden in diesem Artikel keine Rolle spielen.
[^6]: Tatsächlich kann die Oberflächenstruktur einer Aussage irreführend sein. (...)
[^7]: Wenn der Beweis letztlich in sauberer Form geschrieben wird, werden logische Symbole von vielen als störend empfunden (Velleman 2006, 92). Beweise werden in ihrer finalen Form deshalb meist weitgehend in natürlicher Sprache und ohne mathematische Notation formuliert.
[^8]: Für unsere Zwecke reicht es völlig aus, Aussagenvariablen, Junktoren und Quantoren in ihrer gewöhnlichen Bedeutung zu verwenden, statt die zugrundegelegte Logik dadurch zu formalisieren, dass explizit die Syntax und Semantik einer Sprache erster Ordnung definiert würde. In einem formalisierten deduktiven System werden neben der Sprache auch die Schlussregeln (rules of inference) definiert, doch auch davon sehen wir hier ab. Stattdessen wenden wir intuitiv Regeln eines Kalküls des natürlichen Schließens an.
[^9]: Im Englischen wird zwischen *assertion* und *assumption* unterschieden: "To assert a statement is to claim that it is true, and such a claim is never acceptable in a proof unless it can be justified. However, the purpose of making an assumption in a proof is not to make a claim about what *is* true, but rather to enable you to find out what *would be* true *if* the assumption were correct." (Velleman 2006, 87) Der letztgenannte Begriff wird im Deutschen zumeist durch *Annahme* ausgedrückt (Quelle???). Wenn Aussagen als wahr vorgetragen werden, kann dem Sprechakt in einigen Fällen die illokutionäre Rolle einer *Behauptung* zugeschrieben werden (Frege XXX). Der semantische Gehalt eines solchen Sprechakts scheint mir damit aber nicht treffend beschrieben. Aus Mangel an einer besseren Idee verwende ich deshalb *Assertion* als wäre es ein deutsches Wort.
