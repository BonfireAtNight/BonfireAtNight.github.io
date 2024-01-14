---
layout: default
title: Neovim für WordPress
---

# Neovim für WordPress
Neovim ist durch externe Tools erweiterbar. Dadurch wird es möglich, darin produktiv auch für WordPress zu entwickeln. In diesem Artikel werde ich über einige dieser Werkzeuge sprechen. Ich werde erklären, inwiefern sie uns bei der Arbeit mit WordPress helfen und wie man sie für eine möglichst komfortable Arbeit in Neovim integriert.

Die verwendete Software wird über den PHP-Paketmanager installiert werden. Da er sich leicht über den systemweiten Paketmanager verschiedener Linux-Distributionen installieren lässt, dürfte diese Anleitung weitestgehend neutral in der Frage der verwendeten Distribution sein. Da ich selbst mit einem Arch-basierten System arbeite, verwende ich im Folgenden dennoch einige Pacman-Kommandos. Es dürfte nicht allzu schwierig sein, die äquivalenten Befehle für APT oder RPM zu finden.

Composer lässt sich auch auf anderen Betriebssystemen installieren. Ich weiß leider nicht, ob die verwendeten Pfade für die Konfiguration unter Windows oder MacOS abgewandelt werden müssen oder ob weitere Unterschiede zu beachten sind.

## Intelephense

### LSP: Funktionen wie in einer integrierten Entwicklungsumgebung
Neovim stellt von Haus aus einen Client für das sogenannte [Language Server Protocol (LSP)](https://microsoft.github.io/language-server-protocol/){: target="_blank"} bereit. Dabei handelt es sich um einen nicht zuletzt von Microsoft beförderten Standard. Unterstützte Texteditoren können um verschiedene Features bereichert werden, indem sie sich mit Language Servers verbinden, die dem Standard gemäß entwickelt wurden.

Genug der allgemeinen Worte, was heißt das für WordPress? Das Content-Management-System selbst wie auch seine Themes und Plugins werden primär in PHP geschrieben. Das heißt wir brauchen einen Language Server für diese Sprache. Wwr haben dabei vor allem zwei Optionen: [phpactor](https://github.com/phpactor/phpactor){: target="_blank"} und [Intelephense](https://intelephense.com/){: target="_blank"}.

Beide werben damit, dass sie eine Reihe von Funktionalitäten herstellen können, wie man sie von einer integrierten Entwicklungsumgebung (IDEs) erwarten würde. Sie versorgen die Entwicklerin mit Informationen aus der offiziellen PHP-Dokumentation, sie geben eine Diagnose und weisen auf Probleme hin, sie bieten eine smarte Rename-Funktion und sie erlauben es, sich schnell durch die Codebasis zu bewegen (go to definition, go to declaration, go to type definition etc.). Dateien können nach Standards wie PSR12 automatisch formatiert werden. Und über sogenannte Code-Actions lassen sich DocStrings und notwendige Importe automatisch einfügen.

Würde es uns nur um PHP gehen, würden die Unterschiede vielleicht keine allzu große Rolle spielen. Für WordPress fällt die Wahl eindeutig auf Intelephense. Der Grund dafür liegt darin, dass dieser Language Server über sogenannte PHP-Stubs lernen kann, welche Bezeichner (für Variablen, Konstanten, Klassen, Methoden etc.) im Rahmen von WordPress vordefiniert wurden. Ohne dieses Wissen würde uns der Language Server hundertfach darauf hinweisen, dass ihm die verwendeten Symbole unbekannt sind. Außerdem können wir so Informationen aus der Dokumentation abrufen, wie wir es für die im PHP-Core eingebauten Komponenten bereits konnten.

### Installation
Somit müssen wir zwei Dinge installieren: Intelephense und die PHP-Stubs für WordPress.[^installation-intelephense] Eine Installation von Intelephense über die Distro-Paketverwaltung ist über das [Arch User Repository](https://aur.archlinux.org/packages/nodejs-intelephense){: target="_blank"} möglich. In den meisten Anleitungen (darunter die [offizielle Dokumentation](https://github.com/bmewburn/intelephense-docs/blob/master/installation.md){: target="_blank"}) wird Intelephense mit [npm](https://www.npmjs.com){: target="_blank"} installiert.
```bash
npm i intelephense -g
```

Zwar ist npm nicht auf Node.js beschränkt und dürfte auf dem Computer vieler Webentwickler installiert sein.[^npm] Im Grunde ist Node aber natürlich ein direkter Konkurrent zu PHP und es sollte möglich bleiben, auf einer Maschine nur mit dem einen oder dem anderen zu arbeiten. Das geht deshalb, weil PHP über einen eigenen Paketmanager verfügt, [Composer](https://getcomposer.org/){: target="_blank"}. Praktischerweise können wir damit auch Intelephense installieren. 

Somit installiern wir zunächst Composer (hier mit Pacman) und installieren damit im Anschluss systemweit Intelephense.
```bash
pacman -S composer
composer global require bmewburn/intelephense
```

Einige Pakete erwarten, über Composer installierte Anwendungen direkt aufrufen zu können. Zumindest zur Sicherheit empfiehlt es sich deshalb, das Unterverzeichnis mit den ausführbaren Binärdateien der `PATH`-Umgebungsvariable hinzuzufügen.[^composer-pfad] Das geschieht in einer Shell-Konfigurationsdatei wie `.bashrc` oder `zshenv`.
```bash
export PATH="$PATH:$HOME/.config/composer/vendor/bin"
```

Über Composer können wir nun auch die für WordPress benötigten PHP-Stubs installieren. Wir könnten sie individuell als Projekt-Dependencies anfordern, indem wir sie der `composer.json` hinzufügen. Der Einfachheit halber installiere ich sie aber global:
```bash
composer global require php-stubs/wordpress-globals php-stubs/wordpress-stubs php-stubs/woocommerce-stubs php-stubs/acf-pro-stubs wpsyntex/polylang-stubs php-stubs/genesis-stubs php-stubs/wp-cli-stubs
```

Davon werden die meisten Benutzer nur einige wenige benötigen: WordPress-Core, WordPress-Globals und vielleicht noch Woocommerce. Es ist aber vielleicht interessant zu wissen, dass es Stubs auch für andere populäre Plugins gibt.

Schließlich brauchen wir Composer noch aus einem anderen Grund. Intelephense wird sich nur dann mit unserem Code-Buffer verbinden, wenn es das Wurzelverzeichnis des Projekts identifizieren kann, zu dem die geöffnete Datei gehört. Dazu wird ein Elternverzeichnis gesucht, in der sich eine `composer.json` befindet. Für kleinere Projekte ohne nennenswerte Dependencies kann die Konfiguration sehr spartanisch sein. Für ernsthaftere Projekte kann über `composer init` ein Setup-Assistent gestartet werden.

Natürlich sind an der Plugin- und Theme-Entwicklung andere Sprachen beteiligt. HTML und CSS spielen notwendig eine Rolle. Für Frontend-Entwicklung wird sicher auch JavaScript zur Anwendung kommen. Hier geht es mir erst einmal nur um WordPress selbst. In Zukunft werde ich sicher weitere Beiträge zum Thema Neovim und Webentwicklung schreiben.

### Intelephense in Neovim nutzen
Als nächsten Schritt müssen wir Neovim für die WordPress-Entwicklung konfigurieren. Das geschieht primär dadurch, dass Intelephense als Language Server für PHP festgelegt wird. Darüber hinaus müssen wir angeben, welche PHP-Stubs verwendet werden sollen und gegebenenfalls, wo im Dateisystem sie sich befinden.

Ich unterstelle hier, dass [lspconfig](https://github.com/neovim/nvim-lspconfig){: target="-blank"} installiert wurde. Die allermeisten Neovim-Nutzer werden vom nativen LSP-Client Gebrauch machen wollen und nur die wenigsten werden die Konfigurationen ohne das Plugin vornehmen.

Um Intelephense und die PHP-Stubs zu verwenden, kann der lspconfig-Konfiguration der folgende Eintrag hinzugefügt werden:
```lua
local lspconfig = require('lspconfig')
local capabilities = require('cmp_nvim_lsp').default_capabilities()

lspconfig.intelephense.setup({
  capabilities = capabilities,
  settings = {
    intelephense = {
      stubs = { 
        "bcmath",
        "bz2",
        "calendar",
        "Core",
        "curl",
        "zip",
        "zlib",
        "wordpress",
        "woocommerce",
        "acf-pro",
        "wordpress-globals",
        "wp-cli",
        "genesis",
        "polylang"
      },
      environment = {
        includePaths = {
          '/home/kai/.config/composer/vendor/php-stubs',
          '/home/kai/.config/composer/vendor/wpsyntex'
        }
      },
        files = {
          maxSize = 5000000,
      };
    };
  }
});
```
Natürlich müssen die beiden Pfade für den eigenen Benutzer abgewandelt werden. Bei den Stubs muss man entscheiden, welche man verwenden möchte. Die `capabilities`-Zeile dient dazu, Completion-Voschläge zuzulassen.

Wenn man nun eine PHP-Datei öffnet, sollte sich der Buffer automatisch mit Intelephense verbinden. Um das zu kontrollieren, kann `:LspInfo` aufgerufen werden. Code-Diganosen und Completion-Vorschläge dürften damit bereits ohne Weiteres angezeigt werden.

Andere Informationen und Aktionen müssen ausdrücklich angefordert bzw. ausgelöst werden. Da man das natürlich nicht über die Kommandoschnittstelle machen möchte, müssen Keybindings definiert werden. Falls man das nicht sowieso schon für die Verwendung eines anderen Language Servers gemacht hat, kann man ein Schema wie dieses verwenden:
```lua
-- Global Mappings
vim.keymap.set('n', '<space>dp', vim.diagnostic.goto_prev)
vim.keymap.set('n', '<space>dn', vim.diagnostic.goto_next)
vim.keymap.set('n', '<space>df', vim.diagnostic.open_float)
vim.keymap.set('n', '<space>dl', '<cmd>Telescope diagnostics<cr>')

vim.api.nvim_create_autocmd('LspAttach', {
  group = vim.api.nvim_create_augroup('UserLspConfig', {}),
  callback = function(ev)
    -- Enable completion triggered by <c-x><c-o>
    vim.bo[ev.buf].omnifunc = 'v:lua.vim.lsp.omnifunc'

    -- Buffer local mappings
    -- See `:help vim.lsp.*` for documentation on any of the below functions
    local opts = { buffer = ev.buf }
    vim.keymap.set('n', 'K', vim.lsp.buf.hover, opts)
    -- Press second time for movement within floating window
    vim.keymap.set('n', 'gd', vim.lsp.buf.definition, opts)
    vim.keymap.set('n', 'gT', vim.lsp.buf.type_definition, opts)
    vim.keymap.set('n', 'gD', vim.lsp.buf.declaration, opts)
    vim.keymap.set('n', '<space>r', vim.lsp.buf.rename, opts)
    vim.keymap.set({ 'n', 'v' }, '<space>ca', vim.lsp.buf.code_action, opts)
    vim.keymap.set('n', '<space>f', function()
      vim.lsp.buf.format { async = true }
    end, opts)
  end,
})
```

Damit haben wir einige Tastenkombinationen, um mit der gegebenen Diagnose zu interagieren. Mit `<Leertaste>+dp` bzw. `<Leertaste>+dn` kann zur vorherigen (*previous*) bzw. nächsten (*next*) Zeile gesprungen werden, für die eine Diagnose-Meldung vorliegt. Falls die Meldung visuell abgeschnitten wird, können wir sie uns mit `<Leertaste>+df` in einem Fenster (*floating window*) anzeigen lassen. Mit `<Leertaste>+dl` werden alle Diagnose-Nachrichten (über Telescope) aufgelistet.

Wenn sich der Cursor über einem Bezeichner befindet, kann man sich per `K`-Tastendruck Information über das bezeichnete Objekt anzeigen lassen. Mit `gd` kann man zur Definition, mit `gD` zur Deklaration und mit `gT` zur Typ-Definition springen. Durch `<Leertaste>+r` können Bezeichner auf eine Weise umbenannt werden, die den Kontext berücksichtigt. So bleiben eventuelle Matches beispielsweise innerhalb eines Strings unangetastet.

### Intelephense-Lizenz
Nun muss leider gesagt werden, dass einige der genannten Features nur für zahlende Kunden zur Verfügung stehen. Intelephense verfolgt ein Freemium-Modell. Welche Funktionen erst freigeschaltet werden müssen, kann der [offiziellen Webseite](https://intelephense.com/){: target="_blank"} entnommen werden. Dort kann eine lebenslang gültige Lizenz für 20€ erworben werden. Wenn man Intelephense viel nutzt, lohnt sich die Anschaffung wohl gerade für die Rename-Funktionalität.

Nun stellt sich natürlich die Frage, wie wir Neovim mitteilen, dass wir über die Lizenz verfügen.[^lizenz] Dazu wird der erhaltene Lizenzschlüssel zunächst in einer Textdatei abgelegt. Im Rahmen von Lua wird der Schlüssel der Datei über eine Funktion entnommen:
```lua
local get_intelephense_license = function ()
    local f = assert(io.open(os.getenv("HOME") .. "/intelephense/license.txt", "rb"))
    local content = f:read("*a")
    f:close()
    return string.gsub(content, "%s+", "")
end
```
Dieses Design hat den Vorteil, dass der Schlüssl nicht im Klartext in der Neovim-Konfiguration auftaucht. So gibt es keine Probleme, die Konfigurationsdateien über ein öffentliches Repository zu verwalten.

Diese Funktion können wir im Anschluss aufrufen, um Intelephense über die Lizenz zu informieren.
```lua
lsp.configure("intelephense", {
    on_attach = on_attach,
    init_options = {
        licenceKey = get_intelephense_license()
    }
})
```
Wenn wir Neovim (neu)starten und eine PHP-Datei öffnen, sollten wir einmalig die Meldung erhalten, dass der Intelephense-Lizenzschlüssel aktiviert wurde.

## Code-Sniffer: Code-Standard-Check und Autoformatter
WordPress definiert auf seiner offiziellen Webseite einen [Standard](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/){: target="_blank"}, dessen Regeln strikt befolgt werden sollten.[^core] Dabei handelt es sich nicht nur um einen Styleguide. Nach ausgedrücktem Selbstverständnis bringen einige der Regeln bewährte Best Practices zum Ausdruck.

Anders als die Language Server einiger anderer Sprachen, scheint Intelephense nicht in der Lage zu sein, die Einhaltung solcher Vorgaben zu prüfen. Deshalb kann uns Neovim dazu leider keine Live-Diagnose bieten. Das WordPress-Dokument selbst verweist auf eine Alternative: [PHP_CodeSniffer](https://github.com/PHPCSStandards/PHP_CodeSniffer/){: target="_blank"} und den dafür verfügbaren [WordPress Coding Standard](https://github.com/WordPress/WordPress-Coding-Standards){: target="_blank"}.

Beide Komponenten können erneut mit Composer installiert werden.
```bash
composer global require "squizlabs/php_codesniffer=*"

composer global config allow-plugins.dealerdirect/phpcodesniffer-composer-installer true
composer global require --dev wp-coding-standards/wpcs:"^3.0"
```
Die erste Zeile gibt uns zwei Anwendungen, die unter `$HOME/.config/composer/vendor/bin/` installiert werden. `phpcs` ist ein externer Linter, den wir manuell aufrufen können und der auf eventuelle Regelverletzungen in einer Datei oder einem Verzeichnis hinweist. Mit `phpcbf` ([PHP Code Beautifier and Fixer](https://phpqa.io/projects/phpcbf.html){: target="_blank"}) können Quellcodedateien automatisch formatiert werden. Im Resultat wurden Abweichungen vom Standard ausgebügelt.[^in-place] 

Mit den beiden anderen Zeilen wird der WordPress-Standard für CodeSniffer installiert. Über ein Kommandozeilenargument wird festgelegt, dass dieser verwendet werden soll:[^direkte-verwendung]
```
~/.config/composer/vendor/bin/phpcs --standard=WordPress <Pfad>
~/.config/composer/vendor/bin/phpcbf --standard=WordPress <Pfad>
```
Es können in beiden Fällen Pfade zu einer oder mehr einzelnen Dateien oder ein Pfad zu einem Verzeichnis übergeben werden.

Als Kommandozeilenwerkzeug ist PHP_CodeSniffer damit bereits relativ hilfreich. Weitaus praktischer wird es, wenn wir es in Neovim integrieren. Dazu können wir einige Anpassungen vornehmen, die nur dann wirksam werden, wenn eine PHP-Datei geöffnet wurde. Dazu wird `~/.config/nvim/after/ftplugin/php.lua` erstellt bzw. bearbeitet.

WordPress erwartet, dass echte Tabs verwendet werden. Damit ist gemeint, dass sich in unseren Quellcodedateien Tabzeichen finden. Demgegenüber bevorzugen viele Entwickler, dass durch Druck auf die `<TAB>`-Taste kein Tabzeichen, sondern eine passende Anzahl von Leerzeichen eingefügt werden. Falls `vim.opt.expandtab` global auf `true` gesetzt wurde, müssen wir die Einstellung für PHP überschreiben:
```lua
opt.expandtab = false
```

Als nächstes können wir der `php.lua` Zeilen hinzufügen, durch die wir die geöffnete Datei per Tastenkombination auf Einhaltung der Standards überprüfen oder automatisch formatieren können.
```lua
vim.api.nvim_set_keymap('n', '<leader>lwp', [[:split | :terminal ~/.config/composer/vendor/bin/phpcs --standard=WordPress %<CR><C-\><C-n>]], { noremap = true, silent = true })
vim.api.nvim_set_keymap('n', '<leader>fwp', [[:!~/.config/composer/vendor/bin/phpcbf --standard=wordpress --ignore=*/vendor/* --ignore=*/tests % <CR>]], { noremap = true, silent = true })
```
Die Diagnose wird in einem neuen Split ausgegeben. Im Falle einer langen Ausgabe kann man sich darin flexibel bewegen und das Fenster im Anschluss durch `:q` schließen.

<!-- ## Treesitter: Verbessertes Syntax-Highlighting -->

## Schlussworte
Es nicht gerade einfach ist, Informationen über das Thema Neovim und WordPress zu finden. Zwar behandeln viele Artikel die Frage, wie man Neovim generell für Webentwicklung einrichtet. Der Fokus liegt dabei aber in der Regel auf dem Frontend. Auf die Entwicklung fürs Backend wird wenig und für Content-Management-Systeme gar nicht eingegangen.

Bezüglich WordPress ist die Situation vielleicht wenig überraschend. Die Schnittmenge zwischen Neovim-Enthusiasten und WordPress-Entwicklern und -Anwendern dürfte überschaubar sein. Die eine Seite repräsentiert eine Tinkerer-Mentalität, die andere Seite erwartet vorgefertigte Lösungen für eine möglichst schnelle und reibungslose Implementierung.

Dennoch findet gerade so etwas wie eine PHP-Rehabilitierung statt. Mittlerweile scheint sich weitgehende Einigkeit eingestellt zu haben: ["PHP doesn't suck (anymore)"](https://youtu.be/ZRV3pBuPxEQ?si=ImwPdc67iN5xabVq){: target="_blank"}. Der vielleicht prominenteste Neovim-Contributor, [TJ DeVries](https://github.com/tjdevries){: target="_blank"}, veröffentlicht dieser Tage sogar eine [Video-Serie](https://youtu.be/xOR3IqrLEyI?si=dDhsSFUEdeM5hhvN){: target="_blank"}, in der er "unironisch" über Laravel spricht.

Im Rahmen von WordPress wird leider weitaus mehr Schrott produziert. Bessere Werkzeuge könnten zuminest ein wenig zu höheren Qualitätsstandards beitragen.

## Fußnoten
[^installation-intelephense]: Ich folge einer [Anleitung](https://daniele.tech/2021/07/neovim-lsp-with-intelephense-for-php-and-wordpress-and-others/){: target="_blank"} von [Daniele Scasciafratte](https://github.com/Mte90){: target="_blank"}. Eine wichtige Ergänzung besteht darin, dass ich der Intelephense-Umgebung einen weiteren Pfad hinzufüge: `vendor/wpsyntex/`. Diesen Zusatz habe ich Danieles [Dotfiles](https://github.com/Mte90/dotfiles/blob/master/.config/nvim/lua/plugin/lsp.lua#L112C1-L112C115){: target="_blank"} entnommen. Um ehrlich zu sein, verstehe ich den Zweck nicht ganz. [wpsyntex](https://packagist.org/packages/wpsyntex/){: target="_blank"} scheint ein Entwickler oder eine Entwicklerin für PHP-Pakete zu sein. In meinem Verzeichnis habe ich das [polylang](https://packagist.org/packages/wpsyntex/polylang){: target="_blank"}-Paket. Ohne den Pfad konnte sich der Buffer zwar mit Intelephense verbinden; bei jeder Änderung wurde die Verbindung aber abgebrochen.
[^npm]: Tatsächlich ist npm *kein* Akronym, das heißt es steht (anders als viele glauben) [nicht](https://en.wikipedia.org/wiki/Npm#History) für "Node Package Manager".
[^composer-pfad]: An dieser Stelle sollte kurz über den Pfad gesprochen werden, an dem Composer-Inhalte installiert werden. Neue Versionen von Composer scheinen dem [XDG-Standard](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html){: target="_blank"} zu folgen. Das heißt, dass Konfigurationsdateien unter `$HOME/.config/<Name der Anwendung>/` abgelegt werden. In einigen Artikeln findet sich statt `$HOME/.config/composer/` noch der alte Speicherort, `$HOME/.composer`. Nach der Installation sollte man deshalb kontrollieren, wo sich die Composer-Dateien befinden und die Pfade im Folgenden gegebenenfalls abgeändert werden.
[^lizenz]: Die folgenden Informationen entnehme ich einer [Anleitung](https://alextheobold.com/posts/intelephense_in_neovim/){: target="_blank"} von Alex Theobold.
[^core]: Wenn man für den WordPress Core entwickeln möchte, dann *müssen* die Regeln eingehalten werden.
[^in-place]: Anders als bei ähnlichen Anwendungen wird die korrigierte Fassung nicht nur auf der Standardausgabe ausgegeben; die Zieldatei selbst wird angepast. Für eine bloße Ausgabe kann ein sogenannter Dry-Run durchgeführt werden: `phpcbf --dry-run path/to/your/directory`.
[^direkte-verwendung]: Wenn `/vendor/bin/` dem `PATH` hinzugefügt wurde, dann können `phpcs` und `phpcbf` direkt (ohne Pfadangabe) aufgerufen werden.
