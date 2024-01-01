---
layout: post
title:  "Nix Pills (11)"
tags: nix nixos
excerpt_separator: <!--more-->
---

# Nix Pills, Chapter 11 (Kommentar)
<div class="hide-excerpt">
Im elften Kapitel der Nix Pills, <a href="https://nixos.org/guides/nix-pills/garbage-collector" target="_blank">Garbage Collector</a>, wird erklärt, wie Store Derivations und Outputs aus dem Nix Store entfernt werden können. Nix nutzt dazu einen Garbage Collector.
</div>
<!--more-->

Im elften Kapitel der Nix Pills, [Garbage Collector](https://nixos.org/guides/nix-pills/garbage-collector){: target="_blank"}, wird erklärt, wie Store Derivations und Outputs aus dem Nix Store entfernt werden können. Wie in vielen modernen Skriptsprachen nutzt Nix dazu einen Garbage Collector.

## Welche Software-Komponenten werden aufgeräumt?
Nix umfasst ein Werkzeug, mit dem nicht mehr länger genutzte Software-Komponenten aus dem Nix-Store entfernt werden. Pakete werden *nicht* dadurch (schon) entfernt, dass sie deinstalliert werden. Zwar sind sie dadurch nicht mehr für einen Benutzer (in einer Nutzerumgebung) verfügbar. Doch erst eine gründliche Bereinigung durch `nix-collect-garbage` löscht Outputs aus dem Store.
```
$ nix-collect-garbage
finding garbage collector roots...
[...]
deleting unused links...
[...]
```
Gargabe Collection ist ein automatischer Vorgang. Es stellt sich deshalb die Frage, wie der Collector entscheidet, ob eine Derivation im Store nicht mehr verwendet wird.

Nix implementiert ein Kriterium analog zu dem in anderen Sprachen mit einem Garbage Collector. Zentral dabei ist die Idee von *GC Roots*.
>  Programming languages with a garbage collector have an important concept in order to keep track of live objects: GC roots. A GC root is an object that is always alive (unless explicitly removed as GC root). All objects recursively referred to by a GC root are live.
>
> Therefore, the garbage collection process starts from GC roots, and recursively mark referenced objects as live. All other objects can be collected and deleted.

In Nix sind GC Roots keine Objekte, sondern Store-Pfade.

Wenn ich die Erklärung richtig verstehe, dann trackt Nix diese besonderen Pfade über Symlinks, die auf die GC Roots zeigen und sich im Verzeichnis `/nix/var/nix/gcroots` finden. Die Nix Pills sprechen diesbezüglich von einer *Liste*. Eine Derivation gilt als aktiv, wenn sie direkt in dieser Liste vorkommt.
>  So we have a list of GC roots. At this point, deleting dead store paths is as easy as you can imagine. We have the list of all live store paths, hence the rest of the store paths are dead.

Wenn ich es richtig lese, dann definiert das Zitat ein Kriterium für GC Roots. Wenn ein Symlink in diesem Verzeichnis auf einen Pfad zeigt, dann handelt es sich (deshalb) um eine GC Root.

Bezüglich dieses Zitats erscheinen mir zwei Dinge überraschend. Zunächst ist mir nicht ohne Weiteres klar, warum die Verzeichnisstruktur als *Liste* konzeptualisiert wird. Nach meinem Verständnis sind Listen hierarchisch flach. Das passt insbesondere nicht zu einem weiteren angeführten Merkmal: "Nix allows this directory to have subdirectories: it will simply recurse directories in search of symlinks to store paths."

Das Zitat liest sich, als ob nur GC Roots nicht vom Garbage Collector aufgeräumt würden. In dem Fall wäre die Rede von *Wurzeln* (*roots*) mehr als überraschend. Wahrscheinlich können GC Roots (über Pfade) auf andere Derivations zeigen und diese Pfade werden ebenfalls nicht gelöscht? Wahrscheinlich ist das damit gemeint, dass "all objects recursivelely referred to by a GC root are life" (wie es im vorausgegangen Zitat hieß).

Diese Interpretation findet sich später bestätigt: "What's left in the nix store (after running `nix-collect-garbage`) is everything being referenced from the GC roots."

Wir erfahren auch, dass der Garbage Collector mit einem Papierkorb arbeitet:
> (...) Nix first moves dead store paths to `/nix/store/trash` which is an atomic operation. Afterwards, the trash is emptied.

Der Hinweis darauf, dass es sich dabei um eine atomare Operation handelt. Damit ist vermutlich gemeint, dass das Verschieben einer Derivation in den Papierkorb völlig unabhängig von anderen Store-Komponenten ist.

Mir ist nicht klar, wann "afterwards" ist. Vermutlich wird der Papierkorb nicht schon während des GC-Vorgangs geleert? Wenn es aber nicht automatisch passiert, wann genau passiert es? Muss dafür manuell ein Befehl ausgeführt werden?

Wir erfahren, dass sich alle GC Roots mit `nix-store --gc --print-roots` anzeigen lassen. Dieser Punkt wird nicht besonders hervorgehoben, aber scheinbar kann die GC Wurzel einer gegebenen Derivation mit einem Unterbefehl von `nix-store` ermittelt werden:
```bash
nix-store -q --roots `which fortune`
```

## Profile und ihre Generationen als GC Roots
Es wurde bereits festgestellt, dass Derivations durch eine Deinstallation nicht aus dem Store entfernt werden.
```
$ nix-env -iA nixpkgs.bsdgames
$ readlink -f `which fortune`
/nix/store/b3lxx3d3ggxcggvjw5n0m1ya1gcrmbyn-bsd-games-2.17/bin/fortune
$ nix-env -e bsd-games
uninstalling `bsd-games-2.17'
$ ls /nix/store/b3lxx3d3ggxcggvjw5n0m1ya1gcrmbyn-bsd-games-2.17
bin  share
```

Es ist gar nicht so einfach, den Garbage Collector dazu zu bewegen, eine Derivation zu entfernen. Um den Grund dafür zu verstehen, muss zunächst bemerkt werden, dass *Nutzerverzeichnisse* (*user environments*) als GC Roots dienen.
```
$ nix-env -iA nixpkgs.bsdgames
$ nix-store -q --roots `which fortune`
/nix/var/nix/profiles/default-9-link
```
In diesem Kontext erfahren wir, dass nicht nur die Symlinks in `/nix/var/nix/gcroots` darüber entscheiden, welche Store-Pfade als GC Roots betrachtet werden. Den gleichen Effekt hat `/nix/var/nix/profiles`, auch die darin enthaltenen Symlinks machen die Ziele zu GC Roots.[^nixos]

Wenn ein Programm für einen Benutzer installiert ist, dann zeigt ein Symlink auf die entsprechende Derivation. Damit ist sie noch aktiv und wird beim Garbage Collection nicht entfernt.
```
$ nix-collect-garbage
[...]
$ ls /nix/store/b3lxx3d3ggxcggvjw5n0m1ya1gcrmbyn-bsd-games-2.17
bin  share
```

Es überrascht vielleicht, dass der Garbage Collector die Derivation auch dann nicht ohne Weiteres entfernt, wenn der aktive Benutzer sie (für sich) deinstalliert. Im Falle eines Mehrbenutzersystems scheint die Erklärung auf der Hand zu liegen. Es könnte sein, dass ein anderes Profil noch auf die Derivation zeigt und dass sie deshalb nicht vom Garbage Collector entfernt wird.

Interessanterweise wird der Garbage Collector für eine Derivation auch bei einem Einbenutzersystem nicht unbedingt wirksam, wenn dieser sie deinstalliert hat. Das heißt das zuletzt zitierte Code-Listing bleibt in diesem Fall möglich und sogar wahrscheinlich.

Der Grund dafür sind *Generationen* von Nutzerumgebungen. Im Beitrag wurde zunächst die `bsd-games`-Derivation installiert. Wie der Pfad anzeigt (`/nix/var/nix/profiles/default-9-link`), führte diese Operation zur neuten Generation der aktiven (Default-)Nutzerumgebung. Wenn in der Zwischenzeit keine weiteren `nix-env`-Operationen ausgeführt wurden, dann ergibt sich aus der Deinstallation die zehnte Generation.

In der zehnten Generation wurde `bsd-games` deinstalliert, das heißt es findet sich in der Umgebung kein Symlink mehr zur Derivation im Store. Die Tatsache, dass der Garbage Collector die Derivation dennoch weiterhin als aktiv erachtet erklärt sich dadurch, dass die neunte Generation und mithin der darin enthaltene Symlink noch immer vorhanden ist.

Wenn wir die neunte Generation entfernen,[^generationen-entfernen] so verschwindet damit (zumindest wie das Beispiel konstruierte wurde) auch der letzte Link auf die `bsd-games`-Derivation im Store. Damit ist sie nicht mehr aktiv und der Garbage Collector wird sie entfernen.
```
$ rm /nix/var/nix/profiles/default-9-link
$ nix-env --list-generations
[...]
   8   2014-07-28 10:23:24
  10   2014-08-20 12:47:16   (current)
$ nix-collect-garbage
[...]
$ ls /nix/store/b3lxx3d3ggxcggvjw5n0m1ya1gcrmbyn-bsd-games-2.17
ls: cannot access /nix/store/b3lxx3d3ggxcggvjw5n0m1ya1gcrmbyn-bsd-games-2.17: No such file or directory
```

## `nix-build`-Outputs sind GC Roots
`nix-build` erzeugt einen `result`-Symlink in dem Verzeichnis, indem der Befehl ausgeführt wurde. Wir erfahren nun, dass das nicht der einzige Nebeneffekt ist. In einem Unterverzeichnis, `/nix/var/nix/gcroots/auto`, werden durch den Befehl automatisch Symlinks zu den `result`-Symlinks erstellt.

Im voraugegangenen Kapitel wurde die GNU Hello Derivation gebaut. Dazu wurde `nix-build hello.nix` im Home-Verzeichnis ausgeführt. Wir erhalten deshalb:
```
$ ls -l /nix/var/nix/gcroots/auto/
total 8
drwxr-xr-x 2 nix nix 4096 Aug 20 10:24 ./
drwxr-xr-x 3 nix nix 4096 Jul 24 10:38 ../
lrwxrwxrwx 1 nix nix   16 Jul 31 10:51 xlgz5x2ppa0m72z5qfc78b8wlciwvgiz -> /home/nix/result/
```

GNU Hello ist ein gutes Beispiel. Da die Derivation ihrer Natur nach keine Dependency eines anderen Pakets sein wird, wird im Store an keiner Stelle auf Hello verlinkt. Da wir das Programm noch ausführen können, nachdem der Garbage Collector ausgeführt wurde, muss es vom Collector als noch aktiv klassifiziert worden sein. Somit muss es sich um eine GC Root handeln. Woraus ergibt sich dieser Status?

Es wurde bereits gesagt, dass auch die Symlinks in Unterverzeichnissen vom `gcroots` die Symlink-Ziele zu GC Roots machen. Nun erfahren wir, dass die Zeile nicht nur "richtige" Dateien, sondern selbst wiederum Symlinks sein können. Wenn ein `gcroots`-Symlink auf einen Symlink zeigt, dann sprechen wir bezüglich des Ziels von einer *indirect GC root*. Vor allem aber lernen wir, dass alle Build-Resultate GC Roots sind und damit vom Garbage Collector nicht ohne Weiteres gelöscht werden.

Das wirkt erstmal unschön. Es ist ziemlich willkürlich, wo `nix-build` ausgeführt wird; dennoch wird der daraus resultierende Symlink zu einem Kriterium, ob die entsprechende Derivation aus dem Store entfernt wird. Wenn wir eine so gebaute Derivation aus dem Store entfernen möchten, müssen wir den Build-Ordner und den darin enthaltenen `result`-Symlink identifizieren.

Das Design ist weniger problematisch, wenn man bemerkt, dass der Garbage Collector auch Dateien in  `/nix/var/nix/gcroots/auto` automatisch entfernt, wenn das Ziel (der `result`-Symlink) entfernt wurde. Wenn das Vorgehen methodisch nicht überzeugen kann, das Resultat hinterlässt zumindest keine unsauberen Rückstände.

Es wird eine andere Methode angeführt, dessen Ergebnis weitaus weniger sauber ist. Wir könnten einfach den Inhalt aus `/nix/var/nix/gcroots/auto` entfernen. Im Falle einer Derivation, die (wie GNU Hello) keine Dependency einer anderen Derivation im Store ist, würde der Garbage Collector sie also entfernen. Doch der `result`-Symlink würde nun ins Nichts zeigen (*dangling reference*). In dem Fall ließe der Garbage Collector Garbage links liegen.

Die Methode scheint mir schon deshalb kaum nennenswert, da man den Symlinks in `/nix/var/nix/gcroots/auto` nicht ohne Weiteres ansieht, welche Derivation sie repräsentieren. Es wird nicht darauf eingegangen, woraus sich ihre Namen ergeben; jedenfalls sind sie nicht sprechend und im Bestenfall zeigt nur das `result`-enthaltene Verzeichnis an, welche Derivation gebaut wurde.

## Das System gründlich aufräumen
In den beiden vorausgeganenen Absätzen wurden zwei Gründe dafür angesprochen, warum viel Müll im Nix-Store entsteht. Vor allem entsteht viel Müll, der vom Garbage Collector nicht ohne Weiteres als solcher identifiziert wird. Zum einen entstehen durch die Verwendung von `nix-env` neue Generationen; zum anderen entstehen durch `nix-build` neue indirekte GC Roots. Beide Mechanismen sorgen dafür, dass Derivations im Stor vor dem Garbage Collector geschützt werden.

Die Nix Pills definieren eine Befehlsfolge, die zu einem (nahezu) sauberen Sytem führen.
```
$ nix-channel --update
$ nix-env -u --always
$ rm /nix/var/nix/gcroots/auto/*
$ nix-collect-garbage -d
```

Die ersten drei Befehle dienen dazu, die GC-behindernden Nebeneffekte von `nix-build` zu beheben. Es werden neue Versionen der Derivation-definierenden Ausdrücke aus Nixpkgs heruntergeladen und angewendet. Dadurch entstehen neue Store-Outputs, die aktuellere Versionen der (für den aktiven Benutzer) installierten Derivations repräsentieren. Nun brauchen wir die mit `nix-build` selbst erzeugten Outputs nicht mehr. Wenn die damit assozierten indirekten GC roots entfernt werden, dann wird der Garbage Collector sie entfernen.

Der Autor gibt zu, dass dieses Vorgehen nicht optimal ist. An den Orden, an denen die Builds ausgeführt wurden, bleiben nun tote `result`-Symolinks zurück. Darüber hinaus habe ich mich gefragt, ob überhaupt garantiert ist, dass wir alle unsere Derivations behalten würden. Kann es nicht sein, dass gar keine aktuellere Version eines Programms vorliegt und dadurch durch das Upgrade kein neuer Build-Output erzeugt würde? Würden wir dann nicht die "alte" Derivation ersatzlos löschen?

Die letzte Zeile (`nix-collect-garbage -d`) hat den Effekt, dass alle alten Generationen gelöscht und dann erst die Garbage Collection durchgeführt wird. Natürlich ist dadurch sichergestellt, dass keine alte Generation mehr verhindern wird, Derivations zu entfernen.

Auch dieser Befehl erscheint mir vergleichsweise drastisch. Wie der Autor selbst sagt, sollten wir dadrauf nur dann zurückgreifen, wenn die aktuelle Generation ganz sicher stabil und voll funktionsfähig ist.

Was bleibt ist ein sicher nicht gänzlich befriedigendes Verfahren, um sein System aufzuräumen.

## Offene Fragen
- Warum spricht der Beitrag beüglich `/nix/var/gcroots` von einer *Liste*?
- Wann wird der Nix-Papierkorb geleert?
- Welche Verzeichnisse bestimmten GC Roots?
- Woraus ergibt sich der Name der Symlinks in `/nix/var/nix/gcroots/auto`? Auf welcher Grundlage werden die Hash-Werte berechnet?
- Ist sichergestellt, dass wir durch die Befehlsfolge `nix-channel --update && nix-env -u --always && rm /nix/var/nix/gcroots/auto/*` wirklich notwendig alle unsere installierten Derivations behalten?

## Fußnoten
[^nixos]: Wir erfahren von einem weiteren Verzeichnis, das über den GC Root Status von Store-Pfaden entscheidet: `/run/booted-system`. Dieses Verzeichnis wird von NixOS genutzt. Es wird keine vollständige Liste dieser bedeutungsvollen Verzeichnisse gegeben.
[^generationen-entfernen]: Es wirkt vielleicht etwas radikal, die Generation einfach mit `rm` zu entfernen. Viele werden sich fragen, ob Nix von der Änderung weiß. Deshalb wird klargestellt: "`nix-env --list-generations` does not rely on any particular metadata. It is able to list generations based solely on the file names under the profiles directory." Das gilt vermutlich für alle auf Generationen bezogene Kommandos.
