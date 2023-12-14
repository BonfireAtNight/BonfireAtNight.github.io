---
layout: post
title:  "Inria Tutorium (5)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix-Tutorium vom Nationalen Forschungsinstitut für Informatik und Automatisierung (INRIA): Teil 5
<div class="hide-excerpt">
Kommentar zum fünften Beitrag in der inria-Serie über Nix, <a href="https://nix-tutorial.gitlabpages.inria.fr/nix-tutorial/experiment-packaging.html" target="_blank">Deploying and Binary Cache</a>. Darin lernen wir:
<ul>
<li>Wie man sich die Pakete anzeigen lassen kann, von denen ein gegebenes Paket abhängt</li>
<li>Wie man gebaute Pakte von einem Nix-Store <i>exportieren</i> und in einem anderen <i>importieren</i> kann</li>
<li>Wie man Pakete vom Nix-Store einer Maschine A in den Nix-Store einer Maschine B kopiert</li>
</ul>
</div>
<!--more-->
Der nächste inria-Beitrag, den ich mir näher anschauen möchte, ist [Deploying and Binary Cache](https://nix-tutorial.gitlabpages.inria.fr/nix-tutorial/experiment-packaging.html){:target="_blank"}. Ich skippe den [Beitrag über Notebooks](https://nix-tutorial.gitlabpages.inria.fr/nix-tutorial/notebook.html){:target="_blank"}. Notebooks sind cool, wenn man seine Forschung oder Lernfortschritte dokumentieren möchte. Aber nicht gerade das spannendste Thema, um darüber zu schreiben.

Im Einzelnen lernen wir:
- Wie man sich die Pakete anzeigen lassen kann, von denen ein gegebenes Paket abhängt
- Wie man gebaute Pakte von einem Nix-Store *exportieren* und in einem anderen *importieren* kann
- Wie man Pakete vom Nix-Store einer Maschine A in den Nix-Store einer Maschine B kopiert

## Closure
Bei der Frage, von welchen Software-Komponenten ein gegebenes Paket abhängt, geht es nicht nur um *unmittelbare* Dependencies. Ein Paket kann auf ein anderes verweisen (*reference*), das wiederum auf andere Pakete zeigt. In einigen Kontexten ist die Menge aller mittelbaren und unmittelbaren Dependencies entscheidend.

Technisch spricht man von der *Abgeschlossenheit* der Menge aller Pakte unter der Referenzrelation (*closure of the reference relation*). Das heißt für jedes Paket-Paar lässt sich entscheiden, ob sie in der Relation stehen oder ob sie es nicht tun. Das gilt in beide Richtungen.

Abgeschlossenheit ist eine Eigenschaft einer Algebra, das heißt einer Struktur bestehend aus der Menge aller Pakete und der Referenzrelation. Wichtiger sind Mengen, die *Closures* genannt werden. Die Closure eines gegebenen Pakets ist die Menge aller Pakete, die rekursiv über Referenzen erreicht werden können.[^artikel] Auch das in Frage stehende Paket selbst ist Element der Menge.

## Auflistung von Dependencies
Um sich die Dependencies eines Pakets anzeigen zu lassen, muss es bereits (mit `nix-build`) gebaut worden sein. Daraus resultiert die `./result`, die mit `nix-store` abgefragt werden kann:
```bash
nix-store --query --references ./result
nix-store --query --requisites ./result
```
Durch den ersten Befehl erhalten wir die *unmittelbaren* Dependencies. Der zweite Befehl können wir uns rekursiv alle Pakete auflisten lassen, auf die ein Paket zeigt (*recursively list the references of a package*).

Nebenbei lernen wir etwas über `nix-build`. Ich glaube der Befehl resultiert immer in der `./result` im Arbeitsverzeichnis. Doch die Derivation wird nur dann tatsächlich gebaut, wenn sich das gebaute Paket nicht bereits im Nix-Store befindet.

Wir erfahren auch, wie diese Frage entschieden wird: "(...) `nix-build` looks into the store whether the package can be found. This is made possible thanks to the hashing of the package inputs (and thanks to the package name)."

Der Zusatz über den Paketnamen wirft die Frage auf, ob Hash-Werte nicht bereits einmalig wären. Könnten zwei Derivations (zumindest theoretisch) exakt gleiche Inputs und damit gleiche Hash-Werte haben, ohne identisch zu sein? Das heißt, sind immer Hash-Wert und Paketname zusammen ausschlaggebend?

## Pakete zwischen Stores übertragen
Je nach Inhalten kann es sehr bis extrem zeitaufwändig sein, Pakete zu bauen. Unter Umständen kann es effektiver sein, ein auf einer Maschine bereits gebautes Paket auf eine *andere* Maschine zu übertragen.[^architektur] Das ist eine Möglichkeit, Software zu *verteilen*.

Der Kontext wird *Software-Deployment* genannt. Nix versteht sich nicht primär als Paket-Manager, sondern als Deployment-System. Es resultierte aus einem wissenschaftlichen Forschungsprojekt, dessen Ergebnisse in Eelco Dolstras Disseration erläuert wurden. Hier die ersten Worte aus der Einleitung von [The Purely Functional Software Deployment Mode](https://edolstra.github.io/pubs/phd-thesis.pdf){:target="_blank"}:
> This thesis is about getting computer programs from one machine to another—and having them still work when they get there. This is the problem of *software deployment*. Though it is a part of the feild of Software Confgiuration Management (SCM), it has not been a subject of academic study until quite recently.

Closures können aus einem Store *exportiert* werden. Dazu kann ein weitere `nix-store`-Unterbefehl verwendet werden:
```bash
nix-store --export $(nix-store --query --requisites ./result) > chord.nar
```
Das Ergebnis ist ein Nix ARchive (NAR). Es wird nichts weiter darüber gesagt, warum dieses Archivformat verwendet wird. Weißt es Besonderheiten gegenüber anderen Archivtypen auf?

Exportierte Archive können wir nun auf beliebigen Wege auf die andere Maschine übertragen und dann in den dortigen Nix-Store *importieren*:
```bash
nix-store --import < chord.nar
```

Dieser Ansatz hat zwei Nachteile. Der Befehl umfasst keinen Übertragungsmechanismus, mit dem das Archiv auch tatsächlich auf die andere Maschine gebracht werden kann. Außerdem "weiß" der Befehl nichts darüber, welche Pakete eventuell bereits auf dem Zielsystem vorhanden sind. Stattdessen wird einfach die *gesamte* Closure ins Archiv integriert. 

Für gewöhnlich dürfte ein anderer Befehl, `nix-copy-closure` vorzuziehen sein. "This command copies a closure from one store to another, but it only transfers the dependencies that are missing on the target host."

Leider lernen wir im Beitrag absolut gar nichts über die Verwendung des Befehls. Das [offizielle Bedienungshandbuch](https://nixos.org/manual/nix/unstable/command-ref/nix-copy-closure.html){:target="_blank"} gibt folgendes Beispiel:
```bash
nix-copy-closure --to alice@itchy.labs $(type -tP firefox)
```

## Binary Cache
Der Titel des Beitrags ist irreführend: "Deploying and *Binary Cache*". Soweit ich sehe wird von einem Binary Cache nur an einer einzigen Stelle gesprochen (STRG+f bestätigt das):
> If the machines have a different architecture, importing the closure might not help the second machine, as the packages would not be available in the architecture it desires. In this case, the second machine would fetch the required packages from a binary cache if possible, or rebuild them otherwise.

Nicht sonderlich erhellend. Es gibt für Nixpkgs einen [Server](https://cache.nixos.org/){:target="_blank"} mit vorgebauten Paketen, von denen Daten unter bestimmten Umständen heruntergeladen werden (*Binary Cache Server*). Überraschenderweise geht der Beitrag darauf mit keinem Wort ein.

Überraschend finde ich gleichermaßen, dass nicht über die *Installation* der (vor)gebauten Pakete gesprochen wird. Gibt es Szenarien, in denen wir gebaute Binaries in unseeren Nix-Store übernehmen wollen, ohne sie auch in einer Nutzerumgebung verfügbar zu machen? Gibt es keine Befehle, die Import und Installation kombinieren?

Das Bedienungshandbuch gibt ein in diesem Kontext hilfreiches zweites Beispiel:
```bash
nix-copy-closure --from alice@itchy.labs \
    /nix/store/0dj0503hjxy5mbwlafv1rsbdiyx1gkdy-subversion-1.4.4
nix-env --install /nix/store/0dj0503hjxy5mbwlafv1rsbdiyx1gkdy-subversion-1.4.4
```
Dadurch erfahren wir wenigstens, wie wir ein kopiertes gebautes Paket in einer Nutzerumgebung installieren.

## Offene Fragen
- Sind Hash-Werte hinreichend, um ein Paket eindeutig zu identifizieren? Oder sind Paketname und Hash-Wert zusammen erforderlich?
- Gibt es Befehle, die gebaute Pakete zugleich importieren und installieren?
- Was zeichnt Nix Archive gegenüber anderen Archivformaten aus?

## Fußnoten
[^architektur]: Das ist nur dann sinnvoll, wenn die beiden Systeme mit der gleichen CPU-Architektur operieren. Ansonsten ist das gebaute Paket auf dem Zielsystem wertlos.
[^artikel]: Es ist *die* Closure, oder? Der Closure? Klingt komisch. Das Closure? Bestimmt nicht.
