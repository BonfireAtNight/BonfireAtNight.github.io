---
layout: post
title:  "Nix Pills (9)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 9 (Kommentar)
<div class="hide-excerpt">
Im neunten Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/automatic-runtime-dependencies" target="_blank">Automatic Runtime Dependencies</a>, geht es um Dependencies. Wir lernen, wie wir uns Build-Dependencies und Runtime-Dependencies anzeigen lassen können. In einigen Fällen werden Dependencies falsch klassifiziert. Es wird erklärt, wie falsch zugeordnete "Runtime"-Dependencies entfernt werden können.
</div>
<!--more-->

Im neunten Kapitel der Nix Pills, [Automatic Runtime Dependencies](https://nixos.org/guides/nix-pills/automatic-runtime-dependencies){: target="_blank"}, geht es um Dependencies. Wir lernen, wie wir uns Build-Dependencies und Runtime-Dependencies anzeigen lassen können. In einigen Fällen werden Dependencies falsch klassifiziert. Es wird erklärt, wie falsch zugeordnete "Runtime"-Dependencies entfernt werden können. Nebenbei erfahren wir etwas über NAR-Archive und Build-Phasen.

Im letzten Abschnitt heißt es, dass der vergleichweise kurze Beitrag in der in der Urlaubszeit geschrieben wurde. Um ehrlich zu, das merkt man dem Text an vielen Stellen auch an. Zusammenhänge werden nicht immer deutlich gemacht und meinem Eindruck nach bleibt das Gesamtziel des Beitrags unklar.

## Build-Dependencies
Die Build-Dependencies eines gegebenen Pakets können seiner Store Derivation (der `.drv`-Datei) entnommen werden. Als Beispiel schauen wir uns GNU Hello näher an. Für diese Anwendung wurde im vorausgegangenen Kapitel eine Deriation geschrieben und in einer Datei (`hello.nix`) gespeichert. In einem früheren Kapitel haben wir auch gelernt, dass die Store Derivation als Nebeneffekt der Instanziierung eines Pakets erstellt wird.
```
$ nix-instantiate hello.nix
/nix/store/z77vn965a59irqnrrjvbspiyl2rph0jp-hello.drv
```

Dieser Datei können wir die Build-Dependencies des Pakets entnehmen. Den dafür notwendigen Befehl haben wir auch schon an früherer Stelle kennengelernt.
```
$ nix-store -q --references /nix/store/z77vn965a59irqnrrjvbspiyl2rph0jp-hello.drv
/nix/store/0q6pfasdma4as22kyaknk4kwx4h58480-hello-2.10.tar.gz
/nix/store/1zcs1y4n27lqs0gw4v038i303pb89rw6-coreutils-8.21.drv
/nix/store/2h4b30hlfw4fhqx10wwi71mpim4wr877-gnused-4.2.2.drv
/nix/store/39bgdjissw9gyi4y5j9wanf4dbjpbl07-gnutar-1.27.1.drv
/nix/store/7qa70nay0if4x291rsjr7h9lfl6pl7b1-builder.sh
/nix/store/g6a0shr58qvx2vi6815acgp9lnfh9yy8-gnugrep-2.14.drv
/nix/store/jdggv3q1sb15140qdx0apvyrps41m4lr-bash-4.2-p45.drv
/nix/store/pglhiyp1zdbmax4cglkpz98nspfgbnwr-gnumake-3.82.drv
/nix/store/q9l257jn9lndbi3r9ksnvf4dr8cwxzk7-gawk-4.1.0.drv
/nix/store/rgyrqxz1ilv90r01zxl0sq5nq0cq7v3v-binutils-2.23.1.drv
/nix/store/qzxhby795niy6wlagfpbja27dgsz43xk-gcc-wrapper-4.8.3.drv
/nix/store/sk590g7fv53m3zp0ycnxsc41snc2kdhp-gzip-1.6.drv
```

Es ist vielleicht nicht offensichtlich, warum die Store Derivation etwas damit zu tun hat. Mir jedenfalls war der Zusammenhang nicht ohne Weiteres klar. (Ich frage ich noch immer, was ihr genauer Zweck ist.) Die Nix Pills erklären:
> (...) (T)he hello.drv file is the representation of the build action to perform in order to build the hello out path, and as such it also contains the input derivations needed to be built before building hello.

Build-Dependencies sind all die Dateien und Derivations, die als Build-Inputs verwendet werden. Praktisch sind es all die Dinge, die in der Argumentmenge der `derivation`-Funktion referenziert werden, die das entsprechende Paket als Outpuut hatte.

Im Falle von GNU Hello kommen sie größtenteils aus der in der allgemeinen `autotools.nix` definierten Attributmenge:
```nix
pkgs: attrs:
  let defaultAttrs = {
    builder = "${pkgs.bash}/bin/bash";
    args = [ ./builder.sh ];
    baseInputs = with pkgs; [ gnutar gzip gnumake gcc coreutils gawk gnused gnugrep binutils.bintools ];
    buildInputs = [];
    system = builtins.currentSystem;
  };
  in
  derivation (defaultAttrs // attrs)
```
Lediglich die Quelldatei (`hello-2.10.tar.gz`) wird in der Paket-spezifischen Attributmenge in `hello.nix` definiert:
```nix
let
  pkgs = import <nixpkgs> {};
  mkDerivation = import ./autotools.nix pkgs;
in 
  mkDerivation {
    name = "hello";
    src = ./hello-2.12.1.tar.gz;
  }
```

## Runtime-Dependencies
Runtime-Dependencies unterscheiden sich von Build-Dependencies dahingehend, dass sie nicht explizit beim `derivation`-Aufruf referenziert werden. "(...) Nix automatically computes all the runtime dependencies of a derivation, and it's possible thanks to the hash of the store paths."

Es wird nicht erklärt, wie genau die Store-Pfade beii der Berechnung der Runtime-Dependencies helfen. Direkt auf das gegebene Zitat folgen "Steps". Doch es scheinen die Schritte zu sein, die notwendig sind, um sich die Runtime-Dependencies *anzeigen* zu lassen. Das heißt, Nix wird dadurch aufgefordert, die Dependencies zu berechnen; wie es das macht, erfahren wir nicht.

Die Schritte sind mir auch nicht völlig durchsichtig. Hier das gesamte Zitat:
1. Dump the derivation as NAR, a serialization of the derivation output. Works fine whether it's a single file or a directory.
2. For each build dependency .drv and its relative out path, search the contents of the NAR for this out path.
3. If found, then it's a runtime dependency.

Wer führt die Schritte aus? Ist das eine Anweisung, was *ich* machen soll? Sind es Dinge, die Nix automatisch macht, wenn ich die darauffolgenden Befehle ausführe? Und muss ich wirklich notwendig eine NAR-Datei erstellen?

Scheinbar lassen sich die Runtime-Dependencies mit der folgenden Befehlfolge anzeigen lassen:
```
$ nix-instantiate hello.nix
/nix/store/z77vn965a59irqnrrjvbspiyl2rph0jp-hello.drv
$ nix-store -r /nix/store/z77vn965a59irqnrrjvbspiyl2rph0jp-hello.drv
/nix/store/a42k52zwv6idmf50r9lps1nzwq9khvpf-hello
$ nix-store -q --references /nix/store/a42k52zwv6idmf50r9lps1nzwq9khvpf-hello
/nix/store/94n64qy99ja0vgbkf675nyk39g9b978n-glibc-2.19
/nix/store/8jm0wksask7cpf85miyakihyfch1y21q-gcc-4.8.3
/nix/store/a42k52zwv6idmf50r9lps1nzwq9khvpf-hello
```
`nix-instantiate` dient erneut dazu, die `.drv`-Datei zu erstellen. Der Zweck von `nix-store -r` wird nicht erklärt. Im vorrausgegangenen Abschnitt war von `nix-store --restore` die Rede: "To create NAR archives, it's possible to use `nix-store --dump` and `nix-store --restore`" Ich weiß nicht, ob `nix-store -r` und `nix-store --restore` äquivalent sind.

Und selbst wenn, ich weiß nicht, was `nix-store --dump` und `nix-store --restore` voneinander unterscheidet. Wahrscheinlich soll der eine Befehl auf Maschine A ausgeführt werden, um eine Archiv-Repräsentation der übergebenen Derivation zu erhalten. Und diese kann dann auf einer Maschine B mit dem zweiten Befehl importiert werden?

Die Hauptfrage ist: Was hat das damit zu tun, dass wir uns Runtime-Dependencies anzeigen lassen wollen? Der Befehl hat eine Ausgabe, die als Argument an `nix-store -q --references` übergeben wird. Ist das die einzige Möglichkeit, diese Information abzurufen?

## Werden die richtigen Runtime-Dependencies verwendet?
Im vorrausgegangenen Abschnitt wurde ein Befehl ausgeführt, um die Runtime-Dependencies unseres GNU Hello Pakets anzuzeigen. Das Paket hängt von einer Bibliothek ab, `glibc`. Ich weiß nicht, was genau diese Bibliothek ist (wahrscheinlich eine C-Standardbibliothek?). Jedenfalls wirkt es, wie eine typische Runtime-Dependency. Eine weitere Dependency war `hello`. Das ist ein wenig überraschend, aber wahrscheinlich sind Programme Runtime-Depencencies von sich selbst? Die dritte vermeintliche Runtime-Depencency, `gcc`, ist mehr als überraschend: Compiler sind *Build*-Dependencies par excellence.

Meine Reaktion darauf wäre gewesen, dass Nix' automatische Runtime-Dependency-Erkennung nicht so zuverlässig funktioniert, wie einleitend versprochen wurde. Die Nix Pills hingegen scheint das nicht weiter zu überraschen. Sie geben folgende Erklärung:
> (...) Nix added `gcc` (to the list of runtime dependencies) because its out path is mentioned in the "hello" binary. Why is that? That's the [ld rpath](http://en.wikipedia.org/wiki/Rpath){: target="_blank"}. It's the list of directories where libraries can be found at runtime. In other distributions, this is usually not abused (used?). But in Nix, we have to refer to particular versions of libraries, thus the rpath has an important role.
>

Das erklärt, warum `gcc` in der Liste ist. Dennoch ist es auch nach Einschätzung des Autors eine missgeleitete Klassifikation. Hier die Begründung dieser Einschätzung:
> The build process adds that gcc lib path thinking it may be useful at runtime, but really it's not. 

Die letzten Absätze des Beitrags erklären, wie wir die Depencency los werden. Dazu wird ein Tool verwendet, das von Nix-Contributors erstellt wurde: `patchelf`. Hier der Befehl, um das Problem mit dem RPfad zu beseitigen:
```bash
find $out -type f -exec patchelf --shrink-rpath '{}' \; -exec strip '{}' \; 2>/dev/null
```

Das Vorgehen wirkt ziemlich hacky und zugegebenermaßen ein wenig ad-hoc. Zwar wird von einer *fixup*-Phase beim Build-Prozess geesprochen. Soweit ich sehe werden die Befehle aber in keinerlei Weise in die Erstellung der Derivation integriert. Stattdessen werden die notwendigen Befehle auf der Kommandozeile ausgeführt. Sicher nicht, was man für jedes Paket per Hand machen möchte.

Es stellt sich die Frage, wie häufig Derivations fehlerhaft den Runtime-Dependencies eines Pakets hinzugefügt werden. Und sind die Bemerkungen bezüglich der Speichernutzung wirklich der Hauptgrund dafür, warum das problematisch ist?

## Nix Archive (NAR)
Wir erfahren, dass es ein Nix-eigenes Archivformat gibt: Nix Archive, oder kurz: NAR. Man kann sich fragen, warum nicht einfach das Tar-Format genutzt wird (wie beispielsweise bei vielen Quelldateien). Dazu schreibt der Autor Folgendes:
> (...) (C)ommonly used archivers are not deterministic. They add padding, they do not sort files, they add timestamps, etc.. Hence NAR, a very simple deterministic archive format being used by Nix for deployment. NARs are also used extensively within Nix (...).

Es werden zwei Befehle angeführt, um NAR-Dateien zu erstellen und zu entpacken (extrahieren): `nix-store --dump` und `nix-store --restore`. Es wird gesagt, dass Pakete in sich geschlossen (*self-contained*) sind. Dementsprechend können so archivierte Pakete auf einer anderen Maschine entspackt und garantiert ausgeführt werden. Diese Verteilung von Software-Komponenten ist eine Form von Deployment, einem von Nix verfolgtem Hauptzweck.

## Build-Phasen
Build-Vorgänge können sehr komplex werden. Es liegt nahe, sie in einzelne Phasen einzusteilen. Dabei handelt es sich nicht nur um konzeptuelle Einteilungen. Nix definiert Phasen mit eigenen Beonderheiten. Leider erfahren wir in diesem Beitrag keine technischen Details.

Der Beitrag spricht von einer Entpackphase (*unpack phase*), einer Konfigurationsphase (*configuration phase*), der eigentlichen Build-Phase (*build phase*) und einer *Installationsphase (*installation phase*). Neu hinzu kommt eine *Fixup-Phase* nach der Installation. Zu dieser Zeit werden unter anderem fälschlich hinzugefügte Runtime-Dependencies entfernt.

Leider wird nicht weiter auf die Besonderheiten der Phasen eingegangen.

## Offene Fragen
- Was genau unterscheidet Build-Dependencies und Runtime-Dependencies?
- Wie genau berechnet Nix die Runtime-Dependencies eines gegebenen Pakets? Wie genau helfen die Hash-Werte der Store-Pfade?
- Warum wird `nix-store -r <Store Derivation>` ausgeführt?
- Wie häufig werden Derivations fehlerhaft als Runtime-Dependencies hinzugefügt?
- Warum ist es problematisch, dass sich Derivations fehlerhaft in der Liste von Runtime-Dependencies eines Pakets finden?
- Welche Build-Phasen gibt es und was sind ihre Besonderheiten?
- Welche anderen Dinge können während der Fixup-Phase passieren?

## Fußnoten
