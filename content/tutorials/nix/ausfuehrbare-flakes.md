# Ausführbare Flakes
Flakes dienen nicht nur dazu, NixOS-Systemkonfigurationen zu beschreiben. Mit ihnen lassen sich Anwendungen im Detail definieren und so ein bestimmtes Setup verteilen. Die integrierten Anwendungen lassen sich über `nix  run` ausführen. Dazu müssen die Anwendungen im Flake-Output dafür vorbereitet werden.

Als Beispiel wird hier eine Minimal-Konfiguration für Neovim besprochen.

## Die Inputs: Welche Neovim-Version soll verwendet werden?
Bei der Verwendung von Flakes werden Build-Inputs explizit deklariert. Dazu werden Repos und Branches angegeben, um die zu verwendende Version zu spezifizieren. Wenn das Flake neben Neovim noch andere Software-Komponenten verwendet, sollte auch Nixpkgs eingebunden werden.

Bei Neovim kann zwischen einem stabilen und einem unstabilen Branch gewählt werden. Bei Nixpkgs wählt man zwischen einem unstabilen Branch oder einem bestimmten Release (oder sogar ein bestimmer Commit über seinen Hash).
```nix
{
  description = "A minimal flake to use Neovim";

  inputs = {
    nixpkgs = {
      url = "github:NixOS/nixpkgs"; # unstable branch
      url = "github:NixOS/nixpkgs/release-23.05"; # take software that is/was current at a certain point in time
    };

    neovim = {
      url = "github:neovim/neovim/stable?dir=contrib"; # stable branch
      # url = "github:neovim/neovim?dir=contrib"; # unstable branch
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = { self, nixpkgs, neovim }: { ... };
}
```
Durch `inputs.nixpkgs.follows` wird festgelegt, dass der `neovim`-Input dem `nixpkgs`-Input "folgen" (darauf begründet sein) soll. Beim Build von Neovim kann dadurch Gebrauch von Paketdefinitionen und Anweisungen gemacht werden, die Nixpkgs bereitstellt. Das gilt vor allem für Dependencies wie Lua.

Um die Input-Definitionen verwenden zu können, geben wir sie als Argumente an den Output.

## App-Definitionen im Output
Im Outputs-Abschnitt muss festgelegt werden, wie auf die Flake-Anwendung über `nix run` zugegriffen werden kann.
