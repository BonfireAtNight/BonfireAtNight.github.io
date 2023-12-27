---
layout: post
title:  "Nix Pills (6)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 6 (Kommentar)
<div class="hide-excerpt">
Das sechste Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/our-first-derivation" target="_blank">Our First Derivation</a>, führt den zentralen Grundbegriff der Nix-Paketverwaltung ein.
</div>
<!--more-->

Das sechste Kapitel der Nix Pills, [Our First Derivation](https://nixos.org/guides/nix-pills/our-first-derivation){: target="_blank"}, führt den zentralen Begriff der Nix-Paketverwaltung ein. Wir erfahren, was Derivations sind und zu welchem Zweck `.drv`-Dateien genutzt werden. Derivations werden durch die `derivation`-Funktion und auf der Grundlage einer Attributmenge erzeugt.

## Der Input der `derivation`-Funktion: eine gewöhnliche Attributmenge
Derivations werden von der `derivation`-Funktion erzeugt. Ihr wird eine Attributmenge als Argument übergeben, die bereits einige Ähnlichkeit mit der Derivation hat, die von der Funktion zurückgegeben wird. Damit eine Derivation erstellt werden kann, sind drei Attribute als Input zwingend erforderlich:
- `Name`: Der hier eingegebene Derivationsname wird Store-Pfaden (*store paths*) und Output-Pfaden (*output paths*) angehängt. In der [offiziellen Dokumentation](https://nixos.org/manual/nix/stable/language/derivations.html#required){: target="_blank} findet sich ein Beispiel. Wenn `name = "hello";` gesetzt wurde, dann ergibt sich ein Pfad zur entsprechenden Store Derivation mit dem Format `/nix/store/<hash>-hello.drv`; und der Output-Pfad hat die Form `/nix/store/<hash>-hello[-<output>]`.
- `system`: Builder sind systemabhängig. Über dieses Attribut wird spezifziert für welchen Systemtyp bzw. für welche Systemarchitektur der Builder konzipiert wurde. Dazu gehört beispielsweise `system = "x86_64-linux";`.[^currentSystem]
- `builder`: Als Wert wird ein Pfad zu einer ausführbaren Datei (*builder executable*) angegeben. Die referenzierte Anwendung baut die Derivation. Das kann eine Shell (`builder = "/bin/bash";`), ein Skript (`builder = ./builder.sh;`) oder eine andere Derivation (`builder = "${pkgs.python}/bin/python";`) sein.

Über optionale Attribute erfahren wir in Chapter 6 nahezu nichts. Das [offizielle Bedienungshandbuch](https://nixos.org/manual/nix/stable/language/derivations.html#optional){: target="_blank"} nennt zwei weitere, die häufig verwendet werden.
- `args`: Damit können dem Builder Kommandozeilenargumente übergeben werden. Es wird ein Beispiel für Bash angeführt: `args = [ "-c" "echo hello world > $out" ];`.
- `outputs`: Eine Liste von Namen, für die Umgebungsvariablen erstellt werden. Auf diese Variablen kann der Builder bei der Ausführung zugreifen. Die Werte dieser Variablen sind die ihnen entsprechenden Store-Pfade. Im Default-Fall gibt es nur einen Output, der `out` genannt wird. Das Resultat dieses Attributs ist nicht trivial. Das Bedienungshandbuch gibt ein recht ausführliches Beispiel dazu.

Dadurch, dass `outputs` im Beispiel aus dem Beitrag nicht definiert wird, ergeben sich spezielle Eigenschaften der erzeugten Derivations. Diese zeigen sich (wie wir unten sehen werden) an den Werten der `out`- und `all`-Attribute.

## Der Output der `derivation`-Funktion: eine Derivation
Derivationen dienen als Input für die `derivation`-Funktion. Der Output dieser Funktion ist selbst wiederum eine Attributmenge. Es wird eine Methode angeführt, wie wir diese Feststellung verifizieren können:
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

Wir erfahren ein paar Dinge zu den neuen Attributen:
- Für das `type`-Attribut wird der Wert `"derivation"` gesetzt. "Nix does add a little of magic to sets with type derivation, but not that much." Das heißt vermutlich, dass Mengen mit diesem Attribut in einigen Kontexten anders behandelt werden als gewöhnliche Attributmengen. Leider wird kein Beispiel dafür angeführt.
- "The `d.drvPath` is the path of the .drv file", also beispielsweise `/nix/store/<Hash-Wert>-myname.drv`.
- "The `outPath` attribute is the build path in the nix store", also beispielsweise `/nix/store/<Hash-Wert>-myname`.
- Wenn das `outputs`-Attribut für die hineingegebene Derivation nicht ausdrücklich definiert wurde (oder `out` als Wert bestimmt wurde), dann ist die Output-Menge und der Wert des `out`-Attributs dieser Menge äquivalent:
```
nix-repl> (d == d.out)
true
```
- Wir erfahren auch, dass der Wert von `all` ein Singleton ist, wenn beim Input nur ein Output verlangt wurde. Statt Singleton sagen wir im Deutschen meist Einermenge. Wahrscheinlich ist `all` aber eine Liste.

## Der Nebeneffekt der `derivation`-Funktion: eine `.drv`-Datei
Wie im vorausgegangen Abschnitt erläutert, erzeugt die `derivation`-Funktion eine Derivation. Das ist der *Rückgabewert* der Funktion. Für eine funktionale Programmiersprache untypisch hat sie darüber hinaus einen *Nebeneffekt*. Es wird eine Store Derivation in Form einer `drv`-Datei erzeugt.

Der Zweck dieser Dateien wird in Analogie mit C erläutert:
- `.nix`-Dateien sind wie `.c`-Dateien.
- `.drv`-Dateien sind wie `.o`-Dateien. Wie dieses sind sie Teil eines Zwischenschritts. Sie beschreiben, wie eine Derivation (der Output der `derivation`-Funktion) gebaut werden kann.
- Output-Pfade sind das "Produkt vom beschriebenen Build-Vorgang" (*the product of the build*). Hier wird kein C-Gegenstück angeführt. Wahrscheinlich eine ausführbare Binärdatei? Wahrscheinlich sind auch nicht die Pfade das Produkt, sondern die Dateien an diesen Pfaden?

`drv`-Dateien enthalten so etwas wie Minimalinformationen. Sie sind lesbar, aber nicht ohne Weiteres strukturiert. Mit `nix derivation show` lassen sie sich pretty-printen. Ihr Inhalt ergibt sich daraus, wie die Attributmenge beschaffen war, die `derivation` als Input übergeben wurde.
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

Es wird betont, dass die entsprechende Derivation damit noch nicht gebaut wurde (darauf wird in diesem Kapitel noch gar nicht eingegangen). Wir haben damit eine Situation, in der Output-Pfade spezifziert wurden; bei der die entsprechenden Verzeichnisse und Inhalte tatsächlich aber (noch) gar nicht existieren.

Die zugrundeliegende Designentscheidung wird folgendermaßen motiviert:
>  Think, if Nix ever built the derivation just because we accessed it in Nix, we would have to wait a long time if it was, say, Firefox. That's why Nix let us know the path beforehand and kept evaluating the Nix expressions, but it's still empty because no build was ever made.

Die Unterscheidung der Schritte hat demnach völlig praktische und recht leicht nachvollziehbare Gründe.

## Offene Fragen
- Was sind Beispiele für Kontexte, in denen Derivations (Mengen mit `type = "derivation";`) anders behandelt werden als gewöhnliche Attributmengen?

## Fußnoten
[^currentSystem]: Um sich den Typ des aktuellen Systems anzuzeigen, kann `builtins.currentSystem` im REPL genutzt werden.
