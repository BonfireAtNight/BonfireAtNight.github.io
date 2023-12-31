---
layout: post
title:  "Nix Pills (10)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 10 (Kommentar)
<div class="hide-excerpt">
Im zehnten Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/developing-with-nix-shell" target="_blank">Developing with nix-shell</a>, wird in den Gebrauch von Shell-Umgebungen eingeführt, die mit Nix definiert wurden.
</div>
<!--more-->

Im zehnten Kapitel der Nix Pills, [Developing with nix-shell](https://nixos.org/guides/nix-pills/developing-with-nix-shell){: target="_blank"}, wird in den Gebrauch von Shell-Umgebungen eingeführt, die mit Nix definiert wurden. Dabei geht es um Umgebungen, in dnene Builds durchgeführt werden.

## Der Zweck von `nix-shell`
`nix-shell` ist ein Werkzeug, mit dem eine Shell-Umgebung betreten werden kann. Wenn eine Derivation als Argument übergeben wird (beispielsweise `nix-shell hello.nix`), dann sind darin Umgebungsvariablen für die Attribute der Input-Menge der `derivation`-Funktion gesetzt. In vielerlei Hinsicht gleicht die Shell-Umgebung also der Build-Umgebung, in der `nix-build` ausgeführt wird.

Der Beitrag geht auf einige bemerkenswerte Aspekte der Shell-Umgebungen ein:
- Die Build-Inputs sind *nicht* ohne Weiteres verfügbar. Symbole wie `make`, `tar` etc. sind nur verfügbar, wenn sie dem PATH hinzugefügt wurden. Die Umgebungsvariablen demgegenüber *sind* verfügbar.
- Der Builder wird *nicht* automatisch ausgeführt. Zwar ist die `$builder`-Umgebungsvariable gesetzt, doch sie repärsentiert Bash; das Builder-Skript ist ein *Argument* für Bash. Das Builder-Skript kann aber per Hand ge-source-ed werden. Durch die darin enthaltene `for`-Schleife werden auch die PATH-Komponenten hinzugefügt.
- In der Shell-Umgebung befinden wir uns im Arbeitsverzeichnis, nicht (wie bei `nix-build`) in einem temporären Verzeichnis.

Die folgende Befehlsfolge ist deshalb sehr ähnlich wie die Ausführung von `nix-build`:
```
$ nix-shell hello.nix
[nix-shell]$ source builder.sh
```
Dadurch wird die PATH-Variable gefüllt, durch Entpacken der Quelldatei wird das `hello-2.10`-Verzeichnis (im Arbeitsverzeichnis) erstellt, es wird in das Verzeichnis gewechselt und es wird `make` ausgeführt. Ob die Installation gelingt (`make install`) hängt davon ab, ob der aktive Benutzer Schreibrechtee für den Nix-Store hat.

## Ein Builder für Nix-Shell
Durch das bisherige Vorgehen wird im Wesentlichen das Verhalten von `nix-build` rekonstruiert. In einem Schritt wird eine sehr ähnliche Umgebung betreten; in einem nächsten Schritt wird der Build-Vorgang (auf einen Schlag) in die Wege geleitet. Das Vorgehen kann verbessert werden, um sinnvolleren Gebrauch von Nix-Shell zu machen. Die Nix Pills identifizieren zwei Probleme.

Beim `source` wird eine Builder-Datei im Arbeitsverzeichnis importiert. Das gelingt vermutlich nur deshalb, weil der Autor das Verzeichnis entsprechend vorbereitet hat. Das ist natürlich kein sehr systematisches Vorgehen und wäre besser, wenn die im Nix-Store lokalisierte Builder-Datei verwendet würde. Dazu wird der `autotools.nix` im Folgenden ein weiteres Attribut hinzugefügt.

Beim `source` wird der *gesamte* Build-Vorgang in die Wege geleitet. Wenn wir uns die Mühe machen, extra eine Shell-Umgebung für den Build zu betreten, dann wollen wir mutmaßlich einzelne *Phasen* schrittweise durchführen. Dazu werden im Folgenden einzelne Funktionen für jede Phase geschrieben.

Das Projekte wird modular so aufgebaut, dass zwischen Shell-Umgebung, Build-Umgebung und den einzelnen Build-Schritten unterschieden wird. 

## Ein Setup-Skript enthält aufrufbare Build-Komponenten
`setup.sh` ist ein Skript, das Definitionen für eine Reihe von Funktionen enthält. Die meisten dieser Funktionen repräsentieren jeweils eine Build-Phase. Eine letzte Funktion, `genericBuild`, ruft diese Funktionen in der richtigen Reihenfolge aus und repräsentiert damit den Build-Vorgang. Das einzige, was dieses Skript tatsächlich *macht* (statt nur definiert) ist das Setzen der PATH-Variable.
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

Wenn wir ohne einen automatischeen Builder arbeiten wollen, können wir *diese* Datei in der Shell-Umgebung importieren:
```
$ nix-shell hello.nix
[nix-shell]$ source $setup
[nix-shell]$
```
Nun könnten wir auf einen Schlag den gesamten Build-Vorgang einleiten (`genericBuild`). Oder wir könnten einzelne Phasen des Build-Vorgangs individuell triggern (`unpackPhase`). Oder wir könnten (da die PATH-Variable gesetzt ist) direkt die Programme wie `make` etc. ausführen.

Damit Dateien aus dem Nix-Store in der Nix-Shell ge-source-ed  werden können, wird der Datei mit der Input-Menge für die `derivation`-Funktion (`autotools.nix`) ein weiteres Attribut hinzugefügt.

## Ein Builder aus dem Nix-Store
Wenn wir für den Build wie bisher einen automatischen Builder verwenden wollen, können wir dazu einen speziell zur Verwendung in einer Nix-Shell schreiben. Natürlich sollte dieser Gebrauch der `setup.sh` machen. Damit ist eigentlich das wichtigste bereits zusammen:
```bash
# builder.sh
set -e
source $setup
genericBuild
```
Durch die zweite Zeile wurde die PATH-Variable gesetzt und die Funktionen verfügbar gemacht. Durch die dritte Zeile werden die Funktionen in der richtigen Reihenfolge aufgerufen; dieser Ablauf repräsentiert den generischen Build.[^set-e]

Wir können nun die Nix-Shell betreten und in einem Schritt den Build-Vorgang einleiten. Ganz so, als würden wir `nix-build` verwenden. Auf diesen Aspekt wird im Beitrag soweit ich sehe überraschenderweise nicht weiter eingegangen. Wenn sich die `builder.sh` (wie zu Beginn des Beitrags) im Arbeitsverzeichnis befindet, so könnten wir den Build völlig wie zuvor ausführen:
```
$ nix-shell hello.nix
[nix-shell]$ source builder.sh
```
Um die Builder-Datei aus dem Store zu verwenden, müsste der `autotools.nix` dafür ein Attribut hinzugefügt werden, dass den Pfad zur Store-Datei durch eine Umgebungsvariable verfügbar macht. Das wird im Beitrag nicht gemacht. Der Schritt wäre aber wohl völlig analog dazu, wie wir das Setup-Skript des Store für die Shell-Umgebung im Folgenden verfügbar machen.

Prinzipiell stellt sich mir aber die Frage, warum wir einen Builder innerhalb einer Nix-Shellumgebung ausführen würden. Gäbe es irgendeinen Unterschied zur Ausführung von `nix-build` (außer dass es geringfügig aufwändiger wäre)?

## Skripte aus dem Store verwenden
Oben wurde das Setup-Skript aus dem *Store* aufgerufen: 
```
[nix-shell]$ source $setup
```
Damit die Umgebungsvariable auch wirklich verfügbar ist, muss dazu noch ein Attribut definiert werden. Da es etwas ist, dass für alle Autotools-Projekte gleichermaßen nützlich ist, sollte es Teil der Default-Menge sein. Dementsprechend können wir der `autotools.nix`-Attributmenge ein Element hinzufügen:
```nix
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
Die hier einzig interessante (und neue) Zeile ist die, in der das `setup`-Attribut definiert wird. Wie zuvor führt dies dazu, dass in der daraus erstellten Shell-Umgebung eine `$setup`-Umgebungsvariable gesetzt ist, dessen Wert die `setup.sh` im Nix-Store ist.

## Offene Fragen
- Wenn ein Builder-Skript und eine Derivation-Datei vorliegt, warum sollte man eine Shell-Umgebung betreten, um sie zu bauen? Warum nicht einfach direkt den Builder ausführen?
- Warum genau wäre es "nervig", `set -e` in der `setup.sh` zu setzen?

## Fußnoten
[^set-e]: Durch die erste Zeile wird sichergestellt, dass jeder Fehler zu einem Abbruch des Build-Vorgangs führt. Es wird gesagt, dass diese Einstellung für die `setup.sh` "nervig" wäre. Warum genau?
