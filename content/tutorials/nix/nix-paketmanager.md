# Nix: Der Paketmanager
Nix ist in erster Linie ein Paketmanager. 

## Alleinstellungsmerkmale
Von ähnlichen Systemen unterscheidet es sich durch die Möglichkeit atomarer Updates and Rollbacks.[^1] Ein Benutzer kann verschiedene Versionen der gleichen Software-Komponente nebeneinander betreiben. Wird eine Software-Komponente von verschiedenen Benutzern des Systems verwendet, kommt es zu keinen parallelen Installationen. Das wird durch ermöglicht durch die Verwendung einmaliger kyrypotgraphischer Hashes in Dateipfaden.

Reproduzierbarkeit.
Man nennt Nix deshalb auch allgemein ein *Deployment System*.

## Nixpkgs

## Channels
Nix nutzt sogenannte *Channels*, um Nix-Ausdrücke und die gegebenenfalls damit verbundenen Binaries zu verbreiten.[^2] Über sie werden "gewöhnliche" Anwendungen wie Firefox, Neovim oder Git installiert und aktualisiert, NixOS auf einen neueren Stand gebracht und Systemkonfigurationen verbreitet. Das NixOS-Projekt betreibt zwei offzielle Kanäle zum Nixpkgs-Repo und seinen Software-Paketen.

`nixpkgs-unstable` für Nix und `nixos-unstable` für NixOS. Das heißt, NixOS-Benutzer nutzen für *alles* den letztgenannten Kanal, Nutzer auf anderen Distributionen nutzen den ersten Kanal. https://nixos.org/manual/nixpkgs/stable/ Siehe Overview.

Als **stabilen Channel** kann man den Kanal zum Branch betrachten, der mit dem aktuellen Release von NixOS verbunden ist.[^stabil]

Über den **stabilen Channel** werden die Pakete vom stabilen Branch des Repos verteilt.  Änderungen auf diesem Branch betreffen lediglich Bugfixes.[^bugfixes] Es ist deshalb unwahrscheinlich, dass Systemupgrades, die über einen stabilen Kanal durchgeführt werden, zu Problemen führen. Es versteht sich, dass Verwendung eines solchen Kanals ist für Produktionsumgebungen stark empfohlen wird.

Man könnte sagen, dass stabile Kanäle einen "LTS"-Zustand von NixOS repräsentieren. Sie werden durch einen Namen identifiziert, der eine Versionsnummer enthält und dem Benennungsschema `nixos-xx.yy` folgt.[^3] Der aktuelle stabile Kanal ist `nixos-25.05`. (https://channels.nixos.org/) führt frühere stabile Kanäle auf.

Das Nixpkgs-Repo wird primär auf dem unstabilen Branch weiterentwickelt. Der **unstabile Channel** (`nixos-unstable`) verteilt Pakete dieses Zweigs und ermöglicht Systemupdatees, die mit einem Rolling-Release anderer Distributionen vergleichbar sind. Da die Änderungen potenziell weniger getestet wurden, sind *breaking changes* nicht ausgeschlossen. Unter bestimmten Umständen kann es sinnvoll sein, einen früheren unstabilen Kanal – früher im Vergleich zum gegenwärtigen Stand von `nixos-unstable` – zu verwenden oder daran festzuhalten.

Daneben gibt es "kleine" Kanäle (*small channels*) zu den stabilen unstabilen Branches. Diese unterscheiden sich von den großen Channels dadurch, dass darüber weitaus weniger Binaries verbreitet werden. Binaries werden von von Quelldateien kompiliert. Das betrifft insbesondere GUI-Applikationen. Sie sind primär für die Verwendung auf Servern konzipiert.

Neben den Channels, die vom NixOS-Projekt selbst bereitgestellt werden, betreiben individuelle Nutzer, Communities und Organisationen ihre eigenen Kanäle. Diese sind entweder öffentlich oder der Zugriff auf sie wird begrenzt. Sie verweisen in ihren Paket-Definitionen zwar für gewöhnlich auf Nixpkgs, verwenden aber zumeist ein unabhängiges Repo um Ausdrücke und eventuell Binaries bereitzustellen. Dazu gehört insbesondere das [Nix User Repository (NUR)](https://github.com/nix-community/NUR) mit seiner Vielzahl von Paketbeschreibungen, mit denen Quelldateien kompiliert werden können.

Channels können über die Kommandozeilenschnittstelle verwaltet werden:
```bash
nix-channel --add <channel-url> <channel-name>
```
Mit diesem Befehl wird der Nix-Konfiguration ein weiterer Kanal hinzugefügt. Der stabile und der unstabile NixOS-Kanal schließen sich nicht wechselseitig aus. Ein Benutzer könnte etwa entscheiden, den stabilen Kanal für die meisten Pakete zu verwenden, bei einer bestimmten Anwendung aber auf Bleeding-Edge-Features angewiesen sein.

Der Kanal-Name kann vom Benutzer selbst festgelegt werden. Dieser wird verwendet, wenn der Kanal später wieder gelöscht werden soll.
```bash
nix-channel --remove <channel-name>
```

Um sich die hinzugefügten Kanäle anzeigen zu lassen, wird `nix-channel --list` verwendet.

## Ressourcen
Der Eintrag zu [Nix-Channels](https://nixos.org/manual/nix/stable/command-ref/nix-channel.html) im offiziellen Bedienungshandbuch.

## Literatur
Dolstra, Eelco. 2006. The Purely Functional Software Deployment Model. Utrecht: Utrecht University.<br>

## Fußnoten
[^1]: Vgl. Dolstra 2006, 14-16.
[^2] https://nixos.org/manual/nixos/stable/#sec-upgrading.
[^3]: Für einige Ausführungen über Namenskonventionen im Channel-Kontext, siehe den Thread [Differences between Nix channels](https://discourse.nixos.org/t/differences-between-nix-channels/13998) im NixOS-Forum.
[^bugfixes]: https://nixos.wiki/wiki/Nix_channels
[^stabil]: https://nixos.wiki/wiki/Nix_channels
