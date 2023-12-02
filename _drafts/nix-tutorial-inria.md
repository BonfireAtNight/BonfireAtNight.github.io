---
layout: post
title:  "Nix Tutorial"
tags: nix nixos
---

# Nix-Tutorium vom Nationalen Forschungsinstitut für Informatik und Automatisierung (INRIA): Teil 1

## Installation von Paketen
Ein Paket zu installieren, involviert zwei Schritte. Ausführbare Dateien und Bibliotheken werden zur Verteilung und Verwaltung zu Paketen zusammengefügt, aber Endanwender interessieren sich für gewöhnlich nur für ihre *Inhalte* (*content*). Paketmanager ziehen sie sich entweder aus einem Cache oder es werden Quelldateien heruntergeladen, aus denen Anwendungen und Bibliotheken selbst gebaut werden.

Installierte Pakete müssen den Benutzern eines Betriebssystems zugänglich gemacht werden. In der Linuxwelt gibt es Standardverzeichnisse, an denen Paketinhalte abgelegt werden, etwa `/usb/bin` für ausführbare Binärdateien, `/usr/lib` für Bibliotheksdateien und `usr/include` für C-Headerdateien. Diese Verzeichnisse sind für gewöhnlich Teil der`PATH`-Variable, einem Hauptmechanismus individueller Benutzerumgebungen, um dem Benutzer Anwendungen auf komfortable Weise zugänglich zu machen.

Nix verwendet eine eigene Verzeichnisstruktur, den sogenannten *Nix-Store*. Der Store findet sich für gewöhnlich unter `/nix/store`. Bei einer Paketinstallation wird ein Unterverzeichnis im Store angelegt. Installierte Pakete - wie man das Unterverzeichnis auf der höchsten Ebene des Nix-Store nennen kann - enthalten Paketinhalte, die in weitere Unterverzeichnisse gegliedert werden. Dazu gehören (das `/usr`-Verzechnis spiegelnd) die Verzeichnisse `bin`, `include`, `lib` oder `share`.

Verzeichnisnamen setzen sich zusammen aus einem Hash-Wert, einem Paketnamen und einer Versionsnummer. Pfade zu diesen Verzeichnissen haben deshalb die Form: `/nix/store/<Hash-Wert>-<Name>-<Version>`. Die Hash-Werte werden auf Grundlage aller Inputs berechnet, mit denen das Paket gebaut wurde. Dadurch wird es möglich, verschiedene Varianten der gleichen Anwendung nebeneinander installiert zu haben, ohne Konflikte fürchten zu müssen.[^varianten] Daraus ergibt sich ein einfach überprüfbares Kriterum für die Identität von Pakten. Zwei Pakete sind identisch genau dann, wenn sich den gleichen Hash-Wert haben.

Am Ende des ersten Artikels wird gesagt, dass der Befehl `nix-store` verwendet werden kann, um mit dem Nix-Store zu interagieren. Im [Bedienungshandbuch](https://nixos.org/manual/nix/stable/command-ref/nix-store.html) heißt es dazu genauer, dass der Store damit "manipuliert" (modifiziert) und Anfragen (*queries*) an ihn gestellt werden können. Insbesondere wird gesagt, dass es im Allgemeinen nicht notwendig ist, auf diese Weise mit dem Store zu interagieren. Mutmaßlich werden `nix-store`-Befehle unter der Hand automatisch ausgeführt, wenn andere Befehle verwendet werden.

## Profile
Im Nix-Store installierte Pakete sind Benutzern zwar prinzipiell zugänglich. Doch ohne weitere Mechanismen müssten lange Pfade vorangestellt werden, um auf die Paketinhalte zuzugreifen. Wie beim traditionellen Ansatz könnte die `PATH`-Variable erweitert werden, um die Shell über die installierten Pakete zu informieren. Es versteht sich, dass es sehr mühselig wäre, diese Anpassungen jedesmal dann vorzunehmen, wenn ein neues Paket installiert wird. Insbesondere, weil sich aus Updates Pfade mit neuen Hash-Komponenten ergeben.

Stattdessen arbeitet Nix mit sogenannten *Profilen*. Wie es im Tutoril heißt, "beschreiben" Profile *Benutzerumgebunden* (*user environments*). Benutzerumgebungen werden als "Mengen" (*sets*) charakterisiert, die all die Pakete umfassen, die einem bestimmten Benutzer zugänglich gemacht wurden. Ich habe bereits an einigen anderen Stellen von Profilen gelesen und glaube, die Rede von Mengen ist nich zu wörtlich zu verstehen.

Scheinbar gibt es so etwas wie ein "Default-Profil". Es wird auch gesagt, das Pakete "auf" (in?) einem Profil installiert werden. Oder dass ein Paket auf dem Default-Profil installiert wird, wenn der Befehl `nix-env -i <Paketname>` genutzt wird. Um sich alle Pakete anzuzeigen, die aktuell - im aktuellen Profil (*current profile*) - installiert wurden, kann `nix-env --query "*"` verwendet werden. Um ein Paket aus dem aktuellen Profil zu entfernen, kann `nix-env -e package_name` verwendet werden.

Es wird nicht darauf eingegangen, was mit dem Default-Profil oder mit dem aktuellen Profil (*current profile*) gemeint ist und ob es sich dabei um zwei Bezeichnungen für den gleichen Status handelt. Tatsächlich wird nebenbei noch eine weitere Komplikation eingeführt: "a new generation is created at each modification of the profile". Wir erfahren, dass durch `nix-env --rollback` zur vorausgegangenen Generation zurückgegangen werden kann.

Mutmaßlich handelt es sich dabei um alle zu einem gegebenen Zeitpunkt installierten Pakete, definiert durch die Inputs, mit denen sie gebaut wurden. Es stellt sich aber die Frage, welche Operationen (Unterbefehle von `nix-env`) eine Modifikation des aktiven Profils und damit eine neue Generation implizieren. Es ist nicht davon auszugehen, dass Profile bereits durch Installation oder Entfernen von Paketen im relevanten Sinne modifiziert wurden?

Das Tutorial geht jedoch nicht weiter auf die genaue Implementierung oder Bedeutung dieses Mechanismus ein. Stattdessen wird auf das [offizielle Bedienungshandbuch](https://nixos.org/manual/nix/stable/package-management/profiles) verwiesen.

## Kanäle
Nixpkgs wird als ein Git-Repository charakterisiert, das Definitionen für all die Pakete enthält, die in Nix verfügbar sind. Von anderen Dingen ausgehend, gehe ich davon aus, dass es sich bei diesen Definitionen um *Derivations* handelt. Git-Repositories haben Branches, auf denen entwickelt werden kann. 

Im Hinblick darauf wird ein neuer Grundbegriff eingeführt: "A **channel**, can be seen as a nixpkgs branch (as in git branch) (...)". Ich glaube Kanäle sind nicht selbst die Branches. Besser wäre es vielleicht zu sagen: Die auf Branches entwickelten Paket-Definitionen werden über Kanäle an Nix übermittelt.

Es wird auch gesagt, dass es sich bei Kanälen um *Nixpkgs*-Branches handeln würde. Pakete können auch auf (den Branches) anderer Repositories definiert werden. Kanäle werden deshalb nicht nur zwischen Nix und den Branches von Nixpkgs geöffnet, sondern auch zwischen Nix und anderen Repositories. Tatsächlich nennt das Tutorial selbst eine weitere Quelle (wenn auch nicht im Abschnitt über Kanäle), die [Nix User Repositories (NUR)](https://nur.nix-community.org/).

Das obige Zitat war etwas verkürzt. Vollständig heißt es: "A channel, can be seen as a nixpkgs branch (as in git branch) that is *validated by a continuous integration system*" (meine Hervorhebung). Es wird nicht erläutert, was ein *continuous system integration* ist. In der Softwareentwicklung gibt es sogenannte Integrationstests, um voneinander Komponenten oder Module in Abhängigkeit voneinander (als Systeme) zu testen. Es gibt Abhängigkeitsbeziehungen zwischen Paketen (Pakete sind *Dependencies* anderer Pakekte), weshalb hier mutmaßlich solche Tests gemeint sind.

Um ehrlich zu sein weiß ich durch das Lesen anderer Dokumente noch ein paar Dinge über diesen Bereich. Nixpkgs unterscheidet zwischen stabilen und unstabilen Kanälen. Das Repository wird auf einem unstabilen Branch kontinuierlich weiterentwickelt. Unstabile Kanäle greifen auf Paketdefinitionen zurück, die bisher noch weniger gründlich getestet wurde. Der für die koninuierlichen Tests der Nixpkgs-Definitionen zuständige Dienst heißt [Hydra](https://github.com/NixOS/hydra).

Scheinbar sind Kanäle selbst etwas, das installiert wird. Im Tutorial heißt es dazu: "To list the channel currently installed **on your system**." Die Hervorhebung wird nicht weiter erläutert, legt aber nahe, dass sie *systemweit* installiert werden. Die Rede einer Installation legt aber nahe, dass Kanäle mehr involieren, als nur eine URL. Werden die Paket-Definitionen heruntergeladen, wenn dem System ein Kanal hinzugefügt wird (mutmaßlich als Teil vom Nix-Store)? Darüber hinaus können Kanäle *geupdated* werden (mit `nix-channel --update`). Es wird nicht erläutert, was bei einem Kanal-Update passiert.

Kanäle werden mit einem eigenen Kommandozielenwerkzeug verwaltet, `nix-channel`. Das Tutorial erläutert eine Reihe von Unterbefehlen. Um (dem System?) einen neuen Kanal hinzuzufügen (zu installieren?), kann `nix-channel --add <URL> <Name>` verwendet werden. Der Name scheint willkürlich gewählt werden zu können.[^subscribe] Im Tutorial wird ein Beispiel gegeben:
```nix
# Add the channel 22.05 with the name nixpkgs.
nix-channel --add https://nixos.org/channels/nixos-22.05 nixpkgs
nix-channel --update
# Add the unstable with the name `unstable`
nix-channel --add https://nixos.org/channels/nixpkgs-unstable unstable
nix-channel --update
```
Nach Darstellung des Tutorials hat der erste Befehl den Effekt, dass der unstabile Kanal durch den ersten Befehl "komplett überschrieben" (*the effect to completely override the unstable channel*). Ich vermute, das hat etwas mit dem verwendeten Namen zu tun? Falls ja, dann stellt sich die Frage, ob der Name `nixpkgs` im Nix-Ökosystem eine besondere Bedeutung hat. Vielleicht ist es eine Konvention, damit einen primär verwendeten oder aktuellsten kanal anzuzeigen? Vielleicht nicht. Vielleicht ist es nur eine gängige Präferenz.

Es wird häufig betont, dass sich Nix von anderen Paketmanagern insbesondere darin unterscheidet, dass verschiedene Versionen eines Pakets zugleich installiert sein können. In Anbetracht dessen fand ich den Hinweis sehr interessant, dass Paketinstallationen automatisch *überschrieben* werden:
```nix
# Installing Simgrid from the stable channel.
nix-env -iA nixpkgs.simgrid
# And installing it from unstable, this will override the current simgrid.
nix-env -iA unstable.simgrid
```
Was wäre passiert, wenn wir die Befehle in umgekehrter Reihenfolge ausgeführt hätten. Es ist davon auszugehen, dass der unstabile Branch eine neuere Version enthält. Würde sich Nix weigern, die neuere Version durch eine ältere Version zu überschreiben?

Es wird nicht gesagt, wie ein hinzugefügter Kanal wieder entfernt werden kann. Aus einem anderen Tutorial weiß ich, dass dazu `nix-channel --remove <Name>` genutzt werden kann. Die hinzugefügten Kanäle können mit `nix-channel --list` aufgelistet werden. Am Ende des Artikels wird gesagt, dass nicht verwendete Pakete (*unused packages*) mit `nix-collect-garbage` entfernt werden können. Es wird nicht darauf eingegangen, wann ein Paket als ungenutzt gilt (wann der Garbage-Collector also wirksam wird). Wahrscheinlich geht es hierbei um Dependencies, also Pakete, die man nur deshalb installiert hat, weil man etwas anderes installieren wollte? Ich meine gelesen zu haben, dass der Garbage-Collector sehr ähnlich arbeitet wie der vieler Skriptsprachen. Pakete gelten als ungenutzt, wenn keien *Referenz* mehr auf sie besteht. Nur weiß ich nicht mehr, wie das festgehalten wird. Vielleicht dient die Nix-Datenbank dazu?

Oben wurde `nix-env -i package_name` verwendet, um Pakete zu installieren. Wenn nun mehrere Kanäle zugleich installiert sein können, stellt sich die Frage, über *welchen* Kanal das Paket dabei installiert wird. Mutmaßlich wird geprüft, über welchen Kanal die aktuellste Version des Pakets verfügbar ist und dieses wird dann verwendet?

Jedenfalls gibt es einen Unterbefehl, um ausdrücklich anzugeben, über welchen Kanal ein Paket installiert werden soll. Dazu kann `nix-env -iA channelname.packagename` verwendet werden.

Das führt unmittelbar zur Frage, woher man weiß, welche Pakete (welche Definitioenn für Pakete?) über einen gegebenen Kanal verfügbar sind. Dazu wird `nix search <Kanalname> <Paketname>` verwendet. Es wird nicht darauf eingegangen, dass es sich bei `nix` um ein weitaus neueres Kommandozeilenwerkzeug handelt. Ich glaube tatsächlich handelt es sich sogar um ein "experimentelles" Feature und muss in der Nix-Konfiguration freigeschaltet werden. Ich glaube es kann zur Suche ein regulärer Ausdruck verwendet werden.

Um Pakete zu updaten, kann `nix-env --upgrade <Paketname>` verwendet werden. Da wie oben gesehen keine verschiedenen Varianten des gleichen Pakets über verschiedene Kanäle installiert werden können, dürfte das Paket über seinen Namen eindeutig identifiziert werden können? Moment, das kann nicht richtig sein. Nix wirbt doch groß damit, dass verschiedene Varianten des gleichen Pakets installiert sein können. Woher weiß `nix-env` in diesem Fall, *welche* Variante geupdated werden soll? Und können Pakete erst geupdated werden, nachdem der Kanal geupdated wurde, von dem das Paket installiert wurde? Und heißt das, dass für installierte Pakete irgendwie festgehalten wird, zu welchem Kanal sie gehören? Falls ja, wo wird das dokumentiert?

## Builds
Ein weiterer Befehl, `nix-build`, dient dazu, das in einem Nix-Ausdruck definierte Paket zu *bauen*. Ich bin mir ziemlich sicher, dass die Nix-Ausdrücke, die Pakete definieren, *Derivations* genannt werden. An vielen anderen Stellen wird gesagt, dass Derivations selbst gebaut werden.

Hier wird dazu jedenfalls nur gesagt, dass das gebaute Paket dadurch dem Nix-Store hinzugefügt wird. Es scheint klar, dass sich dieser Vorgang von einer Installation unterscheidet. Mir ist klar, warum ich ein Paket installieren würde; nicht klar ist mir jedoch, warum ich ein Paket bauen aber *nicht* installieren wollen würde. Oder worin der Unterschied genau besteht. Mutmaßlich landet das gebaute Paket am Ende im Nix-Store, ganz so, wie bei einer Installation? Wahrscheinlich wird es nicht dem Profil oder der Benutzerumgebung zugewiesen (nicht für den Benutzer aktiviert)?

## Nix-Shells
Es steht wohl außer Frage, dass sich Nix primär an Entwickler wendet. Ein Hauptfeature dürfte für viele Anwender deshalb sein, eigene Entwicklungsumgebungen zu definieren. Wie das Tutorial erklärt, sind diese vergleichbar mit virtuellen Umgebungen, wie man sie vielleicht von Python kennt.

Falls man sie nicht kennt wird die Idee nicht weiter erläutert. Im Python-Ökosystem werden Pakete durch `pip` installiert. Für einige wenige Python-Pakete ist es vielleicht praktisch, sie systemweit einzurichten. Die meisten dürften jedoch nur speziell für eine Anwendung relevant sein. Insbesondere wollen wir vielleicht für verschiedene Anwendungen verschiedene Versionen eines Pakets als Dependency verwenden. Um Konflikte zwischen den Paketen zu vermeiden, kann für ein Projekt eine virtuelle Umgebung erstellt werden. Pakete können dann für die Umgebung installiert werden. Wenn die Umgebung aktiviert sind, sind dann auch die Pakete verfügbar.

Auf ähnliche aber globalere Weise lassen sich mit Nix Entwicklungsumgebungen definieren. In diesem sehr kurzen Abschnitt wird nur gesagt, dass die definierten Umgebungen mit `nix-shell` betreten werden können. Das Konzept wird nicht weiter motiviert und es wird (noch) nicht erklärt, wie Entwicklungsumgebungen definiert werden können.

## Offene Fragen
- In welchem Sinne beschreiben Profile Benutzerumgebungen?
- Warum ist die weitere Abstraktion von Profilen notwendig? Warum würden Benutzerumgebungen nicht ausreichen?
- In welchem Sinne umfassen Benutzerumgebungen die Pakete, die einem bestimmten Benutzer zugänglich gemacht wurden?
- Sind "current profile" und `default profile" Bezeichnungen für den gleichen Status?
- Welche Profil-Operationen implizieren eine Profil-Modifikation und damit eine neue Generation?
- Wie ist die Beziehung zwischen Profilen und Generationen implementiert?
- Was genau ist damit gemeint, einen Kanal zu installieren? Und was passiert bei Kanal-Updates?
- Über welchen Kanal werden Pakete installiert, wenn der Kanal nicht explizit beim Installationsbefehl angegeben wird. Bei `nix-env -iA channelname.packagename` wird das über seinen Namen identifizierte Paket über den angegebenen Kanal installiert; aber über welchen Kanal wird das Paket installiert, wenn nur `nix-env -i package_name` verwendet wird?
- Hat `nixpkgs` als Kanal-Bezeichnung im Nix-Ökosystem eine besondere Bedeutung?
- Warum würde man ein Paket bauen, ohne es zu installieren?
- Wann gilt ein Paket als ungenutzt? Unter welcher Bedingung wird es vom Garbage-Collector bereinigt?

## Fußnoten
[^varianten]: Einige Anwendungen bieten Nutzern verschiedene Installationsoptionen für die gleiche Anwendung an. Deshalb können sich Installationen eines Pakets auch dann voneinander unterscheiden, wenn es sich um die gleiche Version handelt.
[^subscribe]: Im offiziellen Bedienungshandbuch wird davon gesprochen, dass Kanäle [abonniert](https://nixos.org/manual/nix/stable/command-ref/nix-channel.html) (*subscribed channels*) werden.
