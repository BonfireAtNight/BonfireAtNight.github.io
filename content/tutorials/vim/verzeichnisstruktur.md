---
title: Neovim Verzeichnisstruktur
date: 2023-10-23 11:29:32 +0200
date_last_mod: 2023-10-23 11:29:36 +0200
---
# Neovim: Verzeichnisstruktur
Neovim legt eine Verzeichnisstruktur fest, desssen Ordner einem vordefinierten Zweck dienen. Hier werde ich nur auf die Verzeichnisse eingehen, die für die meisten Anwender von Bedeutung sein werden.

## Dateien beim Programmstart laden
Vim wurde traditionell dadurch initialisiert, dass die `.vimrc` beim Programmstart automatisch ausgeführt wurde. Neovim folgt der neueren [XDG Spezifikation](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html), nach der das Verzeichnis für Programmkonfigurationen durch die `XDG_CONFIG_HOME`-Umgebungsvariable festgelegt wird. Vor Version 0.5 bedeutete das für die meisten Benutzer, dass `~/.vimrc` durch `~/.config/nvim/init.vim` ersetzt wurde.

Seit der Einführung von Lua als Alternative zu Vimscript kann `init.lua` verwendet werden, um Neovim zu initialisieren. Alternative heißt hier, dass sich `init.lua` und `init.vim` wechselseitig ausschließen. Befinden sich bei Programmstart beide Dateien im Neovim-Konfigurationsverzeichnis wird eine Fehlermeldung ausgegeben (`Conflicting configs`). Im Folgenden setze ich voraus, dass Neovim mit Lua konfiguriert werden soll und ignoriere dementsprechend `init.vim` und alle Aspekte, die Vimscript betreffen.

Wer es einfach mag kann alle seine Einstellungen in `init.lua` vornehmen. Will man jedoch Gebrauch machen von weiterführenden Features wie LSP, Treesitter Spellchecking sollten Konfigurationen schon aus Gründen der Übersichtlichkeit auf verschiedene Dateien und Verzeichnisse aufteilen. 

In vielen Guides auf YouTube oder Blogs wird das `lua`-Unterverzeichnis genutzt, un die anderen Konfigurationen vorzunehmen. Daneben gibt es aber noch weitere Unterverzeichnisse, deren Skripte wie (und nach) `init.lua` automatisch bei Programmstart ausgeführt werden. Die Verzeichnisstruktur entscheidet darüber, zu welchem Zeitpunkt des Initialisierungsvorgangs die Konfigurationsdateien ausgeführt werden.

Im Folgenden beschränke ich mich auf Ordner, deren Inhalte *ausgeführt* werden sollen. Daneben können im Unterverzeichnis `autoload` mit Helper-Funktionen und Bibliotheken abgelegt werden, die zunächst nur *geladen*, aber erst on-Demand aufgerufen werden sollen. Dies erfolgt unmittelbar nach Ausführung der `init.lua`.

## `plugin/` und `after/plugin/`
Nach `init.lua` werden die Skripte im `plugin/`-Unterverzeichnis aufgerufen. Es ist deshalb ein guter Ort, um seine allgemeinen Konfigurationen vorzunehmen: Tastenzuordnungen, Colorschemes, Line Numbers etc. Wenn man sehr viele solche Einstellungen vornimmt, empfiehlt es sich natürlich, sie auf verschiedene Dateien aufzuteilen (`options.lua`, `keymaps.lua` etc.).

Ich habe einige Artikel gesehen, denen zufolge `plugin/` (primär) für Vimscript-Skripte genutzt werde oder (primär) dafür vorgesehen sei. In der offiziellen Dokumentation konnte ich dazu leider nichts finden. Soweit ich sehe sprichts nichts dagegen in dem Verzeichnis auch Lua-Skripte abzulegen.

Das Unterverzeichnis `after/plugin/` wird genutzt, wenn man das Default-Verhalten bestehender Plugins anzupassen, ohne ihren Quellcode antasten zu müssen. Plugins meint in diesem Kontext insbesondere auch, an was die meisten Nutzer dabei denken werden: Telescope, NERDTree, Comment und ähnliche Anwendungen. Ein triviales aber realistisches Beispiel sind Tastenbelegungen: Wenn ein Plugin Default-Keymappings festlegt, können wir sie durch unsere eigenen Tastenbelegungen in einer `after/plugin/`-Datei überschreiben. 

Ich denke es ist eine gute Praxis, für jedes verwendete Plugin eine Konfigurationsdatei in `after/plugin/` anzulegen.

## `ftplugin/` und `after/ftplugin/`
Mit den Skripten dieser Unterverzeichnisse verhält es sich sehr ähnlich wie mit `plugin/` bzw. `after/plugin/`, mit dem Unterschied, dass die vorgenommenen Änderungen nur für einen bestimmten Dateityp wirksam werden (`ft`steht für *file type*). Dazu muss das Skript wie die Sprache benannt werden (`python.lua`, `lua.lua`, `markdown.lua` etc.).

Ein naheliegendes Beispiel ist die Frage, durch wie viele Leerzeichen ein Tab dargestellt werden soll. Style-Guides verschiedener Sprachen machen verschiedene Vorgaben. In Python sind vier Leerzeichen üblich, in Nix nur zwei. Auch Tastenbelegungen kann man so abhängig von der Sprache machen.

### Das `lua`-Unterverzeichnis
Das `lua`-Unterverzeichnis dient dazu, die Neovi-Konfiguration modular und flexibel aufzubauen. *Falls* die Skripte dieses Verzeichnisses aufgerufen werden, geschieht das nach den bisher besprochenen automatischen Aufrufen. *Damit* sie beim Programmstart ausgeführt werden, ist `require(<dateiname>)` notwendig.[^1] Der naheliegendste Ort für diese Imports ist `init.lua`.

Ein Beispiel dazu. Durch `require(plugins)` wird die Datei `lua/plugins.lua` ausgeführt. Für Dateien in Unterverzeichnissen kann ein Slash oder ein Punkt verwendet werden: `require(kai.plugins)` und `require(kai/plugins)` sind äquivalent und führen die Datei `lua/kai/plugins.lua` aus. Durch eine `init.lua` in einem Unterverzeichnis können komfortabel alle darin festgelegten Dateien ausgeführt werden. Enthält `lua/kai/` eine `init.lua` können die darin importierten Dateien ausgeführt werden, indem wir in der obersten `init.lua` `require(kai)` einfügen.

Durch direkte Unterverzeichnisse (wie `lua/kai/` im Beispiel) können Profile erstellt werden. So ist man nicht nur flexibel bezüglich einzelner Dateien, sondern man kann auf einfache Weise zwischen verschiedenen Konfigurationen wechseln.

## Fußnoten
[^1]: Es gibt einen Unterschied zwischen `require()` und einem manuellen Skriptaufruf durch `:source`. Mit `require()` wird ein Skript nicht ein zweites Mal aufgerufen (sondern cached).
