---
---
# Pattern Matching mit regulären Ausdrücken
Viele Anwendungen (darunter Webformulare) machen es erforderlich, Benutzereingaben mit bestimmten Formatvorgaben abzugleichen.
Die folgenden zwei Beispiele illustrieren diese Form der Eingabevalidierung.

Wenn sich etwa jemand bei einem Online-Dienst registriert, muss ein Benutzername und Passwort gewählt werden.
Dies Anmeldeinformationen (*login credentials*) erlauben es dem Benutzer später, sich beim Dienst einzuloggen.
Aus Gründen der Sicherheit müssen bestimmte Anforderungen an das Passwort gestellt werden.
Typischerweise darf es eine bestimmte Mindestanzahl an Zeichen nicht unterschreiten und sollte sich aus Buchstaben, Ziffern und Sonderzeichen zusammensetzen.
Es ist ratsam, dass sich Zeichen nicht wiederholen.
Die Passworteingabe des Benutzers ist eine Zeichenkette, die auf die Einhaltung dieser Vorgaben hin überprüft werden kann.

Literaturverwaltungsprogramme wie EndNote oder Zotero erlauben es, Einträge über ihre ISBN (*International Standard Book Number*) zur lokalen Bibliothek hinzuzufügen.
Dabei handelt es sich um eine Folge von Ziffern – der derzeitige Standard, ISBN-13, schreibt 13 Ziffern vor – zur eindeutigen Kennzeichnung von Büchern.
Die Nummer setzt sich zusammen aus dem dreistelligen Präfix (978 oder 979), einer einstelligen Gruppen- bzw. Ländernummer, einer fünfstelligen Verlagsnummer, einer dreistelligen Titel- oder Bandnummer und einer finalen Prüfziffer.
Zusätzlich kann gefordert werden, dass die Blöcke durch einen Bindestrich verbunden werden.
Entspricht eine Eingabe nicht dieser Vorlage, wo wird eine Fehlermeldung ausgegeben.

Muster eignen sich nicht nur dazu, Formatvorlagen zu definieren.
Sie können genutzt werden, um komplexe Suchanfragen zu stellen.
So kann `grep` des Betriebssystems GNU dazu genutzt werden, um in Textdokumenten all die Zeilen zu finden, in denen Textabschnitte dem erfragten Muster entsprechen.
GNU `sed` und ähnliche Programme ermöglichen es, die gefundenen Textabschnitte durch einen bestimmten String zu ersetzen.
Muster sind auch [erforderlich](https://stackoverflow.com/a/3487295/18286715), um Parser zur Analyse von Quelltexten zu schreiben.

Diese Beispiele aus der Webentwicklung, Systemadministration, und speziellen und allgemeineren Softwareentwicklung demonstrieren, dass Entwickler in der Lage sein sollten, mit Mustern dieser Art zu arbeiten.

## Reguläre Ausdrücke zur Definition von Mustern
Bei den obigen Beispielen habe ich allgemein von *Mustern* gesprochen.
Es versteht sich, dass Muster prinzipiell auf verschiedene Weise definiert werden könnten.
Glücklicherweise verwenden die allermeisten Umgebungen zur Definition von Mustern sogenannte *reguläre Ausdrücke* (*regular expressions*, *regex*).

Es versteht sich, dass Muster gewöhnliche Buchstaben, Ziffern und Schriftzeichen wie "!" (Satzzeichen) oder "@" (Sonderzeichen) enthalten können müssen.
Wenn man in einem längeren Textdokument beispielsweise nach "Rezept" sucht, so möchte man die Textstellen erhalten, an denen genau dieses Wort steht.
Die Zeichen in regulären Ausdrücken, die für sich selbst stehen, werden als *Literale* (*literals*) oder *Literalzeichen* (*literal characters*) bezeichnet.
Reguläre Ausdrücke zeichnen sich dadurch aus, dass sie darüber hinaus Zeichen mit besonderer Bedeutung enthalten.
Dadurch kann man beispielsweise festlegen, dass an einer bestimmten Stelle ein Großbuchstabe stehen muss (wie zu Beginn einer IBAN), dass beliebig viele Wörter folgen können oder dass ein Zeichen optional ist.

Reguläre Ausdrücke wurden in den 1950er Jahren in die theoretische Informatik eingeführt, um Mengen von Zeichenketten (*strings*)[^1] zu beschreiben.[^2]
Auf Grundlage expliziter Vereinbarungen (oder Konventionen) steht etwa der reguläre Ausdruck `a|b` für die (endliche) Menge $\{a, b\}$ und der Ausdruck `a+` steht für die (unendliche) Menge bestehend aus `a`, `aa`, `aaa`,...
In der praktischen Anwendung von regulären Ausdrücken als Muster sagt man, dass jeder String der Menge mit dem regulären Ausdruck bzw. dem Muster *matcht*.[^3]
Nutzt man den regulären Ausdruck `a+` als Suchanfrage in Bezug auf den Satz `Peter stehen die Haare zu Berge`, so matcht das `aa` aus `Haare` mit dem Suchmuster.
Das `+` im eben angeführten Beispiel ist etwa als "ein oder mehr des vorausgegangenen Zeichens" (`a` im Beispiel) zu verstehen.

Es gibt weitgehend übereinstimmende Festlegungen und Regeln, wie Zeichen kombiniert werden dürfen (Syntax) und welche Mengen von Zeichenketten dadurch denotiert werden (Semantik).
Der auf diese Weise vereinbarte Standard bildet eine *Sprache regulärer Ausdrücke* (*regular expression language*).
Dieser Artikel gibt einen Überblick über die wichtigsten Elemente dieser Notation.

## Abweichende Regex-Standards
Genau genommen wäre es falsch von *der* Sprache regulärer Ausdrücke zu sprechen.
Verschiedene Umgebungen legen Standards zugrunde, die in Details unterscheiden.
[GNU](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap09.html) unterscheidet zwischen *Basic Regular Expressions* (BRE) und *Extended Regular Expressions* (ERE), die Teil des [POSIX](https://en.wikipedia.org/wiki/POSIX)-Standards sind.
Die Programmiersprache Perl verwendet eine [abweichende Syntax](https://perldoc.perl.org/perlre); wenn ein regulärer Ausdruck korrekt von Perls Regex-Parser interpretiert werden kann, dann ist er *Perl-kompatibel* (PCRE).
Im Jahr 2002 veröffentlichte Java ein Paket für reguläre Ausdrücke ([java.util.regex](https://docs.oracle.com/javase/7/docs/api/java/util/regex/package-summary.html)).
Seit C++ 2011 definiert die Sprache eine [Bibliothek für reguläre Ausdrücke](https://en.cppreference.com/w/cpp/regex)
Microsoft definiert [*.NET Regular Expressions*](https://learn.microsoft.com/en-us/dotnet/standard/base-types/regular-expression-language-quick-reference)

Wie in der Programmierung allgemein sollte das Ziel also darin bestehen, Ideen und Grundbegriffe zu verstehen; ihre individuelle Implementierung ist zweitrangig.
Nicht zuletzt deshalb, weil reguläre Ausdrücke schnell so kompliziert werden, dass man Details wohl eh regelmäßig nachschlagen muss.

### Zeichen-Klassen (*character classes*)
Mit eckigen Klammern
Listen

### Anchor-Zeichen
Die Metazeichen `^` und `$` matchen mit *Positionen* (in einer Zeile oder einem String), nicht mit tatsächlichen Textzeichen.
Sie werden deshalb als *Anker* (anchors) bezeichnet.

## Tipps
1. Wenn man reguläre Ausdrücke in der Shell verwendet (etwa als erstes Argument für `grep`) sollten sie in einfache Anführungszeichen gesetzt werden. Dadurch wird die Shell angewiesen, das Innere weitestgehend zu ignorieren. Doppelte Anfürungszeichen hätten zur Folge, dass einige Zeichen als *Metazeichen der Shell* interpretiert würden.
2. Reguläre Ausdrücke sollten linear konstruiert und verarbeitet werden (Friedl 2006, 8).

## Notizen
Will man mit regulären Ausdrücken arbeiten, muss man also nicht nur die vielen Metazeichen 

Aufgrund der vielen Metazeichen und ihrer vielseitigen Kombinierbarkeit von schnell überwältigender Komplexität definiert werden.


Muster (wie ein Muster-Parser) verstehen und schreiben zu können, reduziert sich also auf das Problem, reguläre Ausdrücke verstehen und schreiben zu können.

Vorgang des Pattern Matching

Unentbehrliches Hilfsmittel bei der maschinellen Textverarbeitung

Verfügbar in sehr vielen Tools, darunter Kommandozeilen-Anwendungen, Texteditoren, Datenbanken und vor allem Programmiersprachen

[^1]: Von einem rein syntaktischem Standpunkt werden Mengen von Zeichenketten nicht selten als (formale) *Sprachen* bezeichnet. *Reguläre Sprachen*, das heißt die Klasse der Sprachen, die durch reguläre Ausdrücke beschrieben werden können, bilden eine Untermenge der Klasse der formalen Sprachen allgemein. Sie bilden einen Forschungsgegenstand der theoretischen Informatik. So wurde beispielsweise festgestellt, dass eine Sprache genau dann mittels eines regulären Ausdrucks beschrieben werden kann, wenn es einen endlichen Automat gibt, der sie akzeptiert. Theoretische Aussagen dieser Art spielen für den Anwendungsbereich von regulären Ausdrücken als Muster eine nur untergeordnete Rolle.
[^2]: Tatsächlich wurden die grundlegenden Ideen vom Mathematiker Stephen C. Kleene zunächst bezüglich neuronaler Netze entwickelt. Die von ihm im maßgeblichen Aufsatz von 1951 analysierten *Ereignisse* sind in dem Sinne regulär, dass sich ihre Strukturen mithilfe bestimmter Formeln beschreiben lassen. Es zeigte sich, dass sich genau diese Formeln – später reguläre Ausdrücke genannt – auch zur Analyse von Zeichenketten verwenden lassen. Dieser historische Abriss demonstriert erneut die überaus vielfältige Anwendbarkeit regulärer Ausdrücke.
[^3]: Es ist nicht ungewöhnlich, das englische *to match* als deutsches Verb zu verwenden (so etwa bei [Wikipedia](https://de.wikipedia.org/wiki/Pattern_Matching)). Wenn Entwickler wie Anwender davon sprechen, dass "ein Match gefunden" wurde, fühlen sich einige vielleicht an Tinder erinnert.


## Zitate
"(Many tools) provide a rich set of extensions to the notation of regular expressions. Some extensions, such as shorthand for “zero or one occurrences” or “anything other than white space,” do not change the power of the notation. Others, such as the ability to require a second occurrence, later in the input string, of the same character sequence that matched an earlier part of the expression, increase the power of the notation, so that it is no longer restricted to generating regular sets. Still other extensions are designed not to increase the expressiveness of the notation but rather to tie it to other language facilities. In many tools, for example, one can bracket portions of a regular expression in such a way that when a string is matched against it the contents of the corresponding substrings are assigned into named local variables." (Scott 2016, 48)
