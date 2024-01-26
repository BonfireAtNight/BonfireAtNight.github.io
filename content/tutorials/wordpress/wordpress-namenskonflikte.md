# Namenskonflikte in WordPress
Das WordPress-Ökosystem umfasst Zehntausende Plugins. Es ist deshalb völlig unerlässlich, dass sich jedes von ihnen an bestimmte Regeln hält, um Kollisionen zu vermeiden. Ein *Namens*konflikt liegt dann vor, wenn eine Variable, Funktion oder Klasse wie Objekte anderer Plugins benannt wird. Deshalb definiert Das Content-Management-System Namenskonventionen und Mechanismen, die von Plugins strikt eingehalten werden müssen.

Das [Plugin Handbook](https://developer.wordpress.org/plugins/plugin-basics/best-practices/){: target="_blank"} stellt die verschiedenen Ansätze vor. Einige werden eher mit einer prozeduralen Programmierweise assoziiert, während andere den Grundprinzipien objektorientierter Programmierung folgen. Diese Strukturierung erscheint mir nicht völlig sachgemäß. Eine bessere Strategie sollte von allen Verfahren Gebrauch machen.

Wir wollen mit möglichst einfachen Mitteln einen Gültigkeitsbereich (*scope*) für die Entitäten unseres Plugins schaffen. Dabei wird einem objektorientiertem Ansatz der Vorrang eingeräumt. Es wird *eine* Klasse definiert, die das Plugin repräsentiert. In ihrer Instanziierung werden die meisten Plugin-bezogenen Daten *eingekapselt* (*encapsulate*).

Zusätzlich werden weitere Klassen unter *Namespaces* erstellt, um vom praktischem [Autoload-Muster](https://www.php.net/manual/de/language.oop5.autoload.php){: target="_blank"} profitieren zu können.[^psr-4] Von Präfixen beim Namen schließlich wird Gebrauch gemacht bei den Dingen, die nicht als Klassen-Mitglieder behandelt werden können. Dazu gehören insbesondere Optionen (die als Schlüssel-Wert-Paare in Datenbanktabellen gespeichert werden) und eigene Hooks (durch die andere Plugins mit unserem Plugin interagieren können).

## Eine Plugin-Klasse
Klassen sind in erster Linie Blaupausen (*blueprints*), denen gemäß *Objekte* erstellt (oder *instanziiert*) werden  können. Variablen und Funktionen, die innerhalb von Klassen deklariert werden, werden in der Regel erst als Eigenschaften (*properties*) und Methoden (*methods*) von Objekten realisiert. Sie werden nur über das individuelle Objekt referenziert.[^statische-methoden]

Klassen bieten also einen Rahmen (oder Gültigkeitsbereich), in dem Bezeichner eindeutig für die Komponenten unseres Plugins stehen. Wir müssen lediglich gewährleisten, dass der Klassenname einmalig ist. Das kann dadurch sichergestellt werden, dass ihm ein Präfix vorangestellt oder der Name des Plugins selbst als Klassenname verwendet wird. Alternativ kann ein Namensraum definiert werden (dazu mehr im nächsten Abschnitt).

Wenn kein Namensraum festgelegt wird, sollte ausdrücklich abgefragt werden, dass der Klassenname noch nicht bereits von einem anderen Plugin verwendet wird. In einem [Beispiel](https://developer.wordpress.org/plugins/plugin-basics/best-practices/#object-oriented-programming-method){: target="_blank"} empfiehlt das Plugin-Handbuch das folgende Muster:
```php
if ( ! class_exists( '<Klassenname>' ) ) {
    class <Klassenname> { [...] }
}
```

Die Klasse umfasst Methoden, mit denen das Plugin initialisiert (`init()`) oder ausgeführt (`run()`) wird. Das geschieht entweder durch Methodenaufrufe nachdem die Klasse instanziiert wurde. In diesem Fall wird nur *eine* Instanz der Klasse erstellt (*singleton*) und das Objekt repräsentiert den Zustand des Plugins zu einem gegebenen Zeitpunkt. Oder es werden zur Initialisierung bzw. Ausführung *statische* Methoden bestimmt. Diesem Muster folgt das Beispiel von eben, wenn wir es uns in vollständiger Form anschauen:
```php
if ( ! class_exists( 'WPOrg_Plugin' ) ) {
    class WPOrg_Plugin {
        public static function init() {
            register_setting( 'wporg_settings', 'wporg_option_foo' );
        }

        public static function get_foo() {
            return get_option( 'wporg_option_foo' );
        }
    }

    WPOrg_Plugin::init();
    WPOrg_Plugin::get_foo();
}
```

Dem [Single-Responsibility-Prinzip](https://de.wikipedia.org/wiki/Single-Responsibility-Prinzip){: target="_blank} folgend sollten weitere Klassen verschiedene Aspekte der Initialisierung und Ausführung übernehmen. So empfiehlt sich etwa eine Klasse, die für alle Dinge verantwortlich ist, die mit dem WordPress-Adminbereich zu tun haben.

## Namensräume
[Namensräume](https://www.php.net/manual/de/language.namespaces.basics.php){: target="_blank"} (*namespaces*) sind ein Feature, das mit Version 5.3.0 in PHP eingeführt wurde. Damit kann zu Beginn einer Datei ein *lokaler* Namensraum definiert werden.[^mehrere-namensraeume] Wenn in der Datei etwa eine Klasse oder Funktion definiert wird, kann sie in einer anderen Datei desselben Projekts über einen Namen referenziert werden, bei dem der Namensraum explizit vorangestellt wird.

Alternativ kann mit dem `use`-Schlüsselwort ein Namensraum bestimmt werden. Damit wird gesagt, dass bei der Auswertung eines Bezeichners (zuerst) in *diesem* Namensraum nach einem Wert gesucht werden soll. Eine explizite Angabe des Namensraum ist im Anschluss nicht mehr notwendig.
```php
foo
```
Dadurch wird nicht nur Schreibaufwand vermieden, sondern der Code wird insgesamt auch leserlicher.

## Automatisches Laden von Klassen

## Namenskonvention: Verwende Präfixe

## Fußnoten
[^psr-4]: [PSR-4](https://www.php-fig.org/psr/psr-4/){: target="_blank} ist ein Code-Standard, der das automatische Laden von Klassen empfiehlt. 
[^statische-methoden]: Statische Methoden einer Klasse existieren unabhängig davon, ob Instanzen von ihr generiert werden. Um sie aufrufen zu können, muss aber der Klassenname vorangestellt werden. Wenn der Klassenname einmalig ist, dann drohen also auch hier keine Kollisionen.
[^mehrere-namensraeume]: Es ist prinzipiell möglich, in *einer* Datei mehr als einen Namensraum festzulegen. Davon ist abzuraten [REF?].
