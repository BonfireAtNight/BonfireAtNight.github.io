---
layout: post
title:  "Nix Pills (7)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 7 (Kommentar)
<div class="hide-excerpt">
Im siebten Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/working-derivation" target="_blank">Working Derivation</a>, werden funktionsfähige Derivations definiert. Wir lernen, wie sie in .nix-Dateien eingefügt und über die Kommandozeile gebaut werden können.
</div>
<!--more-->

Im siebten Kapitel der Nix Pills, [Working Derivation](https://nixos.org/guides/nix-pills/working-derivation){: target="_blank"}, werden funktionsfähige Derivations definiert. Wir lernen, wie sie in `.nix`-Dateien eingefügt und über die Kommandozeile gebaut werden können.

## Shell-Skripte als Builder
Wenn ein Programm von Hand gebaut wird, dann wird auf der Kommandozeile eine Folge von Befehlen ausgeführt. Um diesen Vorgang zu automatisieren, können sie in einem Shell-Skript zusammengefasst werden. Ein solches Skript kann einem Paket als *Builder* hinzugefügt werden.

In Nix ist die Situation komplizierter als man denken könnte. Das Skript soll von einem bestimmten Interpreter ausgeführt werden. Wenn man (wie im Beispiel) Bash nutzen möchte, dann würde man auf der Kommandozeile `bash builder.sh` ausführen. Um eine Datei automatisch mit einem bestimmten Interpreter auszuführen, wird für gewöhnlich eine Shebang-Zeile eingefügt. Das Problem in Nix ergibt sich daraus, dass wir beim Schreiben vom Skript nicht wissen, *welches* Bash genutzt werden wird. Wie alle Anwendungen findet sich Bash (als Build-Dependency) im Store; aber den Pfad kennen wir nicht ohne Weiteres.

Unser Ziel ist es, den Input der `derivation`-Funktion so zu definieren, dass Bash als Builder genutzt und `builder.sh` als Argument übergeben wird. Daraus ergeben sich zwei Probleme: Wie lässt sich Bash definieren? Und wie können wir die `builder.sh` als Argument übergeben?

## Import von Derivations
Um Bash verwenden zu können, müssen wir eine Derivation verwenden. Nixpkgs ist ein Repository, in dem sich sehr viele Derivations finden. Installierte Kanäle entscheiden dann hinter den Kulissen darüber, *welche* Version der importieren Pakete verwendet werden wird.

Um Nixpkgs-Pakete im Nix-REPL verwenden zu können, können sie importiert werden:
```
nix-repl> :l <nixpkgs>
Added 3950 variables.
nix-repl> "${bash}"
"/nix/store/ihmkc7z2wqk3bbipfnlh0yjrlfkkgnv6-bash-4.2-p45"
```
`<nixpkgs>` ist ein Pfad zum Repository. Die Variablen können genutzt werden, um auf ein bestimmtes Bash im Store zu verweisen:
```
derivation { ...; builder = "${bash}/bin/bash"; ... }
```

Für gewöhnlich werden Derivations in einer `.nix`-Datei, nicht im REPL definiert. In einem früheren Kapitel wurde erklärt, dass mit `import` Dateien geparst und ihr Inhalt ausgewertet werden kann. `<nixpkgs>` zeigt auf eine Datei, die einen Lambdaausdruck enthält. `import <nixpkgs>` evaluiert entsprechend zu einer Funktion. Mit `<nixpkgs> {}` wird dieser Funktion die leere Menge als Argument übergeben.[^zwei-funktionen]
```nix
let
  pkgs = import <nixpkgs> {};
in
  pkgs.stdenv.mkDerivation {
    [...]
    builder = "${pkgs.bash}/bin/bash";
    [...]
}
```
Es wird nicht im Detail erläutert, warum genau der Funktion die leere Menge übergeben wird. Es wird nur gesagt:
> Calling `import <nixpkgs> {}` into a `let`-expression creates the local variable `pkgs` and brings it into scope. This has an effect similar to the `:l <nixpkgs>` we used in nix repl, in that it allows us to easily access derivations such as `bash`, `gcc`, and `coreutils`, but those derivations will have to be explicitly referred to as members of the `pkgs` set (e.g., `pkgs.bash` instead of just `bash`).

Damit ist das erste Problem gelöst. Ein Bash als Builder ist nun verfügbar.

## Argumente an einen Builder übergeben
Das zweite Problem besteht darin, unserem Builder (Bash) das Build-Skript als Argument zu übergeben. Im vorausgegangen Kapitel wurde gesagt, dass für die Menge, die der `derivation`-Funktion als Argument übergeben wird, *optionale* Attribute definiert werden können. Dazu gehört `args`, mit dem Kommandozeilenargumente für den Builder bestimmt werden können.

Im REPL:
```
derivation { [...] builder = "${bash}/bin/bash"; args = [ ./builder.sh ]; [...] }
```

Und in einer `.nix`-Datei:
```nix
let
  pkgs = import <nixpkgs> {};
in
  pkgs.stdenv.mkDerivation {
    [...]
    builder = "${pkgs.bash}/bin/bash";
    args = [ ./simple_builder.sh ];
    [...]
}
```

Bemerkenswert ist der Umstand, dass wir in beiden Fällen einen *Pfad* und keinen String nutzen (`./builder.sh` statt `"./builder.sh"`). Dazu wird gesagt:
> This way, it is parsed as a path, and Nix performs some magic which we will cover later. Try using the string version and you will find that it cannot find builder.sh. This is because it tries to find it relative to the temporary build directory.

## Die Build-Umgebung: ein REPL-Beispiel
Bisher wurde vom Inhalt des Skripts abstrahiert. Das Beispiel ist trivial: es erstellt im Nix-Store eine Datei, die das Wort `foo` enthält. Außerdem sollen während des Build-Vorgangs die Umgebungsvariablen ausgegeben werden. Das hat vermutlich didaktische Gründe; der Autor der Nix Pills möchte über die Werte einige dieser Variablen sprechen.

`builder.sh`:
```bash
declare -xp
echo foo > $out
```
`declare` ist ein in Bash eingebautes Kommando. Wenn der Befehl mit den gegebenen Flags ausgeführt wird, werden alle in der gegebenen Umgebung exportierten Variablen (mit ihren Werten) aufgelistet. Uns wird im Folgenden interessieren, woher das Skript die `out`-Variable kennt (sie kommt von Nix).

Die Derivation wird im REPL erzeugt (mit der `derivation`-Funktion) und mit mit REPL-Besonderheit `:b` gebaut.
```
nix-repl> d = derivation { name = "foo"; builder = "${bash}/bin/bash"; args = [ ./builder.sh ]; system = builtins.currentSystem; }
nix-repl> :b d
[1 built, 0.0 MiB DL]

this derivation produced the following outputs:
  out -> /nix/store/gczb4qrag22harvv693wwnflqy7lx5pb-foo
```
Die Ausführung hat keine Ausgabe. Dass der Build erfolgreich war, erkennen wir daran, dass sich im Anschluss im Store eine `foo` Datei mit dem erwarteten Inhalt (`foo`) befindet. Im Beitrag findet sie sich unter ` /nix/store/w024zci0x1hh1wj6gjq0jagkc1sgrf5r-foo`.

Im Shell-Skript war darüber hinaus ein Befehl, der exportierte Umgebungsvariablen ausgibt. Um uns anzuzeigen, welche Ausgaben der Build-Vorgang hatte, kann bezüglich eines Outputs eine Log-Datei verlangt werden:
```
$ nix-store --read-log /nix/store/gczb4qrag22harvv693wwnflqy7lx5pb-foo
declare -x HOME="/homeless-shelter"
declare -x NIX_BUILD_CORES="4"
declare -x NIX_BUILD_TOP="/tmp/nix-build-foo.drv-0"
declare -x NIX_LOG_FD="2"
declare -x NIX_STORE="/nix/store"
declare -x OLDPWD
declare -x PATH="/path-not-set"
declare -x PWD="/tmp/nix-build-foo.drv-0"
declare -x SHLVL="1"
declare -x TEMP="/tmp/nix-build-foo.drv-0"
declare -x TEMPDIR="/tmp/nix-build-foo.drv-0"
declare -x TMP="/tmp/nix-build-foo.drv-0"
declare -x TMPDIR="/tmp/nix-build-foo.drv-0"
declare -x builder="/nix/store/q1g0rl8zfmz7r371fp5p42p4acmv297d-bash-4.4-p19/bin/bash"
declare -x name="foo"
declare -x out="/nix/store/gczb4qrag22harvv693wwnflqy7lx5pb-foo"
declare -x system="x86_64-linux"
```
Diese Variablen beschreiben die Build-Umgebung, es ist deshalb nicht unintressant, uns ihre Werte im Detail anzugucken. Die Nix Pills geben folgende Erklärungen:

## Die Build-Umgebung: ein C-Beispiel
Für ein etwas realistischeres Beispiel, wird im Beitrag ein Paket für ein (sehr einfaches) C-Programm zusammengestellt. Das Programm selbst ist in `simple.c` beschrieben:
```c
void main() {
  puts("Simple!");
}
```
Die zweite Paketdatei ist ein eigenes Build-Skript (`simple_builder.sh`):
```bash
export PATH="$coreutils/bin:$gcc/bin"
mkdir $out
gcc -o $out/simple $src
```
Hier finden sich neben `out` noch weitere Symbole, für die nicht ohne Weiteres auf der Hand liegt, wo sie eingeführt werden: `gcc` und `src`.

Die Pakete werden nun in einem Paket zusammengefasst, das in der `simple.nix`-Datei definiert wird:
```nix
let
  pkgs = import <nixpkgs> {};
in
  pkgs.stdenv.mkDerivation {
    name = "simple";
    builder = "${pkgs.bash}/bin/bash";
    args = [ ./simple_builder.sh ];
    gcc = pkgs.gcc;
    coreutils = pkgs.coreutils;
    src = ./simple.c;
    system = builtins.currentSystem;
}
```
Um das Paket schließlich zu bauen, wird `nix-build simple.nix` verwendet. Der Build-Output wird im Store erstellt. Es handelt sich um eine ausführbare Datei, die `Simple!` ausgibt. Im Arbeitsverzeichnis wird der Symlnk `result` erstellt, der auf die ausführbare Datei im Store zeigt.

## Offene Fragen
- Mit `import <nixpkgs>` wird der Lambdaausdruck in der referenzierten Datei geparst und ausgewertet. Was genau ist die Bedeutung von `import <nixpkgs> {}`, das heißt was wird damit bezweckt, der denotierten Funktion beim Aufruf die leere Menge zu übergeben?
- Warum wird ein Pfad statt eines Strings verwendet? Was genau sind die hier relavanten Besonderheiten vom einem Pfad?

## Fußnoten
[^zwei-funktionen]: Es wird betont, dass es sich beim Ausdruck um *zwei* Funktionsaufrufe handelt. Dies wird deutlicher, wenn man den Ausdruck mit Klammern paraphrasiert: `(import <nixpkgs>) {}`.
