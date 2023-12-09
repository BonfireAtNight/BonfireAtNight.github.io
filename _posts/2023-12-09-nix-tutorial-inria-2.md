---
layout: post
title:  "Inria Tutorium (2)"
tags: nix nixos
---

# Nix-Tutorium vom Nationalen Forschungsinstitut für Informatik und Automatisierung (INRIA): Teil 2
Nach einer Woche Pause geht es weiter mit Teil 2, [Hacking Your First Package](https://nix-tutorial.gitlabpages.inria.fr/nix-tutorial/first-package.html). Was es dieses Mal zu lernen gibt:
- Wie man ein Paket in der Nix Expression Language definiert
- Wie man ein Paket baut
- Wie man eine Shell betritt, die die Build-Umgebung eines gegebenen Pakets repräsentiert

## Pakete und Derivations
Relativ zu Beginn des Beitrags wird gesagt, dass von Derivations *statt* von Paketen (*packages*) gesprochen werden wird.Könnte man deshalb sagen, dass der Paketbegriff auf verschiedene Weisen konzeptualisiert werden kann und dass Derivations die Konzeption der Nix-Paketverwaltung repräsentieren?

Für mich stellt sich nun die Frage: Gibt es im Nix-Ökosystem eine Idee von Paketen, die nicht als Derivations zu deuten sind? Ich bin mir ziemlich sicher, ich habe in einigen anderen Artikeln und Quellen gelesen, dass sich Pakete *und* Derivations im Nix-Store finden. Vielleicht sind Pakete etwas Abstraktes, das durch Derivations in Nix (also in der Nix Expression Language) *beschrieben* werden? In diesem Sinne können sie gebaut und installiert werden. Vielleicht sind *installierte* Pakete gemeint, wenn gesagt wird, dass sich neben Derivations noch "Pakete" im Store finden?[^resulting-package] 

Der so wichtige Grundbegriff wird nur relativ vage definiert: "A derivation is a function that describes a build process." Funktionsausdrücke haben in Nix die folgende syntaktische Form: `<Parameter>: <Wert>`. Derivations werden nach dem Vorausgesagten durch Ausdrücke der gleichen Form denotiert. Nicht ohne Weiteres klar ist es, wie solche Ausdrücke (oder die damit bezeichneten Derivations) Build-Vorgänge beschreiben. Das im Artikel unmittelbar darauf folgende Beispiel vermittelt aber zumindest ein gutes Grundverständnis. Dazu sofort mehr.

Für eine präzise Definition wird auf das [offizielle Bedienungshandbuch](https://nixos.org/manual/nix/stable/language/derivations.html) verwiesen. Dort wird eine Funktion dargestellt, `derivation`. Doch auch wenn es sich bei Derivations um Funktionen handelt - eine Charakterisierung die ich zumindest noch mit einem Fragezeichen versehen würde - ist nicht unmittelbar klar, welcher Zusammenhang zwischen Derivationen-als-Funktionen und der `derivation`-Funktion besteht.

Das folgende Beispiel nutzt im Funktionskörper eine Funktion namens `mkDerivation`. Es ist davon auszugehen, dass zwischen `derivation` und `mkDerivation` ein Zusammenhang besteht. Jedenfalls scheinen sie bei der Definition von Derivationen-als-Funktionen eine zentrale Rolle zu spielen.

## Nix-Paketdefinitionen: Ein Beispiel
Exemplarisch verpackt wird ein Projekt, das in C++ implementiert wurde und [SimGrid](https://simgrid.frama.io/) sowie die C++-Bibliothek [Boost](https://www.boost.org/) nutzt. Scheinbar wird ein [Chord-verteilter Hash-Algorithmus](https://en.wikipedia.org/wiki/Chord_(peer-to-peer)) simuliert. Ich habe keine Ahnung, was diese Dinge bedeuten. Spielt für die eigentliche Paketdefinition hoffentlich keine allzu große Rolle.

Hier das Beispiel:
```nix
{
  pkgs ? import (fetchTarball {
    url = "https://github.com/NixOS/nixpkgs/archive/4fe8d07066f6ea82cda2b0c9ae7aee59b2d241b3.tar.gz";
    sha256 = "sha256:06jzngg5jm1f81sc4xfskvvgjy5bblz51xpl788mnps1wrkykfhp";
  }) {}
}:
pkgs.stdenv.mkDerivation rec {
  pname = "chord";
  version = "0.1.0";

  src = pkgs.fetchgit {
    url = "https://gitlab.inria.fr/nix-tutorial/chord-tuto-nix-2022";
    rev = "069d2a5bfa4c4024063c25551d5201aeaf921cb3";
    sha256 = "sha256-MlqJOoMSRuYeG+jl8DFgcNnpEyeRgDCK2JlN9pOqBWA=";
  };

  buildInputs = [
    pkgs.simgrid
    pkgs.boost
    pkgs.cmake
  ];

  configurePhase = ''
    cmake .
  '';

  buildPhase = ''
    make
  '';

  installPhase = ''
    mkdir -p $out/bin
    mv chord $out/bin
  '';
}
```

## Der Input: pkgs
Derivations wurden als Funktionen charakterisiert und Funktionsausdrücke (Lambdas) haben die Form `<Parameter>: <Rückgabewert>`. Weniger abstrakt betrachtet haben sie die Form `<Attributmenge>: pkgs.stdenv.mkDerivation rec {...}`.

Um zu gewährleisten, dass `pkgs` zur Verfügung steht, wird der notwendige Funktionsinput vorausgesetzt. `pkgs` repräsentiert das das Nixpkgs-Repository. Wir lernen: "Nixpkgs (...) contains functions to help in building packages and a (big) set of packages." `pkgs.stdenv.mkDerivation` dürfte eine solcher Helper-Funktion sein? 

Wahrscheinlich enthält das Repository genau genommen keine Pakete, sondern *Definitionen* von Paketen (oder Derivations). Also Ausdrücke, die strukturell dem obigen Beispiel ähneln. Richtiger ist es wohl auch zu sagen, dass `pkgs` zu einem gegebenen Zeitpunkt (zum Zeitpunkt der Ausdrucksauswertung?) einen bestimmten *Snapshot* von Nixpkgs repräsentiert. 

Das Beispiel zeigt, dass über Commits auf solche Zeitpunkte verwiesen werden kann. Dies wird genutzt, um einen Repo- oder Branch-Snapshot als Fallback zu bestimmen. Dazu wird mit `pkgs ? ...` ein Defaultwert für das `pkgs`-Attribut festgelegt. Der Snapshot vom unstabilen Nixpkgs-Branch findet sich [hier](https://github.com/NixOS/nixpkgs/tree/4fe8d07066f6ea82cda2b0c9ae7aee59b2d241b3). Es wird betont, dass diese Praxis reproduzierbarer ist als die Verwendung von Kanälen für den Zugriff auf das Nixpkgs-Repo.

Die Details der Input-Definition sind mir nicht völlig klar. Wenn ich das richtig lese, dann ist der Defaultwert für `pkgs` der Rückgabewert der `import`-Funktion. Wie das Bedienungshandbuch [erklärt](https://nixos.org/manual/nix/stable/language/builtins.html#builtins-import) wird `import` ein Pfad zu einer Nix-Datei übergeben, die genau einen Ausdruck enthält. Der Rückgabewert der Funktion ist der Wert des Ausdrucks in der Datei.

Wir brauchen demnach einen Pfad. Das Handbuch erklärt bezüglich [`fetchTarball`](https://nixos.org/manual/nix/stable/language/builtins.html?highlight=fetchtarball#builtins-fetchTarball): "Download the specified URL, unpack it and return the path of the unpacked tree." Ich weiß nicht, was der "path of the unpacked tree" genau ist. Verzeichnisse und Repos sind (wie alle Verzeichnisstrukturen) baumformig aufgebaut. Und im Tutorium heißt es auch: "(...) `pkgs` is the top-level tree of Nixpkgs (...)." Aber ich verstehe dennoch nicht völlig, was hier gesagt wird. Wie dem auch sei, zumindest dürfte der Rückgabewert von `fetchTarball` ein passendes Argument für `import` zu sein.

Das Handbuch nennt zwei Weisen, wie `fetchTarball` verwendet werden kann. Beim einfachen Gebrauch wird lediglich eine URL übergeben:
```nix
with import (fetchTarball https://github.com/NixOS/nixpkgs/archive/nixos-14.12.tar.gz) {};
```
Interessant sind die leeren geschweiften Klammern am Ende des Funktionsaufrufs. Diese finden sich auch im obigen Beispiel. Leider wird nicht darauf eingegangen, was sie bedeuten.

Das Beispiel verwendet eine komplexere Form des Funktionsaufrufs. Über einen Hash-Wert wird geprüft, dass die heruntergeladenen Daten wirklich diejenigen sind, die erwartet wurden:
```nix
pkgs ? import (fetchTarball {
  url = "https://github.com/NixOS/nixpkgs/archive/4fe8d07066f6ea82cda2b0c9ae7aee59b2d241b3.tar.gz";
  sha256 = "sha256:06jzngg5jm1f81sc4xfskvvgjy5bblz51xpl788mnps1wrkykfhp";
}) {}
```

Das sei nicht die "gewöhnliche Methode" um Archivinhalte zu prüfen. "The usual method is to pass `lib.fakeSha256` (...) to fetchTarball and try to build the package. Nix will then return an error indicating the actual sha256 of the tarball." Das wirkt alles ziemlich hacky. Vielleicht erstmal auch nicht so wichtig.

Das wichtigste nochmal zusammengefasst:
- "`pkgs` is the top-level tree of Nixpkgs, which contains functions to help in building packages and a (big) set of packages."
- "[`pkgs`] is an alternative of using a channel, and is much more reproducible as the tarball can be fixed to a specific version as it is done here."
- "The `pkgs` imported here is a snapshot of the *unstable* nixpkgs channel (branch?) on the `4fe8d07066f6ea82cda2b0c9ae7aee59b2d241b3` commit."

## Der Output
Der Funktionskörper enthält nur einen Ausdruck, und zwar einen Funktionsauf von `mkDerivation`. Der Rückgabewert der Funktion ist demnach zugleich der Rückgabewert der Derivation. Oder nicht? Leider lernen wir zu dieser zentralen Funktion nur sehr wenig: "`mkDerivation` takes a *set* as input and expects many *attributes* within it." Der Beitrag geht daraufhin nur auf die vielen Attribute ein. Über den Rückgabewert der Funktion erfahren wir nichts. Einleitend wurde zitiert, dass Derivations Pakete repräsentieren und *Build-Vorgänge* beschreiben. Vor diesem Hintergrund sind die vorgestellten Attribute zumindest im Allgemeinen nachvollziehbar.

An dieser Stelle ist es vielleicht angebracht, nochmal darüber zu sprechen, was Derivations sind. Ich habe definitiv Stellen im Netz gefunden, nach denen die *Attributmengen* (die Argumente für `mkDerivation`) Derivations genannt werden. Im Grunde vielleicht nur Nomenklatur ohne substanzielle Bedeutung. Dennoch würde mich interessieren, was die *richtige* Verwendung vom Wort "Derivations" wäre.

Das sind die erläuterten Attribute:
- `pname` und `version`: Wie die Namen nahelegen handelt es sich um Name und Version des Pakets. Scheinbar wird auf ihrer Grundlage automatisch ein anderes Attribut definiert. Der Wert von `name` sind die miteinander verketteten Werte von `pname` und `version` (und einem Bindestrich dazwischen). Das heißt: `chord` + `0.1.0` = `chord-0.1.0`.
- `src` erwartet als Wert ein Verzeichnis, das den zu bauenden Quellcode enthält. Wahrscheinlich könnte man mit einem Pfad auf der Verzeichnis hinweisen, etwa wenn man etwas lokal entwickelt hat? Es wird gesagt, dass im Beispiel `fetchurl` verwendet würde; tatsächlich wird `pkgs.fetchgit` genutzt. Es ist deshalb wenig überraschend, dass die URL auf ein Git-Repo verlinkt. Scheinbar kann man einen Commit-Hash als Wert für `rev` setzen, um den entsprechenden Repo-Snapshot herunterzuladen. Wie bereits bei der anderen Fetcher-Funktion oben gesehen, kann mit `sha256` die Integrität des heruntergeladenen Inhalts überprüft werden. In jedem Fall ist klar: Der Ordner mit den Quellcode-Dateien kann über Fetcher bereitgestellt werden. Mutmaßlich werden die dabei heruntergeladenen Dateien im Nix-Store abgelegt.
- Beim `buildInputs` Attribut wird eine Liste mit Build-Dependencies angegeben (über ihren `pname`). Zum Bauen des Pakets notwendig sind die bereits oben genannten SimGrid und Boost, aber auch CMake. Sie sind verfügbar im Nixpkgs-Repo.
- Nix unterscheideet beim Build-Vorgang verschiedene Phasen (*phases*). Für weitere Informationen dazu wird wieder auf das [Bedienungshandbuch](https://nixos.org/manual/nixpkgs/stable/#sec-stdenv-phases) verwiesen. Im Beispiel werden drei Phasen mit eigenen Anweisungen überschrieben: `configurePhase`, `buildPhase` und `installPhase`. Es wird darauf hingewiesen, dass das tatsächlich nicht notwendig gewesen wäre und nur dem pädagogischen Zweck dient, die Phasen bzw. ihren genauen Ablauf dadurch für den Leser explizit zu machen. Stattdessen führt Nix eigentlich automatisch einen sachgemäßen Build-Vorgang durch, dessen genauer Ablauf von den Paketen in `buildInputs` abhängt.

## Bauen von Paketen
Derivations können gebaut werden. Dazu kann `nix-build` genutzt werden. Wenn der obige Ausdruck in der Datei `chord_example.nix` gespeichert wurde, dann resultiert `nix-build chord_example.nix` in einem neuen Verzeichnis im Nix-Store mit den gebauten Dateien. Im Build-Verzeichnis wird ein Link angelegt, der auf das Verzeichnis im Store zeigt (`./result`).

In der Build-Umgebung sind nur die Build-Dependencies (spezifiziert als `buildInputs`) verfügbar. Mit `nix-shell` kann eine Shell betreten werden, in der (nur) die `buildInputs` einer angegebenen Derivation verfügbar sind. Um die Shell wieder zu verlassen, kann STRG+D gedrückt oder `exit` ausgeführt werden. Mit `--command` kann ein Befehl in der Shell ausgeführt werden, ohne sie zu betreten:
```bash
nix-shell chord_example.nix --command 'mkdir build && cd build && cmake .. && make'
```

Im Defaultfall werden die gegenwärtig gesetzten Umgebungsvariablen für die neue Shell übernommen. Will man dieses Verhalten unterbinden, kann das `--pure`-Flag gesetzt werden.
```bash
nix-shell --pure chord_example.nix
```
Davon unabhängig werden einige Umgebungsvariablen automatisch gesetzt. Dabei handelt es sich um Variablen, die für die `buildInputs` wichtig sind. CMake beispielsweise erwartet `$CMAKE_LIBRARY_PATH`.

Es wird gesagt, dass `nix-shell` zunächst dafür eingeführt wurde, die Build-Umgebung eines Pakets zu betreten (primär für Debugging-Zwecke). Für Entwickler dürfte ein anderer Verwendungszweck noch interessanter sein. Mit `mkShell` kann eine eigene Umgebung für generelle Zwecke definiert werden. Dafür wird für gewöhnlich eine `shell.nix` angelegt. Hier ein Beispiel:
```nix
# This shell defines a development environment for the Chord project.
{
  pkgs ? import (fetchTarball {
    url = "https://github.com/NixOS/nixpkgs/archive/4fe8d07066f6ea82cda2b0c9ae7aee59b2d241b3.tar.gz";
    sha256 = "sha256:06jzngg5jm1f81sc4xfskvvgjy5bblz51xpl788mnps1wrkykfhp";
  }) {}
}:
pkgs.mkShell rec {
   buildInputs = with pkgs; [
    cmake
    boost
    simgrid

    # debugging tools
    gdb
    valgrind
   ];
}
```

Wie das Beispiel zeigt, fungiert `mkShell` auf analoge Weise zu `mkDerivation`. So würde man wohl auch seinen Editor, Linter, Autoformatter, die GNU Core Utils, und all die anderen Dinge festlegen, die man für eine Entwicklung an dem Projekt benötigt?

Was mich daran nur irritiert ist die Rede von `buildInputs`. Debugging Tools sind keine Build-Inputs, zumindest nicht im obigen Sinne von Build-Dependencies. Oder etwa doch? Ich meine mich zu erinnern, dass Ian Henry in seiner Nix-Serie feststellt, dass Nix nicht strikt zwischen Runtime- und Build-Dependencies unterscheidet.[^build-runtime-dependencies] Aber ich weiß nicht recht, ob dieser Punkt hier wirklich eine Rolle spielt.

## Offene Fragen
- In welchem begrifflichen Zusammenhang stehen Pakete und Derivations?
- Welcher Zusammenhang besteht zwischen den Funktionen `derivation` und `mkDerivation`?
- Was genau ist der Output einer Derivation-Funktion? Was gibt `mkDerivation` zurück?
- Was ist mit "path of the unpacked tree" oder dem "top-level tree of Nixpkgs" genau gemeint?
- Was besagen die leeren geschweiften Klammern beim Aufruf von `import`?
- Hätte `rev` auch mit `fetchurl` statt `fetchgit` funktioniert?
- Was wäre das idiomatische Vorgehen, um seinen Editor und andere zur Entwicklung benötigte Anwendungen in einer `shell.nix` zum Teil der Entwicklungsumgebung zu erklären?

## Fußnoten
[^resulting-package]: Tatsächlich findet sich im Beitrag selbst ein Satz, der das nahelegt: "The resulting package resides in the nix store." Also das Paket, das daraus resultiert, dass eine entsprechende Derivation gebaut wird.
[^build-runtime-dependencies]: Siehe "Does Nix not explicitly distinguish between runtime and build-time dependencies?" in [Beitrag 40](https://ianthehenry.com/posts/how-to-learn-nix/open-questions/) seiner "How to Learn Nix"-Serie.
