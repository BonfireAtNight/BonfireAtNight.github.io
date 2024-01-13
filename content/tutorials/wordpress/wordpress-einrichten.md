---
layout: default
title: WordPress einrichten (Arch Linux)
---

# WordPress einrichten (Arch Linux)
Heute habe ich WordPress auf einem System mit Arco Linux eingerichtet. Wie immer gibt es im Arch Wiki gute [Anleitungen](https://wiki.archlinux.org/title/Wordpress){: target="_blank"}. Da aber nicht alle Beiträge auf dem neuesten Stand sind und man bei mehreren Artikeln leicht etwas übersieht, wird das Vorgehen hier in geordnete Bahnen gebracht.

Um WordPress zum Laufen zu bringen, werden drei Komponenten benötigt: [Apache HTTP Server](https://wiki.archlinux.org/title/Apache_HTTP_Server){: target="_blank"}, [PHP](https://wiki.archlinux.org/title/PHP){: target="_blank"} und [MariaDB](https://wiki.archlinux.org/title/MariaDB){: target="_blank"}. Apache serviert die Seite und MariaDB stellt die Datenbank bereit, in der Seiteninformationen gespeichert werden. PHP ist nötig, da WordPress selbst in PHP geschrieben ist.

Mir geht es nur um den puren Ablauf. Ich werde nicht auf die Hintergründe eingehen, warum bestimmte Einstellungen zu setzen sind. Wir wollen lediglich schnell und umkompliziert zu einer funktionierenden WordPress-Umgebung auf dem Localhost zu gelangen. Wenn es einmal läuft, dann spielen die Einrichtungsdetails keine große Rolle.

## Manuelle Installation von WordPress
Die erste Komponente ist natürlich WordPress selbst. In Arch-basierten Systemen hat man die Wahl zwischen einer automatischen Installation über den Paketmanager oder einer manuellen Installation. Im ersteren Fall würde Pacman auch die Updates übernehmen.

Dennoch ist dieses Vorgehen nicht zu empfehlen. Themes und Plugins könnten nur dann über das WordPress-Admin-Panel installiert werden, wenn man Anpassungen an den Rechten vornimmt. Außerdem hätte man nur eine, global installierte WordPress-Instanz. Es wäre mit extra Aufwand verbunden, wenn man an mehreren Seiten parallel arbeiten wollen würde. Außerdem ist WordPress selbst in der Lage, sich und die installierten Themes und Plugins zu aktualisieren; ein Paketmanager wird nicht benötigt.

Für eine manuelle Installation sollte zunächst die aktuelle WordPress-Version [heruntergeladen](https://wordpress.org/download/){: target="_blank"} werden. Das heruntergeladene Archiv muss entpackt werden. Den Inhalt verschiebt man in ein Unterverzeichnis des Ordners, den Apache im Default-Fall nutzt (`/srv/http`).[^apache-document-root]
```bash
mkdir /srv/http/my-new-website
cd /srv/http/my-new-website
wget https://wordpress.org/latest.tar.gz
tar xvzf latest.tar.gz
```

Für eine bessere Sicherheit empfiehlt es sich gegebenenfalls, einige Anpassungen bei den Rechten des Verzeichnisses vorzunehmen. Auf der offiziellen WordPress-Seite wird ein Schema [vorgeschlagen](https://wordpress.org/documentation/article/hardening-wordpress/#file-permissions){: target="_blank"}

## PHP
PHP wird Server-seitig ausgeführt. Da der Entwicklungscomputer beim Localhost selbst der Server ist, muss PHP darauf installiert sein. Das Paket ist in den offiziellen Paket-Repositories verfügbar. Ich meine mich zu erinnern, dass in einigen Fällen auch `gd` notwendig wird, weshalb es vielleicht nicht schadet, das kleine Paket direkt mitzuinstallieren.
```bash
pacman -S php
pacman -S php-gd
```

PHP wird über die `/etc/php/php.ini` konfiguriert. Darin sollte zunächst die Zeitzone gesetzt werden (`date.timezone = Europe/Berlin`). Für WordPress wichtig sind einige Einstellungen, bei denen zu kleine Werte zu Problemen führen. Dazu gehören `max_execution_time`, `max_input_time`, `post_max_size` und `upload_max_filesize`. In einigen Fällen weist WordPress selbst auf Engpässe hin. Auch viele Themes informieren über Mindestanforderungen.

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

Es kann festgelegt werden, über welche URL die Webseite erreichbar sein soll. Dazu wird eine neue Datei angelegt: `/etc/httpd/conf/extra/httpd-wordpress.conf`
```
Alias /<Zugang> "<Pfad zur Seite>"
<Directory "<Pfad zur Seite>">
	AllowOverride All
	Options FollowSymlinks
	Require all granted
</Directory>
```
Für `<Zugang>` ist ein Name einzusetzen, über den man die Seite erreichen möchte. Wenn man den Platzhalter durch `myblog` ersetzt, dann kann man die Seite über `localhost/myblog` aufrufen. Für `<Pfad zur Seite>` ist ein absoluter Pfad zum Verzeichnis einzutragen, in dem die WordPress-Dateien abgelegt wurden. Bei einer manuellen Installation ist das vermutlich so etwas wie `/srv/http/meine-seite`.

Die Datei muss noch in der Apache HTTP Server Konfiguration (`/etc/httpd/conf/httpd.conf`) geladen werden:
```
# WordPress sites
Include conf/extra/httpd-wordpress.conf
```
Gegen Ende der Konfigurationsdatei sind eine Reihe von Includes, dort findet auch der neue Eintrag seinen Platz.

In derselben Datei sind weitere Änderungen vorzunehmen. Falls man nur lokal arbeitet, kann `Listen 127.0.0.1:80` statt `Listen 80` eingestellt werden. Für WordPress-Seiten muss der Server eine `index.php` statt einer `index.html` zurückgeben. Deshalb muss folgende Anpassung vorgenommen werden:
```
<IfModule dir_module>
    DirectoryIndex index.php
</IfModule>
```

Apache ist nicht ohne Weiteres fähig, PHP zu verarbeiten. Wir haben über Pacman bereits `php-apache` installiert. Nun müssen noch die entsprechenden Einstellungen in der Apache-Konfiguration vorgenommen werden.[^libphp] Ein Modul muss auskommentiert und ein anderes dafür geladen werden. So soll es aussehen:
```
#LoadModule mpm_event_module modules/mod_mpm_event.so
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
```
Am Ende der `LoadModule`-Liste müssen noch zwei weitere Einträge hinzugefügt werden:
```
LoadModule php_module modules/libphp.so
AddHandler php-script .php
```
Schließlich muss auch den Includes noch eine Datei hinzugefügt werden:
```
Include conf/extra/php_module.conf
```

Nun kann man über systemd den Apache-Dienst starten: `systemctl start httpd`. Falls man den Service bereits gestartet hatte, werden die in der Zwischenzeit vorgenommenen Einstellungen durch einen Neustart übernommen: `systemctl restart httpd`. Über `systemctl status httpd` kann man prüfen, ob alles wie gewünscht aktiviert wurde.

Mit `systemctl enable httpd` bzw. `systemctl disable httpd` kann festgelegt werden, ob der Dienst beim Boot ins System automatisch gestartet werden soll. Da es sich nicht um einen "echten" Webserver handelt und ich Apache nicht dauerhaft brauche, bevorzuge ich den manuellen Start. Das gleiche gilt für MariaDB.


## MariaDB
WordPress arbeitet mit einer Datenbank. Deshalb muss für jede WordPress-Instanz eine Datenbank eingerichtet werden, auf die sie zugreifen kann. Wir arbeiten mit MariaDB, einem Fork von MySQL mit FOSS-Garantie. Zur Installation können wir wieder Pacman nutzen: `pacman -S mariadb`.

Bevor der Service gestartet wird, müssen einige Einstellungen vorgenommen werden. Das erfolgt über die Kommandozeile:
```bash
mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```
Den Dienst können wir wieder über systemd starten: `systemctl start mariadb`.

Nach dem vorausgegangenen Befehl können wir uns *mit Root-Rechten* in das Verwaltungstool einloggen. Es wird *kein Passowrt* genötigt, doch `sudo` ist vorausgesetzt.
```bash
sudo mariadb -u root -p
```

Für unsere WordPress-Seite wird eine eigene Datenbank erstellt. Außerdem erstellen wir einen Benutzer, der (mit einem Passwort) auf die Datenbank zugreifen darf.  
```
MariaDB> CREATE DATABASE <Datenbankname>;
MariaDB> GRANT ALL PRIVILEGES ON <Datenbankname>.* TO "<Benutzername>"@"localhost" IDENTIFIED BY "<Passwort>";
MariaDB> FLUSH PRIVILEGES;
MariaDB> EXIT
```

Bei Benutzername und Passwort handelt es sich nur um Zugangsdaten, mit denen eine WordPress-Instanz auf eine für sie eingerichtete Datenbank zugreift. Sie werden (eigentlich nur) beim anfänglichen Setup benötigt. Es ist deshalb nicht unbedingt nowedngi, sie einem Passwort-Manager hinzuzufügen.

Es handelt sich hier demnach *nicht* um die Informationen, die beim Login in den WordPress-Adminbereich verwendet werden. Diese werden im nächsten Schritt, bei der Einrichtung von WordPress selbst, festgelegt.

## WordPress einrichten
Wenn wir jetzt die Seite im Browser aufrufen (`localhost/<Alias>`), sollte uns der WordPress-Installationsassistent begrüßen. Sobald man auf `Let's go` klickt, können die Daten für die eben erstellte Datenbank eingetragen werden. Daraus wird automatisch die `wp-config.php` erstellt.

Zumindest sollte es so sein. In meinem Fall bekam ich die Meldung, dass die Datei nicht erstellt werden konnte. Das ist nicht weiter bedenklich; auch im Arch Wiki steht, dass das zu erwarten ist. Stattdessen präsentiert uns der Assistent mit dem Code, den wir (mittels eines Texteditors) per Hand in die `wp-config.php` im Wurzelverzeichnis unser WordPress-Instanz kopieren können. Im Anschluss daran können wir die Installation über den graphischen Assisten fortsetzen.

Nachdem Benutzername und Passwort gewählt wurden, sind wir fertig. Wir können die Seite nun aufrufen und uns in dem Adminbereich einloggen.

## Weitere Einstellungen

### Debugging
Wenn man seine WordPress-Instanz für Entwicklungszwecke nutzt, dann sollte man den Debug-Modus aktivieren. Dadurch erhält man Einblick in die eventuell von PHP ausgegebenen Fehlermeldungen. Dazu muss die folgende Einstellung (vor `/* That's all, stop editing! Happy blogging. */`) in der `wp-config.php` hinzugefügt werden:
```php
// Enable WP_DEBUG mode
define( 'WP_DEBUG', true );
```

Dabei gibt es zwei Möglichkeiten. Durch die eben gesetzte Einstellung werden Fehlermeldungen im Adminbereich angezeigt. Zusätzlich kann eine Logdatei erstellt werden:
```php
// Enable Debug logging to the /wp-content/debug.log file
define( 'WP_DEBUG_LOG', true );
```
Um die obersten Einträge einzusehen, hilft folgendes Kommando:
```bash
tail -f /path/to/your/wordpress/wp-content/debug.log
```

Wer die Fehlermeldungen und Warnungen im Adminbereich irritierend findet, kann sie ausschalten:
```php
// Disable display of errors and warnings
define( 'WP_DEBUG_DISPLAY', false );
@ini_set( 'display_errors', 0 );
```

In einigen Fällen kann vielliecht auch Apache nützliche Debugging-Informationen liefern. Die Logdatei kann man bequemerweise wieder mit `tail` einsehen:
```bash
sudo tail -f /var/log/httpd/error_log
```

### Symlinks
In einigen Fällen kann es hilfreich sein, in WordPress mit symbolischen Links zu arbeiten. So könnte man etwa ein Plugin oder ein Theme an einem Ort speichern, an dem ein Benutzer generell seine Projekte der Softwareentwicklung abspeichert.

So findet man seine Projekte nicht nur leichter wieder. Wenn man mit verschiedenen WordPress-Seiten arbeitet, dann weiß man ohne Weiteres, wo man die *aktuellste* Version findet. Zugleich kann man über Symlinks gewährleisten, dass alle WordPress-Instanzen diese aktuellste Version nutzen.

Symlinks sind leicht erstellt:
```bash
ln -s ~/Repositories/my-new-plugin/ /srv/http/wordpress-development-environment/wp-content/plugins/
```
Wenn wir das Verzeichnis *kopiert* hätten, dann würde es zweifellos in der Liste der Plugins erscheinen. Damit auch ein verlinktes Plugin erkannt wird, müssen einige Voraussetzungen erfüllt sein.

Aus Sicherheitsgründen sind sowohl WordPress als auch Apache dazu angehalten, Symlinks nicht einfach zu folgen. Damit ist der Link für sie nicht einfach das Zielverzeichnis. Für dieses Verhalten sind weitere Einstellungen zu setzen.

In WordPress muss dazu die `wp-config.php` im Wurzelverzeichnis bearbeitet werden.
```php
/* Add any custom values between this line and the "stop editing" line. */
define('ALLOW_SYMLINKS', true);

/* That's all, stop editing! Happy publishing. */
```

Für Apache können wir erneut die globale Konfigurationsdatei (`/etc/httpd/conf/httpd.conf`) bearbeiten.
```apache
<Directory "/srv/http/wordpress-development-environment/wp-content/plugins">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```
Danach ist ein Neustart des Servers notwendig: `systemctl restart httpd.service`.

Die `FollowSymLinks`-Option wird *nicht* rekursiv gesetzt. Das heißt, auch wenn man bereits eine entsprechende Einstellung für `/srv/http` hat (wie es wahrscheinlich der Fall ist), muss sie für jedes Unterverzeichnis erneut gesetzt werden.

Schließlich benötigt der Server-Benutzer (in Arch-Systemen: `httpd`) Lese- und Ausführungsrechte für den Link wie auch für das Zielverzeichnis. Insbesondere werden die Rechte auch für alle Elternverzeichnisse benötigt. Ob das der Fall ist, kann der Reihe nach kontrolliert werden:
```bash
ls -ld /home
ls -ld /home/kai
ls -ld /home/kai/Repositories
ls -ld /home/kai/Repositories/my-new-plugin
```
Sollte die `r-x` für andere Benutzer (neben dem Besitzer und der zugeordneten Gruppe) nicht gesetzt sein, müssen die Rechte hinzugefügt oder es muss dem Server-Benutzer auf anderem Wege Zugriff gewährt werden. Das geht beispielsweise durch das folgende Kommando:
```bash
chmod o+rx /home/kai
```

## Fußnoten
[^apache-document-root]: Falls gewünscht kann das Verzeichnis in der Apache-Konfigurationsdatei durch die Einstellung `DocumentRoot <Verzeichnis>"`geändert werden. Das Arch-Wiki warnt jedoch, dass daraufhin weitere Einstellungen notwendig werden.
[^libphp]: Ich nutze hier der Einfachheit halber `libphp`. Falls Performance eine Rolle spielt, sollten stattdessen `apache2-mpm-worker` und `mod_fcgid` eingebunden werden. Letzteres ist im AUR verfügbar. Für weitere Erklärungen, siehe den Abschnitt im [Arch Wiki](https://wiki.archlinux.org/title/Apache_HTTP_Server#PHP){: target="_blank"}
