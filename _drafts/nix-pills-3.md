---
layout: post
title:  "Nix Pills (3)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 3 (Kommentar)
<div class="hide-excerpt">
Im dritte Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/why-you-should-give-it-a-try" target="_blank">Enter the Environment</a>, geht es primär um Benutzerumgebungen. Wir lernen, wie Programme installiert und wie wir zwischen Generationen wechseln können. Zusätzlich lernen wir, wie man direkt mit dem Store interagieren kann und von welchen Quellen Pakete in der Regel installiert werden.
</div>
<!--more-->

Das dritte Kapitel der Nix Pills ist mit [Enter the Environment](https://nixos.org/guides/nix-pills/enter-environment) übertitelt. Wie zu erwarten ist, lernen wir ein paar Dinge über Benutzerumgebungen (*user environments*) und wie wir sie mit dem `nix-env`-Tool verwalten.

Der Fokus liegt auf der *Installation* von Programmen. Vor diesem Hintergrund werden einige weitere Themen angesprochen, die über Umgebungen hinauszeigen. Programme zu installieren, bedeutet dass wir für einen Benutzer (in ihrem *Profil*) verfügbar gemacht werden. Um Änderungen der Nutzerumgebung *rückgängig* machen zu können, führt jede Umgebungsoperation zu einer neuen *Generation*. Wir lernen, wie man zwischen Generationen wechseln kann.

Noch bevor Programme einem Benutzer zugänglich gemacht werden, bedeutet sie zu installieren zunächst, dass sie gebaut und die Outputs in einem geeigneten Verzeichnis abgelegt werden. Wir haben bereits gesehen, dass Build-Outputs im Nix-Store gespeichert werden. Wir lernen, wie direkt mit dem Store interagiert werden kann. Demnach vereinfachen Umgebungen die Interaktion, sind aber keine essentielle Komponente.

Als ein Beispiel wird eine Situation angesprochen, in der Nix aus der gegebenen Nutzerumgebung entfernt wurde. Wir können eine voll funktionsfähige Nix-Umgebung wieder herstellen, indem direkt auf die noch immer im Store verfügbaren Tools zugegriffen wird.

## Installation von Paketen
Spezifische Versionen bzw. Varianten von Programmen können installiert werden, indem die entsprechenden Pakete in einer gegebenen Benutzerumgebung installiert werden. Zur Verwaltung von Umgebungen dient das CLI-Tool `nix-env`.

Zur Installation von Programmen kann der folgende Unterbefehl verwendet werden:
```bash
nix-env -i <Paketname>
```

Wie in der Einleitung angedeutet, involviert die Installation von Programmen zwei Schritte. Zunächst wird das entsprechende Paket *gebaut* und der Output im Nix-Store abgelegt. Der Beitrag betont, dass Installationen immer *für einen bestimmten Benutzer* vorgenommen werden. Deshalb wird im zweiten Schritt die Umgebung dieses Benutzers angepasst wird, um das Programm für den Benutzer zugänglich zu machen.

Beide Schritte finden sich in der Ausgabe der Installation auf der Kommandozeile. Hier das im Beitrag angeführte Beispiel:
```
$ nix-env -i hello
installing 'hello-2.10'
[...]
building '/nix/store/0vqw0ssmh6y5zj48yg34gc6macr883xk-user-environment.drv'...
created 36 symlinks in user environment
```
GNU Hello ist ein sehr einfaches Werkzeug, um `Hello world` auf der Standardausgabe anzuzeigen. Es versteht sich, dass es primär für Testzwecke verwendet wird.

Es wird betont, dass Programme über den Derivationsnamen installiert werden. Und "betont" ist hier wörtlich zu nehmen:
> We installed `hello` by derivation name minus the version. I repeat: we specified the **derivation name** (minus the version) to install it.

Ich habe das Gefühl, ich verstehe nicht recht die angedeutete Wichtigkeit dieses Punkts. Soll damit gesagt werden, dass die Versionsnummer nicht angegeben werden muss? Im Anbetracht der Tatsache, dass bei Nix verschiedene Versionen des gleichen Programms zugleich installiert sein können, ist das vielleicht überraschend.

Die Ausgabe zeigt an, welche Version bzw. welche Derivation installiert wird. Da die Version nicht explizit ausgewählt wurde, stellt sich die Frage, warum genau sich der Paketmanager für diese Version entschieden hat. Es ist zu vermuten, dass die festgelegten *Kanäle* darüber entscheiden. Zwar gibt es einen Abschnitt über Kanäle; doch diese Frage wird darin nicht explizit beantwortet.

Die Ausgabe wirft für mich zwei weitere Fragen auf:
- Bei der Installation wird eine `.drv`-Datei gebaut. Die Endung steht vermutlich für *Derivation* und wir wissen bereits, dass Derivations gebaut werden können. In der inria-Serie haben wir Derivations aber als *Funktionen* kennengelernt, und diese Funktionen wurden in `.nix`-Dateien abgespeichert. Was sind also `.drv`-Dateien und in welcher Beziehung stehen sie zu Derivations?
- `hello` ist ein sehr einfaches Programm. Der Umstand, dass dafür sage und schreibe 36 Symlinks in der Nutzerumgebung erstellt werden, wird deshalb überraschen. Was sind all diese Links?

## Umgebungskomponenten zeigen auf den Nix-Store
In Kapitel 2 haben wir bereits gesehen, dass Komponenten der Nutzerumgebungen Symlinks sind, die auf Outputs im Nix-Store zeigen. Bisher wurde die etwas untypische Situation angesprochen, die direkt nach der Nix-Installation eingerichtet wird. Der Standardfall stellt sich ein, sobald neben `nix` selbbst noch weitere Pakete in der Umgebung eines Benutzers installiert werden.

Das aktive Profil wird durch `~/.nix-profile` repräsentiert. Wenn ich das vorausgegangene Kapitel richtig verstanden habe, dann wird ein (*das*?) Default-Profil verwendet, wenn Nix für nur einen Benutzer eingerichtet wird. Die Umgebung des Benutzers, der durch dieses Profil repräsentiert wird, erreichen wir über eine Serie von Verlinkungen.

Bisher war es so, dass `~/.nix-profile` (unter anderem) eine *Verlinkung* namens `bin` enthielt. Die Verlinkung zeigte auf ein Unterverzeichnis des einzigen installierten Pakets, `nix`.[^version] Nach dem vorausgegangenen Abschnitt haben wir wenigstens ein weiteres Programm, das für den aktiven Benutzer installiert wurde: `hello`. Dieser Umstand transformiert die Verzeichnisstruktur.

`bin` ist nun ein richtiges Verzeichnis, kein Symlink zu einem Verzeichnis.[^kein-link] Die prinzipielle Struktur bleibt aber erhalten: `bin` ist nun ein Verzeichnis, dass Symlinks zu Binärdateien aller ausführbaren Programme im Store enthält, die für den aktiven Benutzer installiert sind.

Hier das Beispiel aus dem Beitrag:[^man]
```
$ ls -l ~/.nix-profile/bin/
[...]
man -> /nix/store/83cn9ing5sc6644h50dqzzfxcs07r2jn-man-1.6g/bin/man
[...]
nix-env -> /nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3/bin/nix-env
[...]
hello -> /nix/store/58r35bqb4f3lxbnbabq718svq9i2pda3-hello-2.10/bin/hello
[...]
```

Nach der Nix-Installation zeigte `bin` auf das gleichnamige Unterverzeichnis von `nix`. Dadurch wurden dem aktiven Benutzer die Anwendungen zugänglich gemacht, die zu Nix selbst gehören (`nix-env`, `nix-store`, `nix-channel` etc.). Diese Anwendungen sind dem Benutzer weiterhin verfügbar.

Das geschieht dadurch, dass das normale `bin` Verzeichnis Symlinks enthält, die direktt auf die ausführbaren Binärdateien zeigen. Wenn wie hier `nix-2.1.3` installiert ist, zeigt `nix-env` beispielsweise auf die Datei `/nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3/bin/nix-env`.

## Direkte Interaktionen mit dem Nix-Store
Wenn ich ihren Zweck richtig verstehe, dann dienen Nutzerumgebungen dazu, verschiedenen Benutzern die im Store verfügbaren Programme auf unkomplizierte Weise zugänglich zu machen. Das heißt auf eine Weise, die von Aspekten wie Versionsnummern und Hash-Werten abstrahiert und es erlaubt, wie gewohnt über Dateinamen auf sie zuzugreifen. Sie sind demnach vergleichbar mit der `PATH`-Umgebungsvariable, zugeschnitten auf die Komplexität der Nix-spezifischen Paketverwaltung.

Das bedeutet nicht, dass Store-Inhalte *nur* über Nix-Umgebungen erreichbar sind. Wenn der aktive Benutzer über die notwendigen Rechte verfügt, dann kann er *direkt* mit dem Nix-Store interagieren. Im Beitrag werden zwei Aspekte angesprochen: Es wird gezeigt, wie Informationen über im Store installierte Pakete *abgefragt* werden können; und es werden Programme unabhängig von Benutzerumgebungen ausgeführt.

Für den zweiten Fall wird ein Szenario angesprochen, bei dem der Benutzer Nix selbst deinstalliert hat. Man könnte denken, dass Nix dadurch in einem grundlegenden Sinne entfernt wurde. Ohne Nix kann man auch nichts mit Nix installieren, richtig? Und wenn man nichts installieren kann, dann kann man auch Nix nicht installieren.

Der Abschnitt *Recovering the hard way* dient wohl primär dem Ziel, das darin ausgedrückte Missverständnis auszuräumen. Programme werden *für Benutzer* installiert und deinstalliert. Wenn ein Benutzer etwas deinstalliert, bedeutet das *nicht*, dass sie dadurch vom Computer entfernt werden.

So auch bei Nix selbst. Wenn ein Benutzer Nix deinstalliert hat, dann sind `nix-env` und ähnliche Anwendungen nicht mehr ohne Weiteres verfügbar. Doch wenn sie nicht radikal aus dem Store gelöscht wurden, dann finden sie sich noch immer an den im vorausgegangenen Abschnitt angesprochenen Orten.

Wir könnten also `nix-env` verwenden, um Nix erneut für den Benutzer zu installieren:
```bash
/nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3/bin/nix-env -i /nix/store/ig31y9gfpp8pf3szdd7d4sf29zr7igbr-nix-2.1.3/bin/nix-env 
```

Das Beispiel demonstriert damit auch den praktischen Nutzen von Benutzerumgebungen.

## Abhängigkeiten zwischen Paketen
Für Store-Abfragen (*queries*) dient ein Unterbefehl des `nix-store`-Tools: `nix-store --query`. Wir erfahren, wie man sich damit Dependencies von Paketen im Store anzeigen lassen kann.

Im gewöhnlichen Fall möchte man sich vielleicht die Dependencies eines gegebenen Pakets anzeigen lassen. Hier ist zwischen zwei Fällen zu unterscheiden: Im ersten Fall interessieren uns die *direkten* Abhängigkeiten. Sie lassen sich abfragen mit:
```bash
nix-store -q --references <Derivation>
```
Der Unterbefehl `-q` steht für `--query`, also für Anfragen an den Store.

`<Derivation>` ist ein Unterverzeichnis des Nix-Store. Da die Paketpfade Hashwerte enthalten, kennt man die Pfade aller Wahrscheinlichkeit nach nicht. Wenn die in Frage stehende Derivation für den aktiven Benutzer installiert ist, können wir sie uns mit `which <Programmname>` anzeigen lassen:
```bash
nix-store -qR `which bash`
```

Häufig interessieren uns nicht nur die direkten Dependencies, sondern wiederum ihre Dependencies, bis hin zu den Paketen, die nicht selbst wiederum von anderen Paketen abhängen. In technischen Worten: uns interessiert die *Closure* einer Derivation.

Dieses Konzept wird im Beitrag wie folgt definiert: "The closures of a derivation is a list of all its dependencies, recursively, including absolutely everything necessary to use that derivation." Meinem Sprachverständnis nach stimmt grammatisch etwas nicht mit dieser Definition: Die Closure*s* (Plural) sind *eine* Liste (Singular). Sind verschiedene Closures je eine Liste?

Vielleicht bin ich voreingenommen durch andere Dinge, die ich darüber gelesen habe. Ich denke korrekt wäre es zu sagen, dass es für jede Derivation *eine* Closure gibt, und dass es sich dabei um eine Liste all der Pakete handelt, von denen sie abhängt. Ich denke die Derivation selbst ist Teil dieser Liste.

Eine bloße Liste gibt keien Auskunft über die Abhängigkeitsstruktur zwischen ihren Elementen. Wir lernen, wie man sich diese unstrukturierte Liste anzeigen lassen kann:
```bash
nix-store -qR <Derivation>
```
Dabei handelt es sich um eine Kurzschreibweise für `nix-store --query --references <Derivation>`.

Wir haben gelernt, dass die darüber ermittelten Pakete aus dem Store *exportiert* und in einem Store auf einer anderen Maschine *importiert* werden können. Der dazu notwendige Unterbefehl wird in den Nix Stores erwähnt, aber diese Deployment-Möglichkeiten werden erst in zukünftigen Beiträgen weiterverfolgt. Die Dependencies-Liste kann als Import für einen anderen `nix-store`-Unterbefehl dienen: `nix-store --export`. Bei `nix-copy-closures` werden automatisch die auf der Zielmaschine noch fehlenden Dependencies ermittelt und eingerichtet.

Mit einem anderen Befehl können die *Abhängigkeitsbeziehugen* (in übersichtlicher Form) abgefragt werden:
```bash
nix-store -q --tree <Derivation>
```

## Kanäle
Bei vielen Paketmanagern können Anwender die Frage ausblenden, von welchen Quellen Paketen installiert werden. Nix expliziert diesen Aspekt über das Konzept der *Kanäle* (*channels*).[^flakes] Kanäle zeigen auf Branches von Paket-Repositories. Sie enthalten Paket-*Definitionen*, *Derivations* im Nix-Jargon. Wir haben an anderer Stelle bereits gelernt, dass Derivations in Nix immer auch Build-Anweisungen enthalten.  

Kanäle werden mit dem `nix-channel`-Tool verwaltet. Im Beitrag wird auf zwei Unterbefehle eingegangen.

Mit `nix-channel --list` können wir uns alle Kanäle auflisten lassen, über die wir gegenwärtig Pakete installieren können. Es wird gesagt, dass in vielen Szenarien nur *ein* Kanal eingerichtet wird. Wir erfahren auch, dass der Unterbefehl im Wesentlichen den Inhalt von `~/.nix-channels` ausgibt.

Wir wird darauf hingewiesen, dass die Kanalnamen der aufgelisteten Kanäle abweichen können, wenn Nix als Teil von NixOS installiert ist. Es wird nicht erläutert, ob sich nur die *Namen* unterscheiden oder ob es sich um vergleichbare aber *verschiedene* Kanäle handelt.

In gewöhnlichen Paketmanagern können alle installierten Pakete mit einem allgemeinen *Update* auf den neuesten Stand gebracht werden. In Nix lassen sich Kanäle *updaten*. Es wird gesagt, dass diese Nix-Operation vergleichbar ist (*quite similar*) mit dem, was bei anderen Paketmanagern bei einem Update passiert. Das heißt vermutlich, dass installierte Anwendungen auf den neuesten Stand gebracht werden.

Im ersten Kapitel wurde betont, dass Nix im Grunde kein Konzept von einem aktuellen Systemstand hat. Es stellt sich deshalb die Frage, was beim Update genau passiert. Hier die Erklärung, die von den Nix Pills gegeben wird:
> (...) `nix-channel --update` (...) will download the new Nix expressions (descriptions of the packages), create a new generation of the channels profile and unpack it under `~/.nix-defexpr/channels`.

Wenn unsere Annahme richtig ist, dann werden die dort abgelegten Definitionen genutzt, um aktuellere Versionen der installierten Programme einzurichten. Durch Kanal-Updates wird lokal der späteste Stand (Commit) eines Branches abgebildet. Und da den Kanälen entsprechende Branches die Idee früherer und späterer Versionen inkorporieren, kann Nix darüber die *aktuellen* Versionen von Paketen identifizieren.

Das klingt alles sehr ähnlich wie das, was beim traditionellen Ansatz passiert. Tatsächlich gilt diese Herangehensweise mittlerweile als überholt. Vor allem wird sie nicht dem Anspruch an perfekte *Reproduzierbarkeit* gerecht, den Nix stellt. In einem späteren Beitrag wird gezeigt, wie *Flakes* und ihre Inputs die Idee von Kanälen ersetzen können.

## Generationen
Mit jeder Änderungen der Umgebung eines Benutzers wird eine neue *Generation* erstellt. Dadurch wird es möglich, zu einem *früheren* Stand zurückzukehren. Falls gewünscht kann danach zu einem *späteren* Stand zurückgekehrt werden. Der Einfachheit halber, werden Generationen Zahlen durchnummeriert.

## Offene Fragen
- Welcher Mechanismus entscheidet darüber, welche Version eines Programms installiert wird, wenn nur der Derivationsname ohne eine Version angegeben wird?
- Was sind `drv`-Dateien und welchem Zweck dienen sie?
- Warum werden für ein so einfaches Programm wie `Hello` 36 Symlinks erstellt?
- Dient `nix-store` nur für Abfragen oder können dadurch auch Änderungen am Store vorgenommen werden?
- Verwendet NixOS lediglich verschiedene Namen für dieselben Kanäle oder handelt es sich um andere aber vergleichbare Kanäle?

## Fußnoten
[^version]: Genauer müsste man vielleicht sagen: `bin` zeigte auf ein Unterverzeichnis einer *bestimmten Version* von `nix`.
[^kein-link]: Dies erkennen wir daran, wenn wir uns den Inhalt des Profil-Verzeichnisses mit `ls` auflisten lassen (`ls -l ~/.nix-profile/`). Anders als beispielsweise `etc` zeigt `bin` auf kein anderes Verzeichnis (kein `->` hinter dem Namen).
[^man]: Im Beitrag wird auf die Installation von `man` eingegangen. Sie ist für die Thematik des Abschnitts nicht wesentlich. Es ist aber vielleicht interessant anzumerken, dass die üblichen Manuals für Nix über den Derivationsnamen `man-db` installiert werden können.
[^flakes]: In neueren Versionen von Nix gibt es ein Feature, das die Paketquellenverwaltung über Kanäle ersetzt. In sogenannten *Flakes* werden die Inputs direkt angegeben.
