---
---

# Wie man GIF-Screencasts aufnimmt
Um Tutorials im Software-Bereich zu illustrieren, können kurze Bildschirmaufnahmen im GIF-Format sehr hilfreich sein.
Als ich an meinem ersten Artikel für diese Website gearbeitet habe, wollte ich schnell einige solche bewegten Bilder erstellen.
Wie sich herausstellte, ist es tatsächlich gar nicht so einfach, eine zufriedenstellende Lösung für diese Aufgabe zu finden.
In zwei Tagen habe ich eine Vielzahl von Anwendungen ausprobiert, wobei die meisten sich aus dem einen oder anderen Grund als unzulänglich herausstellten. 
Ich habe mich deshalb entschieden, meinen ersten Artikel auf dieser Webseite diesem Problem zu widmen.

Im Folgenden werde ich zunächst präzisieren, was ich von der gesuchten Anwendung erwarte.
Aufgrund der sehr spezifischen Anforderungen und Voraussetzungen, kann dieser Artikel vielleicht nur wenigen potenziellen Lesern von Nutzen sein.
Das liegt in der Natur der Sache.
Vielleicht kann man sogar sagen, dass sich die [Unix-Philosophie](http://www.catb.org/~esr/writings/taoup/html/ch01s06.html) auch auf das Schreiben von Artikeln übertragen lässt: Es ist besser, ein klar umgrenztes Problem zu lösen, als den Versuch zu unternehmen, eine *one size fits all*-Antwort für unterbestimmte Anforderungen zu formulieren.

Im Anschluss daran werde ich einen Überblick über die populärsten Screencast-Anwendungen geben, die für Linux zur Verfügung stehen.
Statt sie im Detail vorzustellen, beschränke ich mich auf knappe Hinweise, warum sie den zuvor definierten Anforderungen nicht gerecht werden.
Für interessierte Leser verlinke ich lediglich auf die entsprechenden Webseiten.

Mit Byzanz ist es sehr einfach, Bildschirmaufnahmen zu erstellen und als GIF zu speichern.
Um seine Bedienung noch komfortabler zu gestalten, werde ich ein kleines Shell-Skript vorstellen.

## Anforderungen
Für eine einfache Handhabbarkeit sollte das Programm möglichst wenig tun, seine wenigen Aufgaben aber zufriedenstellend erfüllen.
Da es mir nur um ein einfaches GIF geht, brauche ich keine umfangreichen Videoaufnahmen, gerade auch aus Gründen der Performance während der Aufnahme.
Im Idealfall kann das Programm über die Kommandozeile ausgeführt werden und beschränkt sich darauf, Bildschirmaktivität aufzunehmen und als GIF-Datei zu speichern.
Als Zusatzoptionen sollte man eine Verzögerung vor der Aufnahme, eine Gesamtaufnahmelänge und den aufzunehmenden Bereich oder das aufzunehmende Fenster festlegen können.

Viele Anwendungen erlauben dadurch ein bequemeres (vielleicht produktiveres) Arbeiten, dass sie vor allem über die Tastatur bedient werden können.
Sollen Tutorials über solche Anwendungen mit Videos illustriert werden, ist es deshalb vonnöten, dass nicht nur der sichtbare Bereich auf dem Bildschirm aufgenommen wird, sondern auch die Tastenanschläge visualisiert werden.
Das gesuchte Programm sollte deshalb auch die Ausgabe eines Programms wie [Screenkey](https://gitlab.com/screenkey/screenkey) aufzeichnen können.

Statt den gängigen Desktops setzen sich in der Linux-Welt und unter MacOS-Nutzern immer mehr Fenstermanager durch.
Daraus können sich besondere Probleme bei der Aufnahme des sichtbaren Bildschirmbereiches ergeben.
Die gesuchte Anwendung sollte deshalb im besten Fall auch in einer Umgebung ohne Desktop einsetzbar sein, nach Möglichkeit ohne großeren Konfigurationsaufwand.
Persönlich nutze ich [i3](https://i3wm.org/) und bei der Auswahl eines Programms habe ich vor allem auf Kompatibilität zu diesem bestimmten Fenstermanager geachtet.

## Überblick über verfügbare Screencast-Apps
Einige Linux-YouTuber (wie [Distrotube](https://www.youtube.com/watch?v=_UNRVybAJTs&t=1054s) oder [Brodie Robertson](https://www.youtube.com/watch?v=TpsfdPj0juA)) verwenden für ihre Screencasts [OBS Studio](https://obsproject.com/).
Auch wenn die freie und quelloffene Anwendung sicher hervorragend ist, so bietet sie weitaus mehr Features, als ich für meine Zwecke benötige.
In vielen Foren wird [FFmpeg](https://ffmpeg.org/) empfohlen, mit dem Videos aufgenommen und im Anschluss zu GIF-Dateien umgewandelt werden können.
Wenn das Programm auch relativ einfach zu handhaben ist und über die Kommandozeile bedient werden kann, so wäre mir eine direktere und auf GIF-Dateien beschränkte Lösung lieber.
Darüber hinaus stellte es sich in einem kurzen Test als eher ressourcenintensiv heraus.
Ähnliches gilt für den nicht länger gewarteten [Kazam Screencaster](https://launchpad.net/kazam), das sich weiter aktiv in Entwicklung befindliche [Vokoscreen-NG](https://github.com/vkohaupt/vokoscreenNG) und den irreführend benannten [SimpleScreenRecorder](https://www.maartenbaert.be/simplescreenrecorder/).
Gemeinsam ist diesen graphischen Tools auch ihre meinem Empfinden nach eher wenig ansprechende Benutzeroberfläche.

Zwei weitere populäre Anwendungen, [Peek](https://github.com/phw/peek) und [Silentcast](https://github.com/colinkeenan/silentcast) vertragen sich schlecht mit meinem Fenstermanager.
Da die von Peek genutzte X Shape Extension von i3 [nicht unterstützt](https://github.com/phw/peek#on-i3-the-recording-area-is-all-black-how-can-i-record-anything) wird, müsste eine Compositor (wie Compton) verwendet werden.
Auch wenn Silentcast eine [Anleitung](https://github.com/colinkeenan/silentcast#tiling-window-managers) gibt, um die i3-Konfigurationsdatei entsprechend den Anforderungen anzupassen, so würde es nicht out-of-the-box funktionieren und gegebenenfalls weitere Probleme nach sich ziehen.
Die Verwendung beider Programme ist beim gegebenen Setting also mit einem nicht geringen Mehraufwand verbunden.

Verschiedene CLI-Anwendungen – darunter das zumeist vorinstallierte [GNU Screen](https://linux.die.net/man/1/screen), [Terminalizer](https://github.com/faressoft/terminalizer), [asciinema](https://asciinema.org/) und [ttystudio](https://github.com/chjj/ttystudio) – dienen dem Zweck, Terminal-Sessions aufzuzeichnen.
In Kombination mit [agg](https://github.com/asciinema/agg) war asciinema eine weitgehend zufriedenstellende Lösung.
In einigen Fällen ist es aber zweckdienlich, auch Tastenanschläge auf dem Bildschirm anzuzeigen.
Um diese Ausgaben zusätzlich zur Terminal-Aktivität aufzuzeichnen, ist eine Aufnahme von mehr als nur dem Terminal nötig.

[Byzanz](https://gitlab.gnome.org/Archive/byzanz) ist ein weiteres Tool für die Kommandozeile, das häufig empfohlen wird.
Es sprach mich zunächst deshalb an, weil es in seiner Selbstdarstellung genau die Prinzipien zum Ausdruck bringt, wie man sie sich vielleicht von jeder Software wünschen würde.
Zwar liegt der letzte Release der CLI-Anwendung schon etliche Jahre zurück, doch es erfüllt das gesetzte Ziel völlig zufriedenstellend.
Im folgenden Abschnitt werde ich die Einzelheiten eines Workflows erläutern, mit dem das hier gestellte Problem auf äußerst komfortable Weise gelöst wird.


## Aufnahmen mit Byzanz
Wie die `man`-Page [erläutert](https://linux.die.net/man/1/byzanz-record), wird eine Aufnahme mit Byzanz durch einen Befehl der folgenden Form gestartet:
```bash
 byzanz-record [options] FILENAME
 ```
`FILENAME` steht dabei für den Namen der GIF-Datei, die erstellt werden soll.

Das Programm unterstützt zwei Ausführungsmodi:
1. Durch die Option `-d=WERT` wird die Länge der Aufzeichnung (*duration*) in Sekunden angeben. Wird dieser Wert nicht ausdrücklich festgelegt, so wird die Aufnahme nach zehn Sekunden beendet.
2. Durch die Option `-e="COMMAND"`[^2] wird ein Befehl übergeben, der mit Beginn der Aufnahme ausgeführt wird (*execute*).[^1] Die Aufzeichnung endet, wenn das aufgerufene Programm beendet wird.

Um sich eine Vorbereitungszeit zu gönnen, kann für beide Modi festgelegt werden, nach wie vielen Sekunden die Aufnahme beginnen soll.
Dazu wird `--delay=WERT` gesetzt.
Der Defaultwert beträgt hier eine Sekunde.


### Aufnahme eines Bildschirmbereiches
Standardmäßig wird durch `byzanz-record` der gesamte sichtbare Bereich des Bildschirms aufgenommen.
Das ist häufig nicht zweckmäßig, da ein zu großer Aufnahmebereich vom Wesentlichen ablenken kann.
Ein eingeschränkter Bereich kann auch genutzt werden, um persönliche Informationen zu verbergen, die sonst in der Aufnahme sichtbar wären.

Vier Byzanz-Optionen dienen dazu, den Aufnahmebereich in Form eines Rechtecks zu definieren.
Mit `-x` und `-y` werden die x- und y-Koordinaten des Rechtecks (in Pixeln) angeben, und mit `-w` (`--width`) und `-h` (`--height`) werden Breite und Höhe des Rechtecks (in Pixeln) festgelegt.
One weitere Hilfsmittel bleibt es der Vorstellungskraft des Nutzers überlassen, ein Rechteck auf dem Bildschirm abzustecken.
Wenn man die Bildschirmauflösung seines Computers kennt, kann man grob abschätzen, wo die Koordinaten des vorgestellten Rechtecks liegen würde.

Zum Glück gibt es Tools, die es dem Nutzer ermöglichen, mit der Maus ein sichtbares Rechteck zu ziehen.
Das minimalistische [xrectsel](https://github.com/ropery/xrectsel), das unter anderem im Arch User Repository [befindet](https://aur.archlinux.org/packages/xrectsel) verfügbar ist, eignet sich hervorragend zu diesem Zweck. 
Führt man `xrectsel` aus, so ändert sich der Mauszeiger und man kann ein Rechteck ziehen; auf der Standardausgabe werden dann die vier Werte ausgegeben, die man `byzanz-record` als Koordinaten und Maße übergeben kann.

Um die Verwendung von Byzanz komfortabler zu gestalten, empfiehlt es sich, die verschiedenen Komponenten in einem Skript zu vereinigen.
Praktischerweise wurde uns diese Arbeit von einigen Nutzern im [askubuntu-Forum](https://askubuntu.com/questions/107726/how-to-create-animated-gif-images-of-a-screencast) bereits abgenommen.
Insbesondere [edouard-lopez/record-gif.sh](https://github.com/edouard-lopez/record-gif.sh) ist sehr hilfreich:
```bash
#!/usr/bin/env bash
delay=3
duration=${1:-10}
save_as=${2:-$HOME/recorded.gif}
area=$(shift 2; echo "$@")

start_audio=/usr/share/sounds/freedesktop/stereo/camera-shutter.oga
end_audio=/usr/share/sounds/freedesktop/stereo/complete.oga

notify() {
    message=${1:-''}
    audio="${2}"

    echo "$message"
    if [[ -f $audio ]]; then
      paplay $audio &
    fi
}

# xrectsel from https://github.com/lolilolicon/xrectsel
select_area() {
  local area
  area="${1}"

  if [[ -z "$area" ]]; then
    area=$(xrectsel "--x=%x --y=%y --width=%w --height=%h") || exit -1
  fi

  echo $area
}

countdown() {
  local delay=$1

  printf "Start recording in: "
  for (( i=$delay; i>0; --i )) ; do
      printf "%s\b" "$i"
      sleep 1
  done
  printf "\r"
}

progress-bar() {
  local duration=${1}


    already_done() { for ((done=0; done<$elapsed; done++)); do printf "▇"; done }
    remaining() { for ((remain=$elapsed; remain<$duration; remain++)); do printf " "; done }
    percentage() { printf "| %s%%" $(( (($elapsed)*100)/($duration)*100/100 )); }
    clean_line() { printf "\r"; }

  for (( elapsed=1; elapsed<=$duration; elapsed++ )); do
      already_done; remaining; percentage
      sleep 1
      clean_line
  done
  printf "\n"
}

remove_existing() {
  if [[ -f $save_as ]]; then
    rm $save_as
    printf "Existing record will be overwritten.\n"
  fi
}

create_replay() {
  local seletected_area="$1"
  local duration=$2
  local save_as="$3"
  local executable="$0"

  echo "$executable $duration $save_as ${seletected_area}" > $HOME/.record.again
  chmod u+x $HOME/.record.again
}
run() {
  printf "Saving in %s%s\n" "$save_as"
  remove_existing
  printf "Recording for %ss.\n\n" "$duration"
  printf "Select Area to record…\n"

  seletected_area="$(select_area "$area")"
  countdown $delay
  create_replay "$seletected_area" "$duration" "$save_as"
  printf "You can replay this recording by running: \n  $ bash %s\n\n" "$HOME/.record.again"
  notify "Recording…                        " "$start_audio"

  progress-bar $duration &
  byzanz-record --verbose --delay=0 ${seletected_area} --duration=$duration $save_as > /dev/null

  notify "Finished!" "$end_audio"
}

run
```
Nun kann eine Aufnahme nur eines Teilbereiches des Bildschirms sehr einfach gestartet werden:
```bash
record-gif.sh DURATION FILENAME
```


### Aufnahme von Mauszeiger, Audioausgabe und Tastatureingaben
In der Standardeinstellung wird von Byzanz lediglich das sichtbare Bild aufgenommen.
Soll auch die Audioausgabe aufgezeichnet werden, kann das `-a`-Flag (`--audio`) gesetzt werden.

Auch Benutzereingaben werden nicht ohne Weiteres aufgenommen.
Soll der Mauszeiger von der Aufnahme erfasst werden, muss das `-c`-Flag (`--cursor`) gesetzt werden. 
Leider verfügt Byzanz über keine Funktion, um die auf der Tastatur gedrückten Tasten zu visualisieren und aufzunehmen.
Da es aber Ausgaben von beliebigen anderen Tools aufzeichnet, kann das besonders populäre [Screenkey](https://gitlab.com/screenkey/screenkey) gestartet werden, bevor man mit der Aufnahme beginnt.
Die ansprechend formatierten Tasten-Ausgaben sind dann Teil der Aufnahme, wie alle anderen sichtbaren Bildschirmelemente.

### Workflow und Beispiel
In diesem Abschnitt fasse ich den Workflow zusammen, dem ich folge, um Aufnahmen zu erstellen.
Zur Veranschaulichung habe ich einen Screencast davon erstellt, wie man einen mit Byzanz eine Bildschirmaufnahme erstellt und als GIF speichert:
1. Da verschiedene Terminal-Fenster benötigt werden, empfiehlt sich die Verwendung von `tmux` (oder einem anderen Terminal-Multiplexer). Man startete also zunächst am besten eine neue TMUX-Session.
2. Dann öffnet man ein neues Terminal-Fenster, in dem Screenkey gestartet wird.
3. Im ersten Fenster startet man das Byzanz-Skript und wählt im Anschluss den Aufnahmebereich aus. Danach verbleiben einige Sekunden, bevor die Aufnahme tatsächlich startet.
4. Im Beispiel unten nutze ich diese Zeit, um ein weiteres Terminal-Fenster zu öffnen, in dem die eigentlich aufzuzeichnende Aktivität stattfinden wird. Darüber hinaus verwende ich die Verzögerung, um Vim zu starten. In der Folge könnte also ein Workflow innerhalb dieses Texteditors veranschaulicht werden. Das Resultat wäre ein Video, an dessem Beginn Vim bereits gestartet ist. Es wird durchgeführt, was veranschaulicht werden soll, und nach der als Länge angegebenen Zeit wird die Aufnahme automatisch beendet und in der spezifizierten Datei gespeichert.

<image src="/assets/images/record-gif.gif" alt="Beispiel einer Byzanz-Aufnahme" />

## Installation
```bash
pacman -S screenkey
pacman -S byzanz
curl https://raw.githubusercontent.com/edouard-lopez/record-gif.sh/master/record-gif.sh > ~/.bin/recorrd-gif.sh # oder in einem anderen Verzeichnis für Binaries

# Installation von xrectsel über AUR
cd ~/builds
git clone https://aur.archlinux.org/xrectsel.git
cd xrectsel
makepkg -si
```

## Anmerkungen
[^1]: Auf der verlinkten Online-Version der `man`-Page wird die Option `-e` bzw. `--execute` nicht genannt, wohl aber in der lokalen Version. Ich konnte nicht  in Erfahrung bringen, ob es sich dabei um eine Nachlässigkeit handelt oder ob verschiedene Releases von Byzanz im Umlauf sind, die sich in ihren Features unterscheiden. 
[^2]: Die doppelten Anführungszeichen sind wichtig, falls dem Befehl selbst wiederum Argumente übergeben werden. Der gesamte `byzanz-record`-Befehl wird von der Shell in mehreren Schritten interpretiert, bevor das Programm aufgerufen wird. In einem Schritt werden die Argumente [separiert](https://www.gnu.org/software/bash/manual/bash.html#Word-Splitting). Als ein *Wort* werden für gewöhnlich (sofern die `IFS`-Umgebungsvariable nicht neu definiert wird) Zeichenketten zwischen Leerzeichen aufgefasst. Im Rahmen des Byzanz-Aufrufs sollen nun jedoch mehrere Wörter *ein* Argument bilden, die auf der tieferen Ebene die verschiedenen Argumente eines anderen Befehls sind. Im Befehl `byzanz-record -e "echo Hello World"` ist `echo Hello World` *ein* Argument; der Befehl bewirkt jedoch, dass `echo Hello World` in einer neuen Subshell aufgerufen wird, und dann sind `Hello` und `World` zwei Argumente des `echo`-Befehls.    
