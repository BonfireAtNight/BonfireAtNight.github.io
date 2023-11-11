---
title: Die Nix-Programmiersprache
date: 2023-11-05 9:22:00 +0200
date_last_mod: 2023-11-05 9:22:00 +0200
---
# Nix: Die Sprache des Nix-Paketmanagers
Nix ist die Sprache, in der der Nix-Paketmanager geschrieben ist und auf die er aufbaut. Ihr Hauptzweck besteht darin, *Derivations* zu erstellen. Dieser Grundbegriff bezeichnet Objekte, mit denen Software-Komponenten gebaut (*build*) werden können. Auch Systemkonfigurationen werden in Nix geschrieben.

## Merkmale
Wie aus der Einleitung deutlich wird, ist Nix eine *Domain-spezifische* Sprache. Insbesondere an den eingebauten Funktionen zeigt sich, dass Nix einzig für die Verwendung mit dem Nix-Paketmanager entwickelt wurde. Anders als Sprachen wie C, Python oder Scheme wurde sie nicht dafür konzipiert, beliebige Software-Projekte damit verfolgen zu können (*general-purpose*).[^1]

Nix kann anhand weiterer Merkmale klassifiziert werden. Eine Reihe von Merkmalen charakterisieren die Sprache eindeutig als *funktional*. Das zeigt sich inbesondere daran, dass Funktionen in Nix wie (andere) Werte behandelt werden. Sie können in Variablen gespeichert, als Argumente an Funktionen übergeben, und von Funktionen zurückgegeben werden. In diesem Fall spricht man von *First-Class-Funktionen* bzw. von Funktionen als *First-Class-Citizens*.[^2] Funktionen eignen sich besonders gut dazu, Pakete, Build-Actions, Konfigurationen und Systemzustände zu definieren und parametrisieren.

Funktionen haben in Nix keine Nebeneffekte. In diesem Sinne sind sie *rein* (*pure*). Das heißt, dass Daten und Variablen durch Funktionsausführungen niemals modifiziert werden. Deshalb führen gleiche Inputs stets zu gleichen Outputs. Das ist wichtig für Zwecke der Paketverwaltung: bei gleichen Inputs sollen Software-Komponenten auf perfekt gleiche Weise gebaut werden. In diesem Sinne gewährleistet Nix *Reproduzierbarkeit*.

Für eine möglichst einfache Handhabung ist Nix *dynamisch typisiert* (*dynamically typed*). Variablen müssen niemals mit Typen annotiert werden. Statt explizit den Typ einer Variable zu deklarieren, erhält sie automatisch den Typ des Wertes, mit dem sie initiiert wird.

Anders als in anderen dynamisch typisierten Sprachen kann sich der Typ einer Nix-Variable *nicht* flexibel zur Laufzeit ändern. Das ergibt sich aus einer weiteren Restriktion, durch die Nebeneffekte vermieden werden. Variablen sind in Nix *unveränderlich* (*immutable*), wodurch ausgeschlossen ist, dass einer Variable zu einem späteren Zeitpunkt ein neuer Wert zugewiesen würde.
```nix
myVar = 42; # myVar ist von Typ int
myVar = "hello"; # verboten!
```
Wenn ihr kein neuer *Wert* zugewiesen werden kann, kann ihr natürlich auch kein Wert eines anderen *Typs* zugewiesen werden. Somit sind dynamische Typwechsel von Variablen unnötig.

Werte werden nur dann berechnet, wenn sie benötigt werden. Die Evaluierung von Ausdrücken in Nix wird deshalb als *lazy* bezeichnet. Hier ein Beispiel, das dieses Verhalten illustriert:
```nix
let attrs = { a = 15; b = builtins.throw "Oh no!"; };
in "The value of 'a' is ${toString attrs.a}"
```
Durch die eingebaute Funktion `throw` wird eine Exception geworfen. Dadurch würde der Programmablauf unterbrochen. Tatsächlich wird die Variable `b` jedoch nicht verwendet und dementsprechend niemals evaluiert. Damit wird `throw` nicht aufgerufen.

Nix ist *deklarativ*. Anders als bei sogenannten imperativen Sprachen werden Algorithmen nicht als Folge von Anweisungen definiert. Stattdessen werden in Nix gewünschte Systemzustände, Paket-Konfigurationen und -Abhängigkeiten und Build-Actions beschrieben. Dieser Ansatz spiegelt sich darin, dass Nix *Ausdrücke* (*expression*) aber keine Anweisungen (*instructions*) kennt. Es ist die Aufgabe des Nix-Paketmanagers, die deklarativen Ausdrücke zu interpretieren und umzusetzen. Ergebnisse werden im Nix-Store abgelegt.

Für eine übersichtliche Modularisierung wird empfohlen, dass eine Nix-Datei nur genau einen Ausdruck enthält. Für gewöhnlich wird dadurch eine Funktion ausgedrückt. Nix nutzt Whitespaces, um lexikalische Tokens abzugrenzen (beispielsweise bei der Definition von Listen). Ansonsten sind Leerzeichen, Zeilenumbrüche und Einrückungen nicht weiter bedeutungsvoll, aber natürlich aus stilistischen Gründen geboten. [Alejandra](https://github.com/kamadorueda/alejandra){:target="_blank"} ist ein populäres Tool zur automatischen Code-Formatierung.

## Datentypen
Nix kennt basale und zusammengesetzte Werte.[^3] Sie können durch Literale dargestellt (*represented literally*) und in Variablen gespeichert werden. In diesem Abschnitt gehe ich auf ihre Besonderheiten und Verwendungszwecke ein.

### Basale Werte
In Nix gibt es **Integers**, die wie üblich durch Ziffernfolgen dargestellt werden (`5`, `322`).[^4] Zahlen werden in Nix vor allem für Indizes und Zählvorgänge bei Iterationsvorgängen verwendet. Versionen *könnten* durch Zahlen dargestellt werden, für gewöhnlich werden aber Strings für [Semantic Versioning](https://semver.org/){:target="_blank"} verwendet (`"1.4.2"`).

**Strings** lassen sich einzeilig oder mehrzeilig darstellen. Im ersten Fall werden die String-Literale von doppelten Anführungszeichen umschlossen, im zweiten Fall von zwei einfachen Anführungszeichen je Seite. Diese Abgrenzungssymbole wurden gewählt, weil mehrzeilige Strings häufig zur Darstellung von Code anderer Sprachen verwendet werden; anders als `"` ist `''` in der Regel nicht bedeutungsvoll. So wird eine exzessive Verwendung von Escape-Zeichen vermieden.

Mehrzeilige String-Literale werden in der Regel in Code-Umgebungen eingefügt, die bereits eingerückt sind. Damit sie selbst entsprechend eingerückt werden können, ohne Tabs und Leerzeichen zu einem Teil des dargestellten Strings zu machen, wird vorangestellter Whitespace auf kluge Weise vom Nix-Interpreter ignoriert. Die Leerzeichen vor dem ersten "richtigen" Zeichen werden nicht in den String aufgenommen. Bei den darauffolgenden Zeilen wird die gleiche Anzahl von Leerzeichen entfernt. Zeilenumbrüche im Code werden als Zeilenumbrüche im String interpretiert.
```nix
''
  This
    is
      a multiline string 
''
```
Dieser Code-Block evaluiert zu `"This\n__is\n____a multiline string"`. Das heißt die erste Zeile beginnt ohne Leerzeichen, die zweite Zeile beginnt mit zwei Leerzeichen und die dritte Zeile mit vier Leerzeichen.

Werte von Ausdrücken können in Strings eingefügt werden. Diese Operation, die sowohl für einzeilige als auch mehrzeilige String-Literale verwendet werden kann, wird *String-Interpolation* genannt. Die Syntax entspricht derjenigen, wie sie vielleicht von Shells vertraut ist:
```nix
let name = "Nix";
in "hello ${name}"
```

**Dateisystempfade** und **URIs** sind spezielle Zeichenfolgen, die lokale Dateien und Verzeichnisse bzw. externe Resourcen (wie Webseiten oder Git-Repositories) identifizieren. Es versteht sich, dass Paketverwaltungssysteme sehr regelmäßigen Gebrauch von ihnen machen. Aus Komfortgründen werden sie deshalb nicht wie gewöhnlich durch Anführungszeichen denotiert, sondern können in "nackter" Form in Nix-Code eingefügt werden.

Obwohl syntaktisch ähnlich kann Nix vor allem aufgrund von Kontextinformationen zwischen ihnen unterscheiden. Ein Feld wie `url` (in einer Paket-Definition) oder eine Funktion wie `fetchurl` erwartet eine URI. Außerdem gibt es klare syntaktische Indikatoren: Ein Literal, das mit `http://` oder `https://` beginnt, repräsentiert eine URI; eines, das mit `/` beginnt, repräsentiert einen (absoluten) Pfad.

Pfad-Literale müssen mindestens ein Slash enthalten, um als solches interpretiert werden zu können. Die Substrings eines Pfads, die durch Slashes abgegrenzt werden, werden *Pfad-Komponenten* genannt.[^5] Absolute Pfade zeichnen sich dadurch aus, dass sie mit einem Slash *beginnen*. Pfade, die nicht mit einem Slash beginnen, werden relativ zu dem Verzeichnis evaluiert, in dem sich die Quellcode-Datei befindet. Der absolute Pfad zum Arbeitsverzeichnis wird automatisch vorangestellt. Wie üblich wird `.` als Pfad zum Arbeitsverzeichnis und `..` als Pfad zum Elternverzeichnis des Arbeitsverzeichnisses verstanden.[^6] Um auf relative Weise auf das Arbeitsverzeichnis zu verweisen, kann `./.` verwendet werden.[^7]

Es gibt einen weiteren Typ, dessen Werte Dateipfade darstellen. Bei Search-Paths handelt es sich um Namen für besonders wichtige Pfade. Um anzuzeigen, dass ein verwendeter Name für einen solchen Pfad steht, wird er von spitzen Klammern umschlossen. Die Namen werden in einer Umgebungsvariable festgelegt: `$NIX_PATH`. Aus diesem Grund ist bei diesen Pfaden Reproduzierbarkeit nicht gewährleistet. [In der offiziellen Dokumentation](https://nix.dev/tutorials/nix-language#search-path){:target="_blank"} wird deshalb von ihrer Verwendung abgeraten.

Die beiden Booleschen Werte (`true` und `false`) und der Nullwert (`null`) sind als eingebaute nullstellige Funktionen implementiert.[^8]

### Funktionen
Wie im Abschnitt über die Merkmale der Sprache betont, ist Nix eine *funktionale* Programmiersprache. Alle Berechnungen betreffen die Werte von Ausdrücken; und die Ausdrücke, die Funktionen oder Funktionswerte denotieren, sind von zentraler Bedeutung.

Funktionen können durch Ausdrücke definiert werden, die im Lambdakalkül *Lambda-Abstraktionen* oder einfach *Lambdas* genannt werden. In Nix haben sie die Form: `<Parameter>: <Funktionskörper>`. Wenn diese Ausdrücke als Funktionsliterale genutzt werden, spricht man von *anonymen* Funktionen. Als First-Class-Citizens können Funktionen in Variablen gespeichert und somit mit einem Namen versehen werden.

Funktionen sind weniger augenfällig als in anderen Sprachen, man sollte sich deshalb einprägen: "Wenn man einen Doppelpunkt sieht, dann hat man es mit einer Funktion zu tun!" Der Funktionskörper kann viele Operationen in einem Code-Block umfassen. Interessanter ist aber der eine Parameter.

Wie die syntaktische Form anzeigt, erwarten Nix-Funktionen in der Regel *genau ein* Argument.[^9] Dieses Grundprinzip funktionaler Programmierung ist primär dadurch motiviert, dass sich Funktionswerte und Argumente dadurch möglichst leicht lesbar in Pipelines einfügen lassen, in denen der Wert einer Funktion das Argument einer anderen Funktion wird. Ich werde gleich auf zwei wichtige Mechanismen zu sprechen können, wie Funktionen mit mehr als einem Argument auf einfache Weise mit Nix' eigenen Mitteln nachgebildet werden können.

Die zuvor besprochenen Ausdrücke bezeichnen Funktionen. Wenn auf einen solchen Ausdruck ein weiterer Ausdruck folgt, dann entsteht ein Gesamtausdruck, der einen *Funktionswert* denotiert. Diese syntaktischen Konstrukte werden im Lambdakalkül etwas irreführend *Function Application* genannt. Die eigentliche Applikation findet erst statt, wenn sie schrittweise umgeformt werden, um den tatsächlichen Funktionswert zu berechnen. Diese Operationen, die im Lambdakalkül *Beta-Reduktionen* genannt werden,[^10] werden in Nix nur bei Bedarf durchgeführt (*lazy evaluation*).

Im einfachsten Fall wird das Argument durch einen Bezeichner identifiziert, der nicht zu einer Funktion oder einer Menge evaluiert.
```nix
let zweitePotenz = x: x * x;
in zweitePotenz 3
```
Der zuletzt angeführte Ausdruck evaluiert zu `9`. Ein wenig komplizierter wird die Syntax, wenn eine anonyme Funktion auf ein Argument angewendet wird. In diesem Fall ist es notwendig, den Funktionskörper mit Klammern zu umschließen:
```nix
(x: x + 1) 1
```
In diesem Beispiel stellen die Klammern klar, was Funktionsliteral und was Argument ist.

Nun zu einer der beiden Möglichkeiten, wie Funktionsdefinitionen mit mehr als einem Parameter auf idiomatische Weise in Nix realisiert werden können. Wie oben bereits betont, können Funktionen *Funktionen* als Werte zurückgeben. Die zurückgegebene Funktion, die das Argument der äußeren Funktion enthält, kann nun selbst wiederum auf ein Argument angewendet werden. Dieses Muster, das nach seinem Urheber *Currying* genannt wird,[^11] kann beliebig oft wiederholt werden. Mit jedem Schritt wird ein weiterer Parameter hinzugefügt.

Das mag in der Theorie kompliziert klingen. In der Praxis vereinfacht die Syntax den Vorgang so sehr, dass einem kaum bewusst wird, dass man es jeweils mit nur einem Argument zu tun hat.

```nix
addTwoNumbers = a: b: a + b;
```
`addTwoNumers` erwartet *ein* Argument, das für `a` eingesetzt wird. Zurückgegeben wird eine Funktion, die selbst wiederum ein Argument erwartet. Wenn wir ein zweites Argument angeben, dann wird die *zurückgegebene* Funktion auf dieses Argument angewendet; das heißt, dass es für `b` eingesetzt wird.
```nix
addTwoNumbers 1 # evaluiert zu `b: 1 + b` (eine Funktion)
addTwoNumbers 1 2 # evaluiert zu `b: (1 + b) 2`, also `3`
```

### Zusammengesetzte Werte
In Nix gibt es (nur) zwei Möglichkeiten, wie einfachere Werte zu komplexen Strukturen zusammengefasst werden können. Im Abschnitt über Merkmale wurde bereits gesagt, dass Funktionen in Nix keine Nebeneffekte haben. Datenstrukturen sind anders als bei einigen anderen Sprachen *unveränderlich* (*immutable*): Elemente können nicht modifiziert, hinzugefügt oder entfernt werden. Deshalb sind sie *strikt* in ihrer Länge. Statt bestehende Strukturen zu verändern, können auf ihrer Grundlage neue Strukturen erstellt werden.

**Listen** (*lists*) werden genutzt, um beispielsweise Dependencies oder Konfigurationsoptionen in geordneter Form zu repräsentieren, etwa um sie iterativ zu verarbeiten. In ihnen können Werte beliebiger Typen nebeneinander stehen. Das ist anders als bei den homogenen *Arrays* anderer Sprachen. Zu den unterstützten Typen gehören insbesondere auch Funktionen. Das ist ein weiter Gesichtspunkt, unter dem sie First-Class-Citizens sind.

Syntaktisch werden Listen durch Literale denotiert, bei denen Ausdrücke von eckigen Klammern umschlossen und mit Leerzeichen (nicht etwa Kommas oder Semikolons) abgegrenzt werden. Zunächst ein einfaches Beispiel:
```nix
[[ 123 "hello" ]]
```
Dieser Ausdruck definiert eine Liste, die zwei Elemente enhält: eine Integer-Zahl und einen String. Bemerkenswert ist, dass Werte auf beliebige Weise dargestellt werden können. Funktionen beispielsweise können durch einen Namen (wie `f`) oder als Lambda-Abstraktion denotiert werden. Die Werte, die sich aus einer Anwendung einer Funktion auf ein Argument ergeben, können durch eine Function Application (im oben erläuterten syntaktischen Sinne) denotiert werden. 
```nix
[[ 123 ./foo.nix "abc" (f { x = y; }) ]
```
Zwei Anmerkungen zum diesem Beispiel. Zum einen sollte hier vielleicht daran erinnert werden, dass Nix-Ausdrücke lazy evaluiert werden: Funktionen werden nur dann tatsächlich auf das Argument anwendet, wenn der Wert auch gebraucht wird. In gewisser Weise ist der Wert also nicht Teil der Liste. Zum anderen sind die runden Klammern um den Funktionswertausdruck nicht optional. Ohne sie würden `f` und `{ x = y; }` als unabhängige Elemente der Liste interpretiert.

**Attribute Sets** sind der Hauptmechanismen, um Daten innerhalb von Nix-Ausdrücken auf sprechende Weise zu strukturieren. So werden beispielsweise Pakete oder ganze Systemkonfigurationen durch Mengen charakteristischer Attribute definiert. Im letzteren Fall werden Attribute wie `packages` (installierte Software), `users` (Benutzer und ihre Gruppen) oder `services` (aktivieren und Konfiguration von Diensten wie `openssh` oder `mariadb`) verwendet. Durch sie lassen sich auch Schlüssenwortparameter für Funktionen definieren.

Attribute sind Paare von Namen und Werten. Attributdefinitionen haben die Form `<Name> = <Wert>`. Innerhalb eines Sets darf jeder Name nur ein Mal vorkommen. Wie in der Mathematik werden Mengen-Literale durch geschweifte Klammern umschlossen; Attributdefinitionen werden dabei durch Semikolons voneinader abgegrenzt. Zur Übersichtlichkeit werden sie häufig eingerückt.
```nix
{
  x = 123;
  text = "Hello";
  y = f { bla = 456; };
}
```
Hier gilt erneut, dass der Funktionswert von `f` für dieses Argument (ein weiteres Attribute Set) erst berechnet wird, wenn `y` gebraucht wird.

Um auf Attribute bzw. Felder innerhalb einer Menge zuzugreifen, wird die Dot-Notation verwendet. Dies ist vergleichbar mit dem Zugriff auf die Attribute von Klasseninstanzen in objektorientierten Sprachen.
```nix
let person1 = { name = "Peter"; alter = 32 };
in person1.name
```
Der zuletzt angeführte Ausdruck evaluiert zu `"Peter"`. Ausdrücke dieser Form werden *Attributpfade* (*attribute paths*) genannt.

Will man auf die Attribute einer Attributmenge zugreifen, ohne den Namen der Menge voranstellen zu müssen, kann `with <Attributmenge>; <Ausdruck>` verwendet werden. Dieses Konstrukt erspart Schreibarbeit in den Fällen, in denen man innerhalb *eines* Ausdrucks mehrmals auf die Attribute eines Sets zugreift.
```nix
let
  a = {
    x = 1;
    y = 2;
    z = 3;
  };
in
with a; [ x y z ]
```

Attributmengen werden sehr häufig ineinander verschachtelt. Das heißt Mengen sind dann selbst wiederum die Werte von Attributen innerhalb einer äußeren Menge.
```nix
let 
  directoryStructure =
  {
    directoryA = {
      file1 = "Content of file 1";
      file2 = "Content of file 2";
    };

    directoryB = {
      file3 = "Content of file 3";
      file4 = "Content of file 4";
    };
  };
in
directoryStructure.directoryA.file2
```
Wie die letzte Zeile illustriert, müssen Attributpfade genau anzeigen, wie man zum gewünschten Attribut gelangt.

Attribute können nicht ohne Weiteres auf andere Attribute desselben Sets zugreifen. Der folgende Code produziert eine Fehlermeldung, da die Variablen `gruss` und `name` in diesem Kontext unbekannt sind.
```nix
{
  gruss = "hallo ";
  name = "peter";
  gesamtergruss = gruss + name; # Fehler!
}
```
Damit Attributpfade innerhalb einer Attributmenge beginnen können, muss bei ihrer Definition das Schlüsselwort `rec` vorangestellt werden. Damit entsteht ein wohlgeformter Ausdruck:
```nix
rec {
  gruss = "hallo ";
  name = "peter";
  gesamtergruss = gruss + name;
}
```
Auch wenn der Attributpfad von Innen beginnt, muss dennoch der genaue Pfad zum gewünschten Attribut angegeben werden.
```nix
rec {
  directoryA = {
    file1 = "Content of file 1";
    file2 = "Content of file 2";
    subdirectoryA = {
      file5 = "Content of file 5";
    }
  };

  directoryB = {
    file3 = "Content of file 3";
    file4 = "Content of file 4";
    file6 = directoryA.subdirectoryA.file5;
  };
};
```
Der Pfad in der letzten bedeutungsvollen Zeile beginnt von einem anderen Attribut und führt über verschiedene Ebenen zum gewünschten Wert.

Es gibt Nix-Ausdrücke, mit denen Pakete im Nixpkgs-Repository bezeichnet werden. Sie stehen für sogenannte *Derivations*, das heißt für Build-Anleitungen für Software-Komponenten. Darunter befinden sich vertraute Anwendungen wie etwa `pkgs.emacs` oder `pkgs.firefox`.[^12] Derivations sind als Attributmengen implementiert. Ich gehe auf diesen Grundbegriff des Nix-Ökosystems in einem eigenen Artikel ein.

## Operationen
Zur Komposition von Ausdrücken werden viele der typischen Operatoren verwendet. Für die arithmetischen Operationen gibt es `+`, `-`, `*` und `/`. Strings lassen sich mit `+` und Listen mit `++` verketten (*concatenation*). Es gibt die üblichen Vergleichsoperatoren (`=`, `<`, `>`, `<=`, `>=`) und die Booleschen Operatoren (`!`, `&&`, `||`).

Interessanter sind die Operationen, die auf Attributmengen definiert sind. Der Punkt-Operator zum Zugriff auf Felder innerhalb einer Attributmenge wurde oben bereits besprochen. Ein Ausdruck der Form `<Attributmenge> ? <Attributname>` evaluiert `true` oder `false`, je nachdem, ob es ein Attribut mit diesem Namen in der Menge gibt. Mit `or` kann ein Default-Wert zurückgegeben werden, falls es in einer Attributmenge kein Feld mit dem Namen gibt, auf den man zuzugreifen versucht.
```nix
let
  attributmenge = { a = 1 };
in 
  attributmenge.a or 0 # evaluiert zu 1
  attributmenge.b or 0 # evaluiert zu 0
```

Attributmengen können mit `//` vereinigt werden. Wenn es keine Überschneidungen zwischen den Mengen gibt, ist ihr Verhalten ziemlich intuitiv:
```nix
{ a = "hello"; } // { b = "world"; }
```
Das Ergebnis dieser Operation ist `{ a = "hello"; b = "world" }`. Wenn ein Name in beiden Mengen vorkommt, dann wird der Name mit dem Wert aus der *rechten* Menge übernommen.
```nix
{ a = "left"; } // { a = "right"; }
```
Dieser Ausdruck evaluiert zu `{ a = "right" }`.

Im offiziellen Handbuch findet sich ein schönes Beispiel dafür, wie der Operator zur Abstraktion und zur Vermeidung von unnötigen Wiederholungen genutzt werden kann.[^13] Wir wollen die folgende Server-Konfiguration vereinfachen:
```nix
{
  services.httpd.virtualHosts =
    { "blog.example.org" = {
        documentRoot = "/webroot/blog.example.org";
        adminAddr = "alice@example.org";
        forceSSL = true;
        enableACME = true;
        enablePHP = true;
      };
      "wiki.example.org" = {
        documentRoot = "/webroot/wiki.example.org";
        adminAddr = "alice@example.org";
        forceSSL = true;
        enableACME = true;
        enablePHP = true;
      };
    };
}
```
Es werden zwei virtuelle Hosts definiert, die sich lediglich bezüglich des Wertes vom `document-Root`-Attributs unterscheiden. Es liegt also nahe, eine Attributmenge mit den anderen Werten zu definieren. Dieses kann mit einem Set vereinigt werden, das nur das `documentRoot`-Attribut mit dem jeweiligen Wert enthält. Diese Konfiguration wird dann zum Wert des virtuellen Host:
```nix
let
  commonConfig =
    { adminAddr = "alice@example.org";
      forceSSL = true;
      enableACME = true;
    };
in
{
  services.httpd.virtualHosts =
    { "blog.example.org" = (commonConfig // { documentRoot = "/webroot/blog.example.org"; });
      "wiki.example.org" = (commonConfig // { documentRoot = "/webroot/wiki.example.com"; });
    };
}
```

## Primary Operations (PrimOps)
Als *Primäroperationen* (*primary operations*) bzw. *PrimOps* bezeichnet man die Funktionen aus der `builtins`-Standardbibliothek, die als Teil der Nix-Sprache ohne Import verwendet werden können.[^14] Die dazu verwendete Operation, `import`, ist die wohl meistgenutzte Primäroperation. Ihr wird ein Pfad übergeben, entweder als "gewöhnliches" Pfad-Literal oder als Search-Path. Da Nix-Dateien (nur) einen Ausdruck enthalten, evaluieren Ausdrücke der Form `import <Pfad>` zum Wert des Ausdrucks in der importierten Datei.

Wichtig sind auch die sogenannten Fetchers. Mit Primäroperationen wie `fetchurl`, `fetchGit` oder `fetchTarball` können Dateien heruntergeladen werden. Die heruntergeladenen Dateien werden im Nix-Store abgelegt. Archive werden automatisch entspackt. Da sich Verfügbarkeit und Inhalt der runterzuladenen Datei im Laufe der Zeit ändern kann, werden die Funktionen als *unrein* (*impure*) betrachtet.

Andere Primäroperationen sind nur in spezielleren Szenarien hilfreich. Beispielsweise werden Daten in vielen Systemen und Diensten im JSON-Format dargestellt. Die Operationen `fromJSON` und `toJSON` können genutzt werden, wenn Nix mit solchn Systemen interagiert. `fromJSON` dient dazu, um empfangene JSON-Darstellungen in Attributmengen zu übersetzen. Auf analoge Weise dient `toJSON` dazu, um Attributmengen für Systeme vorzubereiten, die mit JSON arbeiten. Tatsächlich unterscheidet sich die Syntax von Attributmengen nur geringfügig von der Darstellung im JSON-Format.

Nix definiert einen Standard zur Darstellung von Pfaden. Beispielsweise werden Backslashes (`\`), wie sie in Windows verwendet werden, durch Vorwärtsslashes (`/`) ersetzt. Relative Pfade werden in absolute Pfade überführt und Punkte werden zu ausgeschriebenen Pfad-Komponenten erweitert. `toPath` wird genutzt, um einen durch einen String dargestellten Pfad ins Standardformat umzuwandeln. Aus diesem Grund sind Pfad-Ltierale nicht bloß String-Literale ohne Anführungszeichen. Sie evaluieren automatisch zu Pfaden in Normalform.

## Namensbindungen und Gültigkeitsbereiche
Es gibt in Nix drei Kontexte, in denen Objekte an Bezeichner (*identifiers*) gebunden werden. Nur innerhalb dieser Kontexte kann über den dadurch eingeführten Namen auf den assoziierten Wert zugegriffen werden.

Wie üblich bilden Funktionen einen Gültigkeitsbereich (*scope*) für ihre lokalen Variablen. Funktionsparameter repräsentieren lokale Variablen, die ihren Namen bei der Funktionsdefinition erhalten. Anders als bei den beiden anderen Kontexten erhalten sie ihren Wert variabel zum Zeitpunkt des Funktionsaufrufs. Eine Variablenbelegung der Form `<Name> = <Wert>` findet hier nicht statt. Da Nix-Dateien für gewöhnlich *einen* Funktionsausdruck (und sonst nichts) umfassen, befinden sich die beiden anderen Kontexte innerhalb des Gültigkeitsbereiches dieser Funktion.

Variablen werden in einem `let`-Block definiert, deren Gültigkeitsbereich sich über einen darauffolgenden `in`-Block erstreckt. Die Schlüsselwörter vermitteln bereits ein intuitives Verständnis ihrer Bedeutung, weshalb sie hier bereits einige Male verwendet wurden.

Obwohl die allermeisten Funktionen in Nix genau genommen genau ein Argument erwarten, kennt Nix ein spezielles Definitionsmuster, das den Schlüsselwortparametern anderer Programmierparadigma ähnelt.
```nix
{ a, b }: a + b
```
Die geschweiften Klammern zeigen an, dass wir es bei diesem Parameter mit einer Attributmenge zu tun haben. Es fehlen jedoch Attributwerte und die Attributnamen werden durch Kommas statt Semikolons abgegrenzt. Die Funktion erwartet eine Attributmenge als Argument und diese Menge muss genau zwei Attribute enthalten und diese Attribute müssen `a` und `b` heißen. Die Reihenfolge, in der sie definiert werden, spielt jedoch keine Rolle.
```nix
let f = { a, b }: a + b
in f { a = 1; b = 2 } # Evaluiert zu `3`
```

Will man der Funktion Attributmengen beliebiger Länge übergeben können, kann bei ihrer Definition `...` angegeben werden.
```nix
let
  f = { a, b, ... }: a + b
in
  f { a = 1; b = 2; c = 3; } # Evaluiert zu `3`
```
Dadurch können der Funktion Attributmegen übergeben werden, die weitere Elemente enthalten. Die Funktionsdefinition legt der Input-Menge damit nur noch Beschränkungen bezüglich der Anzahl und der Namen der Attribute auf. Es versteht sich, dass so definierte Funktionen weitaus flexibler werden und in mehr Szenarien verwendet werden können.

Will man auf Attribute der übergebenen Menge zugreifen, für die es keinen entsprechenden Schlüsselwortparameter gibt, benötigt man Zugriff auf das Attributset selbst. Dazu kann ihm mit `@` ein Name gegeben werden. Die beiden folgenden Definitionen sind äquivalent:
```nix
args@{ a, b, ... }: a + b + args.c
<=>
{ a, b, ... }@args: a + b + args.c
```
Wie das Beispiel zeigt, kann so auf ein Attribut der übergebenen Menge zugegriffen werden, das `c` heißt. Da erst aus dem Funktionskörper statt aus der Funktionssignatur klar wird, welche Attributnamen beim Aufruf der Funktion bedeutungsvoll sind, handelt es sich nicht gerade um den cleansten Code. Doch durch Namen wird es zumindest möglich, dem Leser den Zweck vom Argument zu kommunizieren.

Durch die Angabe von *Default-Werten* wird es möglich, einer Funktion Attributmengen mit *weniger* Elementen zu übergeben. Dazu wird `?` verwendet:
```nix
let f = { a, b ? 2 }: a + b;
in f { a = 1; } # Evaluiert zu `3`
```

Wollen wir eine Variablen nach Vorlage eines Attributs erstellen, kann `inherit (<Attributmenge>) <Name> ...` verwendet werden. Die Ellipse zeigt an, dass damit beliebig viele Variablen übernommen werden können. Die beiden folgenden Definitionen sind deshalb äquivalent:
```nix
inherit (a) x y 
<=> 
x = a.x; y = a.y
```

Andersherum können mit `inherit` auch Attribute auf der Grundlage von bereits im äußeren Gültigkeitsbereich befindlichen Namensbindungen definiert werden.
```nix
let
  x = 1;
  y = 2;
in
  { inherit x y }
```
Die letzte Zeile ist äquivalent zu `{ x = x; y = y }`, wobei links vom Gleichheitszeichen ein neuer Attributname eingeführt und rechts vom Gleichzeitszeichen ein bestehender Variablenname verwendet wird. Es entsteht das Set `{ 1, 2 }`.

## Links
Für diese Artikel habe ich primär drei Quellen verwendet, die im [NixOS & Flakes Book](https://nixos-and-flakes.thiscute.world/the-nix-language/){:target="_blank"} empfohlen werden:<br>
[Nix Language Basics \(nix.dev\)](https://nix.dev/tutorials/first-steps/nix-language){:target="_blank"}<br>
[Nix, A One-Pager \(tazjin auf GitHub\)](https://github.com/tazjin/nix-1p){:target="_blank"}<br>
[Die offizielle Dokumentation](https://nixos.org/manual/nix/stable/language/){:target="_blank"}<br>

In einer früheren Version habe ich in Fußnoten gezielt auf die Abschnitte verwiesen, aus denen ich Informationen oder Beispiele entnommen habe. Ich denke es ist eine vertretbare und weit verbreitete Praxis bei einem Thema wie diesem nur allgemein auf die genutzten Seiten zu verlinken, statt Schuldigkeiten im einzelnen auszuweisen. So kann wenigstens die Fülle an Fußnoten vermieden werden.

## Literatur
Dolstra, Eelco. 2006. The Purely Functional Software Deployment Model. Utrecht: Utrecht University.<br>
Scott, Michael Lee. 2016. Programming Language Pragmatics. 4. Aufl. Waltham, MA: Morgan Kaufmann.

## Fußnoten
[^1]: Natürlich ist dieser Ansatz keineswegs alternativlos. [GNU Guix](https://guix.gnu.org/){:target="_blank"} ist eine Distribution, die stark von NixOS inspiriert wurde, die sich aber für Guile Scheme als Programmiersprache entschieden hat.
[^2]: Es werden drei Status von Werten unterschieden. Für die Details von *first-class*-, *second-class*- und *third-class*-Werten, siehe beispielsweise Scott 2016, 155.
[^3]: Die Rede von *basalen* Werten ist bewusst vage. In einem [Thread auf Stack Overflow](https://stackoverflow.com/questions/6623130/scalar-vs-primitive-data-type-are-they-the-same-thing){:target="_blank"} werden präzisere Begriffe wie *skalar*, *nativ* oder *atomar* erläutert.
[^4]: Genau genommen gibt es in Nix neben Integers noch Floating-Point Numbers. Diese dürften von keiner sehr großen Wichtigkeit sein.
[^5]: Vgl. Dolstra 2006, 67.
[^6]: Die Pfad-Literale `relative/path` und `./relative/path` dürften in den meisten Fällen äquivalent sein. Es gibt aber feine Unterschiede.
[^7]: [nix.dev](https://nix.dev/tutorials/nix-language#file-system-paths) hebt diesen Ausdruck gesondert hervor. Die Motivation dafür erschließt sich mir ehrlicherweise nicht recht.
[^8]: Vgl. Dolstra 2006, 71.
[^9]: Es gibt einige wenige Funktionen, die ohne Argument aufgerufen werden können (*nullary functions*). `true`, `false` und `null` wurden bereits angeführt.
[^10]: Die andere zentrale Umformungsoperation des Lambdakalküls, die Alpha-Conversion genannt wird und durch die gebundene Variablen umbenannt werden, spielt in Nix keine große Rolle.
[^11]: Haskell Curry hat die Methodik popularisiert. Moses Schönfinkel oder sogar Gottlob Frege haben sie wohl bereits vor ihm in veröffentlichten Schriften angesprochen.
[^12]: <a href="https://nixos.org/manual/nixos/stable/#sec-configuration-file" target="_blank">https://nixos.org/manual/nixos/stable/#sec-configuration-file</a>.
[^13]: <a href="<https://nixos.org/manual/nixos/stable/#sec-module-abstractions>" target="_blank">https://nixos.org/manual/nixos/stable/#sec-module-abstractions</a>. Für den gleichen Zweck hätte auch eine Funktion definiert werden können, die den `documentRoot`-Wert als Argument nimmt.
[^14]: Vgl. Dolstra 2006, 80.
