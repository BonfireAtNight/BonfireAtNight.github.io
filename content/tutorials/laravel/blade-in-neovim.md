# Blade in Neovim
[Blade](https://laravel.com/docs/11.x/blade){:target="_blank"} ist Laravels hauseigene Template-Sprache. Sie wird nicht von Neovim ohne Weiteres unterstützt. Besonders frustrierend: Wenn man die entsprechenden Dateien öffnet, bleibt Syntax-Highlighting ausgeschaltet. Nicht nur für Blade selbst - auch für PHP und HTML.

Die Arbeit ohne farbliche Hervorhebungen ist heutzeutage unzumutbar, deshalb wollen wir zunächst das Syntax-Highlighting zum Laufen bringen. Im Anschluss daran wird (gewissermaßen als Bonus) gezeigt, wie sich Blade-Code mithilfe von [conform.nvim](https://github.com/stevearc/conform.nvim){:target="_blank} formatieren lässt.

Ich folge hier zwei Anleitungen, die sich im Grunde leicht finden lassen, aber bei denen man vielleicht leicht etwas übersieht. Für die Tree-sitter-Grammatik ein [GitHub-Issue](https://github.com/EmranMR/tree-sitter-blade/discussions/19){:target="_blank"} und für die Autoformatierung [@jogarcias Anleitung](https://medium.com/@jogarcia/laravel-blade-on-neovim-ee530ff5d20d){:target="_blank"}.

## Tree-sitter für Blade
Viele Features von Neovim stützen sich auf Parsing durch [Tree-sitter](https://github.com/tree-sitter/tree-sitter){:target="_blank}. Blade ist leider nicht Teil der Sprachen, die sich einfach durch `TSInstall X` installieren lassen. @EmranMR hat jedoch eine [Grammatik](https://github.com/EmranMR/tree-sitter-blade/tree/main){:target="_blank"} geschrieben, die wir Tree-sitter manuell hinzufügen können. Dazu müssen wir mehrere Dinge tun.

1. Zunächst müssen wir der Tree-sitter-Konfiguration die Grammatik hinzufügen. Vermutlich haben die meisten Nutzer bereits eine `require'nvim-treesitter.configs'.setup{...}`-Konfiguration; neue Grammatiken werden mit `require "nvim-treesitter.parsers".get_parser_configs()` hinzugefügt.

```lua
require'nvim-treesitter.configs'.setup{...}

local parser_config = require "nvim-treesitter.parsers".get_parser_configs()
parser_config.blade = {
  install_info = {
    url = "https://github.com/EmranMR/tree-sitter-blade",
    files = {"src/parser.c"},
    branch = "main",
  },
  filetype = "blade"
}

-- Set the *.blade.php file to be filetype of blade
vim.filetype.add({
    pattern = {
        [".*%.blade%.php"] = "blade",
    },
})
```
Nun muss die Grammatik noch installiert werden:
```
TSInstall blade
```

2. Um Syntax-Highlighting zu ermöglichen, muss `~/.config/nvim/after/queries/blade/highlights.scm` mit folgendem Inhalt erstellt werden:

```Query
(directive) @function
(directive_start) @function
(directive_end) @function
(comment) @comment
((parameter) @include (#set! "priority" 110)) 
((php_only) @include (#set! "priority" 110)) 
((bracket_start) @function (#set! "priority" 120)) 
((bracket_end) @function (#set! "priority" 120)) 
(keyword) @function
```

3. Durch Tree-sitters [Language Injection](https://tree-sitter.github.io/tree-sitter/syntax-highlighting#language-injection){target="_blank"} Feature wird festgelegt, dass auch der PHP-Code in den gleichen Dateien berücksichtigt wird. Ansonsten würden nur Blades eigenen Symbole und Directives eingefärbt. Dazu wird `~/.config/nvim/after/queries/blade/injections.scm` mit folgendem Inhalt erstellt:[^injections]

```Query
((text) @injection.content
    (#not-has-ancestor? @injection.content "envoy")
    (#set! injection.combined)
    (#set! injection.language php))
```

## Code-Formatting
[Blade-formatter](https://github.com/shufo/blade-formatter){:target="_blank"} dient der automatischen Formatierung. Um es (global) zu installieren, kann NPM genutzt werden:

```bash
sudo npm install -g blade-formatter
```

Um das Tool für die Formatierung anzuwenden, wird es der Conform-Konfiguration hinzugefügt:

```lua
require("conform").setup({
  formatters_by_ft = {
    blade = { "blade-formatter" }, -- run `sudo npm install -g blade-formatter`
  },
})
```

[^injections]: In der entsprechenden [Datei auf GitHub](https://github.com/EmranMR/tree-sitter-blade/blob/main/queries/injections.scm){:target="_blank"} finden sich weitere Injections. Diese könnten in einigen Fällen notwendig sein.
