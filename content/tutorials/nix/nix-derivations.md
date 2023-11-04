# Derivations

## Beispiel: curl
Das folgende Listing zeigt eine vereinfachte Version einer Derivation für `curl`, wie sie sich in den Nixpgks findet:

```nix
{ lib, fetchurl, openssl, zlib }:

with lib;

stdenv.mkDerivation rec {
  name = "curl-7.80.0";
  version = "7.80.0";

  src = fetchurl {
    url = "https://curl.se/download/curl-${version}.tar.gz";
    sha256 = "...";  # Hash of the source archive
  };

  nativeBuildInputs = [ makeWrapper ];

  buildInputs = [ openssl zlib ];

  meta = with stdenv.lib; {
    description = "cURL is a command-line tool for transferring data with URLs.";
    license = licenses.mit;
  };
}
```
Es handelt sich um eine Funktion, die Parameter durch ein Attribute Set definiert. Der Hauptteil ist ein Funktionsaufruf von `stdenv.mkDerivation`, die eine Derivation auf der Grundlage einer Reihe von Attributen berechnet.

- `name` und `version` spezifizieren (wenig überraschend) den Namen und die Version des Pakets. Auf ähnliche Weise werden im `meta`-Attribut generelle Informationen über das Paket festgehalten.

- Durch `fetchurl` wird gesagt, dass der Quellcode von der angegebenen URL heruntergeladen (TODO: wo gestored?) werden soll. Im `sha256` wird der Hash-Code des Quellarchives gespeichert, um zu gewährleisten, dass wirklich die richtige Datei runtergeladen wird (*integrity*).

- `makeWrapper` ist ein Werkzeug, das beim Build-Vorgang sehr vieler Software-Komponenten genutzt wird. Es erzeugt ein Skript, das wiederum eine in sich geschlossene Build-Umgebung generiert. Dadurch wird gewährleistet, dass die Build-Dependencies während des Build-Prozesses gefunden werden. Build-Dependenices werden in Nix in verschiedene Kategorien unterteilt. `nativeBuildInputs` ist eine davon. Andere sind `buildInputs` und `propagateBuildInputs`.

- `openssl` und `zlib` sind weitere Software-Komponenten, die zur Build-Zeit benötigt werden. `curl` nutzt OpenSSL und seine Bibliotheken im Umgang mit sicheren Netzwerkprotokollen wie HTTPS. `zlib` dient der Datenkompression, und `curl` kann es nutzen, um komprimierte Daten zu übertragen. Hier geht es um Build-Time-Dependencies, doch beide können auch zur Laufzeit (bei Ausführung des `curl`-Binary) benötigt werden.
