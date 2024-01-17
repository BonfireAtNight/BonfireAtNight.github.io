---
layout: post
title:  "Nix Pills (19)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 19 (Kommentar)
<div class="hide-excerpt">
Im neunzehnten Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/fundamentals-of-stdenv" target="_blank">Fundamentals of Stdenv</a>, wird auf eine der wichtigsten Derivations im Nixpkgs-Repository eingegangen. <code class="language-plaintext highlighter-rouge">stdenv</code> stellt viele Tools bereit, die beim Bauen sehr vieler Software-Komponenten benötigt werden.<br><br>

In weniger elaborierter Form haben wir viele dabei involvierte Komponenten bereits in früheren Kapiteln kennengelernt. Dazu gehören vor allem: ein Builder-Skript, über das Pakete gebaut werden können; ein Setup-Skript, mit dessen Hilfe die Build-Umgebung hergestellt wird; und eine <code class="language-plaintext highlighter-rouge">mkDerivation</code>-Funktion, die für die vielen Pakete verwendet werden kann, die mit den Autotools gebaut werden.
</div>
<!--more-->

Im neunzehnten Kapitel der Nix Pills, [Fundamentals of Stdenv](https://nixos.org/guides/nix-pills/fundamentals-of-stdenv){: target="_blank"}, wird auf eine der wichtigsten Derivations im Nixpkgs-Repository eingegangen. `stdenv` stellt viele Tools bereit, die beim Bauen sehr vieler Software-Komponenten benötigt werden.

*Umgebung* ist hier im gleichen Sinne wie zuvor zu verstehen. Es handelt sich um eine Shell-Umgebung, die durch gesetzte Umgebungsvariablen und die darin installierten Build-Werkzeuge definiert ist. Wie zuvor kann dazu `nix-build` verwendet werden oder die Umgebung kann mit `nix-shell` explizit betreten werden.

In weniger elaborierter Form haben wir viele dabei involvierte Komponenten bereits in früheren Kapiteln kennengelernt. Dazu gehören vor allem: ein Builder-Skript, über das Pakete gebaut werden können; ein Setup-Skript, mit dessen Hilfe die Build-Umgebung hergestellt wird; und eine `mkDerivation`-Funktion, die für die vielen Pakete verwendet werden kann, die mit den Autotools gebaut werden.[^make]

## Überblick
Natürlich gibt es einen Zusammenhang zwischen den eben erwähnten Komponenten. `stdenv` ist ein Paket, das die Skriptdatei umfasst. `mkDerivation` ist ein Wrapper über die `derivation`-Funktion, durch die Derivation erstellt werden. Sie definiert ein Attribut für die Skriptdatei, wodurch eine `$skript`-Umgebungsvariable in die Umgebung aufgenommen wird. 

Das Builder-Skript ist ein Fallback-Argument. Es wird verwendet, falls die der `mkDerivation`-Funktion übergebene Menge kein Attribut für das Builder-Argument enthält. Das Builder-Skript macht zwei Dinge: es importiert das Setup-Skript (wodurch  die Build-Umgebung hergestellt wird) und es führt den generischen Build-Vorgang durch.

Das Skript kann auch genutzt werden, um eine Shell-Umgebung zu betreten und selbst zu entscheiden, wie die verschiedenen Phasen des Build-Vorgangs auszuführen sind. Dazu enthält es Funktionen, die die Phasen repräsentieren.

## Was ist `stdenv`?
Damit haben wir zumindest ein gewisses Vorverständnis vom Zweck der `stdenv`-Derivation. Wir können uns den Inhalt des Pakets annähern, wenn wir es bauen.
```
$ nix-build '<nixpkgs>' -A stdenv
/nix/store/k4jklkcag4zq4xkqhkpy156mgfm34ipn-stdenv
$ ls -R result/
result/:
nix-support/  setup

result/nix-support:
propagated-user-env-packages
```
Die Derivation umfasst demnach zwei Dateien: `/setup` and `/nix-support/propagated-user-env-packages`. Auf letztere wird nicht näher eingegangen. Es wird lediglich gesagt, dass es eine leere Datei ist. Wozu dient sie dann?

Uns interessiert die `setup`-Datei. Dabei handelt es sich um ein [Bash-Skript](https://github.com/NixOS/nixpkgs/blob/master/pkgs/stdenv/generic/setup.sh){: target="_blank"}.

## Wiederholung: `builder.sh` und `setup.sh`
Funktional wird die `stdenv`-Setupdatei mit der `builder.sh` verglichen, die in einem früheren Kapitel entwickelt wurde. 
>  Remember our generic `builder.sh` in [Pill 8](https://nixos.org/guides/nix-pills/generic-builders){: target="_blank"}? It sets up a basic `PATH`, unpacks the source and runs the usual autotools commands for us.

Wichtiger ist vielleicht eigentlich das [zehnte Kapitel](https://nixos.org/guides/nix-pills/developing-with-nix-shell){: target="_blank"}. Darin werden Änderungen vorgenommen, die für die Verwendung von `nix-shell` notwendig sind. Darin wird zwischen der `builder.sh` und der `setup.sh` unterschieden. Die `builder.sh` wurde als Argument an Bash (dem eigentlichen Builder) übergeben und im Wesentlichen importierte sie nur die `setup.sh` und führte die (im Setup-Skript definierte) `genericBuild`-Funktion aus.
```bash
# builder.sh
set -e
source $setup
genericBuild
```

Für die `stdenv`-Setupdatei interessanter ist (wenig überraschend) die `setup.sh`. Durch *dieses* Skript wurden Umgebungsvariablen gesetzt (insbesondere `PATH`), Werkzeuge für Autotools-Pakete bereitgestellt, ein Archiv mit Quelldateien entpackt und die üblichen zum Build der heruntergeladenen Software notwendigen Schritte ausgeführt. Dieser Vorgang wurde in Phasen eingeteilt: `unpackPhase`, `configurePhase`, `buildPhase`, `checkPhase`, `installPhase` und `fixupPhase`. 
```bash
# setup.sh
unset PATH
for p in $baseInputs $buildInputs; do
  export PATH=$p/bin${PATH:+:}$PATH
done

function unpackPhase() {
  tar -xzf $src

  for d in *; do
    if [ -d "$d" ]; then
      cd "$d"
      break
    fi
  done
}

function configurePhase() {
  ./configure --prefix=$out
}

function buildPhase() {
  make
}

function installPhase() {
  make install
}

function fixupPhase() {
  find $out -type f -exec patchelf --shrink-rpath '{}' \; -exec strip '{}' \; 2>/dev/null
}

function genericBuild() {
  unpackPhase
  configurePhase
  buildPhase
  installPhase
  fixupPhase
}
```

## Wiederholung: `mkDerivation`
Die Ausdrücke, durch die Derivations erzeugt werden (Aufrufe der `derivaition`-Funktion), wurden so abgewandelt, dass automatisch das Setup-Skript importiert wurde. Das geschah dadurch, dass ein `setup`-Attribut definiert wurde.
```nix
# autotools.nix
pkgs: attrs:
  let defaultAttrs = {
    builder = "${pkgs.bash}/bin/bash";
    args = [ ./builder.sh ];
    setup = ./setup.sh;
    baseInputs = with pkgs; [ gnutar gzip gnumake gcc coreutils gawk gnused gnugrep binutils.bintools patchelf findutils ];
    buildInputs = [];
    system = builtins.currentSystem;
  };
  in
  derivation (defaultAttrs // attrs)
```
Attribute werden in der erzeugten Shell-Umgebung automatisch zu Umgebungsvariablen. Dadurch ist insbesondere gewährleistet, dass der Aufruf des Skripts in der `builder.sh` (`source $setup`) funktioniert.

Dieser in dieser Datei definierte Lambdaausdruck wird aufgerufen, um die Wrapper-Funktion `mkDerivation` zu definieren. In früheren Kapiteln geschah dies in den einzelnen Paketdateien.
```nix
# hello.nix
let
  pkgs = import <nixpkgs> {};
  mkDerivation = import ./autotools.nix pkgs;
in mkDerivation {
    name = "hello";
    src = ./hello-2.12.1.tar.gz;
  }
```

## Die Skriptdatei und `mkDerivation` der `stdenv`
Das Setup-Skript der Nix-Standardumgebung macht im Prinzip die gleichen Dinge, wie die obige `setup.sh` (wenn auch in extrem viel elaborierterer Form). Das heißt, es werden wieder Umgebungsvariablen gesetzt (insbesondere `PATH`) und es werden wichtige Werkzeuge zur Verfügung gestellt, die für zum Build der Autotools-Pakete brauchen. Sie sind (in einem offenbar nicht trivialen Sinne) ins Skript hardcoded:
```
$ head result/setup
export SHELL=/nix/store/zmd4jk4db5lgxb8l93mhkvr3x92g2sx2-bash-4.3-p39/bin/bash
initialPath="/nix/store/a457ywa1haa0sgr9g7a1pgldrg3s798d-coreutils-8.24 ..."
defaultNativeBuildInputs="/nix/store/sgwq15xg00xnm435gjicspm048rqg9y6-patchelf-0.8 ..."
```

Um heruntergeladene Quelldateien in der darin definierten Umgebung und mit den darin definierten Funktionen zu bauen, können wir die Bash-Datei direkt (mit `source`) importieren.[^unset] 
```
$ nix-shell -E 'derivation { name = "fake"; builder = "fake"; system = "x86_64-linux"; }'
nix-shell$ unset PATH
nix-shell$ source /nix/store/k4jklkcag4zq4xkqhkpy156mgfm34ipn-stdenv/setup
nix-shell$ tar -xf hello-2.10.tar.gz
nix-shell$ cd hello-2.10
nix-shell$ configurePhase
...
nix-shell$ buildPhase
...
```

Natürlich wollen wir aber nicht Bash, sondern Nix selbst nutzen, um Pakete mithilfe der Setupdatei zu bauen. Dazu wird eine Wrapper-Funktion definiert, die funktional völlig unserer eigenen `mkDerivation` entspricht. Die Dateien, in denen Derivations (durch Funktionsaufruf) erzeugt werden, sehen deshalb ganz genauso aus wie zuvor.
```nix
with import <nixpkgs> {};
stdenv.mkDerivation {
  name = "hello";
  src = ./hello-2.10.tar.gz;
}
```
Wie zuvor können wir die Datei nun an `nix-build` übergeben, um die entsprechende Derivation zu instanziieren und zu realisieren.
```
$ nix-build hello.nix
...
/nix/store/6y0mzdarm5qxfafvn2zm9nr01d1j0a72-hello
$ result/bin/hello
Hello, world!
```

## Der Builder der `mkDerivation` (Nixpkgs)
Natürlich ist die [Definition](https://github.com/NixOS/nixpkgs/blob/master/pkgs/stdenv/generic/make-derivation.nix){: target="_blank"} der `mkDerivation` im Nixpkgs-Repository weitaus komplexer als unsere Version.

Bei diesem sehr langen Dokument (über 600 Zeilen Code) wird nur ein Aspekt hervorgehoben:
```nix
{
  ...
  builder = attrs.realBuilder or shell;
  args = attrs.args or ["-e" (attrs.builder or ./default-builder.sh)];
  stdenv = result;
  ...
}
```
`default-builder.sh` sieht im Grunde ganz genauso aus wie unsere `builder.sh`, nur natürlich mit der `stdenv`-Setupdatei:
```bash
source $stdenv/setup
genericBuild
```

Ansonsten wird nur noch auf die Umgebungsvariablen eingegangen, die die beim Aufruf von `mkDerivation` erzeugte Build-Umgebung definieren:
```
$ nix derivation show $(nix-instantiate hello.nix)
warning: you did not specify '--add-root'; the result might be removed by the garbage collector
{
  "/nix/store/abwj50lycl0m515yblnrvwyydlhhqvj2-hello.drv": {
    "outputs": {
      "out": {
        "path": "/nix/store/6y0mzdarm5qxfafvn2zm9nr01d1j0a72-hello"
      }
    },
    "inputSrcs": [
      "/nix/store/9krlzvny65gdc8s7kpb6lkx8cd02c25b-default-builder.sh",
      "/nix/store/svc70mmzrlgq42m9acs0prsmci7ksh6h-hello-2.10.tar.gz"
    ],
    "inputDrvs": {
      "/nix/store/hcgwbx42mcxr7ksnv0i1fg7kw6jvxshb-bash-4.4-p19.drv": [
        "out"
      ],
      "/nix/store/sfxh3ybqh97cgl4s59nrpi78kgcc8f3d-stdenv-linux.drv": [
        "out"
      ]
    },
    "platform": "x86_64-linux",
    "builder": "/nix/store/q1g0rl8zfmz7r371fp5p42p4acmv297d-bash-4.4-p19/bin/bash",
    "args": [
      "-e",
      "/nix/store/9krlzvny65gdc8s7kpb6lkx8cd02c25b-default-builder.sh"
    ],
    "env": {
      "buildInputs": "",
      "builder": "/nix/store/q1g0rl8zfmz7r371fp5p42p4acmv297d-bash-4.4-p19/bin/bash",
      "configureFlags": "",
      "depsBuildBuild": "",
      "depsBuildBuildPropagated": "",
      "depsBuildTarget": "",
      "depsBuildTargetPropagated": "",
      "depsHostBuild": "",
      "depsHostBuildPropagated": "",
      "depsTargetTarget": "",
      "depsTargetTargetPropagated": "",
      "name": "hello",
      "nativeBuildInputs": "",
      "out": "/nix/store/6y0mzdarm5qxfafvn2zm9nr01d1j0a72-hello",
      "propagatedBuildInputs": "",
      "propagatedNativeBuildInputs": "",
      "src": "/nix/store/svc70mmzrlgq42m9acs0prsmci7ksh6h-hello-2.10.tar.gz",
      "stdenv": "/nix/store/6kz2vbh98s2r1pfshidkzhiy2s2qdw0a-stdenv-linux",
      "system": "x86_64-linux"
    }
  }
}
```
Neben der individuellen `src` findet sich darin auch der Pfad zur `stdenv`-Derivation.

Bemerkenswerterweise wird auf viele Aspekte nicht eingegangen. Es wird zwischen einem "echten" Builder und einer Shell als Builder unterschieden. Wie genau zwischen `attrs.realBuilder or shell` entschieden wird, wird nicht erklärt. 

`default-builder.sh` scheint eine Art Fallback-Argument zu sein. Natürlich wird es nicht verwendet, wenn der Benutzer `args` expliizit in der Input-Attributmenge definiert. Ehrlich gesagt verstehe ich nicht recht, was es mit dem zusätzlichen `attrs.builder` auf sich hat.

Damit bleiben viele Details im Dunkeln. Für weitere Ausführungen wird erneut auf die [Dokumentation](https://nixos.org/manual/nixpkgs/stable/#chap-stdenv){: target="_blank"} verwiesen.

## Offene Fragen
- Was ist der Zweck von `/nix-support/propagated-user-env-packages`?
- Wie genau sind die beiden obigen Zeilen `  builder = attrs.realBuilder or shell;` und `args = attrs.args or ["-e" (attrs.builder or ./default-builder.sh)];` zu verstehen?

## Fußnoten
[^make]: Daher wohl auch der Name, `mkDerivation` für `make Derivation`. Das soll wahrscheinlich heißen: Derivations, die mit `make` und verwandter Tools gebaut werden.
[^unset]: Der Befehl `unset PATH` dient Demonstrationszwecken. Es soll gezeigt werden, dass die durch die Setupdatei aufgebaute Umgebung in sich geschlossen ist und (dennoch) ausreicht, ein von den Autotools-abhängiges Paket wie GNU Hello zu bauen.
