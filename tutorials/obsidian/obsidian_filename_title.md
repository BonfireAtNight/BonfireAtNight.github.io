---
aliases: 
tags: 
cssclass: 
publish: 
date created: October 9th 2022, 10:02:10 am
date modified: October 9th 2022, 10:02:53 am
---

# Titel und Dateinamen in Obsidian
Notizen in Obsidian sind nichts anderes als gewöhnliche Markdown Dateien und Obsidian Vaults sind nichts anderes als gewöhnliche Verzeichnisse (Linux) bzw. Ordner (Windows).
Dieser Umstand macht es sehr einfach, seine Projekte zu verwalten, Backups zu erstellen oder auf verschiedenen Geräten zu synchronisieren.
Um umfangreiche Änderungen vorzunehmen, können Skripte geschrieben werden.

Aus dieser Designentscheidung ergeben sich aber auch einige Schwierigkeiten.
Markdown-Dokumente haben für gewöhnlich einen Titel ([H1](https://www.markdownguide.org/basic-syntax/#headings)) und es stellt sich die Frage, wie man die Beziehung zwischen Titel und Dateinamen auffassen möchte.
Der Dateiname entspricht der [ID](https://www.markdownguide.org/basic-syntax/#headings) in einem physischen Zettelkasten-System.

## Ein Zeitstempel als Dateiname?
Einige Nutzer verwenden einen Zeitstempel als Namen für ihre Obsidian-Notizdateien. 
So könnte man seine Datei der gängigen [ISO 8601-Norm](https://www.iso.org/iso-8601-date-and-time-format.html) folgend `202210091033` nennen, wenn man sie (wie diese Notiz) am 09. Oktober 2022 um 10:33 erstellt hat.
In der Vergangenheit gab es dazu das Core-Plugin [Zettelkasten Prefixer](https://jackiegeek.gitee.io/obsidian-docs/en/Plugins/Zettelkasten%20prefixer/), heute ersetzt durch den Unique Note Creator.

Neben dem ursprünglichen Erstellungsdatum (*creation date*) dürfte die Information, wann eine Notiz zuletzt bearbeitet wurde, von größerem Interesse sein.
Hinzu kommt, dass Zeitstempel nicht sehr leicht lesbare Dateinamen sind.
Ich ziehe es deshalb vor, die beiden Daten an einem anderen Ort zu festzuhalten.
Dazu eignet sich der [Front-Matter](https://help.obsidian.md/Advanced+topics/YAML+front+matter)-Bereich am Anfang einer jeden Datei.

Für das Erstellungsdatum könnte man das integrierte [Templates](https://help.obsidian.md/Plugins/Templates)-Plugin verwenden.
Besser noch ist die Verwendung des [Linter](https://github.com/platers/obsidian-linter)-Plugins, mit dessen Hilfe (neben vielen anderen Dingen) automatisch Erstellungsdatum (`date created`) und Datum der letzten Bearbeitung (`date modified`) eingefügt werden können.
Dazu werden die entsprechenden Optionen im `YAML Timestamp`-Abschnitt der Plugin-Konfigurationsdatei gesetzt.[^1]

## Dateinamen und Titel synchronisieren
In der [Vergangenheit](https://forum.obsidian.md/t/use-h1-or-front-matter-title-instead-of-or-in-addition-to-filename-as-display-name/687) nutze Obsidian den Dateinamen für sehr viele Zwecke: unter anderem als Ziel für Links und Backlinks, in der Graphenansicht (*graph view*), und bei der Anzeige von Suchergebnissen.
In vielen Bereichen nutzt Obsidian heute primär den Titel der Datei, ein Ansatz, der den meisten Nutzern sachgemäßer erscheinen dürfte.

Dadurch hat der Titel den Dateinamen in seiner Bedeutung weitgehend verdrängt.
Im Grunde könnte man deshalb eine aufsteigende Nummer als unique ID bzw. Dateinamen verwenden.
Als ich meine [zkn3](http://www.zettelkasten.danielluedecke.de/)-Notizen für die Verwendung in Obsidian als Markdown-Dateien exportiert habe, habe ich sie einfach beginnend mit `1.md` benannt.
Diese Praxis hat mir später noch lange (und zunehmend) Kopfzerbrechen bereitet.   

Denn für einige Zwecke bleibt ein sprechender Dateiname dennoch wichtig.
So halten die Entwickler aus für mich nicht ganz einsichtigen Gründen beim Quick Switcher am Dateinamen fest (vielleicht eine Performance-Entscheidung?).
Bisher gibt es keine Option, um auf eine Verwendung des Titels umzustellen.
Abhilfe schafft der [switcher-obsidian-plus](https://github.com/darlal/obsidian-switcher-plus), der darüber hinaus einige weitere nette Features bietet.
Und auch wenn man mit seinen Markdown-Dateien *als Dateien* im Dateisystem arbeitet, hilft es sehr, wenn man vom Dateinamen auf ihren Inhalt schließen kann.

Nun will man Dateinamen und Titel natürlich nicht immer doppelt eingeben.
Für den Dateinamen könnte man vielleicht einen sprechenden Kurztitel verwenden.
Will man Dateiname und Titel jedoch synchron halten, sollte man auf Plugins zurückgreifen.
Beim ersten Erstellen der Datei könnte man ein Template verwenden (`# {{title}}`).
Das Problem ist, dass sich Titel häufiger nochmal ändern können, und man müsste dann trotzdem den Dateinamen per Hand ins Feld für den Dateinamen kopieren.

Weitaus einfacher macht es ein Community-Plugin, [obsidian-filename-heading-synch](https://github.com/dvcrn/obsidian-filename-heading-sync).
Wird der Dateiname oder Titel geändert, so wird automatisch auch das jeweils andere Feld angepasst.
Enthält der Titel ein Zeichen, das in Dateinamen nicht erlaubt ist (wie ":" oder "/"), so wird es im Dateinamen ausgelassen.
In neueren Versionen des Plugins können darüber hinaus  Zeichen aus dem Titel festgelegt werden, die für den Dateinamen ausgelassen werden sollen.


## Empfehlenswerte Dateinamen bei automatischer Umbenennung
Für ordnungsliebende Nutzer bleibt ein letztes Problem. Es ist eine gute Praxis, beim Benennen seiner Dateien einigen Regeln zu folgen.[^2]
Demnach sollte man nicht nur bestimmte Sonderzeichen, sondern auch Leerzeichen und vielleicht sogar Großbuchstaben in Dateinamen vermeiden.
Am einfachsten wäre es natürlich, wenn *filename-heading-synch*-Plugin das Feature implementieren würde.
Ich habe dafür ein [Feature Request](https://github.com/dvcrn/obsidian-filename-heading-sync/issues/62) gestellt.

Für eine externe Lösung habe ich ein [Python-Skript](https://github.com/BonfireAtNight/rename_markdown_files) geschrieben, das Dateinamen automatisch dem Titel anpassen kann.
Eine Erklärung seines Gebrauchs findet sich auf der GitHub-Seite.

## Anmerkungen
[^1]: Bisher haben nur vier Attribute im Rahmen von Obsidian eine besondere Bedeutung: `tags`, `aliases`, `cssclass` und `publish`. Sollte in der Zukunft ein offizieller Name für Dates festgelegt werden (beispielsweise `Created` stat `date created`) erlaubt es Linter, die Änderungen für alle Dateien auf einen Schlag vorzunehmen.
[^2]: Für einige einfache Konventionen siehe beispielsweise ["Best practice for naming files in Linux"](https://www.inmotionhosting.com/support/server/linux/best-practice-for-naming-files-in-linux/) oder ein vor kurzem veröffentlichtes [Video](https://www.youtube.com/watch?v=Wu0CxdflECY) von DistroTube. 
