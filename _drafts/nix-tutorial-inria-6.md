---
layout: post
title:  "Inria Tutorium (5)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix-Tutorium vom Nationalen Forschungsinstitut für Informatik und Automatisierung (INRIA): Teil 6
<div class="hide-excerpt">
Kommentar zum sechsten Beitrag in der Nix-Serie vom inria, <a href="https://nix-tutorial.gitlabpages.inria.fr/nix-tutorial/experiment-packaging.html" target="_blank">Convenient Management of Inputs with Nix Flakes</a>. Darin lernen wir:
<ul>
<li>Wie man Flakes und das Kommandozeilenwerkzeug `nix` aktiviert</li>
</ul>
</div>
<!--more-->
Im letzten inria-Beitrag, [Convenient Management of Inputs with Nix Flakes](https://nix-tutorial.gitlabpages.inria.fr/nix-tutorial/flakes.html){: target="_blank"}, geht es über Flakes. Dabei handelt es sich um ein relativ neues Feature, um den Umgang mit Build-Inputs zu vereinfachen und perfekte Reproduzierbarkeit zu gewährleisten.

Darüber hinaus erfahren wir zunächst sehr wenig bis gar nichts darüber, was Flakes sind. Im Prinzip scheint es sich um Dateien zu handeln, in der Derivations definiert werden. Also ganz so, wie wir es seit dem zweiten Beitrag gemacht haben. Ich denke es handelt sich nicht um etwas wesentlich Neues.[^flakes] Es handelt sich eher um einen weiteren Schritt, um unsere Praxis zu verfeinern.

Im Einzelnen lernen wir:
- Wie man Flakes und das Kommandozeilenwerkzeug `nix` aktiviert

## Flakes aktivieren
Auch wenn Flakes in der Nix-Community mittlerweile eine sehr breite Verbreitung gefunden haben, gelten sie noch immer als experimentelles Feature. Sie müssen ausdrücklich aktiviert werden, um verwendet werden zu können. Auch das Werkzeug, über das sie verwendet werden (der neue `nix`-Befehl) muss freigegeben werden.

Eine Möglichkeit wäre, sie bei Verwendung auf der Kommandozeile freizuschalten. Dazu dient ein spezielles Flag:
```bash
 --experimental-features 'nix-command flakes'
```

Nur die wenigsten werden das alles bei jeder Verwendung tippen wollen.

Der Nix-Paketmanager kann über eine Konfigurationsdatei konfiguriert werden. Für einzelne Benutzer findet sich die Datei in `~/.config/nix/nix.conf`. Daneben gibt es eine Datei, mit der Nix systemweit konfiguriert werden kann (`/etc/nix/nix.conf`). Um die entsprechenden Features zu aktivieren, kann folgende Zeile hinzugefügt werden:
```nix
experimental-features = nix-command flakes
```

Leider wird im Beitrag nicht weiter auf die Konfigurationsdatei eingegangen. Welche anderen Einstellungen können darin vorgenommen werden?

## Flake-Templates
Wir erfahren, dass neue Flakes auf der Grundlage von *Templates* erzeugt werden können. Von Haus aus kommt Nix mit einer Reihe solcher Vorlagen. Diese können wir uns mit einem Befehl anzeigen lassen:
```bash
nix flake show templates
```
Darüber hinaus können selbst neue Templates definiert werden. Der Beitrag sagt jedoch nichts dazu, wie wir das machen würden.

Um ein neues Flake auf der Grundlage einer vorhandenen Vorlage zu erstellen, kann folgender Befehl verwendet werden:
```bash
nix flake init -t templates#<Template-Name> <Verzeichnis>
```

Wenn kein Verzeichnis-Pfad übergeben wird, dann wird das neue Flake im Arbeitsverzeichnis erstellt. Es wird im Beitrag gesagt, dass wir ein Default-Template definieren können und dass standardmäßig `trivial` als Default gesetzt ist. Wenn wir das Flake im Arbeitsverzeichnis und auf der Grundlage der `trivial`-Vorlage erstellen wollen, kann der obige Befehl also verkürzt werden:
```bash
nix flake init -t templates
```

Leider erfahren wir nicht, wie der Default-Wert geändert werden kann. Mutmaßlich könnten wir dazu wieder die Nix-Konfigurationsdatei verwenden?

Tatsächlich ist `-t templates` selbst wiederum ein Default. Bei passenden Einstellungen kann der Befehl zum Erstellen eines Flakes demnach reduziert werden auf:
```bash
nix flake init
```

Flakes müssen notwendig in ein Versionsverwaltungssystem eingecheckt werden. Das bedeutet, dass wir nur dann mittels `nix flake` mit einem Flake interagieren können, wenn die Datei der Git Staging Area hinzugefügt wurde. Es versteht sich, dass das Verzeichnis, in dem sich die Flake-Dateien befinden, ein Git-Repo enthalten muss.
```bash
git init
git add flake.nix
```

## Interaktion mit Flakes
Das `nix`-Werkzeug stelle eine Reihe von Unterbefehlen bereit, um mit Flakes zu interagieren. Wir einleitend vermutet wurde, sind Flakes ein Mechanismus, um Pakete (Derivations) zu definieren. Es gibt einen neuen Befehl, um alle im Flake enthaltenen Pakete aufzulisten:
```bash
nix flake show
```

Bisher haben wir Pakete mit `nix-build` gebaut. Wir erfahren nun, dass wir die in einem Flake enthaltenen Pakete mit dem Unterbefehl `nix build` bauen können. Es wird nicht ausdrücklich auf die Gemeinsamkeiten und Unterschide zwischen `nix-build` und `nix build` eingegangen. Die Syntax ist ein wenig anders:
```bash
nix build <Verzeichnis>#<Paketname>
```
Wenn sich im Arbeitsverzeichnis eine `flake.nix` befindet, die ein Paket namens `hello` enthält, dann können wir das Paket folgendermaßen bauen:
```bash
nix build .#hello
```
Wir erfahren, dass auch durch den neuen Unterbefehl eine Verlinkung erzeugt wird, die auf den Build-Output im Nix-Store zeigt (`./result`).

Viele Pakete umfassen ausführbare Binärdateien. Wenn wir ein Flake gebaut haben, finden sich diese Dateien im Unterverzeichnis, das daraufhin im Nix-Store erstellt wurde. Das neue Kommandozeilenwerkzeug erlaubt es uns, diese Hintergründe zu vergessen. Stattdessen können wir uns vorstellen, wir würden die in Flakes enthaltenen Pakete direkt ausführen.
```bash
nix run <Verzeichnis>#<Paketname>
```
Die Syntax dabei ist völlig analog zur Verwendung von `nix build`

Wir erfahren, dass bereits `nix-build` einen Zugang bot, um die in einem Paket enthaltenen Binärdateien auszuführen. Wie nun oft gesagt result der Build von Paketen in Unterverzeichnissen im Nix-Store, die gegebenenfalls ausführbare Binärdateien enthalten. Der Link `./result` verweist auf dieses Unterverzeichnis. Für gewöhnlich finden sich die Binärdateien im Unterverzeichnis `/bin`.

Das `hello`-Paket hat genau diesen Aufbau. Um es auszuführen, kann somit folgender Befehl verwendet werden:
```bash
./result/bin/hello
```

Das gilt sowohl für `nix-build` wie für `nix build`.

## Der Aufbau eines Flake
Wie zuvor wollen wir eine Datei schreiben, in der ein oder mehr Pakete (Derivations) definiert werden. Bisher haben wir dazu eine *Funktion* definiert. In einer `flake.nix` wird nun eine *Attributmenge* definiert. Für gewöhnlich werden dabei genau drei Attribute festgelegt.[^flake-attribute]

Eines der drei Attribute, `outputs`, enthält als Wert eine Funktion, die stark dem ähnelt, was wir schon kennen. Mit dem `inputs`-Attribut werden Dependencies für unsere Pakete gesetzt. Durch das `description`-Attribut schließlich kann der Zweck unserer Flakes (in Form eines Strings) beschrieben werden.

Hier das im Beitrag diskutierte Flake:
```nix
{
  description = "A very basic flake";

  inputs = {
        nixpkgs.url = "github:nixos/nixpkgs/22.05";
  };

  outputs = { self, nixpkgs }: {

        packages.x86_64-linux.hello = nixpkgs.legacyPackages.x86_64-linux.hello;

        defaultPackage.x86_64-linux = self.packages.x86_64-linux.hello;

  };
}
```

## Inputs
Der Zweck des `inputs`-Attributs liegt darin, die spezifische Version einer Dependency anzupinnen. Im Beispiel wird festgelegt, dass Version `22.05` vom Nixpkgs-Repo als Input verwendet werden soll.

Wie bei vielen Paketverwaltungssystemen werden die spezifischen Dependency-Versionen, die als Inputs verwendet werden, in einer `lock`-Datei dokumentiert. Nix verwendet dazu `flake.lock`. Für das Beispiel-Projekt sieht diese Datei folgendermaßen aus:
```yaml
{
  "nodes": {
        "nixpkgs": {
          "locked": {
                "lastModified": 1653936696,
                "narHash": "sha256-M6bJShji9AIDZ7Kh7CPwPBPb/T7RiVev2PAcOi4fxDQ=",
                "owner": "nixos",
                "repo": "nixpkgs",
                "rev": "ce6aa13369b667ac2542593170993504932eb836",
                "type": "github"
          },
          "original": {
                "owner": "nixos",
                "ref": "22.05",
                "repo": "nixpkgs",
                "type": "github"
          }
        },
        "root": {
          "inputs": {
                "nixpkgs": "nixpkgs"
          }
        }
  },
  "root": "root",
  "version": 7
}
```

Die `flake.nix` scheint automatisch aktualisiert zu werden, wenn wir die Inputs unserer `flake.nix` anpassen. Leider wird im Beitrag nicht darauf eingegangen, wann das genau passiert. Wenn `flake.nix` seit unserer letzten Verwendung von `nix flake` verändert wurde, scheint eine erneute Verwendung des Unterbefehls zu einer Anpassung der Lock-Datei zu führen. Wir erhalten folgende Warnung:
```bash
nix flake show
warning: updating lock file '/tmp/tuto-nix/flake.lock':
```

## Inputs aktualisieren
Wir erfahren, dass Inputs *aktualisiert* werden können. Wir können entweder alle Inputs zugleich aktualisieren oder einzelne Inputs.
```bash
nix flake update
nix flake lock --update-input <Input-Name>
```

Leider wird nicht ausgeführt, was das genau bedeutet. Mutmaßlich werden daraufhin neuere Version der Dependencies verwendet, wenn wir die als Output definierten Derivations bauen oder ausführen.

Auf welcher Grundlage entscheidet Nix, ob eine neuere Version vorhanden ist? Wenn ich das richtig verstehe, dann *ersetzen* Flakes das Konzept von Kanälen. Ich denke die gesetzten Kanäle entscheiden deshalb nicht über den Ablauf der Aktualisierung.

Es stellt sich auch die Frage, was das Resultat einer Aktualisierung ist? Wird die `flake.nix` oder die `flake.lock` angepasst? Oder vielleicht beide?

## Outputs
Über das `outputs`-Attribut erfahren wir Folgendes:
> The `outputs` field is a function taking as input the inputs and returning a set. This set should have a specific hierarchy. First the type of output (...), then the target architecture (...) and finally the name of the output.

Der Rückgabewert der Output-Funktion ist demnach eine Menge. Im Nix-Ökosystem heißt "Menge" mutmaßlich Attributmenge. "Outputs" (Plural) legt nahe, dass die Menge auf der obersten Ebene Elemente enthält, die jeweils einen Output repräsentieren?

Jeder Output scheint drei Dinge zu haben:
- Sie haben einen Output-Typ. Daraus folgt, dass in Flakes nicht nur Pakete definiert werden können. Wir erfahren von drei möglichen Typen: `packages`, `devShells` und `checks`. Die beiden ersten kennen wir bereits. Was sind `checks`? Und welche weiteren Typen gibt es?
- Sie haben eine Ziel-Architektur. Anwendungen werden für bestimmte CPU-Architekturen gebaut. Dazu gehören beispielsweise `x86_64-linux`, `aarch64-linux` und `x86_64-darwin`.
- Outputs erhalten einen Namen. Wir haben oben bereits gesehen, dass sie darüber von Außen referenziert werden (wie bei `nix build` und `nix run`).

Soweit so gut. Leider scheinen sich diese Ausführungen nicht ohne Weiteres auf das gegebene Beispiel übertragen. Ich sehe, dass die Architektur über das `system`-Attribut definiert wird. Output-Name und -Typ werden weniger offensichtlich durch Attribute repräsentiert.

Wie gesagt gibt die Funktion eine Attributmenge zurück. Wenn ich das Beispiel richtig deute, dann handelt es sich dabei um diese rekursive Menge
```nix
rec {
    chord = pkgs.callPackage ./pkgs/chord {};
    chord_custom_sg = pkgs.callPackage ./pkgs/chord { simgrid = custom_simgrid; };
    custom_simgrid = pkgs.callPackage ./pkgs/simgrid/custom.nix {};
};
```

Wie oben vermutet, scheint diese Menge Elemente zu enthalten, die (in unserem Fall) jeweils ein Paket repräsentieren. Der Paketname scheint der Attributname zu sein.

Der Wert der Pakete ergibt sich daraus, dass (mit `calPackage`) Derivations aufgerufen werden, die an *anderer* Stelle definiert wurden. Dieses Design-Pattern kennen wir prinzipiell bereits aus einem früheren Beitrag.

Dort wurde jedoch ein Mini-Paketrepo verwendet, dass wir (oder die inria-Mitwirkenden) selbst definiert haben. Hier scheinen Derivation-Definitionen aus dem Nixpkgs-Repo verwendet zu werden. `pkgs` wird folgendermaßen definiert:
```nix
pkgs = import nixpkgs { inherit system; };
```

Wir scheinen nun drei Quellen zu haben: `inputs` zum Flake, Attribute zur `outputs`-Funktion und Importe (deren Rückgabe im `let`-Teil einer Variable zugewiesen werden kann). Leider wird nicht erläutert, unter welchen Umständen wir auf welche dieser Komponenten zugreifen würden.

Die eben definierte rekursive Attributmenge wird als Wert für das `packages`-Attribut festgelegt. Daraus dürfte folgen, dass es sich bei den Elementen der Menge um Pakete handelt (Typ der Outputs). Tatsächlich finden wir hier auch die im obigen Zitat angedeutet Hierarchie:
```nix
packages.${system} = rec {
    chord = pkgs.callPackage ./pkgs/chord {};
    chord_custom_sg = pkgs.callPackage ./pkgs/chord { simgrid = custom_simgrid; };
    custom_simgrid = pkgs.callPackage ./pkgs/simgrid/custom.nix {};
};
```
`packages` ist eine Attributmenge, innerhalb derer Pakete definiert werden (Output-Typ). Tatsächlich werden die Pakete der `packages`-Menge aber nicht direkt hinzugefügt. Stattdessen enthält `packages` für eine oder mehr CPU-Architekturen ein Attribut. Jedes Architektur-Attribut enthält als Werte eine Attributmenge, in der Pakete für eben diese Architektur definiert werden. Die Attributname innerhalb dieser Menge zeigen jeweils die Paketnamen an.

Mutmaßlich könnten wir im selben Flake also Pakete für weitere Architekturen definieren. Und wir könnten neben Paketen noch `devShells` oder andere Dinge definieren.
```nix
outputs = { self, nixpkgs }: {
    packages = {
        x86_64-linux {
            paket_1_fuer_architektur_1 = ...;
            paket_2_fuer_architektur_1 = ...;
        }
        x86_64-darwin {
            paket_1_fuer_architektur_2 = ...;
            paket_2_fuer_architektur_2 = ...;
        }
    };
    devShells = ...
};
```

## Abwärtskompatibilität
Es wird darauf hingewiesen, dass `nix-build` nicht ohne Weiteres verwendet werden kann, um Flakes zu bauen. Es gibt jedoch eine Möglichkeit, um Abwärtskompatibilität zum "alten" Ansatz herzustellen.

Dazu muss dem Flake-Verzeichnis eine `default.nix`-Datei mit dem folgenden Inhalt hinzugefügt werden:
```nix
(import (
  fetchTarball {
        url = "https://github.com/edolstra/flake-compat/archive/12c64ca55c1014cdc1b16ed5a804aa8576601ff2.tar.gz";
        sha256 = "0jm6nzb83wa6ai17ly9fzpqc40wg1viib8klq8lby54agpl213w5"; }
) {
  src =  ./.;
}).defaultNix
```

Es wird nicht weiter auf den Inhalt der Datei oder darauf eingeangen, warum genau durch sie Abwärtskombailität hergestellt wird. Ist vielleicht auch nicht so wichtig. Allgemein frage ich mich, unter welchen Umständen wir auf den alten Befehl zurückgreifen wollen würden.

Tatsächlich wird angemerkt, dass wir nun noch immer nicht völlig wie bisher mit den definierten Derivations interagieren können. Wenn wir ein Paket mit `nix-build` bauen möchten, dass in einer Flake-Datei definiert wurde, dann müssen wir den genauen Attributpfad angeben.

Für das `hello`-Paket im ersten Beispiel wäre das etwa:
```bash
nix-build -A packages.x86_64-linux.hello
```

Bei einer "gewöhnlichen" `default.nix`, wie wir sie in den vorausgegangen Beiträgn kennengelernt haben, reichte ein weitaus kürzerer Befehl (`nix-build -A hello`).

## Offene Fragen
- Welche Einstellungen können in der Nix-Konfigurationsdatei vorgenommen werden?
- Wie können wir eigene Flake-Templates definieren?
- Wie legen wir ein Default-Template für Flakes fest?
- Reicht es, `flake.nix` der Git Staging Area hinzuzufügen oder müssen weitere Flake-Dateien eingecheckt werden?
- Was sind die Unterschiede und Gemeinsamkeiten zwischen `nix-build` und `nix build`?
- Unter welchen Bedingungen und zu welchen Zeitpunkten wird die `flake.lock`-Datei aktualisiert?
- Was genau passiert bei einem Update der Flake-Inputs?
- Was sind `checks` (einer der Output-Typen)?
- Welche weiteren Output-Typen gibt es (neben `packages`, `devShells` und `checks`)?
- Was ist die Beziehung zwischen Flake-Inputs, den möglichen Argumenten der Outputs-Funktion und den Werten von `import`?

## Fußnoten
[^flakes]: Das stimmt vielleicht nicht ganz. Zuvor haben wir Dateien geschrieben, die eine Funktion definierten. Nun schreiben wir Dateien, die eine Attributmenge definieren. Dazu unten mehr.
[^flake-attribute]: Ich denke prinzipiell kann jedes der drei Attribute ausgelassen werden.
