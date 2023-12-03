---
layout: post
title:  "Nix Tutorium"
tags: nix nixos
---

# Nix-Tutorium vom Nationalen Forschungsinstitut für Informatik und Automatisierung (INRIA): Teil 1

## Einleitung: Blog-Posts über Blog-Posts
Der Plan war, den Nix-Grundbegriffen eigene Artikel zu widmen und ihren Zweck systematisch zu erklären. Doch nach Wochen von Recherche erscheint es mir noch immer überwältigend schwierig, aus verschiedenen Quellen gesammelte Informationen überzeugend zusammenzuführen. Deshalb gehe ich das Thema nun vorerst anders an.

Am letzten Wochenende habe ich viele der [Blog-Posts](https://ianthehenry.com/posts/how-to-learn-nix/introduction/) gelesen, in denen Ian Henry seine Erfahrungen beim (Neu-)Einstieg in Nix dokumentiert. Er versteht es als Protokoll vom Stream of Consciousness beim Lesen der einschlägigen Literatur. Mich erinnert es sehr an die textnahe Exegese akademischer Philosophie, insbesondere der Mathematik- und Logik-affinen Schule.

Vielleicht ist es das perfekte Format, um darüber zu reflektieren, was man verstanden und gerade auch, was man nicht verstanden hat. Am Ende jedes Artikels schließt Ian mit offenen Fragen. Nix ist in aktiver Entwicklung und das Benutzungshandbuch, Tutorials und Blog-Posts spiegeln nicht immer den aktuellen Stand. So ergibt sich eine gerade für Einsteiger verwirrende und trügerische Landschaft, bei der nichts so ganz für bare Münze genommen werden kann. Einige Dinge niederzuschreiben, bringt wohl nicht den herrlichen Nix-Seelenfrieden. Es hilft aber sicherlich, Lernfortschritte systematisch festzuhalten und der weiteren Recherche eine Richtung zu geben.

Als ich gestern Morgen diesen Plan gefasst habe, stand zufällig das [Nix-Tutorium](https://nix-tutorial.gitlabpages.inria.fr/nix-tutorial/getting-started.html) vom Nationalen Forschungsinstitut für Informatik und Automatisierung (*Institut national de recherche en informatique et en automatique*; INRIA) auf dem Programm. Vielleicht nicht unbedingt die prominenteste Ressource. Das Institut richtet sich primär an Wissenschaftler der Informatik, die Nix zum Aufbau reproduzierbarer Versuchsumgebungen verwenden wollen. Dennoch kann man denke ich einiges davon lernen.

Ian geht es in seinen Artikeln darum, *von Beginn an* jeden Lernfortschritt festzuhalten. Außerdem hält er sich sehr strikt an die Struktur der Dokumente, die er kommentiert. Ich habe nun wie gesagt bereits einige Zeit darauf verwendet, mehr über Nix und NixOS nachzulesen. Ich werde dementsprechend eher versuchen, das Gelesene in einen Kontext zu setzen und im Lichte anderer Informationen zu deuten.

## Installation von Paketen
Nix ist primär ein Paketmanager. Systeme wie Homebrew, RPM, APT und Winget helfen Anwendern dabei, Software zu installieren und auf dem neusten Stand zu halten. Die inria-Artikelserie beginnt mit einer generellen Darstellung, welche Schritte Paketmanager ausführen, um Endnutzern angeforderte Anwendungen und Bibliotheken verfügbar zu machen.

Ausführbare Dateien und Bibliotheken werden zur Verteilung und Verwaltung zu Paketen zusammengefügt, deren *Inhalte* (*content*) sie sind. Um sie an Benutzer auszuliefern, stehen verschiedene Mechanismen zur Verfügung. Pakete halten Quellen fest, von denen Paketmanager notwendige Komponenten ziehen können. Viele Systeme arbeiten mit einem Software-Cache für vorgebaute (*pre-built*) Binärdateien. Steht für eine Software-Komponente kein vorgebautes Paket zur Verfügung (oder besteht keine oder keine hinreichend schnelle Leitung zum Cache), können Quelldateien verwendet werden, um Anwendungen und Bibliotheken selbst zu bauen. Die Pakete enthalten die dazu notwendigen Informationen. Das betrifft Build-Skripte und die notwendigen Build-Dependencies.

Die Benutzer eines Betriebssystems können prinzipiell auf installierte Pakete zugreifen. Zur einfachen Handhabung sind weitere Maßnahmen erforderlich. In der Linuxwelt gibt es Standardverzeichnisse, an denen Paketinhalte abgelegt werden, etwa `/usb/bin` für ausführbare Binärdateien, `/usr/lib` für Bibliotheksdateien und `usr/include` für C-Headerdateien. Diese Verzeichnisse sind für gewöhnlich Teil der`PATH`-Variable einer individuellen Benutzerumgebung. Dadurch wird der Shell gesagt, an welchen Orten des Dateisystems nach installierten Pakten zu suchen ist.

## Der Nix-Store
Nix weicht ab vom gewöhnlichen [Filesystem Hierarchy Standard](https://de.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) und verwendet eine eigene Verzeichnisstruktur, an denen Pakete bei der Installation abgelegt werden. Das dabei für gewöhnlich verwendete Verzeichnis, `/nix/store`, wird *Nix-Store* genannt. Für jedes installierte Paket wird ein Unterverzeichnis auf der höchsten Store-Ebene angelegt. Das `/usr`-Verzeichnis spiegelnd, enthalten installierte Pakte weitere Unterverzeichnisse, darunter `bin`, `include`, `lib` und `share`.

Verzeichnisnamen setzen sich zusammen aus einem Hash-Wert, einem Paketnamen und einer Versionsnummer. Pfade zu diesen Verzeichnissen haben somit die Form: `/nix/store/<Hash-Wert>-<Name>-<Version>`. Die Hash-Werte werden auf Grundlage aller Inputs berechnet, mit denen das Paket gebaut wurde. Dadurch wird es möglich, verschiedene Varianten der gleichen Anwendung nebeneinander installiert zu haben, ohne Konflikte fürchten zu müssen.[^varianten] Daraus ergibt sich ein einfach überprüfbares Kriterum für die Identität von Paketen. Zwei Pakete sind identisch genau dann, wenn sie den gleichen Hash-Wert haben. Darüber kann beispielsweise ermittelt werden, ob ein Paket in vorgebauter Form in [Nix' eigenem Software-Cache](https://cache.nixos.org/) vorliegt.

Am Ende des ersten Artikels wird gesagt, dass das Kommandozeilenwerkzeug `nix-store` eine Reihe von Unterbefehlen bereitstellt, um mit dem Nix-Store zu interagieren. Im [Bedienungshandbuch](https://nixos.org/manual/nix/stable/command-ref/nix-store.html) heißt es dazu genauer, dass der Store damit "manipuliert" (modifiziert) und Anfragen (*queries*) an ihn gestellt werden können. Insbesondere wird aber gesagt, dass es im Allgemeinen nicht notwendig ist, auf diese Weise manuell mit dem Store zu interagieren. Mutmaßlich werden `nix-store`-Befehle unter der Hand automatisch ausgeführt, wenn andere Befehle verwendet werden.

## Profile
Im Nix-Store installierte Pakete sind Benutzern zwar prinzipiell zugänglich. Doch es gilt, was bereits oben allgemein sagt wurde. Ohne weitere Mechanismen müssten lange Pfade vorangestellt werden, um auf die Paketinhalte zuzugreifen.

Wie beim traditionellen Ansatz könnte die `PATH`-Variable erweitert werden, um die Shell über die installierten Pakete zu informieren. Es versteht sich, dass es sehr mühselig wäre, diese Anpassungen jedesmal dann vorzunehmen, wenn ein neues Paket installiert wird. Die Herangehensweise wird ein wahrer No-Starter wenn man bedenkt, dass jedes Software-Update zu Pfaden mit neuen Hash-Komponenten führt.

Stattdessen arbeitet Nix mit *Profilen* und *Benutzerumgebungen* (*user environments*). Wie es im Tutorium heißt "beschreiben" Profile die verschiedenen Benutzerumgebunden. Benutzerumgebungen werden als "Mengen" (*sets*) charakterisiert, die all die Pakete umfassen, die einem bestimmten Benutzer zugänglich gemacht wurden. Ich habe bereits an einigen anderen Stellen von Profilen gelesen und glaube, die Rede von Mengen ist nicht zu wörtlich zu verstehen. Es hat eher etwas mit einem Netz von Symlinks zu tun.

Scheinbar gibt es so etwas wie ein "Default-Profil". Es wird auch gesagt, das Pakete "auf" (in?) einem Profil installiert werden. Oder dass ein Paket auf dem Default-Profil installiert wird, wenn der Befehl `nix-env -i <Paketname>` ausgeführt wird. Um sich alle Pakete anzuzeigen, die aktuell - im aktuellen Profil (*current profile*) - installiert wurden, kann `nix-env --query "*"` verwendet werden. `nix-env -e <Paketname>` dient dazu, um ein Paket aus dem aktuellen Profil zu entfernen.

Es wird nicht darauf eingegangen, was mit dem Default-Profil oder mit dem aktuellen Profil (*current profile*) gemeint ist und ob es sich dabei um zwei Bezeichnungen für den gleichen Status handelt. Tatsächlich wird nebenbei noch eine weitere Systemkomponente eingeführt: "a new generation is created at each modification of the profile". Wir erfahren, dass durch `nix-env --rollback` zur vorausgegangenen Generation zurückgegangen werden kann.

Mutmaßlich handelt es sich dabei um alle zu einem gegebenen Zeitpunkt installierten Pakete, definiert durch die Inputs, mit denen sie gebaut wurden. Es stellt sich aber die Frage, welche Operationen eine Modifikation des aktiven Profils und damit eine neue Generation nach sich ziehen. Es ist nicht davon auszugehen, dass Profile bereits durch Installation oder Entfernen von Paketen im relevanten Sinne modifiziert wurden?

Das Tutorium geht nicht weiter auf die genaue Implementierung oder Bedeutung dieses Mechanismus ein. Stattdessen wird auf das [offizielle Bedienungshandbuch](https://nixos.org/manual/nix/stable/package-management/profiles) verwiesen.

## Kanäle
[Nixpkgs](https://github.com/NixOS/nixpkgs) ist eine extrem umfangreiche Paket-Sammlung, die eine zentrale Bedeutung im Nix-Ökosystem spielt. Sie wird im ersten Artikel als ein Git-Repository charakterisiert, das Definitionen für all die Pakete enthält, die in Nix verfügbar sind. Nach dem zu schließen, was ich an anderer Stelle über Nix und Nixpkgs gelesen habe, gehe ich davon aus, dass es sich bei diesen Definitionen um *Derivations* handelt. Da Derivations aber in dem Artikel keine Erwähnung finden, rede ich im Folgenden neutral von "Paket-Definitionen".

Git-Repositories haben Branches, auf denen entwickelt werden kann. So auch bei Nixpkgs. Im Hinblick darauf wird ein neuer Grundbegriff eingeführt: "A **channel**, can be seen as a nixpkgs branch (as in git branch) (...)". Wahrscheinlich ist die Analogie nicht perfekt präzise? Ich glaube Kanäle sind nicht selbst die Branches. Besser wäre es vielleicht zu sagen: Die auf Branches entwickelten Paket-Definitionen werden über Kanäle an Instanzen des Nix-Paketmanagers übermittelt.

Es wird auch gesagt, dass es sich bei Kanälen um *Nixpkgs*-Branches handeln würde. Ich glaube das Konzept von Kanälen ist nicht auf Nixpkgs beschränkt. Pakete können auch auf den Branches anderer Repositories definiert werden. Kanäle werden nicht nur zwischen Nix und den Branches von Nixpkgs eröffnet, sondern auch zwischen Nix und anderen Repositories. Tatsächlich nennt das Tutorial selbst eine weitere Quelle (wenn auch nicht im Abschnitt über Kanäle), nämlich die [Nix User Repositories (NUR)](https://nur.nix-community.org/).

Ich habe das obige Zitat zunächst ein wenig gekürzt. Vollständig heißt es: "A channel, can be seen as a nixpkgs branch (as in git branch) that is *validated by a continuous integration system*" (meine Hervorhebung). Es wird nicht erläutert, was ein *continuous system integration* ist. In der Softwareentwicklung gibt es sogenannte Integrationstests, um Komponenten oder Module in Abhängigkeit voneinander (als Systeme) zu testen. Es gibt Abhängigkeitsbeziehungen zwischen Paketen (Pakete sind *Dependencies* anderer Pakekte), weshalb hier mutmaßlich ihrem Zweck nach sehr ähnliche Tests gemeint sind.

Von anderen Dokumenten weiß ich noch ein paar andere Dinge über Nixpkgs und über die Kanäle, mit denen das Repository arbeitet. Nixpkgs unterscheidet zwischen stabilen und unstabilen Kanälen. Das Repository selbst wird auf einem unstabilen Branch kontinuierlich weiterentwickelt. Unstabile Kanäle greifen auf Paketdefinitionen zurück, die bisher noch weniger gründlich getestet wurde. Der für die koninuierlichen Tests der Nixpkgs-Definitionen zuständige Dienst heißt [Hydra](https://github.com/NixOS/hydra).[^hydra]

Scheinbar sind Kanäle selbst etwas, das installiert wird. Im Tutorium findet sich der Ausdrck: "the channel currently installed **on your system**." Die Hervorhebung wird nicht weiter erläutert, legt aber nahe, dass sie *systemweit* installiert werden. Die Rede einer Installation impliziert, dass Kanäle mehr involieren, als nur eine URL. Doch was genau bedeutet das? Werden die Paket-Definitionen heruntergeladen, wenn dem System ein Kanal hinzugefügt wird? Es wird auch gesagt, dass Kanäle (mit `nix-channel --update`) *geupdated* werden können. Leider wird nicht erklärt, was bei einem Kanal-Update passiert.

Kanäle werden mit einem speziellen Kommandozielenwerkzeug verwaltet, `nix-channel`. Das Tutorium erläutert eine Reihe von Unterbefehlen. Um einen neuen Kanal hinzuzufügen, wird `nix-channel --add <URL> <Name>` verwendet. Der Name scheint willkürlich gewählt werden zu können.[^subscribe] Es wird ein Beispiel gegeben:
```nix
# Add the channel 22.05 with the name nixpkgs.
nix-channel --add https://nixos.org/channels/nixos-22.05 nixpkgs
nix-channel --update
# Add the unstable with the name `unstable`
nix-channel --add https://nixos.org/channels/nixpkgs-unstable unstable
nix-channel --update
```

Laut dem Artikel hat der erste Befehl den Effekt, dass der unstabile Kanal "komplett überschrieben" wird (*the effect to completely override the unstable channel*). Ich vermute, das hat etwas mit dem verwendeten Namen zu tun? Falls ja, dann stellt sich die Frage, ob der Name `nixpkgs` im Nix-Ökosystem eine besondere Bedeutung hat. Vielleicht ist es aber auch nur eine Konvention, damit einen primär verwendeten oder vielleicht den aktuellsten Kanal anzuzeigen?

Es wird häufig betont, dass sich Nix von anderen Paketmanagern dahingehend unterscheidet, dass verschiedene Versionen eines Pakets zugleich installiert sein können. Die Funktionsweise, nach der Paketinstallationen automatisch *überschrieben* werden, ist vor diesem Hintergrund (gelinde gesagt) überraschend:
```nix
# Installing Simgrid from the stable channel.
nix-env -iA nixpkgs.simgrid
# And installing it from unstable, this will override the current simgrid.
nix-env -iA unstable.simgrid
```
Was wäre passiert, wenn wir die Befehle in umgekehrter Reihenfolge ausgeführt hätten? Es ist davon auszugehen, dass der unstabile Branch eine neuere Version enthält. Würde sich Nix weigern, die neuere Version durch eine ältere Version zu überschreiben? Muss ich später mal ausprobieren.

Es wird nicht gesagt, wie ein hinzugefügter Kanal wieder entfernt werden kann. Aus einem anderen Tutorium weiß ich, dass dazu `nix-channel --remove <Name>` dient. Die hinzugefügten Kanäle können mit `nix-channel --list` aufgelistet werden. Am Ende des Artikels wird gesagt, dass nicht verwendete Pakete (*unused packages*) mit `nix-collect-garbage` entfernt werden können.

Interessant wäre nun natürlich zu wissen, unter welchen Bedingungen der Garbage-Collector wirksam wird. Was genau muss geschehen, dass ein Paket für Nix nicht mehr länger "genutzt" erscheint? Wahrscheinlich geht es dabei um Dependencies, also um Pakete, die man nur deshalb installiert hat, weil man etwas anderes installieren wollte? Ich meine gelesen zu haben, dass ein Paket für den Nix Garbage Collector als ungenutzt gilt, wenn keine *Referenz* mehr darauf besteht. Referenzen haben mit Dateipfaden zu tun. Vielleicht unterscheidet sich die Funktionsweise nicht wesentlich davon, wie der Garbage-Collector vieler Skriptsprachen arbeitet.[^garbage-collection] Jedenfalls stellt sich diee Frage, wie Referenzen zwischen Paketen festgehalten werden. Vielleicht hat die Nix-Datenbank etwas damit zu tun?

Oben wurde `nix-env -i package_name` verwendet, um Pakete zu installieren. Wenn nun mehrere Kanäle zugleich installiert sein können, stellt sich die Frage, über *welchen* Kanal das Paket dabei installiert wird. Mutmaßlich wird geprüft, über welchen Kanal die aktuellste Version des Pakets verfügbar ist und dieses wird dann verwendet? Vielleicht auch nicht so wichtig. In den meisten Fällen dürfte ein anderer Unterbefehl vorzuziehen sein: Mit `nix-env -iA channelname.packagename` kann ausdrücklich angeben werden, über welchen Kanal ein Paket installiert werden soll.

Das führt unmittelbar zur Frage, woher man weiß, welche Pakete (welche Definitionen von Paketen?) über einen gegebenen Kanal verfügbar sind. Dazu wird `nix search <Kanalname> <Paketname>` verwendet. Es wird dabei nicht darauf eingegangen, dass `nix` ein vergleichsweise neueres Kommandozeilenwerkzeug ist. Wenn ich mich nicht täusche handelt es sich um ein "experimentelles" Feature, das erst in der Nix-Konfiguration freigeschaltet werden muss? Ich glaube die Restriktion wurde erst in einer späteren Version von Nix eingeführt. Vielleicht wurde der Artikel geschrieben, als man den Befehl noch ohne Weiteres verwendet konnte? Egal, wichtiger ist: Für die Suche kann ein regulärer Ausdruck für den Paketnamen  verwendet werden.

Um Pakete zu updaten, kann `nix-env --upgrade <Paketname>` genutzt werden. Da wie oben gesehen keine verschiedenen Varianten des gleichen Pakets über verschiedene Kanäle installiert werden können, dürfte das Paket über seinen Namen eindeutig identifiziert werden können? Moment, das kann nicht richtig sein. Nix wirbt doch groß damit, dass verschiedene Varianten des gleichen Pakets zugleich installiert sein können. Woher weiß `nix-env` in diesem Fall, *welche* Variante geupdated werden soll? Und können Pakete erst auf  den neustenn Stand gebracht werden, nachdem der Kanal geupdated wurde, von dem das Paket installiert wurde? Und heißt das, dass für installierte Pakete irgendwie festgehalten wird, zu welchem Kanal sie gehören? Falls ja, wo wird die Information gespeichert?

## Builds
Ein weiterer Befehl, `nix-build`, dient dazu, das in einem Nix-Ausdruck definierte Paket zu *bauen*. Wie bereits gesagt bin ich mir relativ sicher, dass die Pakete definierenden Nix-Ausdrücke *Derivations* genannt werden. An vielen anderen Stellen ist die Rede davon, dass Derivations selbst gebaut werden. Hier wird dazu jedenfalls nur gesagt, dass das gebaute Paket am Ende dem Nix-Store hinzugefügt wird. Das scheint der Zweck der ganzen Aktion zu sein.

Es scheint klar, dass sich dieser Vorgang von einer Installation unterscheidet. Mir ist klar, warum ich ein Paket installieren würde. Doch warum würde ich ein Paket bauen, wenn ich es *nicht* installieren wollen würde. Worin besteht überhaupt der Unterschied? Mutmaßlich landet das gebaute Paket am Ende im Nix-Store, ganz so, wie bei einer Installation? Wahrscheinlich wird es nicht dem Profil oder der Benutzerumgebung zugewiesen (nicht für den Benutzer aktiviert)?

## Nix-Shells
Es steht wohl außer Frage, dass sich Nix primär an Entwickler wendet. Ein Hauptfeature dürfte für viele Anwender deshalb sein, eigene Entwicklungsumgebungen zu definieren. Wie das Tutorium erklärt, sind diese vergleichbar mit virtuellen Umgebungen, wie man sie vielleicht von Python kennt.

Falls man sie nicht von Python kennt (und wer auch mit Docker nicht vertraut ist) schaut bei diesem Abschnitt in die Röhre. Wenn ich es recht verstehe, dann werden im Pakete im Python-Ökosystem über den hauseigenen Paketmanager `pip` installiert. Für einige wenige Pakete kann es praktisch sein, sie systemweit einzurichten. Doch es ist von fundamentaler Wichtigkeit, Dependencies im Entwicklungsprozess nicht willkürlich zu ändern. Wenn Updates installiert oder neue Dependencies eingeführt werden, dann sollte das relativ zu Projekten geschehen.

Diese Idee wird durch Pythons virtuelle Umgebungen modelliert, durch deren Verwendung Konflikte und unvorhergesehene Effekte vermieden werden. Pakete werden innerhalb dieser Umgebungen definiert und geupdated; und Projekte werden in einer konkreten Umgebung weiterentwickelt und getest. Dadurch ergeben sich isolierte Umgebungen und Projekte kommen sich bezüglich ihrer Dependencies nicht ins Gehege.

Auf ähnliche aber globalere Weise lassen sich mit Nix Entwicklungsumgebungen definieren. In diesem sehr kurzen Abschnitt wird nur gesagt, dass die Umgebungen mit `nix-shell` betreten werden können. Das Konzept wird an dieser Stelle (noch) nicht weiter motiviert oder erklärt.

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
[^hydra]: Ian [spekuliert](https://ianthehenry.com/posts/how-to-learn-nix/open-questions/) in einem Artikel darüber, ob Hydra auch dafür verantwortlich ist, vorgebaute Pakete in den Cache einzuspeisen. Das dürfte stimmen.
[^garbage-collection]: Wen das Reference Counter Modell von Garbage Collection näher interessiert, es wird recht ausführlich von Michael Scott behandelt. Vgl. Abschnitt 8.5.3 in Scott, Michael Lee. 2016. Programming Language Pragmatics. 4. Aufl. Waltham, MA: Morgan Kaufmann.
