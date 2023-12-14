---
layout: post
title:  "Inria Tutorium (4)"
tags: nix nixos
---

# Nix-Tutorium vom Nationalen Forschungsinstitut für Informatik und Automatisierung (INRIA): Teil 4
Im vierten Artikel, [Experiment Packaging: Don’t Repeat Yourself](https://nix-tutorial.gitlabpages.inria.fr/nix-tutorial/experiment-packaging.html){:target="_blank"}, wird ein Mini-Repository mit Paketen aufgebaut. Die Sammlung enthält die Dinge, die von verschiedenen Experimenten gebraucht werden. So lassen sich unnötige Wiederholungen vermeiden.

Das Repo, das auf GitHub oder einer ähnlichen Seite gehostet werden kann, enthält primär zwei Dinge. Zunächst wird ein Einstiegspunkt (*entry point*) ins Repo in Form einer `default.nix`-Datei festgelegt. In ihr werden alle Pakete *aufgerufen* (*call*). Zur Erinnerung: Pakete sind Derivations und Derivations sind Funktionen. Damit sind sie etwas Aufrufbares.

Zum anderen umfasst das Repo ein Verzeichnis (`pkgs`), in dem Unterverzeichnisse für die einzelnen Pakete angelegt werden. In diesen Unterverzeichnissen findet sich für jedes Paket eine Datei, in der es definiert wird. Dazu wird erneut eine `default.nix` angelegt (beispielsweise `pkgs/chord/default.nix`).

Das als Beispiel verwendete Repo findet sich [hier](https://gitlab.inria.fr/nix-tutorial/packages-repository){:target="_blank"}.

## Der Einstiegspunkt in ein Paket-Repository
Der Beitrag folgt einem bestimmten Design-Muster (*design pattern*), bei dem eine `default.nix` im Repo-Wurzelverzeichnis als Einstiegspunkt dient. Hier das im Beitrag vorgestellte Beispiel:
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
    chord = callPackage ./pkgs/chord {};
    chord_custom_sg = callPackage ./pkgs/chord { simgrid = custom_simgrid; };
    custom_simgrid = callPackage ./pkgs/simgrid/custom.nix {};

    inherit pkgs; # similar to `pkgs = pkgs;` This lets callers use the nixpkgs version defined in this file.
  };
in
  packages
```

Wie zuvor werden wieder verschiedene Komponenten in einer Attributmenge zusammengefasst. Entscheidend ist offensichtlich die `callPackage`-Funktion. Leider erfahren wir darüber überraschend wenig. Es wird lediglich gesagt, dass sie den Aufruf von Paketen vereinfacht (*simplifies the way to call a derivation*). An einer späteren Stelle lernen wir, dass sie den Aufruf vereinfacht, weil sie scheinbar automatisch die Attribute übergibt, die die Derivation erwartet. Welche Attribute das sind, lassen sich an der Funktionssignatur ablesen.

Interessant ist, dass `pkgs` in die definierte Attributmenge aufgenommen wird. Wie der Kommentar erklärt, ist `inherit pkgs` äquivalent zu `pkgs = pkgs`, wobei der Wert auf der rechten Seite vom Input kommt. Doch was genau ist der Effekt davon? "This lets callers use the nixpkgs version defined in this file." In dieser Datei wird eine Funktion definiert, es sind also wohl die Callers dieser Funktion gemeint. Heißt das, man könnte über das "kleine" Repo auch auf alle Pakete von Nixpkgs zugreifen?

Bisher haben wir Derivations *gebaut*. Was passiert bei ihrem *Aufruf*? Dazu müssten wir wahrscheinlich wissen, was genau `mkDerivation` zurückgibt. Für diese Hintergründe wird auf den [Nix Pills-Beitrag](https://nixos.org/guides/nix-pills/callpackage-design-pattern.html){:target="_blank"} verwiesen. Werde ich unbedingt lesen, sobald ich mit dieser Serie fertig bin!

Okay, ein bisschen was über `callPackage` erfahren wir doch. Es wird gesagt, dass die Funktion zwei Argumente verlangt: einen Pfad zu einer Nix-Datei oder einem Verzeichnis mit einer `default.nix`; und eine Attributmenge, auf die zurückgegriffen werden kann, wenn man die Inputs des aufgerufenen Pakets überschreiben möchte. Ich weiß nicht recht, was das bedeutet oder warum ich das tun wollen würde. Erneut wird auf die [Nix-Pills](https://nixos.org/guides/nix-pills/override-design-pattern.html){:target="_blank"} verwiesen. Wird soweit ich sehe im Beispiel an keiner Stelle verwendet. Das heißt es wird immer die leere Menge als zweites Argument übergeben.

Wie oben gesagt, werden die Derivations in `default.nix`-Dateien im jeweiligen Paket-Unterverzeichnis definiert. Vorgestellt wird `pkgs/chord/default.nix`:
```nix
{ stdenv, fetchgit, cmake, simgrid, boost }:

stdenv.mkDerivation rec {
  pname = "chord";
  version = "0.1.0";

  src = fetchgit {
    url = "https://gitlab.inria.fr/nix-tutorial/chord-tuto-nix-2022";
    rev = "069d2a5bfa4c4024063c25551d5201aeaf921cb3";
    sha256 = "sha256-MlqJOoMSRuYeG+jl8DFgcNnpEyeRgDCK2JlN9pOqBWA=";
  };

  buildInputs = [
    cmake
    simgrid
    boost
  ];
}
```
Die Definitionen unterscheiden sich vor allem darin vom vorausgegangenen Beispiel, dass nicht mehr das gesamte `pkgs` Teil der Inputs ist. Stattdessen werden präziser genau die Inputs bestimmt, die benötigt werden.

Es wird betont, dass die Inputs keinen Default-Wert erhalten (wie es zuvor bei `pkgs` mit dem Fallback-Snapshot getan wurde). Wie "gewöhnliche" Funktionen erhalten sie ihren Wert durch die Argumente, die beim Aufruf übergeben werden. Also durch die Argumente, die `callPackage` beim Aufruf in der `default.nix` im Wurzelverzeichnis bereitstellt.

Wie sieht es aus, wenn Pakekte mit `nix-build` gebaut werden, welche Pakete erhalten die Inputs dann? Dadurch würde vielelicht klarer werden, was der Unterschied zwischen *Pakten bauen* und *Paketen aufrufen* ist. Wird hier aber leider nicht weiter thematisiert.

Paketmerkmale werden als Attributmengen dargestellt. Am Ende des Beitrags wird gezeigt, wie wir vorhandene Pakete unseren Anforderungen entsprechend anpassen können, indem Attributwerte mit eigenen Werten überschrieben werden (mit `overrideAttrs`). Die modifizierten Pakete können wir in unser eigenen Paket-Repo aufnehmen.

Hier das Beispiel einer eigenen SimGrid-Variante:
```nix
{ simgrid, fetchFromGitLab } :

simgrid.overrideAttrs(oldAttrs: rec {
  version = oldAttrs.version + "-custom";
  src = fetchFromGitLab {
    domain = "framagit.org";
    owner = "simgrid";
    repo = "simgrid";
    rev = "fbd3494dc9a7b377cccbc749586313d0f75c15cd";
    sha256 = "sha256-qr/ocxlxMw/UXKAkr1puirW6sttwvmjrE1pH/PIAJF4=";
  };
})
```

## Verwendung der eigenen Paket-Sammlung
Ein eigenes Paket-Repo kann in verschiedenen Kontexten praktisch sein. Tatsächlich können wir die darin enthaltenen Pakete direkt bauen:
```bash
nix-build https://gitlab.inria.fr/nix-tutorial/packages-repository/-/archive/master/packages-repository-master.tar.gz -A chord
```
Diese Verwendung ist äquivalent zu derjenigen, in der wir das Repo zunächst klonen und `nix-build` dann den Pfad zum Verzeichnis mit der `default.nix` geben, die als Einstiegspunkt ins Repo festgelegt wurde (also statt der URL im eben zitierten Beispiel).

Für die Zwecke der Beitragsreihe wichtiger ist der Umstand, dass alle Experimente nun das kleine Paket-Repo als Input nehmen können. Als Beispiel wird eine `shell.nix` für das zweite Experiment definiert:
```nix
{
  tinypkgs ? import (fetchTarball {
    url = "https://gitlab.inria.fr/nix-tutorial/packages-repository/-/archive/master/packages-repository-8e43243635cd8f28c7213205b08c12f2ca2ac74d.tar.gz";
    sha256 = "sha256:09l2w3m1z0308zc476ci0bsz5amm536hj1n9xzpwcjm59jxkqpqa";
  }) {}
}:

tinypkgs.pkgs.mkShell rec {
  buildInputs = [
    tinypkgs.chord
  ];
}
```

## Offene Fragen
- Was ist der Unterschied zwischen *Pakete definieren* und *Pakete bauen*?
- Was genau macht `callPackage`? Was geben Derivations (als Funktionen) zurück, wenn sie aufgerufen werden?
- Welche Werte werden von `callPackage` übergeben? Woher weiß die Funktion, mit welchen Werten ein Paket aufgerufen werden soll?
- Warum wird `pkgs` in die Attributmenge aufgenommen, die im `default.nix`-Einstiegspunkt definiert wird?
