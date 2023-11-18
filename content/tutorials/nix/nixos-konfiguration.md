# NixOS: Konfiguration des Betriebssystems (traditioneller Ansatz)
NixOS wird über eine deklarative Konfigurationsdatei eingerichtet. Diese Datei enthält *einen* Nix-Ausdruck, der den gewünschten Zustand des Systems insgesamt beschreibt. Dieser Zustand wird durch Ausführung von `nixos-rebuild switch` im laufenden Betrieb hergestellt.[^nixos-rebuild]

Bei diesem Vorgang wird der Ausdruck vom Interpreter des Nix-Paketmanagers ausgewertet. Wenn dabei Teilausdrücke für Derivationen stehen, werden sie gebaut, bevor die eigentliche Aktivierung der neuen System-Generation beginnt. Das heißt es werden die für die AKtivierung benötigten Binaries und Bibliotheken erzeugt und in den lokalen Nix-Store integriert. Schließlich werden die Instruktionen ausgeführt, die zum gewünschten Systemzustand führen. Das betrifft Systembereiche wie Benutzer und Gruppen, Symlinks, Bootmanager, Dienste und ähnliches. [REF?]

Traditionell werden NixOS-Konfigurationen in `/etc/nixos/configuration.nix` beschrieben und von `nixos-rebuild` automatisch angewendet. In diesem Artikel wird dieser Ansatz im Detail dargestellt. Die Verwendung sogenannter Flakes erlaubt heute größere Flexibilität und ist mit weiteren Vorteilen insbesondere bei der Reproduzierbarkeit der Konfiguration verbunden. Auf diese moderne Herangehensweise, die als konzeptuelle Abstraktion des traditionellen Schemas betrachtet werden kann, wird ein weiterer Artikel eingehen.

## Aufbau der Konfiguration
NixOS-Konfigurationsdateien enthalten genau einen Nix-Ausdruck. Dabei handelt es sich um eine Lambda-Abstraktion [REF?]. Die damit ausgedrückte Funktion nimmt eine Attributmenge als Argument und gibt eine Attributmenge zurück. Die Attribute der zurückgegebenen Menge beschreiben den gewünschten Systemzustand. Da die Attribute über ihre Namen im Nix-Ökosystem als *Optionen* interpretiert werden können, kann der Nix-Paketmanager auf ihrer Grundlage die notwendigen Operationen durchführen, um den Zielzustand herzustellen.

Zunächst ist vielleicht nicht offensichtlich, wie die Funktion aufgerufen wird, woher sie ihre  Inputs erhält und wie der evaluierte Rückgabewert in der Folge verwendet wird. Der eigentliche Entwicklungsprozess abstrahiert von diesen Details. Argumente werden automatisch bereitgestellt, wenn `nixos-rebuild` ausgeführt wird. Ebenso wird der Rückgabewert automatisch verwendet, um einen entsprechenden Systemzustand herzustellen. Für den Entwickler bedeutet das, dass lediglich die vom Interpreter verwendeten Namen verwendet werden müssen. Der Nix-Paketmanager kümmert sich um die eigentliche Systemeinrichtung.

Systemkonfigurationen beginnen mit einer Funktionssignatur. Zwei Attribute werden sehr häufig verwendet: `config` und `pkgs`. Soll eine Attributmenge mit weiteren Feldern übergeben werden können, ohne die erwarteten Attribute ausdrücklich zu benennen, kann `...` eingefügt werden. Eine konventionelle Minimalsignatur sieht deshalb folgendermaßen aus:
```nix
{ config, pkgs, ... }:
```

Ein weiteres wichtiges Attribut ist `lib`, das den Zugriff auf eine Standardbibliothek im Nixpkgs-Repository erlaubt. Zu den enthaltenen Funktionen gehören beispielsweise `lib.optionals <condition> <thenValue> <elseValue>`, mit der einer von zwei Werten gewählt wird, je nachdem ob eine bestimmte Bedingung erfüllt ist; `lib.splitString <Delimiter> <String>`, womit ein String in eine Liste von Teilstrings umgewandelt wird; oder `lib.removeAttrs <Liste von Attributen> <Attributmenge>`, womit die Listenelemente (repräsentiert als String) aus der Attributmenge entfernt werden. Häufig sind die Funktionsnamen sprechend genug, um zumindest passiv ihre Funktionsweise zu verstehen oder sie aus Online-Beispielen herauszukopieren.
TODO: Auf Attribute eingehen wie `modulesPath`, `specialArgs`, `options`

## config
`config` ist der Name des Moduls, über das Kerneinstellungen vorgenommen werden können. Module und Modularität werden unten besprochen. Hier soll es zunächst um die Rolle von `config` bei der Systemkonfiguration und die zugrundeliegende Logik seiner Verwendung gehen.

Das Resultat der Funktion ist eine Attributmenge, die den Zielzustand beschreibt. Dazu gehören gerade auch diese Kerneinstellungen, die auch beim Output im `config`-Attribut festgehalten werden. Auf den ersten Blick scheint der Aufbau deshalb zirkulär zu sein. Wie können im Funktionskörper Werte verwendet werden, die sich erst als Output aus der Evaluation der Funktion ergeben?

Man könnte vielleicht annehmen, dass der Input eine *vorausgegangene* Generation der Systemkonfiguration betrifft, im Funktionskörper auf der linken Seite aber Optionen für die *kommende* Generation definiert werden. Das ist *nicht*, was tatsächlich passiert. Stattdessen kommt es zu einer Art Bootstrapping, die sich die Lazy-Eigenschaft der Nix-Sprache zunutze macht.

Hier ein Ausschnitt aus einem Beispiel, das unten mehr im Detail besprochen werden wird:
```nix
hardwareConfig = import (machineHardwareConfig config.networking.hostName);
```
Hier geht es zunächst nur um `config.networking.hostName`. Der Wert dieses Ausdrucks ergibt sich aus seiner Definition an einer anderen Stelle *derselben* Konfiguration. Insbesondere kann das verwendete Attribut an einer (lexikalisch) *späteren* Stelle definiert werden. Um den Namen in den Gültigkeitsbereich der Funktion einzuführen, muss `config` als Parameter in der Funktionssignatur angegeben werden.

Ein solcher Aufbau wird dadurch möglich, dass die Auswertung des Ausdrucks zunächst zurückgestellt wird, falls dem Interpreter noch kein Attributwert bekannt ist. Panik wird erst dann ausgelöst, wenn das Attribut an *keiner* Stelle in der Konfiguration definiert wird. So ermöglicht die lazy Evaluation, dass sich eine Input-Menge und eine Output-Menge ihre Attribute teilen.

Die Verwendung von `config` kann noch aus einem anderen Grund irreführend sein. Online finden sich viele Beispiele, in denen `config` im Funktionskörper gar nicht definiert wird. Das `config`-Attribut *kann* explizit bestimmt werden. Syntaktischer Zucker erlaubt jedoch eine sehr gängige Kurzschreibweise.[^discourse-1] Die Werte von Optionen aus direkten `config`-Submodulen können definiert werden, ohne explizit auf das Hauptmodul zu verweisen.
```nix
{ config, pkgs, ... }: 
{
  users.users.johndoe.isNormalUser = true;
}
```
Der Nix-Interpreter versteht, dass es sich bei Attributen wie diesen um Elemente der `config`-Menge handelt. Der vorausgegangene Code wird deshalb als äquivalent zum folgenden Konfigurationsfragment aufgefasst:
```nix
{ config, pkgs, ... }:
{
  config = { 
    users.users.johndoe.isNormalUser = true;
  };
}
```

## Modularität
NixOS-Konfigurationen werden modular aufgebaut. Es kann zwischen zwei Gesichtspunkten der Modularität unterschieden werden. Zum einen betrifft Modularität eine Dateistruktur, bei der Inhalte anderer Dateien importiert werden. Zum anderen betrifft Modularität die Verwendung von *Modulen* in einem technischen Sinne.

Module werden vom NixOS-Projekt unter `nixos/modules` im Nixpkgs-Repo definiert. Darin werden Optionen festgelegt, mit denen das System konfiguriert werden kann und die der Nix-Paketmanager "versteht". Kerneinstellungen des Betriebssystems werden über die Optionen des `config`-Moduls vorgenommen. Genauer enthält es eine Reihe von Submodulen, wie `users`, `networking` oder `services`, mit denen verschiedene Systembereiche eingerichtet werden. Daneben gibt es weitere Module, die andere Systembereiche betreffen, darunter etwa `boot` zur Konfiguration des Bootmanagers.

Systemkonfigurationen werden in Nix in *einer* Datei beschrieben. Zur besseren Übersichtlichkeit und zur Verdeutlichung der modularen Logik ist es aber eine sehr empfehlenswerte Praxis, für verschiedene Konfigurationsbereiche eigene Dateien anzulegen und sie in der Hauptkonfigurationsdatei zu *importieren*.[^manual-modularity] 

Der folgende Aufbau repräsentiert eine NixOS-Verzeichnisstruktur, wie sie im Rahmen des traditionellen Ansatzes durchaus typisch ist.
```lua
/etc/nixos/
|-- configuration.nix
|-- modules/
|   |-- networking.nix
|   |-- services.nix
|   `-- users.nix
`-- hardware-configuration.nix
```
Hier spiegelt die Verzeichnis- und Dateistruktur die Modulstruktur des Nixpkgs. Tatsächlich sind eben genannten Gesichtspunkte der Modularität aber völlig unabhängig voneinander und Konfigurationen können prinzipiell in beliebiger Weise auf verschiedene Dateien aufgeteilt werden.

In den Modulen werden eigene Lambda-Ausdrücke definiert, deshalb sie haben im Wesentlichen die Form:
```nix
{ config, lib, ... }:

{
  # Networking/Users/Services configurations...
}
```

In größeren Konfigurationen kann es sinnvoll sein, Attribute wie `users`, `networking` oder `fileSystems` in die Funktionssignatur aufzunehmen. Dabei handelt es sich um Teilbereiche der Systemkonfiguation, die prinzipiell auch in `config` enthalten sind. Dadurch entsteht ein modularisierterer Aufbau der Konfigurationsdatei und dem Leser wird bereits in der Signatur am Dateibeginn kommuniziert, auf welche Aspekte der Fokus gelegt wird.

Die Einstellungen werden dadurch wirksam, dass sie in die Hauptkonfigurationsdatei (`configuration.nix`) in eine Liste integriert werden, die im Nix-Ökosystem `imports` genannt wird.
```nix
{ config, lib, ... }:

{
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
      ./modules/networking.nix
      ./modules/services.nix
      ./modules/users.nix
    ];

  # Other configurations...
}
```
Wenn der Nix-Interpreter auf das `imports`-Attribut trifft, werden die Listenelemente importiert. Dazu macht er Gebrauch von einer eingebauten Funktion, `import`. Diese Funktion nimmt einen Dateipfad als Argument und gibt das Resultat der Auswertung des Dateiinhalts zurück. Unter gewöhnlichen Umständen ergeben sich Attributmengen mit Konfigurationen, die vom Interpreter in eine baumförmige Gesamtkonfiguration vereinigt werden.

Die Konfiguration auf verschiedene Dateien aufzuteilen, ist aus mehreren Gründen sinnvoll. Zunächst sind kleinere Dateien für gewöhnlich leichter in der Handhabung, da die Verzeichnis- und Dateistruktur im Idealfall einer inneren Logik folgt. Eine solche Herangehensweise ist bereits aus der Konfiguration weniger umfassender Systeme vertraut (siehe beispielsweise die zum Teil vordefinierte Neovim-Verzeichnisstruktur) [REF].

Eine Modularisierung erlaubt es, einzelne Aspekte der Konfiguration auf verschiedenen Systemen *wiederzuverwenden*. Insbesondere können verschiedene Konfigurationen in *eine* Gesamtkonfiguration integriert werden, bei der bestimmte Dateien nur auf bestimmten Maschinen oder in bestimmten Umgebungen geladen werden. Für Hardware-spezifische Kernel-Einstellungen, Treibr und ähnliches verwendet NixOS konventionellerweise eine Datei, die `hardware-configuration.nix` genannt und für gewöhnlich automatisch erzeugt wird. Für verschiedene Maschinen benötigt man also jeweils eine eigene Hardware-Konfigurationsdatei.

Angenommen es wurde auf zwei verschiedenen Laptops ein Hardware-Scan durchgeführt und die daraus resultierenden Dateien wurden in Verzeichnissen abgelegt, die für die verschiedenen Maschinen angelegt wurden.
```lua
/etc/nixos/
|-- configuration.nix
|-- modules/
|   |-- networking.nix
|   |-- services.nix
|   `-- users.nix
|-- machines/
|   |-- kai-thinkpad/
|   |   `-- hardware-configuration.nix
|   `-- kai-surfacego/
|       `-- hardware-configuration.nix
```
Nun benötigt man nur noch ein Unterscheidungskriterium, auf dessen Grundlage entschieden werden kann, auf welcher Maschine man sich befindet und welche Hardware-Konfiguration entsprechend geladen werden soll. Sehr häufig wird dazu der Hostname verwendet.
```nix
# configuration.nix

{ config, lib, ... }:

let
  machineHardwareConfig = hostname: if hostname == "kai-thinkpad" then ./machines/otto-thinkpad/hardware-configuration.nix
                                    else if hostname == "kai-surfacego" then ./machines/otto-surfacego/hardware-configuration.nix
                                    else throw "Unsupported hostname: " + hostname;

  hardwareConfig = import (machineHardwareConfig config.networking.hostName);

in

{
  imports = [ hardwareConfig ];

  # ...
}
```
In diesem Beispiel werden einige Sprachkonstrukte verwendet, die vielleicht neu sind. `machineHardwareConfig` ist eine Variable, dessen Wert durch einen Lambda-Ausdruck definiert wird. Die Funktion gibt einen Dateipfad zur jeweiligen Hardware-Konfigurationsdatei zurück. Welcher Dateipfad zurückgegeben wird, hängt vom übergebenen Argument ab.[^hostname-parameter]

Die Abhängigkeit wird durch eine `if ... else...`-Konstruktion ausgedrückt. Der Einfachheit halber wurden die Hostnamen hardcoded: Die Funktion kann deshalb nur dann verwendet werden, wenn `kai-thinkpad` und `kai-surfacego` als Hostnamen verwendet werden. Wenn der übergebeen String keinem dieser beiden Fälle entspricht (oder gar kein String übergeben wid), dann wird eine sprechende Exception geworfen, die die im `else`-Teil definierte Nachricht enthält.

Die definierte Funktion wird in der nächsten Zeile verwendet, um den Inhalt der Hardware-Konfigurationsdateien zu importieren. Sie dient primär dazu, die `imports`-Liste im Funktionskörper nicht zu überfrachten. Im Nichtfehlerfall evaluiert der Ausdruck `(machineHardwareConfig config.networking.hostName)` zum Pfad der Datei, dessen Inhalt von `import` ausgewertet wird. Wie oben bereits erläutert evaluiert `config-networking.hostName` zum Hostnamen des Systems, auf dem wir uns befinden. Auch wenn der Dateiinhalt damit in einer Variable gespeichert wurde, muss der Konfigurationsteil noch an den Nix-Interpreter vermittelt werden. Das geschieht wie oben erklärt über das `imports`-Attribut.

## Pkgs und Kanäle
`pkgs` ermöglicht es, auf die Pakete der Nixpkgs-Sammlung zuzugreifen. Dies ist wichtig, um die Pakete und Versionen festlegen zu können, die im System bzw. für bestimmte Benutzer verfügbar sein sollen. 
```nix
environment.systemPackages = with pkgs; [
  vim
  git
  htop
];
```

Ein Hauptunterschied zwischen dem traditionellen Ansatz und der Verwendung von Flakes liegt darin, wie die zu verwendenden Quellen festgelegt werden. In Flakes werden die Repositories und Branches, die als Input verwendet werden sollen, zu Beginn der Datei explizit angegeben. Dagegen macht der traditionelle Ansatz weitgehenden Gebrauch von *Kanälen* (*channels*). Da die verwendeten Kanäle unäbhängig von der Konfigurationsdatei verwaltet werden, beschreiben Konfigurationen keine perfekt reproduzierbaren Systemzustände.

Nix nutzt Kanäle, um Nix-Ausdrücke und die gegebenenfalls damit verbundenen Binaries zu verbreiten.[^2] Über sie werden "gewöhnliche" Anwendungen wie Firefox, Neovim oder Git installiert und aktualisiert, NixOS auf einen neueren Stand gebracht und Systemkonfigurationen verbreitet. Das NixOS-Projekt betreibt zwei offzielle Kanäle zum Nixpkgs-Repo und seinen Software-Paketen.

Über den **stabilen Channel** werden die Pakete vom stabilen Branch des Repos verteilt.  Änderungen auf diesem Branch betreffen lediglich Bugfixes. Es ist deshalb unwahrscheinlich, dass Systemupgrades, die über einen stabilen Kanal durchgeführt werden, zu Problemen führen. Es versteht sich, dass Verwendung eines solchen Kanals ist für Produktionsumgebungen stark empfohlen wird.

Man könnte sagen, dass stabile Kanäle einen "LTS"-Zustand von NixOS repräsentieren. Sie werden durch einen Namen identifiziert, der eine Versionsnummer enthält und dem Benennungsschema `nixos-xx.yy` folgt.[^3] Der aktuelle stabile Kanal ist `nixos-25.05`. (https://channels.nixos.org/) führt frühere stabile Kanäle auf.

Das Nixpkgs-Repo wird primär auf dem unstabilen Branch weiterentwickelt. Der **unstabile Channel** (`nixos-unstable`) verteilt Pakete dieses Zweigs und ermöglicht Systemupdatees, die mit einem Rolling-Release anderer Distributionen vergleichbar sind. Da die Änderungen potenziell weniger getestet wurden, sind *breaking changes* nicht ausgeschlossen. Unter bestimmten Umständen kann es sinnvoll sein, einen früheren unstabilen Kanal – früher im Vergleich zum gegenwärtigen Stand von `nixos-unstable` – zu verwenden oder daran festzuhalten.

Daneben gibt es "kleine" Kanäle (*small channels*) zu den stabilen unstabilen Branches. Diese unterscheiden sich von den großen Channels dadurch, dass darüber weitaus weniger Binaries verbreitet werden. Binaries werden von von Quelldateien kompiliert. Das betrifft insbesondere GUI-Applikationen. Sie sind primär für die Verwendung auf Servern konzipiert.

Neben den Channels, die vom NixOS-Projekt selbst bereitgestellt werden, betreiben individuelle Nutzer, Communities und Organisationen ihre eigenen Kanäle. Diese sind entweder öffentlich oder der Zugriff auf sie wird begrenzt. Sie verweisen in ihren Paket-Definitionen zwar für gewöhnlich auf Nixpkgs, verwenden aber zumeist ein unabhängiges Repo um Ausdrücke und eventuell Binaries bereitzustellen. Dazu gehört insbesondere das [Nix User Repository (NUR)](https://github.com/nix-community/NUR) mit seiner Vielzahl von Paketbeschreibungen, mit denen Quelldateien kompiliert werden können.

Channels können über die Kommandozeilenschnittstelle verwaltet werden:
```bash
nix-channel --add <channel-url> <channel-name>
```
Mit diesem Befehl wird der Nix-Konfiguration ein weiterer Kanal hinzugefügt. Das bedeutet, dass die Nix-Ausdrücke, die sich am anderen Ende des Kanals befinden, heruntergeladen und lokal abgespeichert werden (in `~/.nix-channels`).[^nixpills-channels] Mit `nix-channel --update` werden neue Ausdrücke heruntergeladen (es passieren noch andere Dinge).

Der stabile und der unstabile NixOS-Kanal schließen sich nicht wechselseitig aus. Ein Benutzer könnte etwa entscheiden, den stabilen Kanal für die meisten Pakete zu verwenden, bei einer bestimmten Anwendung aber auf Bleeding-Edge-Features angewiesen sein.

Der Kanal-Name kann vom Benutzer selbst festgelegt werden. Dieser wird verwendet, wenn der Kanal später wieder gelöscht werden soll.
```bash
nix-channel --remove <channel-name>
```
Um sich die hinzugefügten Kanäle anzeigen zu lassen, wird `nix-channel --list` verwendet.

## Fußnoten
[^nixos-rebuild]: Der Befehl kann verwendet werden, ohne dass auf die neue Version gewechselt wird oder ohne sie zum Boot-Default zu machen. Siehe REF.
[^discourse-1]: <https://discourse.nixos.org/t/configuration-nix-config/20779>.
[^else]: Dabei ist zu beachten, dass der `else`-Block in Nix nicht ausgelassen werden kann.
[^manual-modularity]: <https://nixos.org/manual/nixos/stable/#sec-modularity>.
[^hostname-parameter]: `hostname` ist ein selbst gewählter Parameter-Name, der den Zweck des Parameters verdeutlichen soll. Es würde in die Verantwortung des Entwicklers fallen, der Funktion beim Aufruf einen Wert zu übergeben, der sinnvoll als Hostname interpretiert werden kann. In der Folge wird der Funktion ein Attribut übergeben werden, dessen Wert im Nix-Ökosystem auf einen Hostnamen festgelegt ist.
[^2] https://nixos.org/manual/nixos/stable/#sec-upgrading.
[^3]: Für einige Ausführungen über Namenskonventionen im Channel-Kontext, siehe den Thread [Differences between Nix channels](https://discourse.nixos.org/t/differences-between-nix-channels/13998) im NixOS-Forum.
[^nixpills-channels]: <a href="https://nixos.org/guides/nix-pills/enter-environment#id1359" target="_blank">https://nixos.org/guides/nix-pills/enter-environment#id1359</a>.
