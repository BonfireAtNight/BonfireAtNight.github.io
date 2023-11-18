# Fragen

## Spielen Kanäle noch eine Rolle bei der Verwendung von Flakes?
Kanäle werden in gewisser Weise global für die Nix-Umgebung definiert. Wenn eine Konfiguration oder Paket-Definition von ihnen Gebrauch macht, hängt das Build-Resultat von *externen* Faktoren ab. Deshalb wird bei Flakes *explizit* festgelegt, welche Build-Inputs verwendet werden sollen.

## Wozu dient inputs.nixpkgs.follow?
In vielen Inputs wird Software, die nicht primär aus dem Nixpkgs-Repo gebaut wird, zusätzlich mit `inputs.nixpgs.follows = "nixpkgs;" definiert. Was genau macht diese Zeile?

## Im Kontext von Nix wird sehr viel von Builds gesprochen; gilt das auch für NixOS und seine Konfigurationen?
NixOS-Konfigurationen sind nicht in einem engeren Sinne etwas, das "gebaut" wird (gebaut werden *Binaries*). NixOS-Konfigurationsdateien enthalten *einen* Ausdruck, und dieser wird *ausgewerdet* [evaluated]. Dabei werden die notwendigen Dependencies heruntergeladen [fetching] und *diese* werden gebaut. Zusätzlich wird eine Verzeichnissstruktur hergestellt und Systemeinstellungen gesetzt. Die Konfiguration kann dann *aktiviert* werden. Das ist es, was der Switch in `nixos-rebuild switch` meint.
