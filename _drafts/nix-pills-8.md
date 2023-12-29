---
layout: post
title:  "Nix Pills (8)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 8 (Kommentar)
<div class="hide-excerpt">
Im achten Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/generic-builders" target="_blank">Generic Builders</a>, wird ein Build-Skript entwickelt, mit dem GNU Autotools-Projekte allgemein gebaut werden können.
</div>
<!--more-->

In vorausgegangenen Kapiteln wurden Builder speziell für einzelne Pakete gebaut. Mit diesen Skripten konnten *nur* die entsprechenden Pakete gebaut werden. Im achten Kapitel der Nix Pills, [Generic Builders](https://nixos.org/guides/nix-pills/generic-builders){: target="_blank"}, werden Pakete mit den GNU Autotools erstellt. Es wird ein Build-Skript entwickelt, das von Besonderheiten einzelner Pakete abstrahiert. Mit dem *einen* Builder können deshalb verschiedene Pakete gebaut werden.

[GNU Autotools](https://en.wikipedia.org/wiki/GNU_Autotools){: target="_blank"} ist das meistgenutzte Build-System. Der Zweck des Werkzeugs liegt darin, Unterschiede zwischen verschiedenen Plattforen (verwendeter C-Compiler, Namen von Header-Dateien, Bestehen von Bibliotheksfunktionen etc.) in den Griff zu bekommen. Mit den Autotools werden Builds für die meisten Programme zu einem zweischrittigen Vorgang: Konfiguration (GNU Autoconf) und Make (GNU Automake).

Im Beitrag wird Darwin (MacOS) angesprochen. Eine Besonderheit der Plattform liegt darin, dass in der Regel `clang` statt `gcc` als C-Compiler verwendet wird. Hier haben wir es demnach mit einem der angedeuteten Unterschiede zwischen Plattformen zu tun. Im Beitrag wird (noch) nicht darauf eingegangen, wie ein Builder geschrieben werden kann, der auch von Merkmalen der Plattform abstrahiert. Auf diesen Gesichtspunkt wird im Folgenden deshalb nicht weiter eingegangen.

## GNU Autotools: ein Builder speziell für ein Paket
Im vorausgegangenen Kapitel wurde ein sehr einfaches C-Programm verpackt. Die dazugehörige `.c`-Datei wurde im Build-Skript mit eine einfachen Aufruf von `gcc` kompiliert. Für ein erstes Beispiel war das in Ordnung. In der Realität nutzen sehr viele Pakete die GNU Autotools.

Hier wird exemplarisch ein sehr einfaches Programm mit diesem Tool verpackt, GNU Hello World. Im ersten Schritt wird ein Builder-Skript wieder speziell für dieses Vorführprogramm geschrieben (`hello_builder.sh`):
```bash
export PATH="$gnutar/bin:$gcc/bin:$gnumake/bin:$coreutils/bin:$gawk/bin:$gzip/bin:$gnugrep/bin:$gnused/bin:$bintools/bin"
tar -xzf $src
cd hello-2.12.1
./configure --prefix=$out
make
make install
```

Das Skript ist in einer Hinsicht untypisch. `--prefix=$out` ist notwendig, um den Output im Nix-Store zu erstellen. Wenn ich die [offizielle Dokumentation](https://www.gnu.org/software/autoconf/manual/autoconf-2.69/html_node/Default-Prefix.html){: target="_blank"} richtig verstehe, dann wird standardmäßig `/usr/local` verwendet. Mit dem Flag wird dieser Wert überschrieben und die Outputs landen im richtigen Unterverzeichnis des Store.

Das Skript nutzt eine Reihe von Umgebungsvariablen (`gnutar`, `gcc`, `gnumake` etc.). Wie zuvor muss eine Derivation erzeugt werden, dessen Input-Menge Attribute umfasst, aus denen entsprechende Variablen für die Build-Umgebung erzeugt werden. Hier eine erste Version der `hello.nix`:
```nix
let
  pkgs = import <nixpkgs> {};
in
  derivation {
    name = "hello";
    builder = "${pkgs.bash}/bin/bash";
    args = [ ./hello_builder.sh ];
    inherit (pkgs) gnutar gzip gnumake coreutils gawk gnused gnugrep;
    gcc = pkgs.clang;
    bintools = pkgs.clang.bintools.bintools_bin;
    src = ./hello-2.12.1.tar.gz;
    system = builtins.currentSystem;
  }
```
Dieses Vorgehen ist aus zwei Gründen suboptimal.

Zum einen müsste für jedes Paket ein eigenes Build-Skript geschrieben werden. Im Folgenden wird ein Build-Skript entwickelt, das von vielen Paket-spezifischen Merkmalen abstrahiert. Dadurch kann es von allen Projekten verwendet werden, deren Inhalte mit den Autotools gebaut werden.

Zum anderen werden beim `derivation`-Aufruf eine Reihe von Dependencies eingeführt, die Autotool-Projekten gemeinsam sind. Im Folgenden wird ein modularisierter Ansatz vorgeführt. Dabei wird eine Derivation geschrieben, die Basisprogramme umfasst. Die Derivations für einzelne Programme (wie GNU Hello World) fügen der Grundmenge die Derivations hinzu, die speziell für sie vorausgesetzt sind.

## Ein generischer Builder für Autotools-Projekte
Unser erstes Ziel ist es demnach, einen Builder für Autotools-Projekte allgemein zu schreiben. Dieses Skript kann den Paketen für diese Projekte beigefügt werden. Hier die für diesen Beitrag finale Version der `builder.sh`:
```bash
set -e
unset PATH
for p in $buildInputs; do
  export PATH=$p/bin${PATH:+:}$PATH
done

tar -xf $src

for d in *; do
  if [ -d "$d" ]; then
    cd "$d"
    break
  fi
done

./configure --prefix=$out
make
make install
```

Es versteht sich, dass einige Grundkenntnisse der Shell-Programmierung und der Autotools vorausgesetzt sind, um den Inhalt dieser Datei zu verstehen. Einige weniger allgemein bekannte Aspekte werden in den Nix Pills erklärt:
- `set -e` legt fest, dass jeder Fehler zum Abbruch des Build-Vorgangs führt.
- Im vorausgegangenen Kapitel wurde darauf hingewiesen, dass die `PATH`-Variable zur Build-Zeit auf `/path-not-set` gesetzt wird. Natürlich ist das kein richtiger Pfad. Mit `unset PATH` wird der Wert fallengelassen. Dadurch können wir im Folgenden einfach Komponenten an den Pfad anhängen (zu Beginn ist es nun der leere String).
- Mit der ersten `for`-Schleife wird das `bin`-Unterverzeichnis aller `buildInputs` dem PATH angehängt. Damit sind die Namen der ausführbaren Binärdateien (`gcc` etc.) zur Build-Zeit verfügbar.
- Der Quellcode wird Paketen sehr häufig in Form eines Archivs hinzugefügt. Mit `tar -xf $src` wird dieses Archiv (in das Temp-Verzeichnis der Build-Zeit) entspackt.
- Wenn ich die Annahmen des Skripts richtig verstehe, dann enthält das temporäre Verzeichnis nur *ein* Unterverzeichnis und dieses Unterverzeichnis enthält (nach dem vorausgegangenen Schritt) die Quelldateien. Die zweite `for`-Schleife iteriert über alle Dateien; wenn die "Datei" ein Verzeichnis ist (`if [ -d "$d" ];`),[^datei] dann enthält es den Quellcode und wir wechseln dorthin (`cd "$d"`).
- Mit den letzten beiden Zeilen werden die beiden Schriite ausgeführt, von denen einleitend die Rede war: Konfiguration und Build.

Damit lässt sich GNU Hello World, aber eben auch viele andere Projekte bauen. Abschließend wird gesagt, dass das Skript noch immer viele Annahmen macht. Es ist aber zweifellos weitaus generischer als das oben angeführte erste Builder-Skript.

## Modularer Input für die `derivation`-Funktion
Derivations werden erstellt, indem der `derivation`-Funktion eine Attributmenge übergeben wird. Sehr viele der dabei definierten Attribute werden für Pakete der allermeisten Autotools-Projekte verwendet.

Sie alle verwenden das eben geschriebene Skript als Builder; `builder` und `args` erhalten deshalb in allen Fällen dieselben Werte. Darüber hinaus gibt es eine Reihe von Werkzeugen, die für den Build dieser Gruppe von Programmen benötigt werden (`baseInputs`). Es liegt deshalb nahe, diese Attribute bereits in einer Menge zusammenzufassen (`defaultAttrs`).
```nix
let
  defaultAttrs = {
    builder = "${pkgs.bash}/bin/bash";
    args = [ ./builder.sh ];
    baseInputs = with pkgs; [ gnutar gzip gnumake gcc coreutils gawk gnused gnugrep binutils.bintools ];
    buildInputs = [];
    system = builtins.currentSystem;
  };
in
  [...]
```
Für die Attribute, die nur von einzelnen Paketen benötigt werden, wird eine eigene Attributmenge erstellt. Für ihre Build-Dependencies kann ein `buildInputs`-Attribut mit einer gefüllten Liste als Wert definiert werden. Für die `baseInputs` werden automatisch Umgebungsvariablen erstellt. Wir haben im Skript oben gesehen, dass für die `buildInputs` PATH-Komponenten hinzugefügt werden. So sind für beide Dependency-Formen Symbole in der Build-Umgebung verfügbar. Falls nötig können weitere Umgebungsvariablen ganz einfach dadurch erzeugt werden, dass der Menge der einzelnen Pakete weitere Attribute hinzugefügt werden.

Der Funktion, die eine Derivation für ein bestimmtes Projekt erstellt, kann dann die *Vereinigung* der Default-Menge und der Projekt-spezifischen Menge als Argument übergeben werden.
```nix
let
  [...]
in
  derivation (defaultAttrs // attrs)
```
Die Rede von einer *Default*-Menge deutet das Verhalten des Merge-Operators (`//`) an. Attribute, die sich in der einen *oder* der anderen Menge finden, werden der resultierenden Menge beide hinzugefügt. Wenn sich ein Attribut in *beiden* Mengen findet, dann wird der Wert der rechtn Menge bevorzugt. Das heißt der Wert der Default-Menge wird *überschrieben*.

Es stellt sich die Frage, warum der Default-Attribbutmenge eine Definition für `buildInputs` mit der leeren Liste als Wert hinzugefügt wird. Der Beitrag geht auf diese Frage nicht ein. Vielleicht will man damit den Fall berücksichtigen, in der später eventuell der Default-Menge schon Build-Inputs hinzugefügt werden? Ich glaube beim Merge der Attributmengen werden auch Listen gemergt, wenn sie denselben Schlüssel haben. Vielleicht gibt es Fälle, in denen für das Attribut ein einzelner Wert und keine Liste als Wert festgelegt wird? Dann würde gewährleistet, dass trotzdem ein Attribut mit einer Liste als Wert resultiert.

## Funktionale Trennung zwischen Basis-Attributen und Paket-spezifischen Attributen
Bei den bisherigen Überlegungen wurde ein wichtiger Aspekt ausgeblendet. In der im vorausgegangenen Abschnitt definieerten `defaultAttrs` werden Symbole für eine Reihe von Derivations verwendet. Die `pkgs`-Variable muss noch mit Leben gefüllt werden. Und auch die Paket-spezifische Attributmenge (`attrs`), mit der in der letzten Zeile gemergt wird, kommt von außen.

Wir transformieren den Mengenausdruck zu einen Lambdaausdruck und machen die Inputs von außen zu Funktionsparametern. Dadurch erhalten wir die finale Version der `autotools.nix`, die eine Basis-Derivation für Autotools-Projekte definiert:
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

Im Kapitel über die `import`-Funktion wurde ein strukturell völlig analoger Fall besprochen. Wir definieren hier eine *zu importierende* Datei und diese Datei enthält einen Funktionsausdruck. In der *importierenden* Datei können Argumente an die importierte Funktion übergeben werden.

In gewisser Weise verlangt die Funktion zwei Argumente (es ist ein Fall von Currying). In der finalen Form der `hello.nix` wird zunächst *ein* Argument bereitgestellt. Die dadurch teilweise gesättigte Funktion (partielle Applikation) wird in einer Variable gespeichert:
```nix
let
  pkgs = import <nixpkgs> {};
  mkDerivation = import ./autotools.nix pkgs;
in 
  [...]
```

Damit sind alle Komponenten zusammen, die von außen kommen und den allermeisten Autotools-Projekten gemeinsam sind. Die Paket-spezifischen Attribute werden an die `mkDerivation`-Funktion übergeben. Hier die fertige finale Version von `hello.nix`:
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
Die Attributmenge, die der Funktion übergeben wird, wird vom `attrs`-Attribut aufgenommen (siehe die Definition in der `autotools.nix`).

GNU Hello World ist ein sehr einfaches Programm, weshalb nur sehr wenige weitere Attribute notwendig sind. Paket-spezifisch sind nur noch der Name und die Quelldateien. Bei komplexeren Programmen könnten weitere Attribute definiert und zusätzliche Build-Dependencies verlangt werden.

## Offene Fragen
- Warum wird der `defaultAttrs` in `autotools.nix` eine Definition für `buildInputs` hinzugefügt?

## Fußnoten
[^datei]: In der Unix-Welt werden Verzeichnisse als Dateien konzeptualisiert. Das Stichwort ist: ["Everything is a file"](https://en.wikipedia.org/wiki/Everything_is_a_file){: target="_blank"}
