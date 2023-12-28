---
layout: post
title:  "Nix Pills (6)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 6 (Kommentar)
<div class="hide-excerpt">
Das sechste Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/our-first-derivation" target="_blank">Our First Derivation</a>, führt den zentralen Grundbegriff der Nix-Paketverwaltung ein. Das Konzept wird über die `derivation`-Funktion erläutert. Derivations sind der Rückgabewert dieser Funktion. Welche Derivation zurückgegeben wird, hängt von Inputs ab, die in Form einer Attributmenge übergeben werden. Store Derivations (`.drv`-Dateien) sind Nebeneffekte des Funktionsaufrufs.
</div>
<!--more-->

Das sechste Kapitel der Nix Pills, [Our First Derivation](https://nixos.org/guides/nix-pills/our-first-derivation){: target="_blank"}, führt den zentralen Begriff der Nix-Paketverwaltung ein. Wir erfahren, was Derivations sind und zu welchem Zweck Store Derivations (`.drv`-Dateien) genutzt werden. Derivations werden durch die `derivation`-Funktion und auf der Grundlage einer Attributmenge erzeugt.

## Der Input der `derivation`-Funktion: eine gewöhnliche Attributmenge
Derivations werden von der `derivation`-Funktion erzeugt. Ihr wird eine Attributmenge als Argument übergeben, die bereits einige Ähnlichkeit mit der Derivation aufweist, die von der Funktion zurückgegeben wird. Damit eine Derivation erstellt werden kann, sind drei Attribute als Input zwingend erforderlich:
- `Name`: Der hier eingegebene Derivationsname wird in Pfaden verwendet. Dabei ist zu unterscheiden zwischen den Pfaden zu Store Derivations (die bei der *Instanziierung* einer Derivation erstellt werden); und den Output-Pfaden (die bei der *Realisierung* einer Derivation erstellt werden). Die Pfade folgen einem vordefinierten Format. Zu den beiden Operationen und den Pfadformaten unten mehr.
- `system`: Builder sind systemabhängig. Über dieses Attribut wird spezifziert für welchen Systemtyp bzw. für welche Systemarchitektur der Builder konzipiert wurde. Dazu gehört beispielsweise `system = "x86_64-linux";`.[^currentSystem]
- `builder`: Als Wert wird ein Pfad zu einer ausführbaren Datei (*builder executable*) angegeben. Die referenzierte Anwendung baut die Derivation. Das kann eine Shell (`builder = "/bin/bash";`), ein Skript (`builder = ./builder.sh;`) oder eine andere Derivation (`builder = "${pkgs.python}/bin/python";`) sein. Der Builder ist verantwortlich, den Output-Pfade und ihre Inhalte zu erstellen.

Über optionale Attribute erfahren wir in Chapter 6 nahezu nichts. Das [offizielle Bedienungshandbuch](https://nixos.org/manual/nix/stable/language/derivations.html#optional){: target="_blank"} nennt zwei weitere, die häufig verwendet werden.
- `args`: Damit können dem Builder Kommandozeilenargumente übergeben werden. Es wird ein Beispiel für Bash angeführt: `args = [ "-c" "echo hello world > $out" ];`.
- `outputs`: Eine Liste von Namen, für die Umgebungsvariablen erstellt werden. Auf diese Variablen kann der Builder bei der Ausführung zugreifen. Die Werte dieser Variablen sind die ihnen entsprechenden Store-Pfade. Im Default-Fall gibt es nur einen Output, der `out` genannt wird. Das Resultat dieses Attributs ist nicht trivial. Das Bedienungshandbuch gibt ein recht ausführliches Beispiel dazu.

Dadurch, dass `outputs` im Beispiel aus dem Beitrag nicht definiert wird, ergeben sich spezielle Eigenschaften der erzeugten Derivations. Diese zeigen sich (wie wir unten sehen werden) an den Werten der `out`- und `all`-Attribute.

## Der Output der `derivation`-Funktion: eine Derivation
Die `derivation`-Funktion gibt eine Derivation zurück. Dieser Output ist selbst wiederum eine Attributmenge. Es wird eine Methode angeführt, wie wir diese Feststellung verifizieren können:
```
nix-repl> d = derivation { name = "myname"; builder = "mybuilder"; system = "mysystem"; }
nix-repl> builtins.isAttrs d
true
```
Natürlich handelt es sich nicht um exakt die gleiche Menge (`derivation` ist natürlich nicht die Identitätsfunktion). Wenn wir uns die Attributnamen der zurückgegebenen Menge anzeigen lassen, dann finden wir einige darunter, die noch nicht in der Inputmenge waren:
```
nix-repl> d = derivation { name = "myname"; builder = "mybuilder"; system = "mysystem"; }
nix-repl> builtins.attrNames d
[ "all" "builder" "drvAttrs" "drvPath" "name" "out" "outPath" "outputName" "system" "type" ]
```
Für eine gegebene Output-Menge erhalten wir die Derivation, die als Input für `derivation` diente, über das `drvAttrs`-Attribut:
```
nix-repl> d.drvAttrs
{ builder = "mybuilder"; name = "myname"; system = "mysystem"; }
```

Wir erfahren ein paar Dinge über die neuen Attribute:
- Für das `type`-Attribut wird der Wert `"derivation"` gesetzt. "Nix does add a little of magic to sets with type derivation, but not that much." Das heißt vermutlich, dass Mengen mit diesem Attribut in einigen Kontexten anders behandelt werden als gewöhnliche Attributmengen. Leider werden die Besonderheiten nicht weiter ausgeführt.
- "The `d.drvPath` is the path of the .drv file", also beispielsweise `/nix/store/<Hash-Wert>-myname.drv`.
- "The `outPath` attribute is the build path in the nix store", also beispielsweise `/nix/store/<Hash-Wert>-myname`.
- Wenn das `outputs`-Attribut für die hineingegebene Derivation nicht ausdrücklich definiert wurde, dann ist die Output-Menge und der Wert des `out`-Attributs dieser Menge äquivalent:
```
nix-repl> (d == d.out)
true
```
- Wir erfahren auch, dass der Wert von `all` ein Singleton ist, wenn beim Input nur ein Output verlangt wurde. Statt Singleton sagen wir im Deutschen meist Einermenge, wahrscheinlich ist `all` aber eine Liste.

Wenn das `outPath`-Attribut definiert ist, dann kann die Menge an die eingebaute `toString`-Funktion übergeben werden, um Pfad zu erhalten.
```
nix-repl> d.outPath
"/nix/store/40s0qmrfb45vlh6610rk29ym318dswdr-myname"
nix-repl> builtins.toString d
"/nix/store/40s0qmrfb45vlh6610rk29ym318dswdr-myname"
```

Um ehrlich zu sein verstehe ich den Zweck von `toString` nicht. Wenn wir den gleichen String-Wert erhalten, könnten wir dann nicht auch einfach `<Attributmenge>.outPath` nutzen?

## Der Nebeneffekt der `derivation`-Funktion: eine `.drv`-Datei
Wie im vorausgegangen Abschnitt erläutert, erzeugt die `derivation`-Funktion eine Derivation. Das ist der *Rückgabewert* der Funktion. Für eine funktionale Programmiersprache untypisch hat sie darüber hinaus einen *Nebeneffekt*. Es wird eine Store Derivation in Form einer `drv`-Datei erzeugt.

Der Zweck dieser Dateien wird in Analogie mit C erläutert:
- `.nix`-Dateien sind wie `.c`-Dateien.
- `.drv`-Dateien sind wie `.o`-Dateien. Wie dieses sind sie Teil eines Zwischenschritts. Sie beschreiben, wie eine Derivation (der Output der `derivation`-Funktion) gebaut werden kann.
- Output-Pfade sind das "Produkt vom beschriebenen Build-Vorgang" (*the product of the build*). Hier wird kein C-Gegenstück angeführt. Wahrscheinlich eine ausführbare Binärdatei bzw. eine Reihe von Outputs in eigenen Unterverzeichnissen? Richtiger ist es wohl. dass nicht die Pfade das Produkt sind, sondern die Dateien und Verzeichnisse an diesen Pfaden?

Ehrlich gesagt verstehe ich den Nutzen von Store Derivations (`.drv`-Dateien) trotzdem nicht. Enthalten die Dateien besondere Informationen, die nicht aus den Input-Mengen abgeleitet werden könnten? Jedenfalls enthalten `drv`-Dateien so etwas wie Minimalinformationen.

Das Dateiformat ist lesbar, aber nicht ohne Weiteres strukturiert. Mit `nix derivation show` lassen sie sich pretty-printen. Ihr Inhalt ergibt sich daraus, wie die Attributmenge beschaffen war, die `derivation` als Input übergeben wurde.
```
$ nix derivation show /nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv
{
  "/nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv": {
    "outputs": {
      "out": {
        "path": "/nix/store/40s0qmrfb45vlh6610rk29ym318dswdr-myname"
      }
    },
    "inputSrcs": [],
    "inputDrvs": {},
    "platform": "mysystem",
    "builder": "mybuilder",
    "args": [],
    "env": {
      "builder": "mybuilder",
      "name": "myname",
      "out": "/nix/store/40s0qmrfb45vlh6610rk29ym318dswdr-myname",
      "system": "mysystem"
    }
  }
}
```

Zu den Umgebungsvariablen (definiert unter `"env"`) wird gesagt: "(...) (T)he environment variables passed to the builder are just those you see in the `.drv` plus some other Nix related configuration (number of cores, temp dir, ...). The builder will not inherit any variable from your running shell, otherwise builds would suffer from non-determinism."

Das Beispiel im Beitrag wird so abgewandelt, dass `true` hinzugefügt wird. Bei dieser Anwendung handelt es sich um eine Komponente der GNU Coreutils. Das Interessante an dieser Modifikation ist, dass die Coreutils dem `"inputDrvs"`-Attribut bei Instanziierung automatisch hinzugefügt wird:
```
$ nix derivation show /nix/store/qyfrcd53wmc0v22ymhhd5r6sz5xmdc8a-myname.drv
{
  "/nix/store/qyfrcd53wmc0v22ymhhd5r6sz5xmdc8a-myname.drv": {
    "outputs": {
      "out": {
        "path": "/nix/store/ly2k1vswbfmswr33hw0kf0ccilrpisnk-myname"
      }
    },
    "inputSrcs": [],
    "inputDrvs": {
      "/nix/store/hixdnzz2wp75x1jy65cysq06yl74vx7q-coreutils-8.29.drv": [
        "out"
      ]
    },
    "platform": "x86_64-linux",
    "builder": "/nix/store/qrxs7sabhqcr3j9ai0j0cp58zfnny0jz-coreutils-8.29/bin/true",
    "args": [],
    "env": {
      "builder": "/nix/store/qrxs7sabhqcr3j9ai0j0cp58zfnny0jz-coreutils-8.29/bin/true",
      "name": "myname",
      "out": "/nix/store/ly2k1vswbfmswr33hw0kf0ccilrpisnk-myname",
      "system": "x86_64-linux"
    }
  }
}
```

Beim Bauen einer Derivation werden zunächst die `.drv`-Dateien der Inputs gebaut, falls die daraus resultierenden Outputs noch nicht im Store vorhanden sind. Erst im Anschluss daran wird die Store Derivation gebaut, dessen `inputDrvs` sie sind.

## Derivation: Instanziierung und Realisierung
Es wird betont, dass eine Derivation durch die `derivation`-Funktion noch nicht gebaut wurde. Wir haben damit eine Situation, in der Output-Pfade spezifziert wurden; bei der die entsprechenden Verzeichnisse und Inhalte tatsächlich aber (noch) gar nicht existieren.

Die zwei Phasen werden durch jeweils eine Nix-Operation eingeleitet: Durch eine *Instanziierung* (`nix-instantiate`) wird der Ausdruck ausgewertet und die `.drv`-Datei erstellt; und durch eine *Realisierung* (`nix-store -r`) wird die Store Derivation (die `.drv`-Datei) gebaut.[^vergleich] Dazu wird sie als Argument übergeben:
```bash
nix-store -r /nix/store/z3hhlxbckx4g3n9sw91nnvlkjvyw754p-myname.drv
```

Die zugrundeliegende Designentscheidung wird folgendermaßen motiviert:
>  Think, if Nix ever built the derivation just because we accessed it in Nix, we would have to wait a long time if it was, say, Firefox. That's why Nix let us know the path beforehand and kept evaluating the Nix expressions, but it's still empty because no build was ever made.

Die Unterscheidung der Schritte hat demnach völlig praktische und recht leicht nachvollziehbare Gründe.

Im REPL werden Derivations mit `:b <Derivation>` gebaut. Es wird nicht darauf eingegangen, wie die Operation in anderen Kontexten ausgeführt wird. Wahrscheinlich kombiniert `nix-build <Derivation>` Instanziierung und Realisierung?



Die Dateien und Verzeichnisse an den Pfadzielen werden erstellt, wenn dafür zuständige Operationen ausgeführt werden. Bei der *Instanziierung* werden Store Derivations (`.drv`-Dateien) erstellt. Diese finden sich unter `/nix/store/<hash>-<name>.drv`. Bei der *Realisierung* werden die Derivations gebaut und die eigentlich interessanten Outputs erzeugt. *Eine* Derivation kann dabei in *mehreren* Outputs (mit ihren eigenen Verzeichnissen) resultieren. Diese finden sich unter `/nix/store/<hash>-<name>[-<output>]`.
Store Derivation-Pfaden (*store derivation paths*) und Output-Pfaden (*output paths*) angehängt. In der [offiziellen Dokumentation](https://nixos.org/manual/nix/stable/language/derivations.html#required){: target="_blank} findet sich ein Beispiel. Wenn `name = "hello";` gesetzt wurde, dann ergibt sich ein Pfad zur entsprechenden Store Derivation mit dem Format `/nix/store/<hash>-hello.drv`; und der Output-Pfad hat die Form `/nix/store/<hash>-hello[-<output>]`. Die letzte (optionale) Pfad-Komponente wird für alle Outputs genutzt, die nicht der Default (`out`) sind.

Das Beispiel aus der Dokumentation illustriert die Pfad-Benennungen, die sich ergeben (`[-<output>]` oben). Bei
```nix
derivation {
  name = "example";
  outputs = [ "lib" "dev" "doc" "out" ];
  # ...
}
```
erhalten wir den Store Derivation-Pfad `/nix/store/<hash>-example.drv` und wir erhalten Pfade für alle Outputs (und den Hauptfall ohne Output-Suffix): `/nix/store/<hash>-example-lib`, `/nix/store/<hash>-example-dev`, `/nix/store/<hash>-example-doc`, `/nix/store/<hash>-example`.

## Offene Fragen
- Was sind Beispiele für Kontexte, in denen Derivations (Mengen mit `type = "derivation";`) anders behandelt werden als gewöhnliche Attributmengen?
- Warum würden wir `builtins.toString <Attributmenge>` verwenden? Wo sind die Vorteile gegenüber `<Attributmenge>.outPath`?
- Welche Änderungen führen dazu, dass der `.drv` einer Derivation neue Inputs (`"inputDrvs"`) hinzugefügt werden?

## Fußnoten
[^currentSystem]: Um sich den Typ des aktuellen Systems anzuzeigen, kann `builtins.currentSystem` im REPL genutzt werden.
[^vergleich]: Die Unterscheidung wird erneut in Analogie zu C erläutert: "Think of it as of compile time and link time like with C/C++ projects. You first compile all source files to object files. Then link object files in a single executable."
