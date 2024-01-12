---
layout: post
title:  "Nix Pills (18)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 18 (Kommentar)
<div class="hide-excerpt">
Im achtzehnten Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/nix-store-paths" target="_blank">Nix Store Paths</a>, werden verschiedene Typen von Store-Pfaden besprochen. Dazu wird Schritt für Schritt erläutert, wie <i>der</i> Store-Pfad für eine gewöhnliche Datei generiert wird, die dem Nix Store hinzugefügt wird.<br><br>

Bei der Berechnung des Store-Pfads werden verschiedene Zwischenergebnisse erstellt.
</div>
<!--more-->

Im achtzehnten Kapitel der Nix Pills, [Nix Store Paths](https://nixos.org/guides/nix-pills/nix-store-paths){: target="_blank"}, geht es um Store-Pfade. Alle Store-Inhalte finden sich unter `/nix/store/.......`. Im Kapitel wird erklärt, wie die entscheidende letzte Komponente berechnet wird.

Das ist zugebenermaßen von völlig theoretischem Interesse. Da Nix für die meisten Benutzer von den Store-Pfaden und Hash-Werten abstrahiert, dürfte es keinen großen praktischen Nutzen haben, in diesem Bereich näher in die Tiefe zu gehen. Der Vollständigkeit halber wollte ich es mir aber dennoch mal anschauen. Also zumindest sowei man es ohne das geringste Verständnis von Kryptographie verstehen kann.

## Beispiel
Als Beispiel wird dem Nix-Store eine extrem einfache Datei hinzugefügt. Die Kommentare, die dabei auf der Standardausgabe ausgegeben werden, geben Aufschluss über die Zwischenergebnisse der Berechnung des Pfads im Store, an dem die Datei letztlich abgelegt wird. Insbesondere sagen sie uns, mittels welcher Kommandos sie erzeugt werden.

Der Inhalt der `myfile`-Datei ist denkbar einfach: sie enthält lediglich den `mycontent`-String. Für den Beitrag von Interesse ist vor allem die Hash-Komponente des Pfads. Wenn ich es richtig verstehe, dann werden Dateien und Verzeichnisse dem Store für *als Teil von Derivations* hinzugefügt.[^add]

Nix kennt den Out-Pfad noch bevor eine Derivation gebaut wurde. Das erkennt man daran, dass bereits die Store-Derivation über die Pfad-Information verfügt. Wie in einem früheren Kapitel besprochen handelt es sich dabei um die `.drv`-Datei, die bei der *Instanziierung* des Pakets (durch Aufruf der `derivation`-Funktion) erzeugt wird. Erst bei der *Realisierung* würde das Paket auch tatsächlich gebaut.

Im einfachsten Fall kann eine Store Derivation auf der Grundlage von drei Informationen erzeugt werden: `name`, `builder`, `system`.[^builder]
```
$ nix repl
nix-repl> derivation { system = "x86_64-linux"; builder = ./myfile; name = "foo"; }
«derivation /nix/store/y4h73bmrc9ii5bxg6i7ck6hsf5gqv8ck-foo.drv»
```
Als Nebeneffekt wird eine Datei erstellt, die auf den Ort der Store Derivation zeigt: `./myfile`. In einem früheren Kapitel wurde bereits gezeigt, wie die Inhalte von Store Derivations ge-pretty-print-et werden können:
```
$ nix derivation show /nix/store/y4h73bmrc9ii5bxg6i7ck6hsf5gqv8ck-foo.drv
{
  "/nix/store/y4h73bmrc9ii5bxg6i7ck6hsf5gqv8ck-foo.drv": {
    "outputs": {
      "out": {
        "path": "/nix/store/hs0yi5n5nw6micqhy8l1igkbhqdkzqa1-foo"
      }
    },
    "inputSrcs": [
      "/nix/store/xv2iccirbrvklck36f1g7vldn5v58vck-myfile"
    ],
    "inputDrvs": {},
    "platform": "x86_64-linux",
    "builder": "/nix/store/xv2iccirbrvklck36f1g7vldn5v58vck-myfile",
    "args": [],
    "env": {
      "builder": "/nix/store/xv2iccirbrvklck36f1g7vldn5v58vck-myfile",
      "name": "foo",
      "out": "/nix/store/hs0yi5n5nw6micqhy8l1igkbhqdkzqa1-foo",
      "system": "x86_64-linux"
    }
  }
}
```
In der Conclusion heißt es: "(...) Nix knows beforehand the out path of a derivation since it only depends on the inputs." Wenn ich die Informationen richtig lese, dann ist der Hash-Wert vom Out-Pfad der gleiche wie der vom einigen Input (unter `inputSrcs`). Wenn der Hash-Wert vom Out-Pfad (nur) auf der Grundlage der Hashes der Inputs berechnet wird, und wenn es nur einen Input gibt, sind die Hash-Werte *deshalb* identisch?[^name] Wäre die Situation anders, wenn wir verschiedene Inputts gehabt hätten?

## Zwischenschritte der Berechnung eines Store-Pfads
Die Frage scheint zunächst zu sein, wie Nix auf den Hash-Wert des *Inputs* gekommen ist. Warum also haben wir für unsere Datei `/nix/store/xv2iccirbrvklck36f1g7vldn5v58vck-myfile` erhalten? Bei der Berechnung lassen sich drei Ergebnisse unterscheiden.

### Ergebnis von Schritt 1: Hash des Dateiinhalts
Im ersten Zwischenschritt wird der Hash des Dateiinhalts berechnet. Ich hoffe ich blamiere mich hier nicht völlig, aber die Hashfunktion nimmt eine beliebig lange Zeichenkette als Argument und gibt eine kürzere Zeichenkette zurück, richtig?

Jedenfalls werden zwei Methoden gegeben, wie wir das Zwischenergebnis berechnen lassen können.
```
$ nix-hash --type sha256 myfile
2bfef67de873c54551d884fdab3055d84d573e654efa79db3c0d7b98883f9ee3

$ nix-store --dump myfile|sha256sum
2bfef67de873c54551d884fdab3055d84d573e654efa79db3c0d7b98883f9ee3
```
Es wird nicht auf die Unterschiede zwischen den beiden Befehlen eingegangen. Es wäre sicher auch nicht der richtige Ort, die Details der [SHA-256-Hashfunktion](https://en.wikipedia.org/wiki/SHA-2){: target="_blank"} zu erklären.

Es werden zwei Typen oder Formate von Dateiinhalten unterschieden: " In general, Nix understands two contents: flat for regular files, or recursive for NAR serializations which can be anything." Was genau sind rekursive Dateiinhalte und was unterscheidet sie von gewöhnlichen Dateiinhalten?

### Ergebnis von Schritt 2: String-Beschreibung
Wir lernen, dass im zweiten Schritt ein besonderer String konstruiert wird. Wir erfahren, dass dieser String drei Komponenten hat: Hash, Pfad-Typ und Dateiname. Leider wird nicht gesagt, was ein Pfad-Typ ist oder welche Typen es gibt. Es wird das folgende Beispiel gegeben:
```
"source:sha256:2bfef67de873c54551d884fdab3055d84d573e654efa79db3c0d7b98883f9ee3:/nix/store:myfile"
```
Hat dieser String wirklich nur drei Komponenten? Mir fällt es nicht leicht, die Komponenten abzugrenzen. Sind manche Teile feststehend?
```
"source:<Hash-Typ>:<Hash-Wert>:<Pfad-Typ>:<Dateiname>"
```
So vielleicht? Ich weiß es nicht.

Insbesondere wird von einer String-*Beschreibung* (*string description*) gesprochen. Seltsamerweise wird in keinem Wort darauf eingegangen, was genau der String beschreibt.

Es wird auch nicht gesagt, wie der String von Nix selbst gebaut wird. Scheinbar gibt es keinen Befehl, der den String als Ergebnis hat? Jedenfalls wird er im Beispiel per Hand erzeugt.
```bash
echo -n "source:sha256:2bfef67de873c54551d884fdab3055d84d573e654efa79db3c0d7b98883f9ee3:/nix/store:myfile" > myfile.str
```
Die Datei ist der Input für den dritten Schritt.

### Ergebnis von Schritt 3: Hash-Wert als Komponente des Store-Pfads unserer Datei
Im dritten Schritt schließlich wird aus dem String die Hash-Komponente des Store-Pfads berechnet. Wir erfahren, mit welchem Befehl wir die Berechnung durchführen lassen können:
```
$ nix-hash --type sha256 --truncate --base32 --flat myfile.str
xv2iccirbrvklck36f1g7vldn5v58vck
```
Gut, dass ist tatsächlich der Wert, den uns die Store Derivation gegeben hat.

Es wird auf einige Details der Berechnung eingegangen: Was wir berechnen ist "the base-32 representation of the first 160 bits (truncation) of a sha256" des obigen Strings. Um ehrlich zu sein, die kryptographischen Details interessieren mich nicht genug, um nähergehend zu recherchieren, was diese Dinge mathematisch genau bedeuten. String als Input, Magie, Hash-Wert als Output, das muss reichen.

## Output-Pfad
Damit haben wir den Hash-Wert des (einzigen) Input-Pfads (*source path*) berechnet. Was uns aber primär interessierte war der *Output*-Pfad der Derivation. Unsere `myfile`-Datei ist nur ein Teil dieses Pakets (wenn auch in unserem erkünstelten Beispiel der einzige Teil).

Im letzten Schritt wird nun schließlich gezeigt, wie der Hash-Wert des Output-Pfads berechnet wird. Oder zumindest mit welchen Befehlen wir ihn uns berechnen lassen können.
>  It's computed in a similar way to source paths, except that the `.drv` is hashed and the type of derivation is `output:out`. In case of multiple outputs, we may have different output:`<id>`.

Gut, vorher haben wir den Inhalt unserer `myfile` als Input an die Hashfunktion gegeben, nun geben wir den Inhalt der `.drv`-Datei.

Es wird aber auch von einem Derivationstyp (*type of derivation*) gesprochen. Was genau ist ein Derivationstyp? Und wo genau werden `output:out` und `output:<id>` eingesetzt? Ich dachte wir wollten einen Hash-Wert berechnen? Okay, ich sehe einige Schritte weiter: hier geht es scheinbar um eine Komponente der String-Beschreibung.

Der nächste Satz ist ähnlich enigmatisch:
> At the time nix computes the out path, the .drv contains an empty string for each out path.

Was ist *der* Out-Pfad und was sind die mehreren Out-Pfade für die unsere `.drv`-Datei zunächst (?) nur einen leeren String enthält? Ich habe das Gefühl, ich habe den Pfaden verloren. Worüber reden wir nochmal?

Jedenfalls wird zunächst der Zustand der Store Derivation simuliert, der zum Zeitpunkt besteht, zu dem der Out-Pfad der Derivation berechnet wird (*the .drv state in which nix is when computing the out path for our derivation*).
```bash
cp -f /nix/store/y4h73bmrc9ii5bxg6i7ck6hsf5gqv8ck-foo.drv myout.drv
sed -i 's,/nix/store/hs0yi5n5nw6micqhy8l1igkbhqdkzqa1-foo,,g' myout.drv
```
Zwar ist hier von der `.drv`-Datei geredet. Gemeint ist aber wahrscheinlich ein Zwischenzustand bei der Instanziierung auf Grundlage der Input-Attributmenge, also ein Zwischenzustand bei der Ausführung der `derivation`-Funktion? Wir nehmen unsere fertige `.drv`-Datei und gehen damit auf einen Zustand zurück, der bestand, *bevor* wir diesen Endzustand erreicht haben.

Mit dieser Zwischendatei machen wir nun genau die drei Schritte, die wir oben mit unserer `myfile`-Datei durchgeführt haben:
```
$ sha256sum myout.drv
1bdc41b9649a0d59f270a92d69ce6b5af0bc82b46cb9d9441ebc6620665f40b5  myout.drv
$ echo -n "output:out:sha256:1bdc41b9649a0d59f270a92d69ce6b5af0bc82b46cb9d9441ebc6620665f40b5:/nix/store:foo" > myout.str
$ nix-hash --type sha256 --truncate --base32 --flat myout.str
hs0yi5n5nw6micqhy8l1igkbhqdkzqa1
```
Moment, irgendwie hatte ich erwartet, wir würden den Pfad berechnen, an dem die `.drv`-Datei abgelegt wird. Was wir berechnet haben, ist der Wert eines anderen Attributs, `out`:
```
"env": {
  "builder": "/nix/store/xv2iccirbrvklck36f1g7vldn5v58vck-myfile",
  "name": "foo",
  "out": "/nix/store/hs0yi5n5nw6micqhy8l1igkbhqdkzqa1-foo",
  "system": "x86_64-linux"
}
```
Gut, das macht irgendwie Sinn, oder?

Nur frage ich mich nun, wie wäre `<Hash-Wert>` in `/nix/store/<Hash-Wert>-foo.drv` berechnet worden, wenn wir mehr als einen Input gehabt hätten? Gibt es Typen von Store-Pfaden, über die gar nicht gesprochen wurde?

## Offene Fragen
- Im Beispiel wird der Store-Pfad für eine Datei berechnet. Gilt völlig Analoges auch für die Berechnung von Store-Pfaden für Verzeichnisse?
- Im Beispiel scheint der Hash-Wert vom Out-Pfad gleich dem Hash-Wert des einzigen Inputs zu sein. Liegt das daran, dass es nur *einen* Input gibt? Ist das, was damit gemeint ist, dass die Hashes (nur) auf der Basis der Inputs berechnet werden?
- Kurz: Wie wird der Hash-Wert im Pfad zur Store Derivation berechnet?
- Was genau ist damit gemeint, dass Nix zwei Formate von Inhalten (*contents*) versteht, "flat for regular files" und "recursive for NAR serializations"? Was genau sind rekursive Dateiinhalte und wie unterscheiden sie sich von gewöhnlichen Dateiinhalten bei der Berechnung von Hashes?
- Was beschreibt die String-Beschreibung im zweiten Schritt?
- Im zweiten Schritt wird von einem Pfad-Typ gesprochen. Was genau ist damit gemeint und welche Typen gibt es?
- Was ist ein Derivationstyp (*type of derivation*).

## Fußnoten
[^add]: Es scheint möglich, eine Datei direkt (ohne sie zu verpacken) dem Store hinzuzufügen. Dafür hätten wir `nix-store --add myfile` nutzen können. Der Hash-Wert wäre der gleiche gewesen.
[^builder]: Die Tatsache, dass `myfile` nicht tatsächlich etwas bauen kann, scheint hier keine Rolle zu spielen.
[^name]: Die Pfade selbst sind nicht identisch. Im einen Fall ist die letzte Pfad-Komponente der Derivation-Name, im anderen Fall der Dateiname.
