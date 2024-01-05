# WordPress einrichten (Arch Linux)
Heute habe ich WordPress auf einem System mit Arco Linux eingerichtet. Wie immer gibt es im Arch Wiki gute Anleitungen. Da aber nicht alle Beiträge auf dem neuesten Stand sind und man bei mehreren Artikeln leicht etwas übersieht, wird das Vorgehen hier in geordnete Bahnen gebracht.

Um WordPress zum Laufen zu bringen, werden drei Komponenten benötigt: [Apache HTTP Server](https://wiki.archlinux.org/title/Apache_HTTP_Server){: target="_blank"}, [PHP](https://wiki.archlinux.org/title/PHP){: target="_blank"} und [MariaDB](https://wiki.archlinux.org/title/MariaDB){: target="_blank"}. Apache serviert die Seite und MariaDB stellt die Datenbank bereit, in der Seiteninformationen gespeichert werden. PHP ist nötig, da WordPress selbst in PHP geschrieben ist.

Mir geht es nur um den puren Ablauf. Ich werde nicht auf die Hintergründe eingehen, warum bestimmte Einstellungen zu setzen sind. Mir geht es lediglich darum, schnell und umkompliziert zu einer funktionierenden WordPress-Umgebung auf dem Localhost zu gelangen. Wenn es einmal läuft, dann spielen die Einrichtungsdetails keine große Rolle.

## Wordpress
Die erste Komponente ist natürlich WordPress selbst. In Arch-basierten Systemen hat man die Wahl zwischen einer automatischen Installation über den Paketmanager oder einer manuellen Installation. Im ersteren Fall würde Pacman auch die Updates übernehmen.

Dennoch ist dieses Vorgehen nicht zu empfehlen. Admin und Themes könnten nur dann über das WordPress-Admin-Panel installiert werden, wenn man Anpassungen an den Rechten vornimmt. Außerdem hätte man nur eine, global installierte WordPress-Instanz. Es wäre mit extra Aufwand verbunden, wenn man an mehreren Seiten parallel arbeiten wollen würde. Außerdem ist WordPress selbst in der Lage, sich und die installierten Themes und Plugins zu aktualisieren; ein Paketmanager wird nicht benötigt.

Für eine manuelle Installation sollte zunächst die aktuelle WordPresss-Version [heruntergeladen](https://wordpress.org/download/){: target="_blank"} werden. Das heruntergeladene Archiv muss entpackt werden. Den Inhalt verschiebt man in ein Unterverzeichnis des Ordners, den Apache im Default-Fall nutzt (`/srv/http`).[^apache-document-root]
```bash
mkdir /srv/http/my-new-website
cd /srv/http/my-new-website
wget https://wordpress.org/latest.tar.gz
tar xvzf latest.tar.gz
```

Für eine bessere Sicherheit empfiehlt es sich gegebenenfalls, einige Anpassungen bei den Rechten des Verzeichnisses vorzunehmen. Auf der offiziellen WordPress-Seite wird ein Schema [vorgeschlagen](https://wordpress.org/documentation/article/hardening-wordpress/#file-permissions){: target="_blank"}

## PHP
PHP wird Server-seitig ausgeführt. Da der Entwicklungscomputer beim Localhost selbst der Server ist, muss PHP darauf installiert sein. Das Paket ist in Pacman verfügbar. Ich meine mich zu erinnern, dass in einigen Fällen auch `gd` notwendig wird, weshalb das kleine Paket direkt mitinstalliert werden kann.
```bash
pacman -S php
pacman -S php-gd
```

PHP wird über die `/etc/php/php.ini` konfiguriert. Darin sollte zunächst die Zeitzone gesetzt werden (`date.timezone = Europe/Berlin`). Für WordPress wichtig sind einige Einstellungen, bei denen zu kleine Werte zu Problemen führen:
TODO

Darüber hinaus können darin einige Extensions eingebunden werden. Ich habe die folgenden Zeilen einkommentiert:
```
extension=gd
extension=pdo_mysql
extension=mysqli
```

## Apache
Es überrascht vielleicht nicht, dass die Konfiguration vom Apache-Server am aufwändigsten ist. Zunächst muss das Paket aber natürlich (falls noch nicht geschehen) installiert werden. Außerdem brauchen wir das PHP-Modul für Apache.
```bash
pacman -S apache
pacman -S php-apache
```

## MariaDB

## WordPress einrichten

## Fußnoten
[^apache-document-root]: Falls gewünscht kann das Verzeichnis in der Apache-Konfigurationsdatei durch die Einstellung `DocumentRoot <Verzeichnis>"`geändert werden. Das Arch-Wiki warnt jedoch, dass daraufhin weitere Einstellungen notwendig werden.
