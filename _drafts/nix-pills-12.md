---
layout: post
title:  "Nix Pills (12)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 12 (Kommentar)
<div class="hide-excerpt">
Im zwöften Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/inputs-design-pattern" target="_blank">Inputs Design Pattern</a>, geht es um Design Patterns. Es wird kurz auf das Single Repository Pattern eingegangen. Der Fokus liegt auf dem Inputs Design Pattern
</div>
<!--more-->

Im zwöften Kapitel der Nix Pills, [Inputs Design Pattern](https://nixos.org/guides/nix-pills/inputs-design-pattern){: target="_blank"}, geht es um Design Patterns. Zwar gibt es in Nix keine vorgegebene Verzeichnisstruktur oder Packaging-Policy. Dennoch lassen sich aus der Struktur vom Nixpkgs-Repository Muster abstrahieren, die sich bewährt haben.

## Paket-Repositories
Bisher wurden einzelne Pakete in Form von Derivations beschrieben. Ab diesem Beitrag soll ein *Repository* angelegt werden, das mehr als eine Paketdefinition umfasst. In seiner Arbeitsweise soll es sich wie das Nixpkgs-Repository verhalten. Zur Erinnerung: das Repository setzt sich zusammen aus Ausdrücken, die Pakete *beschreiben*.

Nix verwendet *ein* Repository für *alle* Pakete. Darin unterscheidet es sich von beispielsweise Debian, wo Pakete über viele kleine Repositories verteilt werden. Dadurch werden einige Probleme vermieden. "(Debian's approach) can make it hard to track interdependent changes and to contribute to new packages."

Die Nix Pills sprechen vom Single Repository Pattern:
> The natural implementation in Nix is to create a top-level Nix expression, and one expression for each package. The top-level expression imports and combines all expressions in a giant attribute set with name -> package pairs.

Auf den ersten Blick könnte man denken, dass es extrem rechenaufwändig wäre, einen alle Pakete umfassenden Ausdruck auszuwerten. Um das zu vermeiden, werden Komponenten nur dann ausgewertet, wenn sie tatsächlich benötigt werden. Das ist ein Hauptaspekt des Lazy-Merkmals der Nix Expression Language, von der in einemm früheren Kapitel die Rede war.

## graphviz verpacken
Zusammen mit GNU Hello wird dem Repository ein weiteres Programm hinzugefügt, [Graphviz](https://graphviz.org/){: target="_blank"}. Das Tool wird genutzt, um Graphen visuell darzustellen. Dabei handelt es sich um eine weitere Software-Komponente, die mit den Autotools gebaut wird. Zur Verpackung kann deshalb unser bisheriger Standard verwendet werden. Hier die `graphviz.nix`, in der die entsprechende Derivation erzeugt wird:
```nix
let
  pkgs = import <nixpkgs> {};
  mkDerivation = import ./autotools.nix pkgs;
in mkDerivation {
  name = "graphviz";
  src = ./graphviz-2.49.3.tar.gz;
}
```

Das Paket könnte mit `nix-build graphviz.nix` gebaut werden. Zwar erhalten wir dadurch eine ausführbare Anwendung. Dabei handelt es sich aber lediglich um den Core; für basale Funktionalitäten (wie die Erzeugng von `.png`-Dateien) müssen weitere Komponenten hinzugefügt werden. Deshalb scheitert der folgende Befehl:
```
$ echo 'graph test { a -- b }'|result/bin/dot -Tpng -o test.png
Format: "png" not recognized. Use one of: canon cmap [...]
```

Der Einfachheit halber wird die [GD Library](https://libgd.github.io/pages/about.html){: target="_blank"} hinzugefügt. Die Bibliothek muss deshalb den Paket-spezifischen Build-Dependencies hinzugefügt werden.

Wenn ich die Ausführungen richtig verstehe, dann werden verschiedene Varianten von graphviz gebaut, je nachdem, welche Flags dem Compiler übergeben werden. Zur Kommunikation mit dem Compiler wird [pkg-config](https://www.freedesktop.org/wiki/Software/pkg-config/){: target="_blank} verwendet. Dieses Tool sagt dem Konfigurationsskript (im Konfigurationsschritt), wo Header und Libraries zu finden sind.

In einem System, das dem POSIX-Standard folgt, erwartet und findet pkg-config die Bibliotheken unter `/usr/lib/pkgconfig`. Mit seinem Store weicht Nix von dem Standard ab. Wir müssen ihm deshalb sagen, wo die Bibliotheken zu finden sind. Das erfolgt über die [`PKG_CONFIG_PATH`](https://askubuntu.com/questions/210210/pkg-config-path-environment-variable){: target="_blank}-Umgebungsvariable.

Wir können für BuildInputs Pfade zum `PKG_CONFIG_PATH` hinzufügen, ganz so, wie wir es für ausführbare Dateien und dem `PATH` gemacht haben. Dazu wird das `setup.sh`-Skript entsprechend angepasst:[^fehler]
```bash
for p in $baseInputs $buildInputs; do
  if [ -d $p/bin ]; then
    export PATH="$p/bin${PATH:+:}$PATH"
  fi
  if [ -d $p/lib/pkgconfig ]; then
    export PKG_CONFIG_PATH="$p/lib/pkgconfig${PKG_CONFIG_PATH:+:}$PKG_CONFIG_PATH"
  fi
done
```
Die `gd`-Bibliothek kann nun den `BuildInputs` vom `graphviz`-Paket hinzugefügt werden:[^split-outputs]
```nix
let
  pkgs = import <nixpkgs> {};
  mkDerivation = import ./autotools.nix pkgs;
in mkDerivation {
  name = "graphviz";
  src = ./graphviz-2.49.3.tar.gz;
  buildInputs = with pkgs; [
    pkg-config
    (pkgs.lib.getLib gd)
    (pkgs.lib.getDev gd)
  ];
}
```
Mit der Variante von `graphviz`, die auf diese Weise erzeugt wird, kann der obige (eine PNG-Datei erzeugende) Befehl erfolgreich ausgeführt werden.

## Der Repository-Ausdruck
Wie bezüglich des Single Repository Pattern gesagt wurde, wird ein Paket-Repository in Nix durch *einen* Ausdruck repräsentiert. Innerhalb des Ausdrucks werden die Derivations in Variablen gespeichert, die nach den entsprechenden Paketen benannt sind. Dazu werden die Dateien importiert, die den `mkDerivation`-Aufruf (die Erzeugung der Derivation als Rückgabewert) enthalten.

Repository ist im Sinne eines Git-Repositories zu verstehen. Wir müssen also ein neues Verzeichnis anlegen und zur Versionsverwaltung initialisieren. Darin wird eine `default.nix` erstellt,[^default] die den Repository-Ausdruck enthält:
```nix
{
  hello = import ./hello.nix; 
  graphviz = import ./graphviz.nix;
}
```
Im Repository enthaltene Pakete können nun gebaut und installiert werden. Der Repository-Ausdruck evaluiert zu einer Attributmenge und mit dem `-A`-Flag kann auf darin enthaltene Attribute zurückgegriffen werden. Durch `nix-env -f <Paket-Repository>` kann ein in einer Datei enthaltenes Paket-Repository bestimmt werden.
```
$ nix-build default.nix -A hello
[...]
$ result/bin/hello
Hello, world!

$ nix-env -f . -iA graphviz
[...]
$ dot -V
```
Wie beim Nixpkgs-Repository können Variablen geladen werden:
```
$ nix repl
nix-repl> :l default.nix
Added 2 variables.
nix-repl> hello
«derivation /nix/store/dkib02g54fpdqgpskswgp6m7bd7mgx89-hello.drv»
nix-repl> graphviz
«derivation /nix/store/zqv520v9mk13is0w980c91z7q1vkhhil-graphviz.drv»
```
Die Variablen repräsentieren also die (durch den `mkDerivation`-Aufruf erzeugten) Derivations.

Damit wurde das Grundverhalten vom Nixpkgs-Repository im (ganz) Kleinen reproduziert.

## Das Inputs Design Pattern
Die bisher geschriebenen Dateien folgen noch nicht dem angekündigten Inputs Design Pattern. Sie weisen drei Probleme auf, die dadurch behoben werden sollen, dass das Muster angewendet wird.
- `hello.nix` und `graphviz.nix` importieren das Nixpkgs-Repository direkt. Besser wäre es, wenn das Repository lediglich als Input (Argument) übergeben würde.[^autotools] Der Grund dafür ist vermutlich, dass wir dadurch flexibel bezüglich der Quelle der Pakete bleiben.
- Wir können nicht flexibel zwischen Graphviz-Varianten wählen. Wir sollten festlegen können, ob mit `gd` kompiliert werden soll.
- Es kann nicht festgelegt werden, welche Version von `gd` verwendet werden soll. Das wäre in Fällen problematisch, in denen wir mit einer bestimmten Version als Dependency experimentieren wollen.

Das Inputs Design Pattern soll Abhilfe schaffen, da der Benutzer oder der Caller über die Inputs entscheiden kann. Das heißt, wir haben eine Parametermenge und es kann beim Aufruf entschieden werden, welche Attributwerte für die Parameter übergeben werden.

Um das erste Problem zu lösen, müssen die Derivation-Dateien sowie die `default.nix` umgeschrieben werden. Zunächst zu letzterer. `hello.nix` und `graphviz.nix` werden so abgewantelt, dass sie nun Lambdaausdrücke enthalten. Der bisherige Hauptteil, der `mkDerivation`-Aufruf, arbeitet mit diesen Inputs. Hier das `graphviz.nix`-Beispiel (`hello.nix` wäre analog):
```nix
{ mkDerivation, lib, gd, pkg-config }:

mkDerivation {
  name = "graphviz";
  src = ./graphviz-2.49.3.tar.gz;
  buildInputs = [
    pkg-config
    (pkgs.lib.getLib gd)
    (pkgs.lib.getDev gd)
  ];
}
```
Bei den Inputs handelt es sich aus Komponenten des Nixpkgs-Repository. Auf der Ebene des Callers (in der `default.nix`) sind diese Symbole kannt, da das Repository importiert wird.

Wenn die beiden Paket-Dateien nun Lambdaausdrücke enthalten, evaluiert ihr Import zu einer Funktion. Dieser müssen die notwendigen Attributmengen übergeben werden, die die Argumente für die Parameter enthalten:[^inherit]
```nix
let
  pkgs = import <nixpkgs> {};
  mkDerivation = import ./autotools.nix pkgs;
in with pkgs; {
  hello = import ./hello.nix { inherit mkDerivation; };
  graphviz = import ./graphviz.nix { inherit mkDerivation lib gd pkg-config; };
}
```

Um das zweite Problem zu lösen, kann ein zusätzlicher Input eingefügt werden. Über diesen kann der Benutzer (oder der Caller) entscheiden, ob die `gd`-Library verwendet werden soll. Als finale `graphviz.nix` erhalten wir deshalb:
```nix
{ mkDerivation, lib, gdSupport ? true, gd, pkg-config }:

mkDerivation {
  name = "graphviz";
  src = ./graphviz-2.49.3.tar.gz;
  buildInputs =
    if gdSupport
      then [
          pkg-config
          (lib.getLib gd)
          (lib.getDev gd)
        ]
      else [];
}
```

Bei Graphviz mit und ohne GD-Support handelt es sich um verschiedene Programme. In Nix werden die Pakete entsprechend als verschieden betrachtet. Daraus ergibt sich also ein drittes Paket in unserem Repository (`default.nix`):
```nix
let
  pkgs = import <nixpkgs> {};
  mkDerivation = import ./autotools.nix pkgs;
in with pkgs; {
  hello = import ./hello.nix { inherit mkDerivation; };
  graphviz = import ./graphviz.nix { inherit mkDerivation lib gd pkg-config; };
  graphvizCore = import ./graphviz.nix {
    inherit mkDerivation lib gd pkg-config;
    gdSupport = false;
  };
}
```

Die Lösung für das dritte Problem ist sehr ähnlich. wie wir `gdSupport = false` hinzugefügt haben, könnten wir `gd = <Version>` hinzufügen. Da Pakete mit verschiedenen Dependency-Versionen von Nix als verschieden betrachtet werden, würde jede Version in einem eigenen Paket resultieren (ähnlich wie die vorausgegangene Unterscheidung `graphviz` und `graphvizCore` nötig machte).

## Offene Fragen
- Warum brauchen wir zwei `gd`-Komponenten, Lib und Dev?

## Fußnoten
[^fehler]: Müsste es nicht `for p in $buildInputs; do` heißen? Kann man in Shell-Skripten über verschiedene Objekte in derselben Schleife iterieren lassen?
[^split-outputs]: Es werden Library- und Development-Outputs hinzugefügt. Das wird damit begründet, dass `gd` ein Paket mit ["split outputs"](https://nixos.org/manual/nixpkgs/stable/#sec-multiple-outputs-){: target="_blank"} ist. Damit ist vermutlich gemeint, dass das Paket mehrere Outputs hat. Was es aber mit der Unterscheidung zwischen Lib und Dev genau auf sich hat und warum auch die Dev-Komponente benötigt wird, bleibt im Dunkeln.
[^default]: Der Name `default.nix` hat im Nix-Ökosystem eine besondere Bedeutung. Viele Befehle fallen auf eine so benannte Datei zurück, wenn nicht audrücklich ein Pfad übergeben wird.
[^autotools]: In der `autotools.nix` hatten wir bereits einen solchen Parameter, der ein (beliebiges) Paket-Repository als Argument entgegennimmt: `pkgs: attrs:`.
[^inherit]: In der `default.nix` wird das `inherit`-Schlüsselwort verwendet. Die Syntax ist irreführend. Bei seiner Interpretation ist zu berücksichtigen, dass das der Attributmenge vorangestellte `with pkgs; <Attributmenge>` einen Einfluss auf seine Bedeutung hat. `... with pkgs; { inherit mkDerivation lib ... }` ist äquivalent zu `{mkDerivation = pkgs.mkDerivation; lib = pkgs.lib; ... }`.
