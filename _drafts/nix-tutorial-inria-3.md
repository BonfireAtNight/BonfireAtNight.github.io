---
layout: post
title:  "Inria Tutorium (3)"
tags: nix nixos
---

# Nix-Tutorium vom Nationalen Forschungsinstitut für Informatik und Automatisierung (INRIA): Teil 3
Im dritten Beitrag, [Packaging Your First Experiment](https://nix-tutorial.gitlabpages.inria.fr/nix-tutorial/first-experiment.html){:target="_blank"}, geht es um einen wissenschaftlichen Verwendungszweck von Nix. Wir lernen wie Experimente so Paketen zusammengefasst werden können, dass sie perfekt wiederholt werden können. Die Einzelheiten dieses Vorgehens sind auch für angehende Nix-Enthusiasten von Interesse, die keine wissenschaftlichen Versuche durchführen werden.

Im Einzelnen lernen wir:
- Wie man Paketdefinitionen (für `nix-build`) und Umgebungsdefinitionen (für `nix-shell`) in *einer* Datei zusammenführt
- Wie man Skripte innerhalb einer bestimmten Umgebung ausführt, in der Anwendungen verfügbar sind, die wir selbst zu Paketen zusammengefasst haben

Auch auf das Zusammenspiel von Nix und Docker wird eingegangen. Da ich vorerst kein primäres Interesse an Docker habe, werde ich auf diese Aspekte im Folgenden ausblenden.

## Warum Paketdefinitionen und Umgebungsdefinitionen zusammenführen?
Im vorausgegangenen Beitrag wurde eine Datei geschrieben, die eine Derivation enthielt, die mit `nix-build` gebaut wurde. In einer *anderen* aber strukturell völlig analogen Datei (`shell.nix`) wurde eine Entwicklungsumgebung beschrieben. Es wird nicht darauf eingegangen, warum diese Situation suboptimal ist. Vielleicht ein paar Punkte dazu.

Zunächst sicherlich, weil es ein Grundprinzip der Programmierung verletzt: Don't repeat yourself. Die beiden Dateien enthalten eine Menge gleichen Code. Durch das Refactoring aus Beitrag 3 lässt sich einiger Boilerplate vermeiden.

Darüber hinaus gibt es einen inhaltlichen Zusammenhang zwischen den beiden Komponenten. Die Umgebung wird für den Zweck aufgebaut, das Paket weiterzuentwickeln oder Fehler daran zu beheben. Die Beziehung wird dadurch verdeutlicht, dass man sie in einer Datenstruktur zusammenbringt.

Die Datei selbst, `default.nix`, spielt eine besondere Rolle im Nix-Ökosystem. In einer Note ganz am Ende vom zweiten Beitrag wird erklärt, dass `nix-shell` nach einer solchen Datei sucht, falls nicht explizit eine Datei angegeben wird (und falls nicht vorher eine `shell.nix` gefunden wird).

## Das Beispiel
Das Experiment-Verzeichnis enthält eine Reihe von Dateien:
- Eine `README.md`, die den Versuch beschreibt und erklärt, wie man ihn ausführt
- Zwei SimGrid-Dateien (`cluster_backbone.xml` und `s4u-dht-chord_d.xml`)
- Ein Skript zum Ausführen des Versuchs (`runner.sh` bzw `runner_shebang.sh`)
- Die bereits erwähnte `default.nix`-Datei

Von Interesse ist für uns nur die letzte Datei. Hier ihr Inhalt:
```nix
{
  pkgs ? import (fetchTarball {
    url = "https://github.com/NixOS/nixpkgs/archive/4fe8d07066f6ea82cda2b0c9ae7aee59b2d241b3.tar.gz";
    sha256 = "sha256:06jzngg5jm1f81sc4xfskvvgjy5bblz51xpl788mnps1wrkykfhp";
  }) {}
}:

with pkgs;

let
  packages = rec {

    # The derivation for chord
    chord = stdenv.mkDerivation rec {
      pname = "chord";
      version = "0.0.in-tuto";
      src = fetchgit {
        url = "https://gitlab.inria.fr/nix-tutorial/chord-tuto-nix-2022";
        rev = "069d2a5bfa4c4024063c25551d5201aeaf921cb3";
        sha256 = "sha256-MlqJOoMSRuYeG+jl8DFgcNnpEyeRgDCK2JlN9pOqBWA=";
      };

      buildInputs = [
        pkgconfig
        simgrid
        boost
        cmake
      ];

    };

    # The shell of our experiment runtime environment
    expEnv = mkShell rec {
      name = "exp01Env";
      buildInputs = [
        chord
      ];
    };

  };
in
  packages
```

Der Unterschied zum vorausgegangenen Ansatz besteht vor allem darin, dass die Datei eine Attributmenge repräsentiert und dass nun über Attribute auf die Derivation und die Shell-Umgebung zugegriffen werden kann. Da `chord` zu den `buildInputs` von `expEnv` gehört, muss es sich um ein *rekursives* Set handeln. Nur so kann innerhalb der Attributmenge auf seine Attribute zugegriffen werden.

Durch das Beispiel frage ich mich nun auch: Sind die Namensbindungen, die in `let` definiert werden, nur für den *einen* Ausdruck gültig, der auf `in` folgt? Mutmaßlich ja. Eine Nix-Datei enthält auch nur einen Ausdruck, geht also gar nicht anders. Oder?

Zuvor hatten wir eine Datei für `nix-build` und eine Datei für `nix-shell`. Nun können beide Werkzeuge mit *einer* Datei interagieren. "However, we *should* tell the commands on which attribute they should work within the set, thanks to the `--attr` (`-A`) command-line option." (meine Hervorhebung) Sollten wir das oder *müssen* wir das? So jedenfalls sollten die entsprechenden Befehle aussehen:
```bash
nix-build default.nix -A chord
nix-shell default.nix -A expEnv
```

## Experimente ausführen
`runner.sh` ist ein Shell-Skript, dessen Ausführung das Experiment repräsentiert. Dabei wird der `chord`-Befehl auf festgelegte Inputs angewendet.
```bash
#!/usr/bin/env bash
chord cluster_backbone.xml s4u-dht-chord_d.xml
```

Der `chord`-Befehl ist nicht ohne Weiteres verfügbar. Es stellt sich deshalb die Frage, wie das Skript ausgeführt werden kann. Dafür werden nicht weniger als vier Möglichkeiten erläutert:
- Das Skript kann innerhalb der definierten Shell-Umgebung manuell ausgeführt werden.
```bash
nix-shell default.nix -A expEnv
./runner.sh
```
- Das Skript kann beim Betreten der Shell-Umgebung automatisch ausgeführt werden. Dazu muss der Menge, die `mkShell` als Argument übergeben wird, ein weiteres Argument hinzugefügt werden:
```nix
shellHook = "./runner.sh";
```
- Das Skript kann manuell innerhalb der Shell-Umgebung ausgeführt werden, ohne sie zu betreten (mit `--command`).
```bash
nix-shell default.nix -A expEnv --command ./runner.sh
```
- Skripte sind ausführbare Dateien. In der UNIX-Welt wird durch eine sogenannte Shebang-Zeichenfolge zu Beginn der Skriptdatei festgelegt werden, welchem Programm die Datei zur Ausführung übergeben wurde. In der obigen Datei, `runner.sh`, wurde Bash zur Ausführung festgelegt (`#!/usr/bin/env bash`). Auch `nix-shell` kann als Interpreter festgelegt werden. Dabei kann sogar eine bestimmte Shell-Umgebung bestimmt werden, innerhalb von der das Skript ausgeführt werden soll.[^shebang] Hier das abgewandelte Skript (`runner_shebang.sh`):
```bash
#!/usr/bin/env nix-shell
#!nix-shell default.nix -A expEnv -i bash
chord cluster_backbone.xml s4u-dht-chord_d.xml
```

## Offene Fragen
- Wie nennt man den Datentyp, den eine `shell.nix`-Datei repräsentiert?
- Welche primäre Rolle spielt die Datei `default.nix` in einem Projekt-/Paketverzeichnis?

## Fußnoten
[^shebang]: Es wird darauf hingewiesen, dass dieser Ansatz weniger flexibel ist. Die Shebang-Zeile ist notwenig relativ zum Speicherort der Skriptdatei.
